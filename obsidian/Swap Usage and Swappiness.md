The Linux kernel provides a tweakable setting that controls how often the swap file is used, called swappiness.

A swappiness setting of **zero** means that the disk will be avoided unless absolutely necessary (you run out of memory), while a swappiness setting of **100** means that programs will be swapped to disk almost instantly.

Ubuntu system comes with a default of 60, meaning that the swap file will be used fairly often if the memory usage is around half of my RAM. You can check your own system's swappiness value by running:

```
one@onezero:~$ cat /proc/sys/vm/swappiness
60
```

As I have 4 GB of RAM I'd like to turn that down to 10 or 15. The swap file will then only be used when my RAM usage is around **80** or **90** percent. To change the system swappiness value, open `/etc/sysctl.conf` as **root**. Then, change or add this line to the file:

```
vm.swappiness = 10
```

Apply the change.

```
sudo sysctl -p
```

You can also change the value while your system is still running with:

```
sysctl vm.swappiness=10
```

You can also clear your swap by running `swapoff -a` and then `swapon -a` as root instead of rebooting to achieve the same effect.

To calculate your swap Formula:

```
free -m (total) / 100 = A

A * 10

root@onezero:/home/one# free -m
             total       used       free     shared    buffers     cached
Mem:          3950       2262       1687          0        407        952
-/+ buffers/cache:        903       3047
Swap:         1953          0       1953
```

> so total is 3950 / 100 = 39.5 * 10 = 395

So what it mean is that when **10 %** (395 MB) of ram is left then it will start using swap.