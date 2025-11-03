First get the Docker image:

```
sudo docker pull vaultwarden/server:latest
```

Next, create the directories:

```
mkdir ~/vaultwarden
mkdir ~/docker-compose/vaultwarden
sudo chmod go-rwx ~/vaultwarden
sudo chmod go-rwx ~/docker-compose/vaultwarden

cd ~/docker-compose/vaultwarden
```

Create the Docker container:

Go to `https://argon2.online/` and create a hashed admin password, using the following parameters:
- Salt: generate a random value by pressing the cogwheel
- Parallelism Factor: 4
- Memory Cost: 65540
- Iterations: 3
- Hash Length: 48
- Plain Text Input: Your admin password

Click on 'Generate Hash' and copy the output in encoded form, e.g.

```
$argon2id$v=19$m=65540,t=3,p=4$YXlScUM2S1ZvN004ZTdlWQ$QB0Xvme/Z+6BnvpV8M/gojZUEnvMGFkIzFvEKk8g0nLK0RbeGAGaocWqSzUjQ7MB
```

Create a file in the Vaultwarden docker compose directory (`~/docker-compose/vaultwarden`) by calling `nano .env` and enter

```
VAULTWARDEN_ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$YXlScUM2S1ZvN004ZTdlWQ$QB0Xvme/Z+6BnvpV8M/gojZUEnvMGFkIzFvEKk8g0nLK0RbeGAGaocWqSzUjQ7MB'
```

Create a `docker-compose.yml` file like this:

```
version: "3"
services:
  vaultwarden:
    container_name: vaultwarden
    environment:
      WEBSOCKET_ENABLED: true
      ADMIN_TOKEN: ${VAULTWARDEN_ADMIN_TOKEN}
    image: "vaultwarden/server:latest"
    ports:
      - "3012:3012"
      - "8091:80"
    restart: "unless-stopped"
    volumes:
      - /home/michael/vaultwarden:/data
```

Create the container by calling

```
sudo docker compose up -d
```

Now you can access Vaultwarden in your browser, e.g. at `https://vaultwarden.home.mittlebenskrise.de/`. When opening it for the first time, you'll need to create an account.

The admin page can be accessed at `https://vaultwarden.home.mittlebenskrise.de/admin`.