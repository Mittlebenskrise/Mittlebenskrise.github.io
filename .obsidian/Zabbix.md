## Zabbix Server


## Zabbix Agent
Find the latest version of the agent on https://www.zabbix.com/download?zabbix=6.4&os_distribution=ubuntu&os_version=22.04&components=agent_2&db=&ws=

Select the Zabbix version, `Ubuntu`, the OS Version and `Agent 2`.

Now follow the steps below:

#### Install and configure Zabbix for your platform

a. Install Zabbix repository

```
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
```

For Raspberry Pi use 

```
wget https://repo.zabbix.com/zabbix/6.4/raspbian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb
```

b. Install Zabbix agent2

```
sudo apt install zabbix-agent2 zabbix-agent2-plugin-*
```

c. Configure Zabbix

After the installation is complete, you need to configure the Zabbix Agent to communicate with the Zabbix server. Edit the Zabbix Agent configuration file using your preferred text editor:

```
sudo nano /etc/zabbix/zabbix_agent2.conf 
```

Find the following lines and replace them with the correct information:

```
Server=<Zabbix_Server_IP>

ServerActive=<Zabbix_Server_IP>

Hostname=<Hostname_Of_Ubuntu_Client>
```

Replace `<Zabbix_Server_IP>` with the IP address of your Zabbix server, and `<Hostname_Of_Ubuntu_Client>` with the hostname of your Ubuntu client (e.g. `Home Server`). Save the changes and close the editor.

d. Add Zabbix user to Docker

For the Zabbix agent to monitor Docker, you’ll need to add the `zabbix` user to the `docker` group:

```
sudo usermod -aG docker zabbix
```

e. Start Zabbix agent process

Start Zabbix agent process and make it start at system boot.

```
sudo systemctl restart zabbix-agent2
sudo systemctl enable zabbix-agent2
```

You can check the status of the Zabbix Agent service with the following command:

```
sudo systemctl status zabbix-agent2
```

The agent will listen on port `10050` for connections from the server. Configure UFW to allow connections to this port:

```
sudo ufw allow 10050/tcp
```
