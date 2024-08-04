# Lab-05
## Работа с LVM
### Цели:
- Научиться создавать и работать с логическими томами.

### Уменьшить том под / до 8G
 <details>
<summary> Создание временного тома: </summary>

```
[root@lvm vagrant]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@lvm vagrant]# pvs
  PV         VG         Fmt  Attr PSize   PFree 
  /dev/sda3  VolGroup00 lvm2 a--  <38.97g     0 
  /dev/sdb              lvm2 ---   10.00g 10.00g
 
[root@lvm vagrant]# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
[root@lvm vagrant]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree  
  VolGroup00   1   2   0 wz--n- <38.97g      0 
  vg_root      1   0   0 wz--n- <10.00g <10.00g

[root@lvm vagrant]# lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
[root@lvm vagrant]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  lv_root  vg_root    -wi-a----- <10.00g                                                    
```
</details>
 <details>
<summary> Создание файловой системы и монтирование: </summary>

```
[root@lvm vagrant]# mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@lvm vagrant]# mount /dev/vg_root/lv_root /mnt

[root@lvm vagrant]# df -h
Filesystem                       Size  Used Avail Use% Mounted on
devtmpfs                         235M     0  235M   0% /dev
tmpfs                            244M     0  244M   0% /dev/shm
tmpfs                            244M  4.5M  239M   2% /run
tmpfs                            244M     0  244M   0% /sys/fs/cgroup
/dev/mapper/VolGroup00-LogVol00   38G  1.6G   36G   5% /
/dev/sda2                       1014M   88M  927M   9% /boot
tmpfs                             49M     0   49M   0% /run/user/1000
/dev/mapper/vg_root-lv_root       10G   33M   10G   1% /mnt                                              
/dev/mapper/vg_root-lv_root       10G   33M   10G   1% /mnt                                              
```
</details>
 <details>
<summary> Копирование всех данных с / раздела в /mnt: </summary>

```
[root@lvm vagrant]#  xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.7 (dump format 3.0)
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.7 (dump format 3.0)
xfsdump: level 0 dump of lvm:/
xfsdump: dump date: Sun Aug  4 10:57:41 2024
xfsdump: session id: 4bc125fa-3f34-41bf-bf8e-d55de55b194c
xfsdump: session label: ""
xfsrestore: searching media for dump
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 1641444352 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description: 
xfsrestore: hostname: lvm
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/VolGroup00-LogVol00
xfsrestore: session time: Sun Aug  4 10:57:41 2024
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: b60e9498-0baa-4d9f-90aa-069048217fee
xfsrestore: session id: 4bc125fa-3f34-41bf-bf8e-d55de55b194c
xfsrestore: media id: 72529cc1-8542-41b0-a22f-42563a3b5cd3
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 4589 directories and 37301 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 1602819008 bytes
xfsdump: dump size (non-dir files) : 1581221008 bytes
xfsdump: dump complete: 76 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 77 seconds elapsed
xfsrestore: Restore Status: SUCCESS                                           
```
</details>
 <details>
<summary> Конфигурация grub: </summary>

```
[root@lvm vagrant]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done

[root@lvm vagrant]#  chroot /mnt/

[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-1160.119.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1160.119.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done

[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;> s/.img//g"` --force; done
sed: -e expression #1, char 17: unknown command: `>'
Executing: /sbin/dracut -v initramfs-3.10.0-1160.119.1.el7.x86_64.img --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-1160.119.1.el7.x86_64.img' done ***
sed: -e expression #1, char 17: unknown command: `>'
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

sed -i 's/rd.lvm.lv=VolGroup00\/LogVol00/rd.lvm.lv=vg_root\/lv_root/g' /boot/grub2/grub.cfg 

reboot

[root@lvm vagrant]# lsblk 
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:1    0 37.5G  0 lvm  
  └─VolGroup00-LogVol01 253:2    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk                                      
```
</details>
 <details>
<summary> Изменение рамзмера LV LogVol00 и перенос /: </summary>

```
[root@lvm vagrant]#  lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
  Logical volume "LogVol00" successfully removed

[root@lvm vagrant]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  lv_root  vg_root    -wi-ao---- <10.00g                                                    

[root@lvm vagrant]#  lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.

[root@lvm vagrant]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-a-----   8.00g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  lv_root  vg_root    -wi-ao---- <10.00g                                                    

[root@lvm vagrant]# mkfs.xfs /dev/VolGroup00/LogVol00
meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@lvm vagrant]#  mount /dev/VolGroup00/LogVol00 /mnt

