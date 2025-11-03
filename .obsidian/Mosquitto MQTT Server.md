## Installation

Get the Docker image by calling

```
sudo docker pull eclipse-mosquitto
```

Create the config file in `~/.config/mosquitto.conf` like this

```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
allow_anonymous true
listener 1883
```

Start the container by calling

```
sudo docker run -it --name mosquitto \
--restart=unless-stopped \
-p 1883:1883 \
-p 9001:9001 \
-v ~/.config/mosquitto.conf:/mosquitto/config/mosquitto.conf \
-v /mosquitto/data \
-v /mosquitto/log eclipse-mosquitto
```

## Send system status to MQTT server

Get the Python script for retrieving the sensors and copy it to `/usr/bin/system_sensors.py`:

```
git clone https://github.com/Sennevds/system_sensors.git

sudo cp system_sensors/src/system_sensors.py /usr/bin/system_sensors.py
sudo chmod a+x /usr/bin/system_sensors.py
```

Create the configuration file `/etc/mqtt_settings.yaml` as follows:

```
mqtt:
  hostname: 192.168.178.21
  port: 1883
deviceName: mini-pc
client_id: mini-pc
timezone: Europe/Berlin
update_interval: 60 #Defaults to 60
check_wifi_strength: true
check_wifi_ssid: false
check_available_updates: true
external_drives:
  Boot: /boot/
  EFI:  /boot/efi/
  SSD:  /media/ssd/
  Book1: /media/book1/
  Book2: /media/book2/
  Backup: /media/backup/
```

Now test the configuration by calling

```
/usr/bin/system_sensors.py /etc/mqtt_settings.yaml
```

Check in Home Assistant if the MQTT data shows up for the configured device.

Create a service file under `/etc/systemd/system`, e.g. by calling

```
sudo nano /etc/systemd/system/mqtt-report.service
```

In that file configure the MQTT report script as follows:

```
[Unit]
Description=MQTT Report

[Service]
User=root
ExecStart=/usr/bin/system_sensors.py /etc/mqtt_settings.yaml
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Start and enable the service:

```
sudo systemctl start mqtt-report
sudo systemctl enable mqtt-report
```

## Send Status of Raspberry Pi

Install the [reporter demon](https://github.com/ironsheep/RPi-Reporter-MQTT2HA-Daemon) on the Raspberry Pi by calling

```
sudo apt-get install git python3 python3-pip python3-tzlocal python3-sdnotify python3-colorama python3-unidecode python3-apt python3-paho-mqtt python3-requests
```

Run `ifconfig` to ensure everything is properly installed. If so, install the demon script:

```
sudo git clone https://github.com/ironsheep/RPi-Reporter-MQTT2HA-Daemon.git /opt/RPi-Reporter-MQTT2HA-Daemon

cd /opt/RPi-Reporter-MQTT2HA-Daemon
sudo pip3 install -r requirements.txt
```

To match personal needs, all operational details can be configured by modifying entries within the file [`config.ini`](https://github.com/ironsheep/RPi-Reporter-MQTT2HA-Daemon/blob/master/config.ini.dist). The file needs to be created first:

```shell
sudo cp /opt/RPi-Reporter-MQTT2HA-Daemon/config.{ini.dist,ini}
sudo nano /opt/RPi-Reporter-MQTT2HA-Daemon/config.ini
```

Locate and configure the following (at a minimum) in config.ini:

```shell
fallback_domain = {if you have older RPis that dont report their fqdn correctly}
# ...
hostname = {your-mqtt-broker}
# ...
discovery_prefix = {if you use something other than 'homeassistant'}
# ...
base_topic = {your home-assistant base topic}

# ...
username = {your mqtt username if your setup requires one}
password = {your mqtt password if your setup requires one}
```

Perform a test run as follows:

```shell
python3 /opt/RPi-Reporter-MQTT2HA-Daemon/ISP-RPi-mqtt-daemon.py --config /opt/RPi-Reporter-MQTT2HA-Daemon
```

In order to start the reporter at runtime, first adjust the daemon user group accordingly:

```shell
# list current groups
groups daemon
$ daemon : daemon

# add video if not present
sudo usermod daemon -a -G video

# list current groups
groups daemon
$ daemon : daemon video
#                 ^^^^^ now it is present
```

_NOTE: Yes, `video` is correct. This appears to be due to accessing the GPU temperatures._

Set up the script to be run as a system service as follows:

```shell
sudo ln -s /opt/RPi-Reporter-MQTT2HA-Daemon/isp-rpi-reporter.service /etc/systemd/system/isp-rpi-reporter.service

sudo systemctl daemon-reload

# tell system that it can start our script at system startup during boot
sudo systemctl enable isp-rpi-reporter.service

# start the script running
sudo systemctl start isp-rpi-reporter.service

# check to make sure all is ok with the start up
sudo systemctl status isp-rpi-reporter.service
```

**NOTE:** _Please remember to run the 'systemctl enable ...' once at first install, if you want your script to start up every time your RPi reboots!_

Use the rpi-monitor-card to display the status as decribed [here](https://github.com/ironsheep/lovelace-rpi-monitor-card).

## Send Status of Docker Containers

Create a network named `mqtt` in Docker. Then run

```
sudo docker run \
--name docker2mqtt \
--network mqtt \
--restart=always \
-e DOCKER2MQTT_HOSTNAME=mini-pc \
-e MQTT_CLIENT_ID=docker2mqtt \
-e MQTT_HOST=192.168.178.26 \
-e MQTT_TOPIC_PREFIX=docker \
-v /var/run/docker.sock:/var/run/docker.sock \
skullydazed/docker2mqtt
```

Now all Docker containers will report their status to home assistant.

In order to better visualize the docker container status, install the Home Assistant plugins [auto-entities](https://github.com/thomasloven/lovelace-auto-entities) and [template-entity-row](https://github.com/thomasloven/lovelace-template-entity-row). Then create a card with a list of all docker containers like this:

```
type: custom:auto-entities
card:
  type: vertical-stack
card_param: cards
filter:
  include:
    - entity_id: binary_sensor.docker_*
      options:
        type: custom:template-entity-row
        name: >-
          {{ state_attr(config.entity, 'friendly_name') | replace('Docker ', '')
          }}
        secondary: 'Image: {{ state_attr(config.entity, ''image'') }}'
        state: '{{ state_attr(config.entity, ''status'') }}'
        active: '{{ is_state_attr(config.entity, ''status'', ''running'') }}'
        color: |-
          {% if is_state_attr(config.entity, 'status', 'running') %}
            green
          {% else %}
            {% if is_state_attr(config.entity, 'status', 'paused') %}
              darkorange
            {% else %}
              {% if is_state_attr(config.entity, 'status', 'stopped') %}
                firebrick
              {% else %}
                gray
              {% endif %}
            {% endif %}
          {% endif %}
show_empty: true
sort:
  method: friendly_name
```