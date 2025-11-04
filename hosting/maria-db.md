# MariaDB

Once Docker is running, let’s first start the database server, since this will be used by the other containers. First, we will need to add the database user name (typically `root`) and password to our secrets file:

    sudo nano /docker/docker-compose/secrets/.env

Add the following lines and save it:

    # Maria-DB
    DB_USER=root
    DB_PASSWORD=<db_root_password>

Now, we can create the Docker Compose folder and file:

    mkdir/docker/docker-compose/mariadb
    nano /docker/docker-compose/mariadb/docker-compose.yml

Copy the following lines in the `docker-compose.yml` file and save it:

    services:
      mariadb:
        container_name: "mariadb"
        environment:
          - "MYSQL_ROOT_PASSWORD=$DB_PASSWORD"
          - "TZ=$TZ"
        image: "mariadb:latest"
        networks:
          - "mysql"
        restart: "always"
    
    networks:
      mysql:
        external: true
        name: "mysql"

This will create a container named “`mariadb`” and an internal network “`mysql`” to access it from other containers. Start it by calling our `docker-compose` script:

    cd /docker/docker-compose/mariadb
    docker-compose

phpMyAdmin
----------

To be able to administer the databases easily, I recommend installing [phpMyAdmin](https://www.phpmyadmin.net/):

    mkdir/docker/docker-compose/phpmyadmin
    nano /docker/docker-compose/phpmyadmin/docker-compose.yml

Paste the following text:

    services:
      phpmyadmin:
        container_name: "phpmyadmin"
        image: "phpmyadmin"
        ports:
          - "8088:80"
        restart: "unless-stopped"
        environment:
          PMA_HOST: mariadb
        networks:
          - mysql
    
    networks:
      mysql:
        external: true

Again, start it by calling our `docker-compose` script:

    cd /docker/docker-compose/phpmyadmin
    docker-compose

Now you can open a browser to log into the database server by entering the URL

    http://<server-ip>:8088/

Log in as user `root` with the password you configured earlier (`<db_root_password>`).

Next, the configuration tables for phpMyAdmin need to be created. For that, a new user `pma` needs to be created on the accounts tab. Download the database script from [GitHub](https://github.com/phpmyadmin/phpmyadmin/blob/master/resources/sql/create_tables.sql). At the top, the following lines need to be activated, and `@localhost` needs to be removed:

    GRANT SELECT, INSERT, DELETE, UPDATE, ALTER ON `phpmyadmin`.* TO
       'pma';

Now paste and execute the modified script in the “SQL” tab.

/info To install additional themes, download the matching ZIP file and unzip it (e.g., by calling `unzip darkwolf-5.2.zip`). 
Then, copy the unzipped folder into the container by calling
```
sudo docker cp darkwolf phpmyadmin:/var/www/html/themes/darkwolf
```
