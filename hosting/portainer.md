# Install Portainer

## Steps
See https://docs.portainer.io/start/install-ce/server/docker/linux

First, create the `docker-compose.yml`:

```bash
mkdir /docker/docker-compose/portainer/
nano /docker/docker-compose/portainer/docker-compose.yml
```

Paste the following text in the editor:

```bash
services:
  portainer:
    container_name: "portainer"
    image: "portainer/portainer-ce:sts"
    network_mode: "bridge"
    environment:
      - TZ=$TZ
    ports:
      - "8000:8000"
      - "9443:9443"
    restart: "always"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "portainer_data:/data"

volumes:
  portainer_data:
    external: true
```

Then, install and run the Portainer Server container:

```bash
cd /docker/docker-compose/portainer/
docker-compose
```

Test by running

```bash
sudo docker ps 
```

The output should look like this:

```console
CONTAINER ID   IMAGE                          COMMAND                  CREATED       STATUS      PORTS                                                                                  NAMES             
de5b28eb2fa9   portainer/portainer-ce:latest  "/portainer"             2 weeks ago   Up 9 days   0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   portainer
```

Now that the installation is complete, you can log into your Portainer Server instance by opening a web browser and going to:

https://localhost:9443

Replace `localhost` with the relevant IP address or FQDN if needed, and adjust the port if you changed it earlier.

You will be presented with the initial setup page for Portainer Server. This must be done right away, or you will receive a timeout. In that case, restart Portainer as follows:

```bash
sudo docker restart portainer
```

Your first user will be an administrator. The username defaults to `admin`, but you can change it if you prefer. The password must be at least 12 characters long and meet the listed password requirements.

![](https://content.gitbook.com/content/tLcRoAdw9BYwwpba4ZAD/blobs/4OY85MUL72xyYCYACqDF/2.15-install-server-setup-user.png)

#### Enabling or disabling the collection of statistics

We use a tool called [Matomo](https://matomo.org/) to collect anonymous information about how Portainer is used. We recommend enabling this option so we can make improvements based on usage.
For more about what we do with the information we collect, read our [privacy policy](https://www.portainer.io/privacy-policy).

During installation, you can enable or disable connection statistics using the checkbox. If you change your mind later, you can easily update this option
under [Settings](/admin/settings) in the Portainer UI.

![](https://content.gitbook.com/content/tLcRoAdw9BYwwpba4ZAD/blobs/xlvsei3pVFudXI47tpB6/2.15-install-server-setup-matomo.png)

#### Connecting Portainer to your environments

Once the admin user has been created, the **Environment Wizard** will automatically launch. The wizard will help get you started with Portainer.

![](https://content.gitbook.com/content/tLcRoAdw9BYwwpba4ZAD/blobs/FtRZmL9hDFUzcV2uypRg/2.15-install-server-setup-wizard.png)

The installation process automatically detects your local environment and sets it up for you. If you want to add additional environments to manage with this Portainer instance,
click **Add Environments**. Otherwise, click **Get Started** to start using Portainer!
