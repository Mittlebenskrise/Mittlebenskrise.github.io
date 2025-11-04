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
