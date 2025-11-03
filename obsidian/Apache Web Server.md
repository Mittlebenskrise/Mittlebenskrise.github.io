### Apache Web Server
The official Apache image on Docker Hub is **`httpd`**. Download the image to your system with the **`docker pull`** command:

```bash
sudo docker pull httpd
```

The command output shows the pull progress and reports when the download finishes.

```console
michael@mini-pc:~$ sudo docker pull httpd
Using default tag: latest
latest: Pulling from library/httpd
1f7ce2fa46ab: Pull complete
424de2a10000: Pull complete
6d9a0131505f: Pull complete
5728e491734b: Pull complete
20d3235e84ad: Pull complete
Digest: sha256:04551bc91cc03314eaab20d23609339aebe2ae694fc2e337d0afad429ec22c5a
Status: Downloaded newer image for httpd:latest
docker.io/library/httpd:latest
```

### Start the Apache Container

Type the **`docker run`** command below to create and start a Docker container based on the **`httpd`** image:

```bash
sudo docker run -d --name [container-name] -p 80:[host-port] httpd
```

The command contains multiple options.

- The **`-d`** option tells Docker to run the container in the **detached mode**, i.e., as a background process.
- The **`--name`** flag lets you name the container. If you exclude this flag, Docker generates a random phrase for a name.
- The **`-p`** option allows you to map the [TCP](https://phoenixnap.com/glossary/transmission-control-protocol-tcp) port **`80`** of the container to a [free open port](https://phoenixnap.com/kb/nmap-scan-open-ports) on the host system.

The following example uses the **`httpd`** image to create and run a container named **`apache`** and publish it to port **`80`** on the host system.

```
sudo docker run -d --name apache -p 80:80 httpd
```

If there are no errors, Docker outputs the ID of the new container.

A better alternative is using a local file for serving the HTML content, that can be easily accessed. To do so, create a folder first:
```
mkdir ~/html
```

Also, for PHP create a folder `.config`.

The run docker with a mapping to the local folder.

```
sudo docker run -d --name apache \
-p 80:80 -p 443:443 \
--restart always \
-v /home/michael/html:/usr/local/apache2/htdocs \
-v /home/michael/.config/php.ini:/etc/php/8.2/apache2/php.ini \
httpd
```

Create a file `index.html` in the local folder (`html`).
### Check if Apache is Running

Test the new Apache installation by opening a web browser and typing the server's [IP address](https://phoenixnap.com/glossary/what-is-an-ip-address):

```
http://[server-ip-address]
```

Alternatively, connect with the Apache installation on the same machine by typing **`localhost`** and the host port you assigned to the container.

## Certificate
This is quite complicated!

Create a folder in the local html folder.

```
mkdir ~/html/.well-known
mkdir ~/html/.well-known/acme-challenge
```

Call  certbot to create the certificates.

```
sudo certbot certonly --manual -d mittlebenskrise.dynv6.net
```

You will be prompted with a text like this:

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Create a file containing just this data:

2V60YYKC5Zqp3pEfwADf9JCAYMgPTb5Tf-DRNlFoCHg.6nrNm5RU8K-Z9kxu0Xe8N3vwoisOm8w4_4dpkllh0tI

And make it available on your web server at this URL:

http://mittlebenskrise.dynv6.net/.well-known/acme-challenge/2V60YYKC5Zqp3pEfwADf9JCAYMgPTb5Tf-DRNlFoCHg

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Now create the matching file with the correct content under `~/html/.well-known/acme-challenge` like this:

```
echo 2V60YYKC5Zqp3pEfwADf9JCAYMgPTb5Tf-DRNlFoCHg.6nrNm5RU8K-Z9kxu0Xe8N3vwoisOm8w4_4dpkllh0tI > ~/html/.well-known/acme-challenge/2V60YYKC5Zqp3pEfwADf9JCAYMgPTb5Tf-DRNlFoCHg
```

Once the file exists, press Enter. The certificate should now be created like this:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/mittlebenskrise.dynv6.net/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/mittlebenskrise.dynv6.net/privkey.pem
This certificate expires on 2024-03-03.
These files will be updated when the certificate renews.
```

Now, copy the certificates to the HTML folder:

```
sudo cp /etc/letsencrypt/live/mittlebenskrise.dynv6.net/fullchain.pem ~/html/server.crt
sudo cp /etc/letsencrypt/live/mittlebenskrise.dynv6.net/privkey.pem ~/html/server.key
sudo chown michael ~/html/server.*
```

In Portainer open a terminal to the docker container. The files should now be in the htdocs folder. Move them to the conf folder:

```
mv htdocs/server.* conf
```

Now activate SSL by running the following command in the Portainer terminal:

```dockerfile
sed -i \
		-e 's/^#\(Include .*httpd-ssl.conf\)/\1/' \
		-e 's/^#\(LoadModule .*mod_ssl.so\)/\1/' \
		-e 's/^#\(LoadModule .*mod_socache_shmcb.so\)/\1/' \
		conf/httpd.conf
```

Now HTTPS to your domain should work. Test it by calling the domain web site:

```
https://mittlebenskrise.dynv6.net/
```

Note:
```
sudo certbot certonly --expand --manual -d home.mittlebenskrise.de,*.home.mittlebenskrise.de
```
### PHP

Open the docker terminal and enter

```
apt-get update
apt-get upgrade
```

Now install PHP.

```
apt-get install php
```

Next, install the PHP module for Apache.

```
apt install libapache2-mod-php
```

To verify your installation, you can run the following PHP `phpinfo` script:

```
<?php
  phpinfo();
?>
```

You can save the content in a file – `phpinfo.php` for example – and place it under the `DocumentRoot` directory of the Apache2 web server. Pointing your browser to `http://hostname/phpinfo.php` will display the values of various PHP configuration parameters.

In `httpd.conf` add the following line (if it does not exist):

```
LoadModule php_module /usr/lib/apache2/modules/libphp8.2.so
```

Find the DirectoryIndex and add `index.php` to it:

```
<IfModule dir_module>
    DirectoryIndex index.html
    DirectoryIndex index.php
</IfModule>
```

At the end of the configuration file include the PHP config file:

```
Include /etc/apache2/mods-available/php8.2.conf
```

#### mysqli

Have the Apache container join the same network as MariaDB (e.g. `mysql`).

In order for PHP to be able to use mysqli, the following needs to be done. First, install mysqli:

```
apt-get install php-mysqlnd
```

Next, enable it:

```
phpenmod mysqli
```

Check if it is installed by calling

```
php -m
```

Restart Apache.
#### intl support

```
apt install php8.2-intl

phpenmod intl
```

Restart Apache.

#### mbstring

```
apt install php8.2-mbstring

phpenmod mbstring
```

#### phpxml

```
apt-get install php-xml

phpenmod xml
```

phpzip

```
apt install php-zip

phpenmod zip
```
##### Potentially necessary

In the `php.ini` file (e.g. at `/etc/php/8.2/apache2/php.ini`) uncomment the `extension=` lines for mysqli and intl.
### Security

see also https://www.linux-magazin.de/ausgaben/2018/07/webserver-security/

In `httpd.conf` search for `<Directory` and add `Options -Indexes` as follows:

```
<Directory "/usr/local/apache2/htdocs">
    Options Indexes FollowSymLinks
    Options -Indexes
</Directory>
```

At the end add the following lines:

```
ServerSignature Off
ServerTokens Prod
```

Find the PHP ini file:

```
php --ini
```

Edit the ini file and search for `expose_php`. Set it to `Off`.

```
expose_php Off
```

Verify that no Apache version is returned anymore:

```
curl -I -L https://mittlebenskrise.dynv6.net
```
