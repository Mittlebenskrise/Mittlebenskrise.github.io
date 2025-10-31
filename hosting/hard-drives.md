# Get Idle Hard Drives to Sleep

At this point, I would like to digress to a topic that was really annoying and took me a bit to resolve. It’s how to spin down hard drives when they are idle. As mentioned [earlier](http://blog.michael-schneider.at/hosting-a-server-at-home), I bought a [5TB USB-3.0 drive from Western Digital](https://amzn.to/4aFJeyc) as an external hard drive for additional storage. When connecting it to the PC, it never stopped spinning. Apart from the wear on the drive, this also produces a constant background noise that I wanted to avoid.

Some research showed that under Ubuntu, some hard drives do not automatically go to sleep when idle. However, there is a way to achieve this. And it works perfectly for me. First, you’ll need to check if the drive’s controller supports `hdparm`. To do so, call

    sudo hdparm -C /dev/sdc

If the drive state is reported as e.g. `active/idle`, it is supported, otherwise not. In my case it reported `drive state is: unknown`.

No matter, if `hdparam` is supported or not, you will need to determine the UID of the drive by calling

    sudo blkid

Find the UUID for the corresponding drive. E.g.

    /dev/sdc1: LABEL="Elements" BLOCK_SIZE="512" UUID="FCD216FAD216B8BA" TYPE="ntfs" PARTLABEL="Elements" PARTUUID="7902be53-dd2a-4613-b580-c89f0525e4fc"

If `hdparam` is supported by your drive, edit `/etc/hdparm.conf` and add the following lines at the end (time is defined in 5-second steps, so here it is 60 \* 5 seconds, i.e. 5 minutes) to set the delay after which the drive shall go to sleep:

    /dev/disk/by-uuid/FCD216FAD216B8BA {
            spindown_time = 60
    }

There are some hard drives out there that do report a correct state when calling `hdparam`, but do not support the spindown time setting. If that is the case with your drive, or if `hdparam` does not work and the drive state is reported as `unknown`, try `hd-idle`. You need to install it by calling

    sudo apt install hd-idle

If this does not work, install it manually by first calling

    wget https://www.foxplex.com/components/uploads/gDzcGMoX-hd-idle_1.05_amd64.deb

to get the package from [here](https://www.foxplex.com/sites/festplatten-standby-im-leerlauf-mit-hdparm-und-hd-idle/) (see section “Einstellung per hd-idle / für harte Fälle”) and then running

    sudo dpkg -i gDzcGMoX-hd-idle_1.05_amd64.deb

Adjust the configuration:

    sudo nano /etc/default/hd-idle

Modify or add the following lines (time for `-i` in seconds, here 5 minutes):

    START_HD_IDLE=true
    
    HD_IDLE_OPTS="-i 0 -a /dev/disk/by-uuid/FCD216FAD216B8BA -i 300 -l /var/log/hd-idle.log"

Start the service and enable it, so that it gets automatically started at boot time from now on:

    sudo systemctl start hd-idle
    sudo systemctl enable hd-idle

Now the hard drive will spin down after the configured time.

To manually spin it down immediately, call

    sudo hdparm -y /dev/sdc

if `hdparam` works or otherwise

    sudo hd-idle -t sdc

**_Note:_**  
_This page contains affiliate links to Amazon. Feel free to order your hardware anywhere, but it would be nice of you to use these links so that I get a small reward for writing all this information up._ 😉
