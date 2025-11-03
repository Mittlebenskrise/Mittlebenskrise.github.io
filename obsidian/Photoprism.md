Create a folder `~/docker-data/photoprism/import/` as import folder. Also, create folders `/media/backup/photoprism/originals/` and `/media/backup/photoprism/storage/` for the Photoprism files.

In mariadb create a new user `photoprism` and a new database `photoprism`. Grant the new user full access to the new database.

Create a `docker-compose.yml` file with the following content:

```
version: '3.5'

services:
  photoprism:
    image: photoprism/photoprism:latest
    container_name: "photoprism"
    restart: unless-stopped
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
    ports:
      - "2342:2342"
    environment:
      PHOTOPRISM_ADMIN_USER: "michael"               # admin login username
      PHOTOPRISM_ADMIN_PASSWORD: "<initialpw>"       # initial admin password (8-72 characters)
      PHOTOPRISM_AUTH_MODE: "password"               # authentication mode (public, password)
      PHOTOPRISM_SITE_URL: "https://photoprism.home.mittlebenskrise.de/"  # server URL in the format "http(s)://domain.name(:port)/(path)"
      PHOTOPRISM_DISABLE_TLS: "false"                # disables HTTPS/TLS even if the site URL starts with https:// and a certificate is available
      PHOTOPRISM_DEFAULT_TLS: "true"                 # defaults to a self-signed HTTPS/TLS certificate if no other certificate is available
      PHOTOPRISM_ORIGINALS_LIMIT: 5000               # file size limit for originals in MB (increase for high-res video)
      PHOTOPRISM_HTTP_COMPRESSION: "gzip"            # improves transfer speed and bandwidth utilization (none or gzip)
      PHOTOPRISM_LOG_LEVEL: "info"                   # log level: trace, debug, info, warning, error, fatal, or panic
      PHOTOPRISM_READONLY: "false"                   # do not modify originals directory (reduced functionality)
      PHOTOPRISM_EXPERIMENTAL: "false"               # enables experimental features
      PHOTOPRISM_DISABLE_CHOWN: "false"              # disables updating storage permissions via chmod and chown on startup
      PHOTOPRISM_DISABLE_WEBDAV: "false"             # disables built-in WebDAV server
      PHOTOPRISM_DISABLE_SETTINGS: "false"           # disables settings UI and API
      PHOTOPRISM_DISABLE_TENSORFLOW: "false"         # disables all features depending on TensorFlow
      PHOTOPRISM_DISABLE_FACES: "false"              # disables face detection and recognition (requires TensorFlow)
      PHOTOPRISM_DISABLE_CLASSIFICATION: "false"     # disables image classification (requires TensorFlow)
      PHOTOPRISM_DISABLE_VECTORS: "false"            # disables vector graphics support
      PHOTOPRISM_DISABLE_RAW: "false"                # disables indexing and conversion of RAW images
      PHOTOPRISM_RAW_PRESETS: "false"                # enables applying user presets when converting RAW images (reduces performance)
      PHOTOPRISM_JPEG_QUALITY: 85                    # a higher value increases the quality and file size of JPEG images and thumbnails (25-100)
      PHOTOPRISM_DETECT_NSFW: "false"                # automatically flags photos as private that MAY be offensive (requires TensorFlow)
      PHOTOPRISM_UPLOAD_NSFW: "true"                 # allows uploads that MAY be offensive (no effect without TensorFlow)
      PHOTOPRISM_DATABASE_DRIVER: "mysql"            # use MariaDB 10.5+ or MySQL 8+ instead of SQLite for improved performance
      PHOTOPRISM_DATABASE_SERVER: "mariadb:3306"     # MariaDB or MySQL database server (hostname:port)
      PHOTOPRISM_DATABASE_NAME: "photoprism"         # MariaDB or MySQL database schema name
      PHOTOPRISM_DATABASE_USER: "photoprism"         # MariaDB or MySQL database user name
      PHOTOPRISM_DATABASE_PASSWORD: "<db-pwd>"       # MariaDB or MySQL database user password
      PHOTOPRISM_SITE_CAPTION: "AI-Powered Photos App"
      PHOTOPRISM_SITE_DESCRIPTION: ""                # meta site description
      PHOTOPRISM_SITE_AUTHOR: ""                     # meta site author
    working_dir: "/photoprism" # do not change or remove
    volumes:
      - "/media/backup/photoprism/originals:/photoprism/originals"
      - "/home/michael/docker-data/photoprism/import:/photoprism/import"
      - "/media/backup/photoprism/storage:/photoprism/storage"
    networks:
      - mysql
      - photoprism_default

networks:
  mysql:
    external: true
  photoprism_default:
    driver: bridge
```

Call `sudo docker compose up -d` to start the container.

How to call the container: https://docs.photoprism.app/getting-started/docker-compose/
