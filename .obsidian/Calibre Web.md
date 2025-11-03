## Installation

(see also https://github.com/linuxserver/docker-calibre-web)

Create the following Docker Compose file in `~/docker-compose/calibre`

```yaml
services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
      - DOCKER_MODS=linuxserver/mods:universal-calibre #optional
      - OAUTHLIB_RELAX_TOKEN_SCOPE=1 #optional
    volumes:
      - /home/michael/docker-data/calibre:/config
      - /media/backup/ebooks:/books
    ports:
      - 8083:8083
    restart: unless-stopped
```

Create the container by calling `sudo docker compose up -d`.

Download an empty `metadata.db` from https://github.com/janeczku/calibre-web/raw/master/library/metadata.db to your ebooks folder (e.g. `/media/backup/ebooks`).

Open the URL (`http://localhost:8083/`) and enter the inital credentials (_Username:_ admin _Password:_ admin123).

Next, enter the location of the Calibre database (usually `/books`). ~~Make sure that you copied the library created by Calibre into that location!~~

If there is the error `No such file or directory: 'xdg-icon-resource'` in the logs and Calibre does not start, call `docker-command calibre sh` and execute

```
apt update

apt install xdg-utils
```

## Setup

In the admin page make the following changes:
### User Name
Change default user name and password!
### Enable Upload of eBooks
In the basic configurarion under feature configuration check the upload feature. Next, in the admin configuration select to configure the user and check the upload permission.
### External Programs
In the basic configuration under external programs set the following:
- **Path for Calibre ebook converter**: /usr/bin/ebook-convert
- **Path for Kepubify ebook converter**: /usr/bin/kepubify
### ~~User Interface
In the user interface configuration select the dark theme.~~
### SMTP
For GMail, first create an app password by going to https://myaccount.google.com/apppasswords 

Then add the following data in http://localhost:8083/admin/view

- **SMTP Hostname**: smtp.gmail.com
- **SMTP Port**: 465
- **Encryption**: SSL/TLS
- **SMTP Login**: Your Gmail Address (e.g. michael.schneider@gmail.com)
- **SMTP Password**: The App password just created
- **Sender Address**: Should be your Gmail address
### Random Books
In your account's settings uncheck the two checkboxes for showing random books.