## Install Maria DB

In Portainer install the MariaDB server template. Set name (e.g. `mariadb`) and password for the `root` user, the click on "Deploy Container".

Add a new network `mysql` based on `bridge` and have the container join it.

## Install phpMyAdmin

Install phpMyAdmin from the following Docker Compose file:

```
version: "3.6"
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
```

Login via Web:

```
http://hostname:8080
```

Default login for MariaDB is user `root` with the password set during installation of MariaDB.

## Database Backup and Restore

(see also https://mariadb.com/kb/en/container-backup-and-restoration/)

Create a backup script as follows:

```
#!/bin/bash

# Script for creating a backup of the databases in MariaDB.
#
# Usage: backup-db.sh [database database ...]
#

MARIA_DB_CONTAINER=mariadb
MARIADB_ROOT_PASSWORD=<SQL Password>
BACKUP_FILE_ALL=/media/backup/mini-pc/sql/backup-all-databases.sql
BACKUP_FILE_BASE=/media/backup/mini-pc/sql/backup-database-
BACKUP_FILE_EXT=.sql


if [ "$1" == "" ]; then
        echo "Backing up all databases"

        sudo docker exec $MARIA_DB_CONTAINER mariadb-dump --all-databases -uroot -p"$MARIADB_ROOT_PASSWORD" > $BACKUP_FILE_ALL
        exit 0
else
        for var in "$@"
        do
                echo "Backing up database $var"

                sudo docker exec $MARIA_DB_CONTAINER mariadb-dump --databases $var -uroot -p"$MARIADB_ROOT_PASSWORD" > $BACKUP_FILE_BASE$var$BACKUP_FILE_EXT
        done

        exit 0
fi
```

You can now backup all databases by calling this script:

```
# Backup all databases
./backup-db.sh

# Backup databases test and another-test
./backup-db.sh test another-test
```

Create a restore script as follows

```
#!/bin/bash

# Script for restoring a backup to Maria DB.
#
# Usage: restore-db.sh backup-file
#

MARIA_DB_CONTAINER=mariadb
MARIADB_ROOT_PASSWORD=<SQL Password>

# check if the parameters have been provided
[[ -z "$1" ]] && { echo "Usage: restore-db.sh backup-file"; exit 1; }

# check if the provided container id actually exists
if [ ! -f "$1" ];
then
        echo "Backup file $1 does not exist";
        exit 1;
fi

echo "Restoring database backup $1"
sudo docker exec -i $MARIA_DB_CONTAINER sh -c 'exec mariadb -uroot -p'"$MARIADB_ROOT_PASSWORD" < $1

```

You can then restore the database by calling

```
# Restore database test from its backup
./restore-db.sh  /media/backup/mini-pc/sql/backup-database-test.sql
```
