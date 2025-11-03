## Installation

Install the smart tools:

```
sudo apt-get install smartmontools
```

Update the database of all drives as follows:

```
sudo update-smart-drivedb
```

Check if the device supports smart status by calling

```
sudo smartctl -i /dev/sda
```

Check the smart status of a device like this:

```
sudo smartctl -a /dev/sda
```

Perform an offline test by calling

```
sudo smartctl -t offline /dev/sda
```

## Don't wake-up sleeping drives

In order to get HDDs to sleep, first determine their UID:

```
sudo blkid
```

Find the UUID for the corresponding drive.

In order to check if `hdparm`  is supported, call

```
sudo hdparm -C /dev/sda
```

If the drive state is reported as e.g. `active/idle`, edit `/etc/hdparm.conf` and add the following lines at the end (time in 5 seconds, here 5 * 120 = 10 minutes):

```
/dev/disk/by-uuid/FCD216FAD216B8BA {
        spindown_time = 120
}
```

If the drive state is reported as `unknown`, try `hd-idle` by calling

```
sudo apt install hd-idle
```

If this should not work, install it manually by first calling

```
wget https://www.foxplex.com/components/uploads/gDzcGMoX-hd-idle_1.05_amd64.deb
```

to get the package (see https://www.foxplex.com/sites/festplatten-standby-im-leerlauf-mit-hdparm-und-hd-idle/) and then installing it by calling

```
sudo dpkg -i gDzcGMoX-hd-idle_1.05_amd64.deb
```

Adjust the configuration by calling

```
sudo nano /etc/default/hd-idle
```

Edit or add the following lines (time for `-i` in seconds, here 5 minutes):

```
START_HD_IDLE=true

HD_IDLE_OPTS="-i 0 -a /dev/disk/by-uuid/FCD216FAD216B8BA -i 300 -l /var/log/hd-idle.log"
```

Start the service and link it to auto start:

```
sudo systemctl start hd-idle
sudo systemctl enable hd-idle
```

Now the hard drive will spin down after the configured time.

To manually spin it down immediately, call

```
sudo hdparm -y /dev/sdc
```

or

```
sudo hd-idle -t sdc
```
