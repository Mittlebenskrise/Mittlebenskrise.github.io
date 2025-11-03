Sane installieren:

```
sudo apt-get install sane sane-utils xsane
```

Manual: https://manpages.ubuntu.com/manpages/xenial/man1/scanimage.1.html

To get a list of devices:

         sudo scanimage -L

       To scan with default settings to the file image.pnm:

         sudo scanimage >image.pnm

       To  scan  100x100  mm  to  the  file  image.tiff  (-x and -y may not be available with all
       devices):

         sudo scanimage -x 100 -y 100 --format=tiff >image.tiff

       To print all available options:

         sudo scanimage -h


TBD

Batch Scanning: https://askubuntu.com/questions/383568/multiple-page-scan-using-scanimage
## Scannerbuttons

Description: https://wiki.ubuntuusers.de/scanbuttond/

Download from https://altlinux.pkgs.org/p10/classic-x86_64/scanbuttond-0.2.3-alt4.x86_64.rpm.html

```
wget https://distrib-coffee.ipsl.jussieu.fr/pub/linux/altlinux/p10/branch/x86_64/RPMS.classic/scanbuttond-0.2.3-alt4.x86_64.rpm
```

Convert for Ubuntu: https://phoenixnap.com/kb/install-rpm-packages-on-ubuntu

```
sudo apt-get install scanbd
```

Description: https://wiki.ubuntuusers.de/scanbd/


Install sane-utils and imagemagick:

```
sudo apt install sane-utils
sudo apt install imagemagick
```

Get script from here: https://github.com/pohape/command-line-scanner

DO NOT INSTALL `scanbd`!!!