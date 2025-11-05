# Authelia Authentication Server

## Installation

First, create the Docker Compose folder and file:

```
mkdir/docker/docker-compose/authelia
nano /docker/docker-compose/authelia/docker-compose.yml
```

Copy the following lines into the `docker-compose.yml` file and save it:

```
secrets:
  JWT_SECRET:
    file: '$DOCKER_DATA_DIR/authelia/secrets/JWT_SECRET'
  SESSION_SECRET:
    file: '$DOCKER_DATA_DIR/authelia/secrets/SESSION_SECRET'
  STORAGE_PASSWORD:
    file: '$DOCKER_DATA_DIR/authelia/secrets/STORAGE_PASSWORD'
  STORAGE_ENCRYPTION_KEY:
    file: '$DOCKER_DATA_DIR/authelia/secrets/STORAGE_ENCRYPTION_KEY'
  IDENTITY_PROVIDERS_OIDC_HMAC_SECRET:
    file: '$DOCKER_DATA_DIR/authelia/secrets/IDENTITY_PROVIDERS_OIDC_HMAC_SECRET'
services:
  authelia:
    container_name: 'authelia'
    image: 'docker.io/authelia/authelia:latest'
    restart: 'unless-stopped'
    networks:
      - nginx_nginx_default
      - mysql
    expose:
      - 9091
    secrets: ['JWT_SECRET', 'SESSION_SECRET', 'STORAGE_PASSWORD', 'STORAGE_ENCRYPTION_KEY', 'IDENTITY_PROVIDERS_OIDC_HMAC_SECRET']
    environment:
      AUTHELIA_IDENTITY_VALIDATION_RESET_PASSWORD_JWT_SECRET_FILE: '/run/secrets/JWT_SECRET'
      AUTHELIA_SESSION_SECRET_FILE: '/run/secrets/SESSION_SECRET'
      AUTHELIA_STORAGE_MYSQL_PASSWORD_FILE: '/run/secrets/STORAGE_PASSWORD'
      AUTHELIA_STORAGE_ENCRYPTION_KEY_FILE: '/run/secrets/STORAGE_ENCRYPTION_KEY'
      TZ: $TZ
    volumes:
      - '$DOCKER_DATA_DIR/authelia/config:/config'

networks:
  nginx_nginx_default:
    external: true
  mysql:
    external: true
```

Start Authelia by calling

```
cd /docker/docker-compose/authelia/
docker-compose
```

## Configure NGinx

Add the information to the NGinx configuration:

```
sudo nano /docker/docker-data/nginx/config/default.conf
```

Add the following lines in the server section and save it:

```
###########################################################################################################
# Setup the Authelia application
###########################################################################################################
server {
    # process the global SSL settings
    include /etc/nginx/conf.d/ssl.config;

    server_name auth.<domain>;

    # URL of the Authelia server inside Docker
    set $upstream http://authelia:9091;

    # allow large file uploads
    client_max_body_size 1g;

    location / {
        # process the default proxy settings
        include /etc/nginx/conf.d/proxy.config;

        proxy_pass $upstream;
    }

    # pass API calls on to Authelia
    location /api/verify {
        proxy_pass $upstream;
    }

    location /api/authz/ {
        proxy_pass $upstream;
    }
}
```

Create the NGinx configuration files for Authelia by calling `nano /docker/docker-data/nginx/config/auth.config` and pasting this text:

```
auth_basic           "Restricted Area";
auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
```

Next, call `nano /docker/docker-data/nginx/config/proxy.config` and paste this text:

```
## Headers
proxy_set_header Host $host;
proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host $http_host;
proxy_set_header X-Forwarded-URI $request_uri;
proxy_set_header X-Forwarded-Ssl on;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;

## Basic Proxy Configuration
client_body_buffer_size 128k;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; ## Timeout if the real server is dead.
proxy_redirect  http://  $scheme://;
proxy_http_version 1.1;
proxy_set_header Connection '';
proxy_cache_bypass $cookie_session;
proxy_no_cache $cookie_session;
proxy_buffers 64 256k;

## Trusted Proxies Configuration
## Please read the following documentation before configuring this:
##     https://www.authelia.com/integration/proxies/nginx/#trusted-proxies
real_ip_header X-Forwarded-For;
real_ip_recursive on;

## Advanced Proxy Configuration
send_timeout 5m;
proxy_read_timeout 360;
proxy_send_timeout 360;
proxy_connect_timeout 360;
```

