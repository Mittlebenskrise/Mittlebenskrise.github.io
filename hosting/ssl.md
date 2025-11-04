# SSL Certificates

Create a folder in the local HTML folder.

```
mkdir ~/html/.well-known
mkdir ~/html/.well-known/acme-challenge
```

Call  certbot to create the certificates (add the wildcard sub-subdomain only if you intend to work with [Sub-Subdomain for NGinx](/hosting/sub-subdomains)).

```
sudo certbot certonly --manual -d <domain>,*.<domain>
```

You will be prompted to create a file on your web server with a certain content, and/or to add text to a TXT entry on the DNS.
For the latter, in Dynv6, go to the zones and select your zone (e.g., `<domain>`). Get to the "Records" tab and select "Add Record".
Select "TXT" and fill in the name and text as provided by `certbot`.

Once done, press Enter. The certificate should now be created like this:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/<domain>/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/<domain>/privkey.pem
This certificate expires on 2024-03-03.
These files will be updated when the certificate renews.
```

Now, copy the certificates to the HTML folder:

```
sudo cp /etc/letsencrypt/live/<domain>/fullchain.pem ~/html/ssl/<domain>/server.crt
sudo cp /etc/letsencrypt/live/<domain>/privkey.pem ~/html/ssl/<domain>/server.key
sudo chown <user> ~/html/ssl/<domain>/server.*
```

Add the certificates to the NGinx configuration file:

```
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl;
    listen [::]:443 ssl;
    ssl_certificate     /var/www/html/ssl/<domain>/server.crt;
    ssl_certificate_key /var/www/html/ssl/<domain>/server.key;
    server_name <domain>;
    ...
```

Restart NGinx, close your browser (to get rid of cached certificates), open it again, and https to your domain should work.
Test it by calling the domain website:

```
https://<domain>/
```

### Renewing Certificates

Due to a potential bug in DynV6, we first need to modify the Zone settings. Change the A record from `*.<domain>` to `www.<domain>`.
Otherwise, you cannot create a TXT record anymore when certbot asks for it.

Create a basic NGinx configuration (move to NGinx setup!) by calling

```
nano /docker/docker-data/nginx/config/default.conf.certbot
```

and paste the following configuration into it

```
#################################################################################
#
# Handle the <domain> domain as a standard web page with PHP
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
error_page 403 https://<domain>/error/403.html;
error_page 404 https://<domain>/error/404.html;
error_page 500 502 503 504 https://<domain>/error/50x.html;


# Handle the HTTP and HTTPS requests
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl;
    listen [::]:443 ssl;
    ssl_certificate     /var/www/html/ssl/<domain>/server.crt;
    ssl_certificate_key /var/www/html/ssl/<domain>/server.key;

    server_name *.<domain>;

    error_page 403 /error/403.html;
    error_page 404 /error/404.html;
    error_page 500 502 503 504 /error/50x.html;

    # root folder of the web page in the Nginx container
    root /var/www/html;

    proxy_set_header Remote-User  "<user>";
    proxy_set_header Remote-Email "<email>";
    proxy_set_header Remote-Name  "<real name>";

    # check HTML files first, then the PHP files (no "-remote" files!)
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

# Treat the local/IP-based host as a standard web page with PHP active
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl;
    listen [::]:443 ssl;
    ssl_certificate     /var/www/html/ssl/<domain>/server.crt;
    ssl_certificate_key /var/www/html/ssl/<domain>/server.key;

    server_name localhost 192.168.178.26;

    error_page 403 /error/403.html;
    error_page 404 /error/404.html;
    error_page 500 502 503 504 /error/50x.html;

    # root folder of the web page in the Nginx container
    root /var/www/html;

    proxy_set_header Remote-User  "<user>";
    proxy_set_header Remote-Email "<email>";
    proxy_set_header Remote-Name  "<real name>";

    # check HTML files first, then the PHP files
    location / {
        index  index.html index.htm index.php;
    }

    # process the PHP settings
    include /etc/nginx/conf.d/php.config;
}
```

Now, save the nginx configuration and replace it with a basic one that does not forward port 80 and does not provide Authelia authentication:

```
cp /docker/docker-data/nginx/config/default.conf /docker/docker-data/nginx/config/default.conf.all
cp /docker/docker-data/nginx/config/default.conf.certbot /docker/docker-data/nginx/config/default.conf
docker-command nginx-webserver nginx -t
docker-restart nginx-webserver
```

Renew by again calling

```
sudo certbot certonly --manual -d <domain>,*.<domain>
```

When asked to do so, add a TXT record named `_acme-challenge` and paste the value as specified by certpot, e.g., for this output

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Please deploy a DNS TXT record under the name:

_acme-challenge.<domain>.

with the following value:

BzVVaEIkJp_1eS-NQzfHOGgA16OCx2IL3DkeLvGIvEw
```

Paste the value `BzVVaEIkJp_1eS-NQzfHOGgA16OCx2IL3DkeLvGIvEw` into the TXT record.
Now verify that it works by opening `https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.<domain>` in a browser. It should show the pasted value.

Next, when asked to create a file, do so in the folder created before. E.g., for this output

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Create a file containing just this data:

hQPszIDHpzp69uufHSLiRwmYIsepRk1XW96RJUNgWLc.6nrNm5RU8K-Z9kxu0Xe8N3vwoisOm8w4_4dpkllh0tI

And make it available on your web server at this URL:

http://<domain>/.well-known/acme-challenge/hQPszIDHpzp69uufHSLiRwmYIsepRk1XW96RJUNgWLc

(This must be set up in addition to the previous challenges; do not remove,

replace, or undo the previous challenge tasks yet.)

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Edit the file as follows and paste the data into it:

```
nano ~/html/.well-known/acme-challenge/hQPszIDHpzp69uufHSLiRwmYIsepRk1XW96RJUNgWLc
```

Now, copy the renewed certificates to the HTML folder:

```
sudo cp /etc/letsencrypt/live/<domain>/fullchain.pem ~/html/ssl/<domain>/server.crt
sudo cp /etc/letsencrypt/live/<domain>/privkey.pem ~/html/ssl/<domain>/server.key
sudo chown michael ~/html/ssl/<domain>/server.*
```

Restore the nginx configuration:

```
cp /docker/docker-data/nginx/config/default.conf.all /docker/docker-data/nginx/config/default.conf
docker-command nginx-webserver nginx -t
docker-restart nginx-webserver
```

See also https://www.devlix.de/lets-encrypt-wildcard-zertifikate-mit-ionos-dns-api-erzeugen/
