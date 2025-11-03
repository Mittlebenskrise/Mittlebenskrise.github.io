## Super User

Start super user mode by entering `sudo -s`.

## Disk Space

```
sudo apt-get install ncdu
sudo ncdu /
```

## User Information

To get user and group ID, call `id username`.

## Redirection

Redirect `stdout` and `stderr` both to a file

```
command &> file
```

## Startup and Shutdown Scripts

^2843d3

### Setting Up a Startup Script

Here’s a simple way to set up a startup script in Ubuntu Linux:

1. **Create a Script**:
Open your favorite text editor and write your script. For instance, if you want to update your system every time it starts up, your script could look something like this:
   
```
#!/bin/bash
apt-get update -y && apt-get upgrade -y
```

Remember to include **`#!/bin/bash`** at the beginning of your script to specify that it should be run using the Bash shell.

2. **Make the Script Executable**:

Save your script to a file, for example, **`update.sh`**, and make it executable by running the following command:

```
chmod +x /path/to/your/script/update.sh 
```

3. **Move the Script to the Startup Directory**:
Move your script into the **`/etc/init.d/`** directory, which is used for startup scripts:

```
sudo mv /path/to/your/script/update.sh /etc/init.d/ 
```

4. **Register the Script**: Lastly, register your script with the system to run at startup. You can do this with the **`update-rc.d`** command:

```
sudo update-rc.d update.sh defaults 
```

The defaults keyword here specifies that the script should be run in the default runlevels.


### Setting Up a Shutdown Script

Setting up a shutdown script is much the same as a startup script, with just a slight difference. Here are the steps:

1. **Create a Script**:
Write your shutdown script. For instance, a script that sends a backup of a specific directory to a remote server upon shutdown might look like this:

```
#!/bin/bash
rsync -av /path/to/directory user@remote:/path/to/backup|
```

2. **Make the Script Executable**:
As before, save your script to a file, for instance, **`backup.sh`**, and make it executable:

```
chmod +x /path/to/your/script/backup.sh 
```

3. **Move the Script to the Shutdown Directory**:
Move your script into the **/etc/rc0.d/** directory, which is used for shutdown scripts:

```
sudo mv /path/to/your/script/backup.sh /etc/rc0.d/ 
```

Similarly, if you want your script to run on reboot, you should move it to the **`/etc/rc6.d/`** directory.

4. **Rename the Script**:
Shutdown and reboot scripts require a specific naming convention: **`KXX<name>`**, where **`XX`** is a two-digit number representing the order in which the scripts are run (lower numbers run first) and is the name of the script:

```
sudo mv /etc/rc0.d/backup.sh /etc/rc0.d/K99backup 
```

### Executing a Bash script at startup using systemd

Systemd is a system and service manager in Ubuntu that provides a standardized way of managing and controlling processes. You can use systemd to execute a Bash script at startup in Ubuntu.

1. Create a service unit file for your script. For example, create a file called `myscript.service` in the `/etc/systemd/system/` directory:

```
sudo nano /etc/systemd/system/myscript.service
```

2. Add the following content to the file:

```config
[Unit]

Description=My Script

After=network.target

[Service]

User=root
ExecStart=/path/to/your/script.sh
Restart=always
RestartSec=10

[Install]

WantedBy=multi-user.target
```

Replace `/path/to/your/script.sh` with the actual path to your Bash script.

3. Save the file and exit the text editor.

4. Reload the systemd daemon to load the new service unit file:

```
sudo systemctl daemon-reload
```

5. Enable the service to start at boot:

```
sudo systemctl enable myscript.service
```

6. Start the service:

```
sudo systemctl start myscript.service
```

Now, your Bash script will be executed at startup.

To check the detailled log from the service script, call

```
journalctl -u myscript.service
```

## Timezone

Check the timezone by calling `timedatectl`. If it is not correct, first get the correct timezone name by calling

```
timedatectl list-timezones | grep Berlin
```

The output will be

```
Europe/Berlin
```

Now change the timezone by calling

```
sudo timedatectl set-timezone Europe/Berlin
```

## Cleanup old Mac Files

Do get rid of Mac files, call

```
find . -type f -name '._*' -delete

find . -type f -name '.DS_STORE' -delete
```

## Compare Files and Directories

For a comparison call

```
diff -qr Directory-1 Directory-2

or

diff File-1 File-2
```

## Check Port Usage

Call 

```
sudo lsof -i tcp:<PORT NUMBER>
```

to check, which process is using a certain port.

## Remove empty Files and Folders

```
# remove all empty directories
find . -empty -type d -delete

# remove all empty files
find . -empty -type f -delete
```

## Rename Files

E.g. rename all PDF files so that the new file name consists of the path (replace `destination`):

```
find . -type f -name '*.pdf' -exec rename -n 's:^./::; s:/:_:g; s:^:destination/:' {} +
```

As soon as it works as expected, remove the `-n` flag.
### Move Files into a Folder

To move e.g. all PDF files into a folder (without the folder structure!) call

```
find . -type f -name '*.pdf' -exec mv -nv -t '/target_folder' -- {} +
```
### Count Files in Folder

To count the number of files in the current folder, call

```
ls -1 | wc -l
```
## Backup a Folder

To backup a folder, use `tar` as follows:

```
tar -zcvf backup.tar.gz folder-to-backup
```

## Check CPU Usage


```
# Display processes and CPU usage
htop

# Display the processes in tree view
htop -t

# Using ps (to also see the process IDs), but only display the top CPU using processes
ps -aux --sort -pcpu | head -n 10
```

To quit the **`top`** function, press **q**. Some other useful commands while **`top`** is running are:

- **`M`** - sort task list by memory usage.
- **`P`** - sort task list by processor usage.
- **`N`** - sort task list by process ID.
- **`T`** - sort task list by run time.

## Check File Usage

Run

```
sudo lsof <file>
```

## Check Memory

```
grep MemTotal /proc/meminfo
```

## Installation Problems

When the installation of a package ends in an error, call

```
sudo apt-get purge PROBLEMPAKETNAME
sudo apt-get autoremove|
sudo apt-get install PROBLEMPAKETNAME
```

## Check Filesystem

Run `sudo fdisk -l` to determine, which drive to check (`/dev/sdXX`).

Run `sudo fsck -f /dev/sdXX` to check the file system.