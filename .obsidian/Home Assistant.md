Add https://blog.cyril.by/en/software/single-sign-on/example-sso-with-authelia-and-home-assistant
## Install Docker Container

Create a folder for the container:

```
mkdir -p /home/michael/docker-data/home-assistant/data
```

Create a `docker-compose.yml`:

```
version: "3.6"
services:
  homeassistant:
    container_name: "homeassistant"
    image: "ghcr.io/home-assistant/home-assistant:stable"
    network_mode: "host"
    restart: "always"
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/run/dbus:/run/dbus:ro"
      - "/home/michael/docker-data/home-assistant/data:/config"
```

The network for Home-Assistant must be set to "--net=host" according to the vendor, but the ports used can also be defined: as an example when using a reverse proxy for access from the Internet, see: [Access from the Internet - SSL Let's Encrypt](https://www.libe.net/en-ha-docker#ssl).

Start the container by calling

```
sudo docker compose up -d
```

After starting the container, Home-Assistant is accessible by default with the IP address of the host and port 8123 in the browser:

 [![](https://www.libe.net/storage/300x371/5fa51e2c580c5.jpg)](https://www.libe.net/storage/440x544/5fa51e2c580c5.jpg)[![](https://www.libe.net/storage/300x371/5fa51e2c782f9.jpg)](https://www.libe.net/storage/440x544/5fa51e2c782f9.jpg)

Those running Docker on the same machine can also use http://localhost:8123 for the call, see calling [localhost: IP address "127.0.0.1", "::1" | what is localhost?](https://www.libe.net/en-localhost)

## Update Docker Container

Backup the container as secribed in [[Backup]].

Check and get the latest version:

```
docker pull ghcr.io/home-assistant/home-assistant:stable
```

If this returns "Image is up to date" then you can stop here, otherwise stop the running container:

```
docker stop homeassistant
```

Then remove it from Docker's list of containers:

```
docker rm homeassistant
```

Finally start the container again as described above in the installation section.
## Copying files to HA

E.g. copy the calendar file for the waste schedule into the `www` folder:

```
sudo docker cp ./abfallkalender.ics e71d4b98934e:/config/www/
```


AirTags: https://aguacatec.es/integrar-airtag-en-home-assistant/
https://community.home-assistant.io/t/airtag-integration-user-friendly-device-tracker/680751

Workday: https://jasinski.info/2021/12/16/workday-sensor-in-home-assistant/
