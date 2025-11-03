	Create `docker-compose.yml` file in `docker-compose/nginx`:

```
version: "3"
services:
  nginx:
    image: nginx
    container_name: nginx-webserver
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /home/michael/html:/var/www/html
      - /home/michael/docker-data/nginx/config:/etc/nginx/conf.d
      - /home/michael/docker-data/nginx/logs:/var/log/nginx/
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
      - /home/michael/html:/var/www/html/
      - /home/michael/docker-data/nginx/logs/php:/var/log/fpm-php.www.log
    networks:
      - nginx_default
      - mysql

networks:
  nginx_default:
    driver: bridge
  mysql:
    external: true
```

Create `Dockerfile` in `docker-compose/nginx/php`:

```
FROM php:8.2-fpm
ADD *.ini /usr/local/etc/php/conf.d/
RUN apt update && apt install -y libicu-dev libxml2-dev libzip-dev && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-configure intl
RUN docker-php-ext-install mysqli pdo pdo_mysql intl xml zip
RUN docker-php-ext-enable mysqli intl xml zip
```

Create `docker-browscap.ini` file in `~/docker-compose/nginx/php`:

```
[browscap]
; https://php.net/browscap
browscap = /usr/local/etc/php/conf.d/browscap.ini
```

Download `browscap.ini` file to `~/docker-compose/nginx/php`:

```
wget https://browscap.org/stream?q=BrowsCapINI -O ~/docker-compose/nginx/php/browscap.ini
```

Start Docker container by caling

```
sudo docker compose up -d
```

If NGinx does not start after rebooting, check if an nginx service is running:

```
sudo systemctl status nginx
```

If so, remove this service by calling

```
sudo systemctl stop nginx
sudo systemctl disable nginx
```

Now, it should work properly after reboot.

