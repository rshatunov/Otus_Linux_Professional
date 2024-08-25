# LAB-06
## ZFS
### Цели
- Научится самостоятельно устанавливать ZFS, настраивать пулы, изучить основные возможности ZFS

### Описание домашнего задания
1. Определить алгоритм с наилучшим сжатием:
	- определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
	- создать 4 файловых системы на каждой применить свой алгоритм сжатия;
	- для сжатия использовать либо текстовый файл, либо группу файлов.
2. Определить настройки пула. С помощью команды zfs import собрать pool ZFS. Командами zfs определить настройки:
	- размер хранилища;
	- тип pool;
	- значение recordsize;
	- какое сжатие используется;
	- какая контрольная сумма используется.
3. Работа со снапшотами:
	- скопировать файл из удаленной директории;
	- восстановить файл локально. zfs receive;
	- найти зашифрованное сообщение в файле secret_message.


### Комментарии
1. С помощью Vagrant и Ansible разварачиваются ВМ
2. Определение алгоритма с наилучшим сжатием
 <details>
<summary> Просмотр списка дисков: </summary>

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





	- Просмотр списка дисков
```bash

```
	- Создание пулов из двух дисков в режиме RAID1
```bash

```
	- Просмотр информации о пулах дисков
```bash
root@client:~# systemctl list-timers  
NEXT                        LEFT          LAST                        PASSED               UNIT                         ACTIVATES                     
Thu 2024-08-22 10:38:30 +05 16s left      Thu 2024-08-22 10:33:30 +05 4min 43s ago         borg-backup.timer            borg-backup.service
root@client:~# 
root@client:~# borg list borg@192.168.11.12:/var/backup/
Enter passphrase for key ssh://borg@192.168.11.12/var/backup: 
etc-2024-08-22_09:58:44              Thu, 2024-08-22 09:58:45 [8b9588a247dc5501515e983dd45d71fa61071b5d0f62e424d7393b5c3ec2672d]
etc-2024-08-22_10:45:30              Thu, 2024-08-22 10:45:31 [ea4c29c62d24c49e5a6f81902b50ea010ef61bd48ccb5e35f4f6d26f3c590b58]
```
	- Просмотр списка дисков
```bash
root@client:~# systemctl list-timers  
NEXT                        LEFT          LAST                        PASSED               UNIT                         ACTIVATES                     
Thu 2024-08-22 10:38:30 +05 16s left      Thu 2024-08-22 10:33:30 +05 4min 43s ago         borg-backup.timer            borg-backup.service
root@client:~# 
root@client:~# borg list borg@192.168.11.12:/var/backup/
Enter passphrase for key ssh://borg@192.168.11.12/var/backup: 
etc-2024-08-22_09:58:44              Thu, 2024-08-22 09:58:45 [8b9588a247dc5501515e983dd45d71fa61071b5d0f62e424d7393b5c3ec2672d]
etc-2024-08-22_10:45:30              Thu, 2024-08-22 10:45:31 [ea4c29c62d24c49e5a6f81902b50ea010ef61bd48ccb5e35f4f6d26f3c590b58]
```
	- Просмотр списка дисков
```bash
root@client:~# systemctl list-timers  
NEXT                        LEFT          LAST                        PASSED               UNIT                         ACTIVATES                     
Thu 2024-08-22 10:38:30 +05 16s left      Thu 2024-08-22 10:33:30 +05 4min 43s ago         borg-backup.timer            borg-backup.service
root@client:~# 
root@client:~# borg list borg@192.168.11.12:/var/backup/
Enter passphrase for key ssh://borg@192.168.11.12/var/backup: 
etc-2024-08-22_09:58:44              Thu, 2024-08-22 09:58:45 [8b9588a247dc5501515e983dd45d71fa61071b5d0f62e424d7393b5c3ec2672d]
etc-2024-08-22_10:45:30              Thu, 2024-08-22 10:45:31 [ea4c29c62d24c49e5a6f81902b50ea010ef61bd48ccb5e35f4f6d26f3c590b58]
```
	- Просмотр списка дисков
```bash
root@client:~# systemctl list-timers  
NEXT                        LEFT          LAST                        PASSED               UNIT                         ACTIVATES                     
Thu 2024-08-22 10:38:30 +05 16s left      Thu 2024-08-22 10:33:30 +05 4min 43s ago         borg-backup.timer            borg-backup.service
root@client:~# 
root@client:~# borg list borg@192.168.11.12:/var/backup/
Enter passphrase for key ssh://borg@192.168.11.12/var/backup: 
etc-2024-08-22_09:58:44              Thu, 2024-08-22 09:58:45 [8b9588a247dc5501515e983dd45d71fa61071b5d0f62e424d7393b5c3ec2672d]
etc-2024-08-22_10:45:30              Thu, 2024-08-22 10:45:31 [ea4c29c62d24c49e5a6f81902b50ea010ef61bd48ccb5e35f4f6d26f3c590b58]
```



3. Проверка логирования скрипта:    
```bash
root@client:~# journalctl -u borg-backup.service -n21
авг 22 10:33:34 client systemd[1]: borg-backup.service: Consumed 1.861s CPU time.
авг 22 10:39:30 client systemd[1]: Starting borg-backup.service - Borg Backup...
авг 22 10:39:31 client borg[7206]: ------------------------------------------------------------------------------
авг 22 10:39:31 client borg[7206]: Repository: ssh://borg@192.168.11.12/var/backup
авг 22 10:39:31 client borg[7206]: Archive name: etc-2024-08-22_10:39:30
авг 22 10:39:31 client borg[7206]: Archive fingerprint: dc9f95fb0fe5020dffcfc756d3e86e9584c522ac287900c8de25dd1b683c86a3
авг 22 10:39:31 client borg[7206]: Time (start): Thu, 2024-08-22 10:39:31
авг 22 10:39:31 client borg[7206]: Time (end):   Thu, 2024-08-22 10:39:31
авг 22 10:39:31 client borg[7206]: Duration: 0.10 seconds
авг 22 10:39:31 client borg[7206]: Number of files: 541
авг 22 10:39:31 client borg[7206]: Utilization of max. archive size: 0%
авг 22 10:39:31 client borg[7206]: ------------------------------------------------------------------------------
авг 22 10:39:31 client borg[7206]:                        Original size      Compressed size    Deduplicated size
авг 22 10:39:31 client borg[7206]: This archive:                1.75 MB            776.09 kB                646 B
авг 22 10:39:31 client borg[7206]: All archives:                5.26 MB              2.33 MB            829.69 kB
авг 22 10:39:31 client borg[7206]:                        Unique chunks         Total chunks
авг 22 10:39:31 client borg[7206]: Chunk index:                     517                 1587
авг 22 10:39:31 client borg[7206]: ------------------------------------------------------------------------------
авг 22 10:39:34 client systemd[1]: borg-backup.service: Deactivated successfully.
авг 22 10:39:34 client systemd[1]: Finished borg-backup.service - Borg Backup.
авг 22 10:39:34 client systemd[1]: borg-backup.service: Consumed 1.875s CPU time.
```