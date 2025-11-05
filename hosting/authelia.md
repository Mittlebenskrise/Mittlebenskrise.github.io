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

Start Authelia by calling

```
cd /docker/docker-compose/nginx/
docker-compose
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
