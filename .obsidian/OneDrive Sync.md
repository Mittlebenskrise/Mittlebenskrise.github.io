## Docker Container
Create a `docker-compose.yml` like this:

```
services:
    onedrive:
        image: driveone/onedrive:edge
        container_name: onedrive-backup
        stdin_open: true # docker run -i
        tty: true        # docker run -t
        restart: unless-stopped
        environment:
            - ONEDRIVE_UID=1000
            - ONEDRIVE_GID=1000
            - ONEDRIVE_UPLOADONLY=1
            # - ONEDRIVE_RESYNC=1
        volumes:
            - /home/michael/docker-data/onedrive-backup/config:/onedrive/conf
            - /media/backup:/onedrive/data
```

- Make sure the folder configured for `/onedrive/data` exists.
- Download the default configuration file from [here](https://raw.githubusercontent.com/abraunegg/onedrive/master/config).
- ~~Adjust the configuration file if necessary (e.g. by setting the `upload_only` or `download_only` options). There is [a list of all available commands](https://github.com/abraunegg/onedrive/blob/master/docs/USAGE.md#all-available-commands) for all possible keys and their default values.~~
- Start the container by calling `sudo docker compose up -d`.
- Get the container's id by calling `sudo docker ps` or using [[Portainer]].
- Stop the container by calling `sudo docker stop <container-id>`.
- Restart the container in interactive mode by calling `sudo docker start <container-id> --interactive`.
- Copy the URL to connect to OneDrive and open it in a web browser.
- Authenticate in the browser and grant the rights to access OneDrive.
- An empty page will appear. Copy the URL and paste it into the container terminal.
- Now, the container should run and start the backup.

***Note:***
You can setup multiple Docker containers with different OneDrive accounts by using multiple `docker-compose.yml` files and multiple configuration folders.

There is also a [list of supported Docker environment variables](https://github.com/abraunegg/onedrive/blob/master/docs/Docker.md#supported-docker-environment-variables).

Further details can be found [here](https://github.com/abraunegg/onedrive) and [here](https://forums.unraid.net/topic/132486-guide-set-up-the-onedrive-client-for-linux-abraunegg-as-docker-container/).

