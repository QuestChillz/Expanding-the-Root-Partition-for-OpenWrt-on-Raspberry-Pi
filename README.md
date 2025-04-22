# Expanding the Root Partition for OpenWrt on Raspberry Pi

For those attempting to run the OpenWrt OS for the first time on their Raspberry Pi or other boards, having sufficient storage volume attached may not resolve an issue with the limitation of root partition sizes of OpenWrt OS images that are shipped with less than 100 MB in size. This limitation can become troublesome after installing some packages and starting the OS. Therefore, in this guide, we will go through the steps to increase the size of the root directory to a desired size, allowing you to install as many software applications and packages as necessary. 

## 1. Prerequisites

To accomplish this, please prepare the following items in advance:

1. **OpenWrt OS (Ext4 format)**: Download the Ext4 format version of the OpenWrt OS that corresponds to your board (in my case, using the Raspberry Pi 4 with processor model BCM 2711 was necessary the download link is: https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2711/ ). Note that sometimes on the OpenWrt website, they promote the SquashFS format version, which is slightly faster than Ext4; however, due to being read-only, it cannot be used for partitioning. The trade-off here is worth using the Ext4 format since the difference is negligible. 

2. **Raspberry Pi (or other board)**: Raspberry Pi board (or any other board you are using to boot up your OpenWrt OS). 

3. **Backup your data**: Backup of all important data (because mistakes during partition resizing and removal can lead to file corruption).

4. **A microSD card reader** (for Raspberry Pi) or an **external drive** connected via USB to a secondary system (your laptop/computer running a Linux system).

If you do not have a laptop/computer running a Linux system as the host, you can use virtualization software such as VMware/VirtualBox/Hyper-V/etc. to install a Linux-based OS, such as Ubuntu. In this guide, I downloaded and used the first and smallest version of Ubuntu, which was Ubuntu 16, on VirtualBox running on my macOS laptop with USB passthrough enabled. Since the USB microSD card reader is connected to my host macOS laptop using the USB port, it was necessary to first install:
The VirtualBox extension package, which can be downloaded from: https://www.virtualbox.org/wiki/Downloads, and then once the guest OS (Ubuntu 16 in this case) is configured and installed, use the commands below on the Ubuntu as the root user:

4.1. VirtualBox Extension Package Installation on Ubuntu
```bash
apt update
apt install virtualbox-ext-pack virtualbox-guest-utils virtualbox-guest-x11 virtualbox-guest-dkms
```  

4.2. Add your user to the vboxusers group to enable USB passthrough
```bash
usermod -aG vboxusers root
```  
and then ```reboot``` for the changes to take effect.

>Note that you should install an extension package that corresponds to the version of VirtualBox that you have on your system; otherwise, it won’t work. Also, during this process, all running VMs must be turned off.

## 2. Accessing the SD Card in Ubuntu VM

Once everything is done, right-click on the Ubuntu VM and hover over the USB setting where you check the **Enable USB controller**. and then select the version that corresponds to yours (it is usually **USB 2** and above, but again, check with your version). Then, from the **USB filter list**, add your device using the plus sign and run the VM.

Once you turn on the VM, from the menu bar of the Ubuntu VM (displayed below), click on the USB icon and select the USB that you added in the previous step. Once this is done, you should be able to see it attached using the command ```lsblk```.

Example output:
```bash
root@Ubuntu:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0    5G  0 disk 
|-sda1   8:1    0    4G  0 part /
|-sda2   8:2    0    1K  0 part 
`-sda5   8:5    0  975M  0 part [SWAP]
sdb      8:16   1   59G  0 disk 
|-sdb1   8:17   1   64M  0 part 
`-sdb2   8:18   1  104M  0 part 
sr0     11:0    1 1024M  0 rom
```  

**To summarize:** <br>
/dev/sdb → full SD card (59 GB)<br>
/dev/sdb1 → boot partition (64 MB)<br>
and /dev/sdb2 → root partition (104 MB; this is what we’ll expand).

# 3. Partitioning:

```
fdisk /dev/sdb
``` 

**Inside ```fdisk```, do this carefully:** <be>

- Type ```p``` → to show current partitions (note the start sector of /dev/sdb2, which is currently created and included with the image).<br>
- Type ```d``` → delete partition.<br>
- Choose partition ```2```.<br>
- Type ```n``` → create new partition.<br>
- Type ```2``` (same number).<br>
- Use the same ```start sector```.<br>
- Accept the ```default ending``` (full size of disk or specify, explained below).<br>
- When prompted about ```removing the signature```: say N for no (if you are asked).<br>
- Type ```w``` → write changes and exit.

> Here, an important thing to consider is the starting and ending sectors for the partition of sdb2 that you will be deleting. When you attempt to create a new one, it should have the same starting sector to avoid overlapping with the boot partition. However, the ending sector could either be the default value, which is usually the entire remaining space on the storage, or if you want this root partition to be set to a specific amount rather than taking the entire storage, you can specify the ending sector. For me, since I had a 64GB SD card, I only wanted 10GB for the root and leave the rest for other use cases such as NAS, File Sharing, etc.

This is how you can calculate the ending sector for the amount of 10GB, assuming you start from the starting sector of 147456, which was my case:

- Start sector is 147456 (you noted this earlier).<be>

