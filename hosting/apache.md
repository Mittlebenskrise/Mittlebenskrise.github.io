# Apache Web Server

## Installation

First, create the docker-compose.yml file:

```
mkdir /docker/docker-compose/apache/
mkdir /docker/docker-data/apache/
nano /docker/docker-compose/apache/docker-compose.yml
```

and paste the following content into it:

```
services:
    apache:
        container_name: apache
        restart: always
        build:
          context: .
          dockerfile: Dockerfile
        ports:
            - 1080:80
            - 1443:443
        volumes:
            - '$HTML_DIR:/var/www/html'
            - '$DOCKER_DATA_DIR/apache/config:/etc/apache2'
            - '$DOCKER_DATA_DIR/apache/config/php:/usr/local/etc/php'
            - '$DOCKER_DATA_DIR/apache/log:/var/log/apache2'
            - '$TANDOOR_MEDIA_DIR:/var/www/html/media/'
        networks:
          - mysql
        environment:
          TZ: $TZ

networks:
  mysql:
    external: true
```

Use a local file for serving the HTML content, which can be easily accessed. To do so, create a folder first (e.g., in the home directory):

```
mkdir ~/html
```

Add the path to `/docker/docker-compose/.env` like this:

```
# general path settings
#  - User
USER_DIR="/home/<user>"
HTML_DIR="$USER_DIR/html"
```

Next, configure PHP by creating a Docker file:

```
nano /docker/docker-compose/apache/Dockerfile
```

and paste the following text into it:

```
# Use an official PHP runtime
FROM php:8.4-apache

# Enable Apache modules
RUN a2enmod rewrite

# install missing packages and activate i18n
RUN apt update && apt install -y libicu-dev libxml2-dev libzip-dev && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-configure intl

# Install any extensions you need
RUN docker-php-ext-install mysqli pdo pdo_mysql intl xml zip
RUN docker-php-ext-enable mysqli intl xml zip

# Set the working directory to /var/www/html
WORKDIR /var/www/html

ADD *.ini /usr/local/etc/php/conf.d/
```

Create `docker-browscap.ini` file in /docker/docker-compose/apache with the following content:

```
[browscap]
; https://php.net/browscap
browscap = /usr/local/etc/php/conf.d/browscap.ini
```

Download `browscap.ini` file to `/docker/docker-compose/nginx/php`:

```
wget https://browscap.org/stream?q=BrowsCapINI -O /docker/docker-compose/apache/browscap.ini
```

## Start the Apache Container

Create a file `index.html` in the local folder (`html`) with some HTML code and start Apache by calling

```
cd /docker/docker-compose/apache/
docker-compose
```

Test the new Apache installation by opening a web browser and typing the server's [IP address](https://phoenixnap.com/glossary/what-is-an-ip-address):

```
http://[server-ip-address]
```

Alternatively, connect with the Apache installation on the same machine by typing **`localhost`** and the host port you assigned to the container.

To verify your installation, save the following content in a file `phpinfo.php` in the HTML folder:

```
<?php
  phpinfo();
?>
```

Point your browser to `http://<domain>/phpinfo.php` and it will display the values of various PHP configuration parameters.

## Certificate

If you want to use `https`, you will need an SSL certificate. The steps are described in the chapter [SSL Certificates](/hosting/ssl)
