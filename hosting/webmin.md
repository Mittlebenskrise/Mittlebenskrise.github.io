# Installation

## Update the package index

To get started, update the local package index to update the list of package information from their repositories.

```sh
sudo apt update
```

The output will be like this:

```
Hit:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
129 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

## Download and run the Webmin setup

Download the setup script using the curl command:

```sh
curl -o webmin-setup-repo.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repo.sh
```

The download will be like:

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 16246  100 16246    0     0   111k      0 --:--:-- --:--:-- --:--:--  111k
```

Verify the script download using the ls command:

```sh
ls -lh
```

The downloaded file should be listed, e.g.:

```
total 16K
-rw-r--r-- 1 root root 16K Mar  3 19:03 webmin-setup-repo.sh
```

Now, run the installation script.

```sh
bash webmin-setup-repo.sh
```

The script automatically sets up the repository and installs GPG keys on your system. Additionally, it provides the Webmin package for installation.

```
Setup Webmin releases repository? (y/N) y
  Downloading Webmin developers key ..
  .. done
  Installing Webmin developers key ..
  .. done
  Cleaning up package priority configuration ..
  .. done
  Setting up Webmin releases repository ..
  .. done
  Cleaning repository metadata ..
  .. done
  Downloading repository metadata ..
  .. done
Webmin and Usermin can be installed with:
  apt-get install --install-recommends webmin usermin
```

Install Webmin using the following command:

```sh
sudo apt install webmin  --install-recommends
```

The command installs Webmin along with all the required packages and dependencies.

## Verify the Webmin installation

Once installed, the Webmin service autostarts. You can confirm the status of the Webmin daemon by running:

```sh
sudo systemctl status webmin
```

The following output will be displayed, proof that Webmin is installed and running.

```
● webmin.service - Webmin server daemon
     Loaded: loaded (/usr/lib/systemd/system/webmin.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-03-03 19:41:14 EET; 1min 21s ago
   Main PID: 4542 (miniserv.pl)
      Tasks: 1 (limit: 2320)
     Memory: 81.5M (peak: 214.5M)
        CPU: 5.765s
     CGroup: /system.slice/webmin.service
```
             
By default, Webmin listens on port 10000 on the host. To confirm this, check it as follows:

```sh
ss -antp |  grep 10000
```

Here is the expected output.

```
LISTEN 0      4096                  0.0.0.0:10000                 0.0.0.0:*     users:(("miniserv.pl",pid=4542,fd=5))
LISTEN 0      4096                     [::]:10000                    [::]:*     users:(("miniserv.pl",pid=4542,fd=6))
```

Having confirmed the successful installation of Webmin, we can now access it.

## Configure UFW for Webmin
If you already have UFW configured, allow inbound traffic to port 1000 as shown.

```sh
sudo ufw allow 10000/tcp
```

```
Rule added
Rule added (v6)
```

Next, reload the firewall to adopt the changes.

```sh
sudo ufw reload
```

```
Firewall reloaded
```

## Access and start using Webmin
To access Webmin, open it in the browser at the URL `https://server-ip:10000`

Webmin ships with a pre-configured self-signed SSL, which explains the warning notification on your browser about a potential attack.
That's fine. The browser flags the SSL certificate since it cannot associate it with a trusted CA. So, switch to "Advanced" and then
to "Proceed to".

On the login screen, use the server's login credentials `<username>` and `<password>`.

The Webmin dashboard will be displayed, offering a top-level view of your system’s metrics and information, e.g., hostname, uptime,
operating system, pending software package updates, etc.

For more details, click on the notification bell in the upper-right corner.

This provides information about system metrics such as CPU and memory usage.

In the navigation panel on the left, you can select the operations to be executed. Please look at the [official documentation](https://webmin.com/docs/) for more information.
