# Ubuntu Installation

Since [www.libe.net](http://www.libe.net/) has a very good installation guide [here](https://www.libe.net/en-docker-mini-server) (I linked to their screenshots in the steps below) and even a video [there](https://www.youtube.com/watch?v=os3ysehVzkY), I will just quickly walk through the steps that I followed and that I consider important. Basically, they follow the official Ubuntu installation guide to install Ubuntu from a stick as described [here](https://help.ubuntu.com/community/Installation/FromUSBStick).

When the installation medium is inserted (DVD or USB stick) and the PC is booted, the GRUB bootloader will appear. If not, you might want to consider [these steps](https://www.libe.net/en-boot).

So, we will select the option to install Ubuntu.

[![GRUB Menu](https://www.libe.net/storage/671x158/63020e808837b.jpg)](https://www.libe.net/storage/671x158/63020e808837b.jpg)

The installation type is a standard Ubuntu server:

[![Installation Type](https://www.libe.net/storage/970x304/63020e80af296.jpg)](https://www.libe.net/storage/970x304/63020e80af296.jpg)

The wizard guides us through the network settings and through setting up the profile:

[![Network Selection](https://www.libe.net/storage/921x209/63020e80e6c41.jpg)](https://www.libe.net/storage/921x209/63020e80e6c41.jpg)

[![Basic Setup](https://www.libe.net/storage/456x372/63020e8125e17.jpg)](https://www.libe.net/storage/456x372/63020e8125e17.jpg)

I went for `mini-pc` as the hostname in my setup, but you can use whatever you like (as long as it is supported by Ubuntu). Here, you also define your username and password.

To be able to administer the server in a headless setup over the network, enable the OpenSSH server:

[![OpenSSH Option](https://www.libe.net/storage/920x266/63020e8139968.jpg)](https://www.libe.net/storage/920x266/63020e8139968.jpg)Docker could be easily enabled as a “featured server snap” during server installation, however, do not do so! We will install it via apt later.

Do not forget to set the time zone on the server! First, check the time zone by calling `timedatectl`. If the time zone is not set correctly, get the correct time zone name (here with Berlin as an example) by calling

    timedatectl list-timezones | grep Berlin

The output will be

    Europe/Berlin

Now change the time zone by calling

    sudo timedatectl set-timezone Europe/Berlin

And you are done! You should now have a working Ubuntu server and we can continue to the next steps.
