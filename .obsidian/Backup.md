## Automated Backup to USB Drive

Setup a USB drive as described in [[Mount USB Drives]] and create a backup folder on it

```
sudo mkdir -p /media/backup/mini-pc
```

Get the ignorelist for files not to backup:

```
wget https://raw.githubusercontent.com/rubo77/rsync-homedir-excludes/master/rsync-homedir-excludes.txt -O /var/tmp/ignorelist
```

Setup rsync to backup the relevant folders to the backup folder.

Each user can backup the home folder like this:

```
rsync -aP --exclude-from=/var/tmp/ignorelist /home/$USER/ /media/backup/mini-pc/home/$USER/
```

Note: The directories must have the correct access rights, so first a 

```
sudo mkdir -p /media/backup/mini-pc/home/$USER
sudo chown $USER /media/backup/mini-pc/home/$USER
```

***MISSING:*** How to restore...
## Backup from remote Linux machine

Follow the steps described in [[Share files with Windows]] to setup Samba on the backup server and create a share for the backups.

On the client machine create the mount point for the backup:

```
sudo mkdir /media/backup
```

Then change `/etc/fstab` by adding the following line:

```
//192.168.178.26/backup /media/backup cifs credentials=/root/.smb 0 0
```

Add the credentials to the file `/root/.smb`:

```
user=<smb-user>
password=<smb-password>
domain=192.168.178.26
```

Use the correct host IP and the backup share.

Now call

```
sudo mount -a
```

Setup rsync as described above to backup to the share by copying to the mount point (`/media/backup`).
## Backup Docker Container
Man kann einen Snapshot des Docker Containers als Image erstellen und sichern.

Zuerst die Liste der Container ermitteln:

```
docker ps
```

Dann sucht man sich den zu sichernden Container heraus und nutzt entweder dessen `CONTAINER ID` oder den Namen:

```
docker commit -p CONTAINER-ID BACKUP-NAME
```

Verifizieren kann man das nun mit folgendem Befehl, in der Liste sollte nun irgendwo der `BACKUP-NAME` auftauchen

```
docker image ls
```

Um das Image nun auf der Festplatte abzuspeichern:

```
docker save -o /media/backup/mini-pc/containers/backup-name.tar BACKUP-NAME
```

Komprimieren kann man die Datei zum Schluss ebenfalls noch

```
gzip /media/backup/mini-pc/containers/backup-name.tar
```

Nun hat man eine komprimierte Version des Docker Containers.

Der Restore des Images erfolgt dann mit `docker load`:

```
docker load -i /media/backup/mini-pc/containers/backup-name.tar
```

Um den Container wieder herzustellen, kann dann einfach das Image mit `docker run` gestartet werden:

```
docker run -d --name NEWCONTAINERNAME BACKUP-NAME
```

### Script for the docker container backup

``` bash
#!/bin/bash

# Script for creating a backup of a docker container.
#
# Usage: backup-container.sh container-id backup-name
#

# check if the parameters have been provided
[[ -z "$1" ]] && { echo "Usage: backup-container.sh container-id backup-name"; exit 1; }
[[ -z "$2" ]] && { echo "Usage: backup-container.sh container-id backup-name"; exit 1; }

# check if the provided container id actually exists
if [ ! "$(sudo docker ps -a -q -f id=$1)" ];
then
        echo "Container with id $1 does not exist";
        exit 1;
fi

# create the backup of the container and store it in the backup folder
sudo docker commit -p "$1" "$2"
sudo docker save -o /media/backup/mini-pc/containers/"$2".tar "$2"
sudo gzip /media/backup/mini-pc/containers/"$2".tar
```