[root@lvm vagrant]# xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.7 (dump format 3.0)
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.7 (dump format 3.0)
xfsdump: level 0 dump of lvm:/
xfsdump: dump date: Sun Aug  4 12:50:23 2024
xfsdump: session id: 1bc883e9-f609-4c25-ad5d-99c49040b831
xfsdump: session label: ""
xfsrestore: searching media for dump
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 1642017728 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description: 
xfsrestore: hostname: lvm
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/vg_root-lv_root
xfsrestore: session time: Sun Aug  4 12:50:23 2024
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: 8d9f85fc-7af0-4736-801e-8b4976fc2d10
xfsrestore: session id: 1bc883e9-f609-4c25-ad5d-99c49040b831
xfsrestore: media id: fce25375-fc26-470e-9b21-9a4a5887e4bc
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 4599 directories and 37416 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 1603119120 bytes
xfsdump: dump size (non-dir files) : 1581445720 bytes
xfsdump: dump complete: 54 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 55 seconds elapsed
xfsrestore: Restore Status: SUCCESS

[root@lvm vagrant]#  for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done

[root@lvm vagrant]#  chroot /mnt/

[root@lvm /]# sed -i 's/rrd.lvm.lv=vg_root\/lv_root/rd.lvm.lv=VolGroup00\/LogVol00/g' /boot/grub2/grub.cfg 

