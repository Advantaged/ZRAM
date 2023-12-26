# ZRAM
ZRAM: Clear instructions, easy to setup, double your RAM-capacity
## Scope of work
* These easy & clear instruction are tailored even for Linux-beginner & allow the "User" to double his RAM-capacity.

* Credits:
[Arch-Linux Wiki](https://wiki.archlinux.org/title/Zram)
* Following the Wiki and due the fact I have 64 GB RAM installed, I will set up a 128 GiB ZRAM, this mean, I will still have always 30,7 GiB RAM free :wink: + 128 GiB ZRAM on top, = 158,7 of total RAM.

### 1. Setup prerequisite
1. Assure you have ROOT-privileges:
```
sudo su
```

2. Install ZRAM, in my BlueStar-Linux is only following packages:
```
pacman -S --needed --noconfirm zram-generator zstd gambas3-gb-compress \
python-pyzstd python-zstandard zarchive lib32-zstd \
lz4 python-lz4 lib32-lz4
```

3. Than, disable zswap temporarily via sysfs:
```
echo 0 > /sys/module/zswap/parameters/enabled
```

### 2. Presettings
ZSwap will prevent to use ZRAM properly, so disable ZSwap permanently, see also [Arch-Wiki](https://wiki.archlinux.org/title/Kernel_parameters) & [Arch-Thread](https://bbs.archlinux.org/viewtopic.php?id=286116):

1. Edit the `/etc/default/grub`, e.g.:
```
nano /etc/default/grub
```
2. There is the fourth line beginning with `GRUB_CMDLINE_LINUX_DEFAULT=`, now add there, between the " " following parameter `zswap.enabled=0` at end, see my complete line below:
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 logo.nologo console=tty1 zswap.enabled=0"
```
3. Update `grub` with:
```
grub-mkconfig -o /boot/grub/grub.cfg
```


### 3. Configure ZRAM provisionally
See also [zram module](https://docs.kernel.org/admin-guide/blockdev/zram.html) and differences between [ZSTD & LZ4](https://engineering.fb.com/2016/08/31/core-infra/smaller-and-faster-data-compression-with-zstandard/).

1. Basic adjustments:

```
modprobe zram

zramctl /dev/zram0 --algorithm zstd --size 128GiB

mkswap -U clear /dev/zram0

swapon --priority 100 /dev/zram0
```

2. To disable it again, either reboot or run:

```
swapoff /dev/zram0

modprobe -r zram

echo 1 > /sys/module/zswap/parameters/enabled
```

### 4. Make Changes permanently using UDEV-rules like every other "Data-Carrier" (DC) too.

1. Create a file for loading new "Service":

```
nano /etc/modules-load.d/zram.conf
```

2. Insert following text and insert an empty line at end with the "Service-Name":

```
zram

```

3. Create another file for the configuration details:

```
nano /etc/udev/rules.d/99-zram.rules
```

4. Insert following text and insert an empty line at end with the configuration-parameters:

```
ACTION=="add", KERNEL=="zram0", ATTR{comp_algorithm}="zstd", ATTR{disksize}="128GiB", RUN="/usr/bin/mkswap -U clear /dev/%k", TAG+="systemd"
```

5. Now open `/etc/sftab`

```
nano /etc/fstab
```

6. Now add `/dev/zram` to your `fstab` with a higher than default priority:

```
/dev/zram0  none    swap    defaults,pri=100    0   0
```

### 5. Settings-Check and -Control

1. Check the ZSWAP-module is disabled with:

```
cat /sys/module/zswap/parameters/enabled
```

* Output should/must be `N`

2. Check all ZRAM-parameters with:
```
zramctl --output-all
```

3. Further possible outputs can be discovered with following command:

```
zramctl --help
```

**Note:** All changes will take effect after a `restart`**!**

4. My output of `zramctl --output-all` after `restart`:

```
NAME       DISKSIZE DATA COMPR ALGORITHM STREAMS ZERO-PAGES TOTAL MEM-LIMIT MEM-USED MIGRATED MOUNTPOINT
/dev/zram0     128G   4K   41B zstd           16          0    4K        0B      16K       0B [SWAP]

```


- [x] **Done!** **&** **ENJOY!** :wink:
