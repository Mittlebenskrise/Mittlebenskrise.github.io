Install `rclone`:

```
sudo apt install rclone
```

Use `rclone` to mount OneDrive as follows:

```
sudo rclone config
```

Select `n` to create a new remote and enter a name for the remote (e.g. `onedrive-mount`). Then pick the number for `Microsoft OneDrive` from the list (was 30 today). Leave client ID and client secret empty (i.e. press Enter). Pick `n` for advanced configuration and then `n` (!!!) for automatic configuration, since the server does not have a UI with a web browser.

Next, go to a Windows machine (or any other machine with `rclone` and a browser) and enter

```
rclone authorize "onedrive"
```

A web browser will open and you will need to login to your OneDrive account. After successful login, the `rclone` command will display the authorization token. Copy it and go back to the server. Paste the token into the waiting `rclone` session.
Now, continue with the setup by selecting `OneDrive Personal` and then the server (select global). Finally the configuration will be displayed and stored after pressing `y`.

Create a new directory where you want to mount the storage, e.g. `/media/onedrive`. Mount OneDrive by calling

```
rclone mount --vfs-cache-mode writes onedrive-mount: /media/onedrive --config /root/.config/rclone/rclone.conf
```

`onedrive-mount:` is the name of the remote before the colon and a path after it (if you leave the path out, it will be the root).

Go to `/etc/systemd/system/` and create a new file by calling

```
sudo nano rclonemount.service
```

and paste the following content into it:

```
[Unit]
Description=rclonemount
AssertPathIsDirectory=/media/onedrive
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount \
        --config=/root/.config/rclone/rclone.conf \
        --vfs-cache-mode writes \
        onedrive-mount: /media/onedrive
ExecStop=/bin/fusermount -u /media/onedrive
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

Now execute it by running `sudo systemctl start rclonemount`. Check if it works by calling `sudo systemctl status rclonemount`.

In order to make it automatically execute on boot, run `sudo systemctl enable rclonemount`.

You should now see your OneDrive content on the specified folder:

```
sudo ls /media/onedrive
```
