# ZRAM
ZRAM: Clear instructions, easy to setup, double your RAM-capacity
## Scope of work
* These easy & clear instruction are tailored even for Linux-beginner & allow the "User" to double his RAM-capacity.

* Credits:
[Arch-Linux Wiki](https://wiki.archlinux.org/title/Zram)
* Following the Wiki and due the fact I have 64 GB RAM installed, I will set up a 128 GiB ZRAM, this mean, I will still have always 30,7 GiB RAM free :wink: + 128 GiB ZRAM on top, = 158,7 of total RAM minimum.

* **Note:** Some Distro, like [CachyOS](https://cachyos.org/download/) already setup ZRAM🟰physical_RAM, in this case you don't need any steps. If you want double your ZRAM under CachyOS… just execute **§ 5.**. In all other case you can setup "RAM*2=ZRAM" with `zram-configurator` using **'§ 1.' till '§ 5.'** (where from § 2. till 4. according to your bootloader = choose only one of three options) and with `udev` excluding **§ 5.** (of course) & the two cahpter not matching your bootloader (§ 2. till 4.).
    * CachyOS have already ZRAM=physical_RAM
    * CachyOS double ZRAM, execute only '§ 5.'
    * Other OS/Distro through `zram-generator`… '§ 1.'➕'§ 2. or § 3. or § 4.' according to you bootloader➕'§ 5.'.
    * Other OS/Distro through `udev`… '§ 1.'➕'§ 2. or § 3. or § 4.' according to you bootloader, exclude '§ 5.' but include the remain chapters ('§ 6.' till '§ 9.').        

### 1. Setup prerequisite
1. Assure you have ROOT-privileges:
```
sudo su

or

sudo -i
```

2. Install ZRAM, in my BlueStar-Linux are only following packages:
```
pacman -S --needed --noconfirm zram-generator zstd gambas3-gb-compress \
python-pyzstd python-zstandard zarchive lib32-zstd \
lz4 python-lz4 lib32-lz4
```

3. Than, disable zswap temporarily via sysfs:
```
echo 0 > /sys/module/zswap/parameters/enabled
```

* **Note:** Don't enable anymore `zswap` if you want to held `zram` for next `reboot` or for ever.

### 2. Presettings **"Grub"**

* **Note: ZSwap** will prevent to use ZRAM properly, so disable ZSwap permanently, see also [Arch-Wiki](https://wiki.archlinux.org/title/Kernel_parameters) & [Arch-Thread](https://bbs.archlinux.org/viewtopic.php?id=286116):

* Don't forget to copy first the original file under .e.g.:  
    `<file-name>.original` , so you can revert &|| undo it at each time!

1. Edit the `/etc/default/grub`, e.g.:
```
nano /etc/default/grub
```
2. There is the fourth line beginning with `GRUB_CMDLINE_LINUX_DEFAULT=`, now add there, between the " " following parameter `zswap.enabled=0` at end, see my complete line below:

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 logo.nologo console=tty1 zswap.enabled=0"
```

* **Note:** Don't forget to save the file after your modifications & assure is an empty line at end of file.

3. Update `grub` with:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

### 3. Presettings **"rEFInd"** 

* **Note: ZSwap** will prevent to use ZRAM properly, so disable ZSwap permanently, see also [Arch-Wiki](https://wiki.archlinux.org/title/Kernel_parameters) & [Arch-Thread](https://bbs.archlinux.org/viewtopic.php?id=286116):

* Don't forget to copy first the original file under .e.g.:  
    `<file-name>.original` , so you can revert &|| undo it at each time!

1. Kernel-CMD-Line modification

* edit file: `/boot/efi/EFI/refind.conf` &|| `/boot/refind_linux.conf` [Arch-Wiki](https://wiki.archlinux.org/title/REFInd) for setting [kernel parameters](https://wiki.archlinux.org/title/Kernel_parameters). 
* Look for the line `menuentry` on which is the title of your Linux-OS: `Arch Linux` or `ArcolinuxD`, `Bluestar Linux`, `EndeavourOS`, `Garuda Linux`, `Manjaro Linux`, etc., to set kernel-options. In the line `options` set as last option `zswap.enabled=0`. See my complete option-line below:

```
options "root=PARTUUID=028fa50-0079-4c40-b240-abfaf28693ea rw add_efi_memmap zswap.enabled=0"

```
* **Note:** Don't forget to save the file after your modifications, no other steps are required after `refind.conf` is saved.

### 4. Presettings **"SystemD-Boot"** &&
ZSwap will prevent to use ZRAM properly, so disable ZSwap permanently, see also [Arch-Wiki](https://wiki.archlinux.org/title/Kernel_parameters) & [EndeavourOS](https://discovery.endeavouros.com/installation/systemd-boot/2022/12/):

* Don't forget to copy first the original file under .e.g.:  
    `<file-name>.original` , so you can revert &|| undo it at each time!

1. Kernel-CMD-Line modification

* edit file: `/etc/kernel/cmdline` & insert after last option `zswap.enabled=0` , here an example:

```
root=UUID=00bfb9fd-a31a-43c6-aea3-02d097db1894 rw quiet zswap.enabled=0

```

* **Note:** The first & last option, in this case is `rw` ... we add between `rw` & `root` our additional option `zswap.enabled=0` , don't forget to save the file after your modifications.

2. Update SystemD-Boot-entries, just execute following command:

```
reinstall-kernels

```

* Now, if you open the files inside `/boot/efi/loader/entries/` you can see inside, of there stored files, all entries are modified exactly as above descrived.

### 5. ZRAM made with `zram-geneartor` & at least as it works…

1. Copy configuration-file at right folder:

`sudo cp /usr/lib/systemd/zram-generator.conf /etc/systemd/zram-generator.conf`

2. Modify destination-file as the user want:

```
[zram0] 
zram-size = ram * 2
compression-algorithm = zstd
max-comp-streams = 16
swap-priority = 180
fs-type = swap
```
* **Notes:**
    * `ram-size`, don't set any value but mathematical expressions like `ram`, `ram / 2` or (in my case) `ram * 2`
    * `compression-algorithm` best algorithm to get 4-5-time compression is `zstd`.
    * `max-compress-streams` set the maximum of threads available on your CPU.
    * `swap-priority` default is 100, 180 is still moderate, some OS set it from 200 on-warts.
    * `fs-type` better to specify fs-type as `swap`

3. Enable "zram-generator" & start it:

`sudo systemctl enable --now systemd-zram-setup@zram0.service`

4. Check ZRAM is working:
    * To check if ZRAM is set up correctly, use: `swapon --show`… output:
      
      ```
      NAME       TYPE        SIZE USED PRIO
      /dev/zram0 partition 125,4G   0B  180
      ```

    * You can also inspect the active ZRAM configuration: `cat /proc/swaps`… output:
      
      ```
      Filename                                Type            Size            Used            Priority
      /dev/zram0                              partition       131473404       0               180
      ```
    
    * Monitor the performance of ZRAM over time using: `zramctl` or `zramctl --output-all`… output:
      
      ```
      NAME       DISKSIZE DATA COMPR ALGORITHM STREAMS ZERO-PAGES TOTAL MEM-LIMIT MEM-USED MIGRATED MOUNTPOINT
      /dev/zram0   125,4G   4K   68B zstd           16          0   20K        0B      20K       0B [SWAP]
      ```

* Further optimization, like in the [Arch-Wiki](https://wiki.archlinux.org/title/Zram) under § 2.3 explained, are made automatically❗️

### 6. Configure ZRAM (udev) provisionally
See also [zram module](https://docs.kernel.org/admin-guide/blockdev/zram.html) and differences between [ZSTD & LZ4](https://engineering.fb.com/2016/08/31/core-infra/smaller-and-faster-data-compression-with-zstandard/), anyway, even linux on ZFS use `zstd` instead of `lz4` & we do it too.

1. Basic adjustments:

```
modprobe zram

zramctl /dev/zram0 --algorithm zstd --size 128GiB

mkswap -U clear /dev/zram0

swapon --priority 100 /dev/zram0
```

* **Note:** you can use also `lz4` as compression algorithm instead of `zstd`, i prefer `zstd` because is much faster, anyway it's depend of your used file-system on which one reside Linux. If you use `ext4` than you can use `lz4` and/or `zstd`, if you use `btrfs` or `zfs` than is (maybe) recommended the use of `zstd`.

2. To disable it again, either reboot or run:

```
swapoff /dev/zram0

modprobe -r zram

echo 1 > /sys/module/zswap/parameters/enabled
```

### 7. Make Changes permanently using UDEV-rules like every other "Data-Carrier" (DC) too.

1. Assure `zswap` kernel-module is disabled...
* Check with:
```
cat /sys/module/zswap/parameters/enabled

```

   * The answer should be `N` for "not enabled".

2. Create a file for loading new "Service":

```
nano /etc/modules-load.d/zram.conf
```

3. Insert following text and insert an empty line at end with the "Service-Name":

```
zram

```

4. Create another file for the configuration details:

```
nano /etc/udev/rules.d/99-zram.rules
```

5. Insert following text and insert an empty line at end with the configuration-parameters:

```
ACTION=="add", KERNEL=="zram0", ATTR{comp_algorithm}="zstd", ATTR{disksize}="128GiB", RUN="/usr/bin/mkswap -U clear /dev/%k", TAG+="systemd"
```

6. Now open `/etc/sftab`

```
nano /etc/fstab
```

7. Now add `/dev/zram` to your `fstab` with a higher than default priority:

```
/dev/zram0  none    swap    defaults,pri=100    0   0
```

### 8. Settings-Check and -Control

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
### 9. Compress Linux-Image too!

1. edit `/etc/mkinitcpio.conf`
2. after `HOOKS` should be the the compressions-parameters like these:

```
# COMPRESSION
# Use this to compress the initramfs image. By default, zstd compression
# is used. Use 'cat' to create an uncompressed image.
COMPRESSION="zstd"
#COMPRESSION="gzip"
#COMPRESSION="bzip2"
#COMPRESSION="lzma"
#COMPRESSION="xz"
#COMPRESSION="lzop"
#COMPRESSION="lz4"
```
* as you can see (in my case), the `zstd` compression is enabled because missing/without `#`, if this part is missing in your `/etc/mkinitcpio.conf`… just add following line:

```
COMPRESSION="zstd"
```
3. don't forget to recreate the kernels with new parameter as unders § "4. Presettings…/2. Update SystemD-Boot-entries…" explained:

```
reinstall-kernels

```
* **Note:** Latest now you should reboot your system❗️


- [x] **Done❗️& ENJOY❗️** :wink:
