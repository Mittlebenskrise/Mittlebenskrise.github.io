https://www.libe.net/en-docker-mini-server (text)
https://www.youtube.com/watch?v=os3ysehVzkY (video)

## Steps
See https://docs.docker.com/engine/install/ubuntu/

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

1. Set up Docker's `apt` repository.
   
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
  
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
   
2. Install the Docker packages.
  
To install the latest version, run:

```console
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verify that the Docker Engine installation is successful by running the `hello-world` image.
 
```console
$ sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits. 

You have now successfully installed and started Docker Engine.

## Fix Network Problem (not enough IP addresses for compose)

Create/open Docker's `daemon.json` by calling

```
sudo nano /etc/docker/daemon.json
```

Edit it like this:

```
{
  "log-driver": "json-file",
  "log-opts": {"max-size": "10m", "max-file": "3"},
  "default-address-pools":[
        {"base":"172.17.0.0/12","size":20},
        {"base":"10.99.0.0/12","size":20},
        {"base":"192.168.0.0/16","size":24}
    ]
}
```

Restart Docker:

```
sudo service docker restart
```
## Changing the Timezone of an Existing Docker Container

To change the timezone in an already running Docker container, you need to perform a few additional steps. First, get into the Docker container’s shell using the **`docker exec`** command.

```
docker exec -it container_id /bin/bash 
```

Once you’re in the container’s shell, install the tzdata package. **tzdata** is a time zone and daylight-saving time database used by several systems (like UNIX systems).

Here is the command to install tzdata:

```
apt-get update && apt-get install -y tzdata 
```

After installing tzdata, you can reconfigure it to set the timezone.

```
dpkg-reconfigure tzdata 
```

This command will open a simple GUI where you can select the geographical area and then the city to set the timezone.

## Overcommit error

If there is an error like this in the log of a container:

```
WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition.
```

You'll need to run this command on the docker host and reboot it:

```
echo "vm.overcommit_memory = 1" | sudo tee /etc/sysctl.d/nextcloud-aio-memory-overcommit.conf
```

## Create Docker Compose file from Container

See https://www.makeuseof.com/create-docker-compose-files-from-running-docker-containers/

## Updating a Container

**Fetch the Latest Image**
Navigate to the Docker Compose directory of the container and run:

```
sudo docker compose pull
```

Docker will retrieve the newest container image.

**Deploy the Updated Container**
Update the conatiner to the new image by calling:

```
sudo docker compose up -d
```

Combined into one call:

```
sudo docker compose pull && sudo docker compose up -d
```

## Docker Command Line

Open a shell:

```
sudo docker exec -it <container-name-or-id> bash
```

Restart a container:

```
sudo docker restart <container-name-or-id>
```

## Docker Compose

Create a file named `docker-compose.yml`. Then call

```
sudo docker compose up -d
```

Force a recreation of the container by calling

```
sudo docker compose up -d --force-recreate
```

In order to turn a `docker run` command into a `docker-compose.yml`, use https://www.composerize.com/.

|                                                                                  |
| -------------------------------------------------------------------------------- |
| `sudo chown root:root ~/docker/secrets`<br><br>`sudo chmod 600 ~/docker/secrets` |
## How to stop all Docker Containers
From Hukot net 12.05.2022

To stop all Docker containers simply run the following command in your terminal:

```
docker kill $(docker ps -q)
```

How It Works
The docker ps command will list all running containers.
The -q flag will only list the IDs for those containers.
Once we have the list of all container IDs, we can simply run the docker kill command, passing all those IDs, and they’ll all be stopped!
## Start Docker only when Mount exists
https://unix.stackexchange.com/questions/337301/how-to-start-a-systemd-service-after-mount-command

Call `sudo systemctl edit docker` and add the following lines at the beginning:

```
[Unit]
ConditionPathIsMountPoint=/media/backup
ConditionPathIsMountPoint=/media/app-data
...
```

## Large Overlay2 folder

see also https://buisteven.medium.com/debugging-docker-overlay2-out-of-space-d1edc2ea412f

Determine, which overlay folder is large by calling

```
sudo ncdu /var/lib/docker/overlay2
```

This will list the folders, but they are hashed. To determine, which container uses which overlay folder, call

```
docker inspect $(docker ps -qa) |  jq -r 'map([.Name, .GraphDriver.Data.MergedDir]) | .[] | "\(.[0])\t\(.[1])"'
```

## DTop

```
curl -sSfL https://amir20.github.io/dtop/install.sh | bash

sudo dtop
```