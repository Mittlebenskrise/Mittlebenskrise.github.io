
See also https://obsidian.md/
## Install CouchDB

### Ini File

Some initial configuration is required. Create a `couchdb.ini` file in an `obsidian` folder to use Self-hosted LiveSync as follows ([CouchDB has to be version 3.2 or higher](https://docs.couchdb.org/en/latest/config/http.html#chttpd/enable_cors), if lower `enable_cors = true` has to be under section `[httpd]`):

```ini
[couchdb]
single_node=true
max_document_size = 50000000

[chttpd]
require_valid_user = true
max_http_request_size = 4294967296
enable_cors = true

[chttpd_auth]
require_valid_user = true
authentication_redirect = /_utils/session.html

[httpd]
WWW-Authenticate = Basic realm="couchdb"
bind_address = 0.0.0.0

[cors]
origins = app://obsidian.md, capacitor://localhost, http://localhost
credentials = true
headers = accept, authorization, content-type, origin, referer
methods = GET,PUT,POST,HEAD,DELETE
max_age = 3600
```

### Docker Compose

Create a `docker-compose.yml` file with the following content:

```
version: "3.6"
services:
  couchdb:
    container_name: "obsidian"
    environment:
      - "COUCHDB_USER=admin"
      - "COUCHDB_PASSWORD=<admin-password>"
    image: "couchdb"
    ports:
      - "5984:5984"
    restart: "always"
    volumes:
      - "/home/michael/obsidian/couchdb.ini:/opt/couchdb/etc/local.ini"
      - "/home/michael/obsidian/data:/opt/couchdb/data"
```

_Remember to replace the path with the path to your obsidian folder_
# Install the Plugin

Under "External Plugins" search in the community plugins for "LiveSync". Install and activate the plugin. Then setup the plugin as described here: https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/quick_setup.md

URL is `https://obsidian.home.mittlebenskrise.de/`, user name is `admin` and password as set above.