Next, call `nano /docker/docker-data/nginx/config/authelia-authrequest.config` and paste this text:

```
## Send a subrequest to Authelia to verify if the user is authenticated and has permission to access the resource.
auth_request /internal/authelia/authz;

## Save the upstream metadata response headers from Authelia to variables.
auth_request_set $user $upstream_http_remote_user;
auth_request_set $groups $upstream_http_remote_groups;
auth_request_set $name $upstream_http_remote_name;
auth_request_set $email $upstream_http_remote_email;

## Inject the metadata response headers from the variables into the request made to the backend.
proxy_set_header Remote-User $user;
proxy_set_header Remote-Groups $groups;
proxy_set_header Remote-Email $email;
proxy_set_header Remote-Name $name;

## Configure the redirection when the authz failure occurs. Lines starting with 'Modern Method' and 'Legacy Method'
## should be commented / uncommented as pairs. The modern method uses the session cookies configuration's authelia_url
## value to determine the redirection URL here. It's much simpler and compatible with the mutli-cookie domain easily.

## Modern Method: Set the $redirection_url to the Location header of the response to the Authz endpoint.
auth_request_set $redirection_url $upstream_http_location;

## Modern Method: When there is a 401 response code from the authz endpoint redirect to the $redirection_url.
error_page 401 =302 $redirection_url;

## Legacy Method: Set $target_url to the original requested URL.
## This requires http_set_misc module, replace 'set_escape_uri' with 'set' if you don't have this module.
# set_escape_uri $target_url $scheme://$http_host$request_uri;

## Legacy Method: When there is a 401 response code from the authz endpoint redirect to the portal with the 'rd'
## URL parameter set to $target_url. This requires users update 'auth.example.com/' with their external authelia URL.
# error_page 401 =302 https://auth.example.com/?rd=$target_url;
```

Finally, call `nano /docker/docker-data/nginx/config/authelia-location.config` and paste this text:

```
set $upstream_authelia http://authelia:9091/api/authz/auth-request;

## Virtual endpoint created by nginx to forward auth requests.
location /internal/authelia/authz {
    ## Essential Proxy Configuration
    internal;
    proxy_pass $upstream_authelia;

    ## Headers
    ## The headers starting with X-* are required.
    proxy_set_header X-Original-Method $request_method;
    proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header Content-Length "";
    proxy_set_header Connection "";

    ## Basic Proxy Configuration
    proxy_pass_request_body off;
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; # Timeout if the real server is dead
    proxy_redirect http:// $scheme://;
    proxy_http_version 1.1;
    proxy_cache_bypass $cookie_session;
    proxy_no_cache $cookie_session;
    proxy_buffers 4 32k;
    client_body_buffer_size 128k;

    # allow large file uploads
    client_max_body_size 1g;

    ## Advanced Proxy Configuration
    send_timeout 5m;
    proxy_read_timeout 240;
    proxy_send_timeout 240;
    proxy_connect_timeout 240;
}
```

Check the NGinx configuration by calling:

```
docker exec -it nginx-webserver nginx -t
```

If needed, correct the configuration file. Now, restart NGinx by calling:

```
docker restart nginx-webserver
```

Now, when accessing the domain, you'll first have to log in to Authelia.

## Password generation

If you want to generate a password for a user file:

```
sudo docker run authelia/authelia:latest authelia crypto hash generate argon2 --password '<password>'
```

Output will be like this:

```
Digest: $argon2id$v=19$m=65536,t=3,p=4$Hjc8e7WYcBFcJmEDUOsS9A$ozM7RyZR1EyDR8cuyVpDDfmLrGPGFgo5E2NNqRumui4
```

Copy the information to the user file.
