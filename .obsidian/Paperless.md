## Installation

1. Login with your user and create a folder in your home-directory to have a place for your configuration files and the files to be processed.

```
$ mkdir -v ~/paperless
```

2. Go to the [/docker/compose directory on the project page](https://github.com/paperless-ngx/paperless-ngx/tree/main/docker/compose) and download `docker-compose.mariadb-tika.yml`. Rename it to `docker-compose.yml`. Download the `docker-compose.env` file and the `.env` file as well. Store all files in the configuration directory `paperless`.
3. Modify `docker-compose.yml` as follows:

```
# Docker Compose file for running paperless from the Docker Hub.
# This file contains everything paperless needs to run.
# Paperless supports amd64, arm and arm64 hardware.
#
# All compose files of paperless configure paperless in the following way:
#
# - Paperless is (re)started on system boot, if it was running before shutdown.
# - Docker volumes for storing data are managed by Docker.
# - Folders for importing and exporting files are created in the same directory
#   as this file and mounted to the correct folders inside the container.
# - Paperless listens on port 8000.
#
# In addition to that, this Docker Compose file adds the following optional
# configurations:
#
# - Instead of SQLite (default), MariaDB is used as the database server.
#
# To install and update paperless with this file, do the following:
#
# - Copy this file as 'docker-compose.yml' and the files 'docker-compose.env'
#   and '.env' into a folder.
# - Run 'docker compose pull'.
# - Run 'docker compose run --rm webserver createsuperuser' to create a user.
# - Run 'docker compose up -d'.
#
# For more extensive installation and update instructions, refer to the
# documentation.

version: "3.4"
services:
  broker:
    image: docker.io/library/redis:7
    restart: unless-stopped
    volumes:
      - redisdata:/data

  webserver:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    user: "0"
    restart: unless-stopped
    depends_on:
      - broker
      - gotenberg
      - tika
    ports:
      - "8089:8000"
    volumes:
      - data:/usr/src/paperless/data
      - /media/backup/paperless/media:/usr/src/paperless/media
      - /media/backup/paperless/export:/usr/src/paperless/export
      - /home/michael/paperless/consume:/usr/src/paperless/consume
    env_file: docker-compose.env
    environment:
      PAPERLESS_REDIS: redis://broker:6379
      PAPERLESS_DBENGINE: mariadb
      PAPERLESS_DBHOST: mariadb
      PAPERLESS_DBUSER: root
      PAPERLESS_DBPASS: <SQL Password>
      PAPERLESS_DBPORT: 3306
      PAPERLESS_TIKA_ENABLED: 1
      PAPERLESS_TIKA_GOTENBERG_ENDPOINT: http://gotenberg:3000
      PAPERLESS_TIKA_ENDPOINT: http://tika:9998
    networks:
      - mysql
      - paperless_default


  gotenberg:
    image: docker.io/gotenberg/gotenberg:7.10
    restart: unless-stopped
    # The gotenberg chromium route is used to convert .eml files. We do not
    # want to allow external content like tracking pixels or even javascript.
    command:
      - "gotenberg"
      - "--chromium-disable-javascript=true"
      - "--chromium-allow-list=file:///tmp/.*"

  tika:
    image: ghcr.io/paperless-ngx/tika:latest
    restart: unless-stopped

volumes:
  data:
  media:
  dbdata:
  redisdata:

networks:
  mysql:
    external: true
  paperless_default:
    external: true

```

4. Modify `docker-compose.env`as follows:
   
```
# The UID and GID of the user used to run paperless in the container. Set this
# to your UID and GID on the host so that you have write access to the
# consumption directory.
USERMAP_UID=0
USERMAP_GID=0

# Additional languages to install for text recognition, separated by a
# whitespace. Note that this is
# different from PAPERLESS_OCR_LANGUAGE (default=eng), which defines the
# language used for OCR.
# The container installs English, German, Italian, Spanish and French by
# default.
# See https://packages.debian.org/search?keywords=tesseract-ocr-&searchon=names&suite=buster
# for available languages.
#PAPERLESS_OCR_LANGUAGES=tur ces

###############################################################################
# Paperless-specific settings                                                 #
###############################################################################

# All settings defined in the paperless.conf.example can be used here. The
# Docker setup does not use the configuration file.
# A few commonly adjusted settings are provided below.

# This is required if you will be exposing Paperless-ngx on a public domain
# (if doing so please consider security measures such as reverse proxy)
PAPERLESS_URL=https://paperless.home.mittlebenskrise.de

# Adjust this key if you plan to make paperless available publicly. It should
# be a very long sequence of random characters. You don't need to remember it.
#PAPERLESS_SECRET_KEY=change-me

# Use this variable to set a timezone for the Paperless Docker containers. If not specified, defaults to UTC.
PAPERLESS_TIME_ZONE=Europe/Berlin

# The default language to use for OCR. Set this to the language most of your
# documents are written in.
PAPERLESS_OCR_LANGUAGE=deu+eng

# Set if accessing paperless via a domain subpath e.g. https://domain.com/PATHPREFIX and using a reverse-proxy like traefik or nginx
#PAPERLESS_FORCE_SCRIPT_NAME=/PATHPREFIX
#PAPERLESS_STATIC_URL=/PATHPREFIX/static/ # trailing slash required
PAPERLESS_FILENAME_FORMAT={correspondent}/{created_year}/{title}
PAPERLESS_FILENAME_FORMAT_REMOVE_NONE=true
```

5. Run `docker compose pull`. This will pull the image.
6. To be able to login, you will need a super user. To create it, execute the following command:

```
sudo docker compose run --rm webserver createsuperuser
```

7. To start the containers, run 

   
```
sudo docker compose up -d
```
   
8. Open the web page `http://myhost:8089`

## Changes to the Environment#

Adjust `docker-compose.env` to make the necessary changes and the call 

```
sudo docker-compose down && sudo docker-compose up -d && sudo docker-compose exec webserver document_renamer
```
## Clients

A moblie client for Android can be found in the [Play Store](https://play.google.com/store/apps/details?id=de.astubenbord.paperless_mobile). 

Labels for Scanning
https://tobiasmaier.info/asn-qr-code-label-generator/
"Regenerate Labels" – "Print Labels"
Print with Chrome
Print–Advanced–Borders–None
Drucken–Weitere Einstellungen–Ränder–Keine