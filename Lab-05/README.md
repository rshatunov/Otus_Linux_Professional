# Lab-05
## Работа с LVM
### Цели:
- Научиться создавать и работать с логическими томами.

### Уменьшить том под / до 8G:
 <details>
<summary>  Создание временного тома: </summary>

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
<summary>  Создание файловой системы и монтирование: </summary>

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
<summary> Копирование всех данных с / раздела в /mnt: : </summary>

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
```
</details>