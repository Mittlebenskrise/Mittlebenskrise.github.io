As first step, we need to install socat, which can be used to setup a socket communication between a TCP port and a script to exchange data. We need to install it by calling `sudo apt install socat`.  Now, we can start setting up the command listener.

First, we create a service by calling `sudo nano /etc/systemd/system/ha-commands.service` and pasting the following code:

```
[Unit]

Description=Execute Commands from Home Assistant

After=network.target

[Service]

User=root
ExecStart=/usr/local/bin/start-ha-listener.sh
Restart=always
RestartSec=10

[Install]

WantedBy=multi-user.target
```

This will setup a service that calls the script `/usr/local/bin/start-ha-listener.sh` to start `socat` as a service.

In order for the systemd daemon to recognize the new service, call

```
sudo systemctl daemon-reload
```

Next, we create the service script by calling `sudo nano /usr/local/bin/start-ha-listener` and entering this code:

```
#!/bin/bash
socat -u tcp-l:7777,fork system:/usr/local/bin/execute-ha-command.sh
```

Finally, create the script executed when Home Assistant sends a message by calling `sudo nano /usr/local/bin/execute-ha-command.sh` and adding this code:

```
#!/bin/bash
read MESSAGE
if [ "$MESSAGE" = "Update" ]; then
        echo "Updating the OS"
        set -e
        rm -rf /var/lib/dpkg/lock-frontend
        rm -rf /var/lib/dpkg/lock
        apt-get update
        apt-get upgrade -y
        apt-get dist-upgrade -y
        apt-get autoremove -y
        apt-get autoclean -y
else
        echo "Unknown command \"$MESSAGE\" received!"
fi
```

Don't forget to set the scripts to be executable:

```
sudo chmod a+x /usr/local/bin/start-ha-listener.sh
sudo chmod a+x /usr/local/bin/execute-ha-command.sh
```

Now, we can enable the service to be started at boot time:

```
sudo systemctl enable ha-commands.service
```

And next, we can start the service:

```
sudo systemctl start ha-commands.service
```

Now, your Bash script will be executed at startup. To check if it's working, call

```
sudo systemctl status ha-commands.service
```

And do see the detailled log, call

```
journalctl -u ha-commands.service
```

Now, in Home Assistant, we need to add shell script support by adding this line to `configuration.yaml`:

```
shell_command: !include shell_commands.yaml
```

And by creating `shell_commands.yaml` as follows:

```
update_home_server: echo -e "Update" | nc 192.168.178.26 7777
```

This sends a command `Update` to port 7777 on our server. This call will be passed on by `socat` to our script. The script will check the command and then run the appropriate code (in this case a full update of all installed components).

The last step in the setup is, creating an automation in Home Assistant like this:

```
alias: Update Home Server
description: ""
trigger: []
condition: []
action:
  - service: shell_command.update_home_server
    metadata: {}
    data: {}
mode: single
```

This can then either be called manually, or e.g. when clicking on an entity that tells the user about updates:

```
type: entities
entities:
  - entity: sensor.mini_pc_host_os
    name: Version
  - entity: sensor.mini_pc_host_architecture
    name: Architecture
  - entity: sensor.mini_pc_updates
    name: Available Updates
    tap_action:
      action: call-service
      service: automation.trigger
      service_data:
        entity_id: automation.update_home_server
title: Operating System
```

Naturally, the script `/usr/local/bin/execute-ha-command.sh` can be extended to execute more commands to trigger further actions.