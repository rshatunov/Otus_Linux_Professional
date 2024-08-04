# Lab-05
## Работа с LVM
### Цели:
- Научиться создавать и работать с логическими томами.

### Уменьшить том под / до 8G:
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
                                        
```
</details>