[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-1160.119.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1160.119.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done

[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; > s/.img//g"` --force; done
sed: -e expression #1, char 18: unknown command: `>'
Executing: /sbin/dracut -v initramfs-3.10.0-1160.119.1.el7.x86_64.img --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-1160.119.1.el7.x86_64.img' done ***
sed: -e expression #1, char 18: unknown command: `>'
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***   
             
###########################################################################################################                    
###  Не перезагружаемся и не выходим из под chroot. Выполняем задание "Выделить том под /var в зеркало" ###
###########################################################################################################

```
</details>

### Выделить том под /var в зеркало
 <details>
<summary> Создание зеркала: </summary>

```
[root@lvm /]# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.

[root@lvm /]# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created

[root@lvm /]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree  
  VolGroup00   1   2   0 wz--n- <38.97g <29.47g
  vg_root      1   1   0 wz--n- <10.00g      0 
  vg_var       2   0   0 wz--n-   2.99g   2.99g

[root@lvm /]# lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.

[root@lvm /]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao----   8.00g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  lv_root  vg_root    -wi-ao---- <10.00g                                                    
  lv_var   vg_var     rwi-a-r--- 952.00m                                    100.00      
```
</details>
 <details>
<summary> Создание ФС и перенос /var: </summary>

```
[root@lvm /]# lvcreate -L 95mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
60928 inodes, 243712 blocks
12185 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=249561088
8 block groups
32768 blocks per group, 32768 fragments per group
7616 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@lvm /]# mount /dev/vg_var/lv_var /mnt

[root@lvm /]# cp -aR /var/* /mnt/

[root@lvm /]# rm -rf /var/*

[root@lvm /]#  umount /mnt
[root@lvm /]# 
[root@lvm /]#  mount /dev/vg_var/lv_var /var
```
</details>
 <details>
<summary> Правим fstab для автоматического монтирования /var: </summary>

```
[root@lvm /]# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab

[root@lvm /]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
#VAGRANT-END
UUID="63151059-77f0-41a8-a2b6-bfc87d8ad7c7" /var ext4 defaults 0 0                                    
```
</details>
 <details>
<summary> Конфигурация grub: </summary>

```
[root@lvm /]# exit

[root@lvm vagrant]# reboot                                      
```
</details>
 <details>
<summary> Удаляем временную VG: </summary>

```
[root@lvm ~]# lvremove  /dev/vg_root/lv_root
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed

[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao----   8.00g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  lv_var   vg_var     rwi-aor--- 952.00m                                    100.00

[root@lvm ~]# vgremove /dev/vg_root
  Volume group "vg_root" successfully removed

[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree  
  VolGroup00   1   2   0 wz--n- <38.97g <29.47g
  vg_var       2   1   0 wz--n-   2.99g   1.12g

[root@lvm ~]# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.
[root@lvm ~]# 
[root@lvm ~]# pvs
  PV         VG         Fmt  Attr PSize    PFree  
  /dev/sda3  VolGroup00 lvm2 a--   <38.97g <29.47g
  /dev/sdc   vg_var     lvm2 a--    <2.00g   1.06g
  /dev/sdd   vg_var     lvm2 a--  1020.00m  64.00m
[root@lvm ~]# 
[root@lvm ~]# df -h
Filesystem                       Size  Used Avail Use% Mounted on
devtmpfs                         235M     0  235M   0% /dev
tmpfs                            244M     0  244M   0% /dev/shm
tmpfs                            244M  4.5M  239M   2% /run
tmpfs                            244M     0  244M   0% /sys/fs/cgroup
/dev/mapper/VolGroup00-LogVol00  8.0G  1.2G  6.9G  15% /
/dev/mapper/vg_var-lv_var        922M  426M  432M  50% /var
/dev/sda2                       1014M   87M  928M   9% /boot
```
</details>

### Выделить том под /home
 <details>
<summary> Создание тома под /home и перенос данных: </summary>

```
[root@lvm ~]#  lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.
[root@lvm ~]# 
[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@lvm ~]# 
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /mnt/
[root@lvm ~]# 
[root@lvm ~]# cp -aR /home/* /mnt/
[root@lvm ~]# 
[root@lvm ~]#  rm -rf /home/*
[root@lvm ~]# 
[root@lvm ~]#  umount /mnt
[root@lvm ~]# 
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /home/
[root@lvm ~]#                                        
```
</details>
 <details>
<summary> Правим fstab для автоматического монтирования /home: </summary>

```
[root@lvm ~]# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
[root@lvm ~]# 
[root@lvm ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
#VAGRANT-END
UUID="63151059-77f0-41a8-a2b6-bfc87d8ad7c7" /var ext4 defaults 0 0
UUID="7cf8d062-4dbd-4428-aafd-bb388aaeb1cd" /home xfs defaults 0 0
[root@lvm ~]# 
[root@lvm ~]# df -h
Filesystem                          Size  Used Avail Use% Mounted on
devtmpfs                            235M     0  235M   0% /dev
tmpfs                               244M     0  244M   0% /dev/shm
tmpfs                               244M  4.5M  239M   2% /run
tmpfs                               244M     0  244M   0% /sys/fs/cgroup
/dev/mapper/VolGroup00-LogVol00     8.0G  1.2G  6.9G  15% /
/dev/mapper/vg_var-lv_var           922M  426M  432M  50% /var
/dev/sda2                          1014M   87M  928M   9% /boot
/dev/mapper/VolGroup00-LogVol_Home  2.0G   33M  2.0G   2% /home                                       
```
</details>

### Работа со снапшотами
 <details>
<summary> Генерируем файлы в /home/: </summary>

```
                                        
```
</details>
 <details>
<summary> Конфигурация grub: </summary>

```
[root@lvm ~]# touch /home/file{1..20}
[root@lvm ~]# 
[root@lvm ~]# ls /home
file1  file10  file11  file12  file13  file14  file15  file16  file17  file18  file19  file2  file20  file3  file4  file5  file6  file7  file8  file9  vagrant                                        
```
</details>
 <details>
<summary> Создание снапшота: </summary>

```
[root@lvm ~]#  lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.
[root@lvm ~]# 
[root@lvm ~]# lvs
  LV          VG         Attr       LSize   Pool Origin      Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00    VolGroup00 -wi-ao----   8.00g                                                         
  LogVol01    VolGroup00 -wi-ao----   1.50g                                                         
  LogVol_Home VolGroup00 owi-aos---   2.00g                                                         
  home_snap   VolGroup00 swi-a-s--- 128.00m      LogVol_Home 0.00                                   
  lv_var      vg_var     rwi-aor--- 952.00m                                         
```
</details>
 <details>
<summary> Удалить часть файлов: </summary>

```
[root@lvm ~]#  rm -f /home/file{11..20}                                       
```
</details>
 <details>
<summary> Процесс восстановления из снапшота: </summary>

```
[root@lvm ~]#  umount /home
[root@lvm ~]# 
[root@lvm ~]# lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%
[root@lvm ~]# 
[root@lvm ~]#  mount /home
[root@lvm ~]#
[root@lvm ~]# ls -la /home/
total 0
drwxr-xr-x.  3 root    root    292 Aug  4 13:30 .
drwxr-xr-x. 18 root    root    239 Aug  4 12:51 ..
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file1
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file10
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file11
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file12
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file13
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file14
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file15
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file16
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file17
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file18
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file19
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file2
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file20
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file3
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file4
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file5
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file6
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file7
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file8
-rw-r--r--.  1 root    root      0 Aug  4 13:30 file9
drwx------.  4 vagrant vagrant 169 Aug  4 09:41 vagrant                                        
```
</details>