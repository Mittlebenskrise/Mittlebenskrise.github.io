#### Step 1: Install Samba on Ubuntu

First, you need to **install Samba** on the Linux operating system. Type the following command to install Samba and press **ENTER**.

```bash
sudo apt-get install samba
```

Type the following command to **assign a password** for the Samba service.

```bash
sudo smbpasswd -a michael
```

#### Step 2: Configure Samba on Ubuntu

Configure the Samba service and make it suitable to share files with Windows. For that, run the given command in the terminal.

```
sudo nano /etc/samba/smb.conf
```

At the beginning in the `global` section add the following lines:

```
   wins support = yes

   client min protocol = SMB3
   client max protocol = SMB3
   restrict anonymous = 2
   encrypt passwords = true
```

At the end add the shares like this:

```
[share]
path = /home/michael/share
browsable = yes
writable = yes
read only = no
available = yes
valid users = michael
public = yes

[book]
path = /media/book3
browsable = yes
writable = yes
read only = no
available = yes
valid users = michael
public = yes
```

Note: To be able to use symbolic links, add...

Run the following command to **start Samba** in Linux.

```bash
sudo systemctl enable smbd
sudo systemctl start smbd
```

Make sure that the owner of the share is the samba account!

```
sudo chown michael /media/book3
```
