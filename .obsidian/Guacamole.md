## Installation

Use the existing [[Maria DB Server]].

Install the MariaDB client by running

```
sudo apt install mariadb-client-core
```

On the command line run

```
sudo docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > guac_initdb.sql
```

Open phpMyAdmin and run the following SQL statements:

```
create database guacamole;
CREATE USER 'guacdb_user' IDENTIFIED BY 'my_strong_password';
GRANT ALL ON `guacamole%`.* TO 'guacdb_user';
flush privileges;
```

Copy the script from above to the container:

```
sudo docker cp guac_initdb.sql mariadb:/
```

In [[Portainer]] open the shell and enter

```
cat guac_initdb.sql | mariadb -u guacdb_user -p guacamole;
```

Enter the password.

After the script ran, enter

```
mariadb -u guacdb_user -p
```

Enter the password. In the MariaDB shell enter

```
use guacamole;
show tables;
```

The output should look like this:

```
+---------------------------------------+
| Tables_in_guacamole                   |
+---------------------------------------+
| guacamole_connection                  |
| guacamole_connection_attribute        |
| guacamole_connection_group            |
| guacamole_connection_group_attribute  |
| guacamole_connection_group_permission |
| guacamole_connection_history          |
| guacamole_connection_parameter        |
| guacamole_connection_permission       |
| guacamole_entity                      |
| guacamole_sharing_profile             |
| guacamole_sharing_profile_attribute   |
| guacamole_sharing_profile_parameter   |
| guacamole_sharing_profile_permission  |
| guacamole_system_permission           |
| guacamole_user                        |
| guacamole_user_attribute              |
| guacamole_user_group                  |
| guacamole_user_group_attribute        |
| guacamole_user_group_member           |
| guacamole_user_group_permission       |
| guacamole_user_history                |
| guacamole_user_password_history       |
| guacamole_user_permission             |
+---------------------------------------+
```

In Portainer create a new network `guacamole` based on `bridge`.

Create a file `docker_compose.yml` on the host like this:

```
version: '3'
services:
  guacd:
    container_name: guacd
    image: guacamole/guacd
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - guacamole
  guacamole:
    container_name: guacamole
    image: guacamole/guacamole:latest
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - 8090:8080
    environment:
      GUACD_HOSTNAME: guacd
      MYSQL_HOSTNAME: mariadb
      MYSQL_DATABASE: guacamole
      MYSQL_USER: guacdb_user
      MYSQL_PASSWORD: <SQL Password>
    depends_on:
      - guacd
    networks:
      - mysql
      - guacamole
networks:
  mysql:
    external: true
  guacamole:
    external: true
```

Create the containers by calling

```
sudo docker compose up -d
```

In a browser open Guacamole (the path is mandatory!):

```
http://192.168.178.26:8090/guacamole/
```

The initial user name and password are both `guacadmin`. For security purposes add a new user under **Settings**->**Users**, granting all permissions. Save the user and logout.

Login with the new user, go back to the settings and remove the `guacadmin` user.

## Adding Hosts

