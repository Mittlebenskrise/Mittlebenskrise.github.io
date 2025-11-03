See also https://linuxconfig.org/howto-mount-usb-drive-in-linux

Connect the USB drive, then find it by calling

```
sudo fdisk -l
```

You can find the drive as e.g. `/dev/sdb1`. Next, get it's UUID by calling

```
ls -l /dev/disk/by-uuid/*
```

In the list, search for the drive (e.g. `/dev/sdb1`) and the UUID will be listed before (e.g. `8765-4321`).

Create the mount point, e.g. by calling

```
sudo mkdir /media/backup
```

Now edit `/etc/fstab` and add the following line

```
/dev/disk/by-uuid/8765-4321    /media/backup         vfat    defaults   0   0
```

Mount the drives like this

```
sudo mount -a
```

The drive should now be mounted at the mount point

```
ls /media/backup
```

## Formatting USB drives

If drives are only used in Linux (or only shared via SMB as described in [[Share files with Windows]]), ext4 should be used as file system.

### Prepare the drive

First, find the drive by calling `lsblk`. The output will look like this:

```
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  74,1M  1 loop /snap/core22/1033
loop1                       7:1    0  73,9M  1 loop /snap/core22/864
loop2                       7:2    0 180,9M  1 loop /snap/lxd/25846
loop3                       7:3    0 152,1M  1 loop /snap/lxd/26200
loop4                       7:4    0  40,9M  1 loop /snap/snapd/20290
loop5                       7:5    0  40,4M  1 loop /snap/snapd/20671
sda                         8:0    0 476,9G  0 disk
└─sda1                      8:1    0 476,9G  0 part /media/ssd
sdb                         8:16   0 476,9G  0 disk
├─sdb1                      8:17   0     1G  0 part /boot/efi
├─sdb2                      8:18   0     2G  0 part /boot
└─sdb3                      8:19   0 473,9G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   100G  0 lvm  /
sdc                         8:32   0 931,5G  0 disk
├─sdc1                      8:33   0   200M  0 part
├─sdc2                      8:34   0 399,9G  0 part /media/book1
└─sdc3                      8:35   0 531,4G  0 part /media/book2
sdd                         8:48   0 931,5G  0 disk
├─sdd1                      8:49   0   200M  0 part
└─sdd2                      8:50   0 931,2G  0 part /media/backup
```

Unmount the "old" partitions by calling

```
sudo umount /dev/sdd2
```

### Create the GPT on the USB Drive

Open the drive in `fdisk`:

```bash
$ sudo fdisk /dev/sdd
```

Enter “_g_” to create the GPT:

```bash
Command (m for help): g
Created a new GPT disklabel (GUID: E98D9321-1A25-4D71-9742-3C7698EABCDB).
```

Create a new partition on it by typing in “_n_” and entering the default values (i.e. one partition covering all free space):

```bash
Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-33554431, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-33554431, default 33554431): 

Created a new partition 1 of type 'Linux filesystem' and of size 16 GiB.
```

Now, write the changes to the USB drive by entering “_w_“:

```bash
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Call `lsblk` again to check:

```
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  74,1M  1 loop /snap/core22/1033
loop1                       7:1    0  73,9M  1 loop /snap/core22/864
loop2                       7:2    0 180,9M  1 loop /snap/lxd/25846
loop3                       7:3    0 152,1M  1 loop /snap/lxd/26200
loop4                       7:4    0  40,9M  1 loop /snap/snapd/20290
loop5                       7:5    0  40,4M  1 loop /snap/snapd/20671
sda                         8:0    0 476,9G  0 disk
└─sda1                      8:1    0 476,9G  0 part /media/ssd
sdb                         8:16   0 476,9G  0 disk
├─sdb1                      8:17   0     1G  0 part /boot/efi
├─sdb2                      8:18   0     2G  0 part /boot
└─sdb3                      8:19   0 473,9G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   100G  0 lvm  /
sdc                         8:32   0 931,5G  0 disk
├─sdc1                      8:33   0   200M  0 part
├─sdc2                      8:34   0 399,9G  0 part /media/book1
└─sdc3                      8:35   0 531,4G  0 part /media/book2
sdd                         8:48   0 931,5G  0 disk
└─sdd1                      8:49   0 931,5G  0 part
```

### Format the partition

Next, format the partition as follows:

```
sudo mkfs.ext4 -m 0 -b 4096 -L backup /dev/sdd1
```

Mount it as described above (changing `vfat` in the `fstab` to `ext4`).

## Resize Partition

Run `growpart /dev/sda 3` to resize `sda3` and then (after mounting) `resize2fs /dev/sda3`.

## Safe Unmount

see https://askubuntu.com/questions/532586/what-is-the-command-line-equivalent-of-safely-remove-drive 

While sudo unmount /dev/sdXY will work, `udisksctl` can do this without root level (sudo) permissions.

If you have a drive /dev/sdXY, mounted, where X is a letter representing your usb disk and Y is the partition number (usually 1), you can use the following commands to safely remove the drive:

```
udisksctl unmount -b /dev/sdXY
udisksctl power-off -b /dev/sdX
```

Example if my drive is /dev/sdb1:

```
udisksctl unmount -b /dev/sdb1
udisksctl power-off -b /dev/sdb
```

Similarly to above, power-off can be used to detach the drive even if there are no partitions mounted, or no partition was ever mounted:

```
udisksctl power-off -b /dev/sdb
```