10 GB in sectors:<br>
- 1 GB = 2,000,000 sectors (since each sector is 512 bytes).<br>
Therefore, 10 GB = 10 × 2,000,000 = 20,000,000 sectors.<be>

End sector for the 10 GB partition:<be>
- End sector = Start sector + 20,000,000 - 1.<be>
- End sector = 147456 + 20,000,000 - 1 = **20147455**


Now, re-check and resize the filesystem:

```bash
e2fsck -f /dev/sdb2
resize2fs /dev/sdb2
```


To verify, you can use:

```bash
sudo lsblk
```


Example output:
```bash
root@Ubuntu:~# sudo lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0    5G  0 disk 
|-sda1   8:1    0    4G  0 part /
|-sda2   8:2    0    1K  0 part 
`-sda5   8:5    0  975M  0 part [SWAP]
sdb      8:16   1   59G  0 disk 
|-sdb1   8:17   1   64M  0 part 
`-sdb2   8:18   1 58.9G  0 part
sr0     11:0    1 1024M  0 rom
```

Lastly, if you want to create an additional partition for other applications like what I did, you can repeat the same partitioning, but make sure that the additional partitions have some gaps in between them for safety, to avoid overlap. For example, since the ending sector for sdb2 was 20147455, you can have the starting sector for sdb3 be 20147455 **+ 1000** (which is 20148455). 

# 4. Mounting third, fourth, and further partitions on OpenWrt

Since OpenWrt only automatically mounts the boot and root partitions created during OS installation, any additional partitions created on a secondary Linux OS (Ubuntu in this case) will not be mounted or shown in OpenWrt. They must be mounted manually using the following commands and steps.

1. List all partitions and identify **mmcblk0p3** as the target (≈49.3 GiB)

```bash
root@OpenWrt:~# cat /proc/partitions
major minor  #blocks  name

   1        0       4096 ram0
   1        1       4096 ram1
   1        2       4096 ram2
   1        3       4096 ram3
   1        4       4096 ram4
   1        5       4096 ram5
   1        6       4096 ram6
   1        7       4096 ram7
   1        8       4096 ram8
   1        9       4096 ram9
   1       10       4096 ram10
   1       11       4096 ram11
   1       12       4096 ram12
   1       13       4096 ram13
   1       14       4096 ram14
   1       15       4096 ram15
 179        0   61798400 mmcblk0
 179        1      65536 mmcblk0p1
 179        2   10000000 mmcblk0p2
 179        3   51724172 mmcblk0p3
```

2. Check if the **MMC/SD kernel module** is installed
```
opkg update
opkg list | grep kmod-mmc
```

4. If not present, install the required packages
```
opkg update
opkg install kmod-mmc kmod-fs-ext4 block-mount
```
5. Install the **file utility** to detect the filesystem
```
opkg update
opkg install file
```
6. Inspect the filesystem on the partition
```file -s /dev/mmcblk0p3```

If unformatted or you wish to reformat:
```mkfs.ext4 /dev/mmcblk0p3```

Example output:
```bash
root@OpenWrt:~# file -s /dev/mmcblk0p3
/dev/mmcblk0p3: Linux rev 1.0 ext4 filesystem data, UUID=d9c427d3-f325-4064-919f-b96afcc5cd56 (needs journal recovery) (extents) (large files) (huge files)
```

6. Create a mount point and mount the partition
```
mkdir -p /mnt/data
mount -t ext4 /dev/mmcblk0p3 /mnt/data
```

8. Confirm with disk free output: ```df -h```

Example output:
```bash
root@OpenWrt:~# df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/root                 9.4G     26.1M      9.4G   0% /
tmpfs                   928.4M     76.0K    928.3M   0% /tmp
/dev/mmcblk0p1           63.9M     17.8M     46.1M  28% /boot
tmpfs                   512.0K         0    512.0K   0% /dev
/dev/mmcblk0p3           48.4G     24.0K     45.9G   0% /mnt/data
```

8. Enable auto‑mount at boot by editing ```/etc/config/fstab```:

Add the following block:
```
cat << 'EOF' >> /etc/config/fstab
config mount
    option target   '/mnt/data'
    option device   '/dev/mmcblk0p3'
    option fstype   'ext4'
    option options  'rw,sync'
    option enabled  '1'
EOF
```


9. Enable and start **fstab service**
```/etc/init.d/fstab enable
/etc/init.d/fstab start
```

11. ```Reboot``` to confirm persistence and after reboot, verify:
```
mount | grep /mnt/data
df -h /mnt/data
```

Example output:
```bash
root@OpenWrt:~# mount | grep /mnt/data
/dev/mmcblk0p3 on /mnt/data type ext4 (rw,sync,relatime)
root@OpenWrt:~# df -h /mnt/data
Filesystem                Size      Used Available Use% Mounted on
/dev/mmcblk0p3           48.4G     24.0K     45.9G   0% /mnt/data
```


## Disclaimer

Please be advised that while the steps outlined in this guide have been carefully reviewed and tested, the author cannot be held responsible for any damage, data loss, or other unintended consequences that may arise from following these instructions. It is strongly recommended to back up all important data before proceeding with any changes to your system. By following this guide, you acknowledge that you do so at your own risk.
