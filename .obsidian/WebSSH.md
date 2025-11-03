Go to the docker compose directory.

Get the latest zip file (here version 1.6.2) and prepare it for Docker Compose:

```
wget https://github.com/huashengdun/webssh/archive/refs/tags/v1.6.2.zip
unzip v1.6.2.zip
rm v1.6.2.zip
cp webssh-1.6.2/docker-compose.yml .
```

Edit `docker-compose.yml`:

```
services:
  web:
    build: webssh-1.6.2
    container_name: webssh
    ports:
    - "8899:8888"
```

Start the container by calling `sudo docker compose up -d`. You can now access WebSSH by calling `<myip>:8899`.