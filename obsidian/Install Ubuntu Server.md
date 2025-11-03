https://www.libe.net/en-docker-mini-server (text)
https://www.youtube.com/watch?v=os3ysehVzkY (video)

## Steps

Install Ubuntu from a stick as described [here](https://help.ubuntu.com/community/Installation/FromUSBStick).

If the installation medium is inserted (DVD or USB stick), the GRUB bootloader will announce itself at the next start, see also: [Starting the computer from USB or DVD | UEFI / BIOS - Boot](https://www.libe.net/en-boot).

![](https://www.libe.net/storage/671x158/63020e808837b.jpg)

![](https://www.libe.net/storage/970x304/63020e80af296.jpg)

The wizard guides us through the network settings and setting up the profile:  
![](https://www.libe.net/storage/921x209/63020e80e6c41.jpg)

![](https://www.libe.net/storage/456x372/63020e8125e17.jpg)

To be able to administer the server over the network, I enabled the OpenSSH server:

![](https://www.libe.net/storage/920x266/63020e8139968.jpg)Docker could be easily enabled as a "featured server snap" during server installation, however, do not do so, but install it via apt later!

Set time zone at the server! 

## Get Free Ubuntu Pro

Follow [Ubuntu Advantage Client - Documentation / Server Guide - Ubuntu Community Hub](https://discourse.ubuntu.com/t/ubuntu-advantage-client/21788)
Register for landscape: [How to register a Landscape client - Landscape](https://landscape.canonical.com/account/jf8pnxuw/how-to-register)

```
LANDSCAPE_ACCOUNT_NAME='jf8pnxuw'
LANDSCAPE_FQDN='landscape.canonical.com'
LANDSCAPE_COMPUTER_TITLE='Mini-PC'
```

```
sudo landscape-config --silent --account-name="${LANDSCAPE_ACCOUNT_NAME}" --computer-title="${LANDSCAPE_COMPUTER_TITLE}" --tags="" --script-users='nobody,landscape,root' --ping-url="http://${LANDSCAPE_FQDN}/ping" --url="https://${LANDSCAPE_FQDN}/message-system"
```
