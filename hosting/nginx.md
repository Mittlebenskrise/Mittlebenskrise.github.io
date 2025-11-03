# Nginx Remote Proxy

With Docker and the database running, it is time to install [nginx](https://nginx.org/). nginx will handle incoming requests and route them to the correct container.

First, create the `docker-compose.yml` file:

```
mkdir /docker/docker-compose/nginx/
mkdir /docker/docker-compose/nginx/php
mkdir /docker/docker-data/nginx/
nano /docker/docker-compose/nginx/docker-compose.yml
```

and paste the following content into it:

```
services:
  nginx:
    image: nginx
    container_name: nginx-webserver
    environment:
      - TZ=$TZ
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - $HTML_DIR:/var/www/html
      - $DOCKER_DATA_DIR/nginx/config:/etc/nginx/conf.d
      - $DOCKER_DATA_DIR/nginx/logs:/var/log/nginx/
      - $TANDOOR_MEDIA_DIR:/media/
    networks:
      - nginx_default

  php:
    build: ./php/
    container_name: nginx-php
    restart: unless-stopped
    ports:
      - "9000:9000"
    expose:
      - 9000
    volumes:
      - $HTML_DIR:/var/www/html/
      - $DOCKER_DATA_DIR/nginx/config/php:/usr/local/etc/php
    networks:
      - nginx_default
      - mysql

networks:
  nginx_default:
    driver: bridge
  mysql:
    external: true
```

Next, configure PHP by creating a Docker file:

```
nano /docker/docker-compose/nginx/php/Dockerfile
```

and paste the following text into it:

```
FROM php:8.4-fpm
ADD *.ini /usr/local/etc/php/conf.d/
RUN apt update && apt install -y libicu-dev libxml2-dev libzip-dev && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-configure intl
RUN docker-php-ext-install mysqli pdo pdo_mysql intl xml zip
RUN docker-php-ext-enable mysqli intl xml zip
```

Create `docker-browscap.ini` file in `/docker/docker-compose/nginx/php` with the following content:

```
[browscap]
; https://php.net/browscap
browscap = /usr/local/etc/php/conf.d/browscap.ini
```

Download `browscap.ini` file to `/docker/docker-compose/nginx/php`:

```
wget https://browscap.org/stream?q=BrowsCapINI -O /docker/docker-compose/nginx/php/browscap.ini
```

Start nginx by calling

```
cd /docker/docker-compose/nginx/ docker-compose
```

You will end up with two new containers, `nginx-webserver` and `nginx-php`.

Now you will find a new file in `/docker/docker-data/nginx/config/` called `default.conf`. Change it to look like this:

```
#######################################################################################
#
# Handle the home.mittlebenskrise.de domain
#
#######################################################################################

# define where log files shall be stored
access_log  /var/log/nginx/host.access.log  main;
error_log   /var/log/nginx/host.error.log   debug;

# define the server used for resolving names
resolver 127.0.0.11;

# define what the proxy upgrade header shall be set to
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

# setup personalized error pages
error_page 403 https://home.mittlebenskrise.de/error/403.html;
error_page 404 https://home.mittlebenskrise.de/error/404.html;
error_page 500 502 503 504 https://home.mittlebenskrise.de/error/50x.html;


# Redirect all http traffic of the domain to https, so that only SSL access is possible from outside
server {
    listen 80;
    listen [::]:80;

    server_name *.home.mittlebenskrise.de;

    return 301 https://$host$request_uri;
}

# Redirect the phpMyadmin subdomain to the phpMyadmin app
server {
    # process the global SSL settings
    include /etc/nginx/conf.d/ssl.config;

    server_name phpmyadmin.home.mittlebenskrise.de;

    # add the Authelia configuration
    include /etc/nginx/conf.d/authelia-location.config;

    # process the global robots.txt settings
    include /etc/nginx/conf.d/robots.config;

    location / {
      proxy_pass http://192.168.178.26:8088/;

      # process the global authorization settings
      include /etc/nginx/conf.d/authelia-authrequest.config;

      # in order for the redirect wo work with phpMyadmin, SSL needs to come from the Home Assistant container
      proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Redirect the domain itself to the standard web page
server {
    # process the global SSL settings
    include /etc/nginx/conf.d/ssl.config;

    server_name home.mittlebenskrise.de;

    # process the default proxy settings
    include /etc/nginx/conf.d/proxy.config;

    location / {
      return 301 http://www.home.mittlebenskrise.de$request_uri;
    }
}

# Redirect all other subdomains to an error page, because they are not (yet) managed
server {
    # process the global SSL settings
    include /etc/nginx/conf.d/ssl.config;

    server_name *.home.mittlebenskrise.de;

    location / {
      return 404;
    }
}


#######################################################################################
#
# Handle the localhost and IP domains
#
#######################################################################################

# Treat the local/IP based host as a standard web page with PHP active
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl;
    listen [::]:443 ssl;
    ssl_certificate     /var/www/html/ssl/home.mittlebenskrise.de/server.crt;
    ssl_certificate_key /var/www/html/ssl/home.mittlebenskrise.de/server.key;

    server_name localhost 192.168.178.26;

    error_page 403 /error/403.html;
    error_page 404 /error/404.html;
    error_page 500 502 503 504 /error/50x.html;

    # root folder of the web page in the Nginx container
    root /var/www/html;

    proxy_set_header Remote-User  "michael";
    proxy_set_header Remote-Email "michael.schneider@gmail.com";
    proxy_set_header Remote-Name  "Michael Schneider";

    # check html files first, then the PHP files (no "-remote" files!)
    location / {
        index  index.html index.htm index.php;
    }

    # process the global authorization settings
    include /etc/nginx/conf.d/php.config;
}
```

## Troubleshooting

If nginx does not start after rebooting, check if another nginx service is already running:

```
sudo systemctl status nginx
```

If so, remove this service by calling

```
sudo systemctl stop nginx sudo systemctl disable nginx
```

Now, it should work properly after reboot.
