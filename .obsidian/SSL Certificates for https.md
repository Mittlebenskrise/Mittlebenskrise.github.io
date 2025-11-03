Create a folder in the local html folder.

```
mkdir ~/html/.well-known
mkdir ~/html/.well-known/acme-challenge
```

Call  certbot to create the certificates (add the wildcard sub-subdomain only if you intend to work with [[Sub-Subdomain for NGinx|sub-subdomains]]).

```
sudo certbot certonly --manual -d home.mittlebenskrise.de,*.home.mittlebenskrise.de
```

You will be prompted to create a file on your webserver with a certain content, and/or to add text to a TXT entry on the DNS. For the latter, in Dynv6 go to the zones and select your zone (e.g. `home.mittlebenskrise.de`). Get to the "Records" tab and select "Add Record". Select "TXT" and fill in name and text as provided by `certbot`.

Once done, press Enter. The certificate should now be created like this:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/home.mittlebenskrise.de/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/home.mittlebenskrise.de/privkey.pem
This certificate expires on 2024-03-03.
These files will be updated when the certificate renews.
```

Now, copy the certificates to the HTML folder:

```
sudo cp /etc/letsencrypt/live/home.mittlebenskrise.de/fullchain.pem ~/html/ssl/home.mittlebenskrise.de/server.crt
sudo cp /etc/letsencrypt/live/home.mittlebenskrise.de/privkey.pem ~/html/ssl/home.mittlebenskrise.de/server.key
sudo chown michael ~/html/ssl/home.mittlebenskrise.de/server.*
```

Add the certificates to the NGinx configuration file:

```
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl;
    listen [::]:443 ssl;
    ssl_certificate     /var/www/html/ssl/home.mittlebenskrise.de/server.crt;
    ssl_certificate_key /var/www/html/ssl/home.mittlebenskrise.de/server.key;
    server_name home.mittlebenskrise.de;
    ...
```

Restart NGinx, close your browser (to get rid of cached certificates), open it again and https to your domain should work. Test it by calling the domain web site:

```
https://home.mittlebenskrise.de/
```

### Renewing Certificates

Due to a potential bug in DynV6, we first need to modify the Zone settings. Change the A record from `*.mydomain.com` to `www.mydomain.com`. Otherwise you cannot create a TXT record anymore when certbot asks for it.

Create a basic NGinx configuration (move to NGinx setup!) by calling

```
nano /docker/docker-data/nginx/config/default.conf.certbot
```

and paste the following configuration into it

```
#################################################################################
#
# Handle the home.mittlebenskrise.de domain as a standard web page with PHP
#
#################################################################################

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


# Handle the HTTP and HTTPS requests
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl;
    listen [::]:443 ssl;
    ssl_certificate     /var/www/html/ssl/home.mittlebenskrise.de/server.crt;
    ssl_certificate_key /var/www/html/ssl/home.mittlebenskrise.de/server.key;

    server_name *.home.mittlebenskrise.de;

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

    # process the PHP settings
    include /etc/nginx/conf.d/php.config;
}


#################################################################################
#
# Handle the localhost and IP domains
#
#################################################################################

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

    # check html files first, then the PHP files
    location / {
        index  index.html index.htm index.php;
    }

    # process the PHP settings
    include /etc/nginx/conf.d/php.config;
}
```

Now, save the nginx configuration and replace it by a basic one that does not forward port 80 and does not provide Authelia authentication:

```
cp /docker/docker-data/nginx/config/default.conf /docker/docker-data/nginx/config/default.conf.all
cp /docker/docker-data/nginx/config/default.conf.certbot /docker/docker-data/nginx/config/default.conf
docker-command nginx-webserver nginx -t
docker-restart nginx-webserver
```

Renew by again calling

```
sudo certbot certonly --manual -d home.mittlebenskrise.de,*.home.mittlebenskrise.de
```

When asked to do so, add a TXT record named `_acme-challenge` and paste the value as specified by certpot, e.g. for this output

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Please deploy a DNS TXT record under the name:

_acme-challenge.home.mittlebenskrise.de.

with the following value:

BzVVaEIkJp_1eS-NQzfHOGgA16OCx2IL3DkeLvGIvEw
```

paste the value `BzVVaEIkJp_1eS-NQzfHOGgA16OCx2IL3DkeLvGIvEw` into the TXT record. Now verify that it works, by opening `https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.home.mittlebenskrise.de` in a browser. It should show the pasted value.

Next, when asked to create a file, do so in the folder created before. E.g. for this output

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Create a file containing just this data:

hQPszIDHpzp69uufHSLiRwmYIsepRk1XW96RJUNgWLc.6nrNm5RU8K-Z9kxu0Xe8N3vwoisOm8w4_4dpkllh0tI

And make it available on your web server at this URL:

http://home.mittlebenskrise.de/.well-known/acme-challenge/hQPszIDHpzp69uufHSLiRwmYIsepRk1XW96RJUNgWLc

(This must be set up in addition to the previous challenges; do not remove,

replace, or undo the previous challenge tasks yet.)

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Call `nano` as follows and paste the data into it:

```
nano ~/html/.well-known/acme-challenge/hQPszIDHpzp69uufHSLiRwmYIsepRk1XW96RJUNgWLc
```

Now, copy the renewed certificates to the HTML folder:

```
sudo cp /etc/letsencrypt/live/home.mittlebenskrise.de/fullchain.pem ~/html/ssl/home.mittlebenskrise.de/server.crt
sudo cp /etc/letsencrypt/live/home.mittlebenskrise.de/privkey.pem ~/html/ssl/home.mittlebenskrise.de/server.key
sudo chown michael ~/html/ssl/home.mittlebenskrise.de/server.*
```

Restore the nginx configuration:

```
cp /docker/docker-data/nginx/config/default.conf.all /docker/docker-data/nginx/config/default.conf
docker-command nginx-webserver nginx -t
docker-restart nginx-webserver
```

See also https://www.devlix.de/lets-encrypt-wildcard-zertifikate-mit-ionos-dns-api-erzeugen/