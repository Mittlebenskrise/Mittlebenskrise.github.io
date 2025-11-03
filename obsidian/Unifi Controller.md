from https://github.com/jacobalberty/unifi-docker

## Setting up, Running, Stopping, Upgrading

### Setting up directories

~~_One-time setup:_ create the `unifi` directory on the Docker host. Within that directory, create two sub-directories: `data` and `log`.~~

```shell
cd
mkdir unifi
mkdir -p unifi/data
mkdir -p unifi/log
```

### Running Unifi-in-Docker

Start the Unifi controller in Docker.

```
sudo docker run -d --init \
   --restart=unless-stopped \
   -p 8080:8080 -p 8443:8443 -p 3478:3478/udp \
   -e TZ='Europe/Berlin' \
   -v unifi:/unifi \
   --user unifi \
   --name unifi \
   jacobalberty/unifi
```

```
   -e "VIRTUAL_HOST=unifi.home.mittlebenskrise.de" \
   -e "VIRTUAL_PORT=8443" \
   -e "VIRTUAL_PROTO=https" \
   -e "LETSENCRYPT_HOST=home.mittlebenskrise.de" \
   -e "LETSENCRYPT_EMAIL=michael.schneider@gmail.com" \
```

Go to [https://docker-host-address:8443](https://docker-host-address:8443/) to complete configuration from the web (initial install) or resume using Unifi Controller.

**Important:** Two points to be aware of when you're setting up your Unifi Controller:

- When your browser initially connects to the link above, you will see a warning about an untrusted certificate. If you are _certain_ that you have typed the address of the Docker host correctly, agree to the connection.
- See the note below about **Override "Inform Host" IP** so your Unifi devices can "find" the Unifi Controller.

### Stopping Unifi-in-Docker

To change options, stop the Docker container then re-run the `docker run...` command above with the new options. _Note:_ The `docker rm unifi` command simply removes the "name" from the previous Docker image. No time-consuming rebuild is required.

```shell
docker stop unifi
docker rm unifi
```

### Upgrading Unifi Controller

All the configuration and other files created by Unifi Controller are stored on the Docker host's local disk (`~/unifi` by default.) No information is retained within the container. An upgrade to a new version of Unifi Controller simply retrieves a new Docker container, which then re-uses the configuration from the local disk. The upgrade process is:

1. **MAKE A BACKUP** on another computer, not the Docker host _(Always, every time...)_
2. Stop the current container (see above)
3. Enter `docker run...` with the newer container tag (see [Current Information](https://github.com/jacobalberty/unifi-docker#current-information) section below.)

## Adopting the Access Points

Access Points:
192.168.178.2/3 (EG/DG)
Login is mlk user name and mv$ password
Call `set-inform http://192.168.178.26:8080/inform` to adopt AP multiple times
