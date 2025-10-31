Now that we have the basics prepared, we can install Docker to run our applications.

Installation
------------

Follow the steps described in the official [installation guide](https://docs.docker.com/engine/install/ubuntu/). Use the latest version of Docker using the [apt repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).

_**Note:**_  
Since I eventually ran into the [overcommit error](https://github.com/nextcloud/all-in-one/discussions/1731), I suggest you make the necessary adjustments right away by calling:

    echo "vm.overcommit_memory = 1" | sudo tee /etc/sysctl.d/nextcloud-aio-memory-overcommit.conf

And now you’ll need to reboot.

    sudo reboot

Directory Structure
-------------------

Next, let’s create the directory structure we need for our setup. This is a suggestion and you can do it completely differently, but all my future posts will work with this setup.

Create two folders for the Docker Compose files and the runtime data:

    cd /
    sudo mkdir docker
    chown <username>:<groupname> docker
    cd docker
    mkdir docker-compose
    mkdir docker-data

Now, in the Docker Compose directory, let’s create a directory for our secrets, e.g. passwords, which we don’t want everybody to be able to read. So, we will put them in a folder, which can only be accessed by the super user:

    cd docker-compose
    mkdir secrets
    sudo chown root:root secrets
    sudo chmod 600 secrets

Next, create the first two secrets, i.e. username and password used to log in to your SMTP server (since several containers will like to send emails).

    sudo nano secrets/.env

In this file, we will store the secrets. So, for now, just copy these two lines in, adjust them to your SMTP server, and save the file:

    SMTP_USERNAME=<username for your SMTP server>
    SMTP_PASSWORD=<password>

Finally, let’s define the global settings, which we will use in our Docker Compose files:

    nano .env

There are several settings, we will constantly use, so paste these lines into the empty file and adjust them to your needs:

    # general server settings
    SERVER_IP=<server-ip>
    PUID=<your UID>
    PGID=<your GID>
    TZ="Europe/Berlin"
    
    # general path settings
    DOCKER_ROOT="/docker"
    DOCKER_COMPOSE_DIR="$DOCKER_ROOT/docker-compose"
    DOCKER_DATA_DIR="$DOCKER_ROOT/docker-data"
    SECRETS_DIR="$DOCKER_COMPOSE_DIR/secrets"
    
    # email settings
    SMTP_SERVER=<your SMTP server, e.g. smtp.gmail.com:587>
    DEFAULT_EMAIL=<email used for sending>

You can determine UID and GID by calling `id` from the shell. The output will look like this:

    uid=<your UID>(<username>) gid=<your GID>(<username>) groups=1000(<username>), …

The last step for this to work is to create a script that uses the global settings when running `docker compose`:

    cd ~
    mkdir -p .local/bin
    echo > .local/bin/docker-compose
    chmod a+x .local/bin/docker-compose
    nano .local/bin/docker-compose

This will create an empty executable file and open it in the editor. Now let’s copy the bash script into it and save it:

    #!/bin/bash
    
    # Script for calling docker compose
    
    sudo docker compose --env-file /docker/docker-compose/.env --env-file /docker/docker-compose/secrets/.env up -d

And we are done! Instead of calling Docker Compose directly to start a container, we will from now on just call `docker-compose`.

Why use a script instead of just calling `docker compose`? Well, for our secrets and general settings to work, we must pass them as environment files. And since I’m lazy and most likely will forget the proper call anyway, it’s easier to just pack that in a script.
