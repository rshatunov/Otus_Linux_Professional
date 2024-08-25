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
root@zfs:~# lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   128G  0 disk 
├─sda1   8:1    0   487M  0 part /boot
├─sda2   8:2    0   1,9G  0 part [SWAP]
└─sda3   8:3    0 125,6G  0 part /
sdb      8:16   0     1G  0 disk 
sdc      8:32   0     1G  0 disk 
sdd      8:48   0     1G  0 disk 
sde      8:64   0     1G  0 disk 
sdf      8:80   0     1G  0 disk 
sdg      8:96   0     1G  0 disk 
sdh      8:112  0     1G  0 disk 
sdi      8:128  0     1G  0 disk                                                     
```
</details>
 <details>
<summary> Создание пулов из двух дисков в режиме RAID1: </summary>

```
root@zfs:~# zpool create otus1 mirror /dev/sdb /dev/sdc
root@zfs:~# zpool create otus2 mirror /dev/sdd /dev/sde
root@zfs:~# zpool create otus3 mirror /dev/sdf /dev/sdg
root@zfs:~# zpool create otus4 mirror /dev/sdh /dev/sdi                                                    
root@zfs:~#
root@zfs:~#
root@zfs:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   960M   116K   960M        -         -     0%     0%  1.00x    ONLINE  -
otus2   960M   108K   960M        -         -     0%     0%  1.00x    ONLINE  -
otus3   960M   114K   960M        -         -     0%     0%  1.00x    ONLINE  -
otus4   960M   122K   960M        -         -     0%     0%  1.00x    ONLINE  -                                                   
```
</details>
 <details>
<summary> Настройка алгоритмов сжатия для файловой системы: </summary>

```
root@zfs:~# zfs set compression=lzjb otus1
root@zfs:~# zfs set compression=lz4 otus2
root@zfs:~# zfs set compression=gzip-9 otus3
root@zfs:~# zfs set compression=zle otus4                                                    
root@zfs:~# 
root@zfs:~#
root@zfs:~# zfs get all | grep compression
otus1  compression           lzjb                       local
otus2  compression           lz4                        local
otus3  compression           gzip-9                     local
otus4  compression           zle                        local                                                   
```
</details>
 <details>
<summary> Загрузка одного и того же файла на файловые системы: </summary>

```
root@zfs:~# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
--2024-08-25 21:03:04--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Распознаётся gutenberg.org (gutenberg.org)… 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Подключение к gutenberg.org (gutenberg.org)|152.19.134.47|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 41070955 (39M) [text/plain]
Сохранение в: «/otus1/pg2600.converter.log»

pg2600.converter.log                      100%[==================================================================================>]  39,17M  7,76MB/s    за 5,9s    

2024-08-25 21:03:11 (6,69 MB/s) - «/otus1/pg2600.converter.log» сохранён [41070955/41070955]

--2024-08-25 21:03:11--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Распознаётся gutenberg.org (gutenberg.org)… 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Подключение к gutenberg.org (gutenberg.org)|152.19.134.47|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 41070955 (39M) [text/plain]
Сохранение в: «/otus2/pg2600.converter.log»

pg2600.converter.log                      100%[==================================================================================>]  39,17M  9,25MB/s    за 5,0s    

2024-08-25 21:03:17 (7,77 MB/s) - «/otus2/pg2600.converter.log» сохранён [41070955/41070955]

--2024-08-25 21:03:17--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Распознаётся gutenberg.org (gutenberg.org)… 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Подключение к gutenberg.org (gutenberg.org)|152.19.134.47|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 41070955 (39M) [text/plain]
Сохранение в: «/otus3/pg2600.converter.log»

pg2600.converter.log                      100%[==================================================================================>]  39,17M  4,86MB/s    за 7,4s    

2024-08-25 21:03:25 (5,29 MB/s) - «/otus3/pg2600.converter.log» сохранён [41070955/41070955]

--2024-08-25 21:03:25--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Распознаётся gutenberg.org (gutenberg.org)… 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Подключение к gutenberg.org (gutenberg.org)|152.19.134.47|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 41070955 (39M) [text/plain]
Сохранение в: «/otus4/pg2600.converter.log»

pg2600.converter.log                      100%[==================================================================================>]  39,17M  10,2MB/s    за 4,7s    

2024-08-25 21:03:31 (8,40 MB/s) - «/otus4/pg2600.converter.log» сохранён [41070955/41070955]

root@zfs:~# 
root@zfs:~# 
root@zfs:~# ls -l /otus*
/otus1:
итого 22084
-rw-r--r-- 1 root root 41070955 авг  2 12:54 pg2600.converter.log

/otus2:
итого 18001
-rw-r--r-- 1 root root 41070955 авг  2 12:54 pg2600.converter.log

/otus3:
итого 10963
-rw-r--r-- 1 root root 41070955 авг  2 12:54 pg2600.converter.log

/otus4:
итого 40136
-rw-r--r-- 1 root root 41070955 авг  2 12:54 pg2600.converter.log
                                                   
```
</details>
 <details>
<summary> Проверка степени сжатия тестового файла: </summary>

```
root@zfs:~# zfs list
NAME    USED  AVAIL  REFER  MOUNTPOINT
otus1  21.7M   810M  21.6M  /otus1
otus2  17.7M   814M  17.6M  /otus2
otus3  10.9M   821M  10.7M  /otus3
otus4  39.3M   793M  39.2M  /otus4
root@zfs:~# 
root@zfs:~# 
root@zfs:~# 
root@zfs:~# zfs get all | grep compressratio | grep -v ref
otus1  compressratio         1.81x                      -
otus2  compressratio         2.23x                      -
otus3  compressratio         3.65x                      -
otus4  compressratio         1.00x                      -                                                    
```
</details>
3. Определение настроек пула
 <details>
<summary> Импорт пула: </summary>

```
root@zfs:~# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download' 
--2024-08-25 21:10:13--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download
Распознаётся drive.usercontent.google.com (drive.usercontent.google.com)… 172.217.20.65, 2a00:1450:4017:804::2001
Подключение к drive.usercontent.google.com (drive.usercontent.google.com)|172.217.20.65|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 7275140 (6,9M) [application/octet-stream]
Сохранение в: «archive.tar.gz»

archive.tar.gz                            100%[==================================================================================>]   6,94M  9,32MB/s    за 0,7s    

2024-08-25 21:10:22 (9,32 MB/s) - «archive.tar.gz» сохранён [7275140/7275140]

root@zfs:~# 
root@zfs:~# 
root@zfs:~# 
root@zfs:~# ls
archive.tar.gz
root@zfs:~# 
root@zfs:~#
root@zfs:~# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
root@zfs:~# 
root@zfs:~#
root@zfs:~# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
	(Note that they may be intentionally disabled if the
	'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
	some features will not be available without an explicit 'zpool upgrade'.
 config:

	otus                         ONLINE
	  mirror-0                   ONLINE
	    /root/zpoolexport/filea  ONLINE
	    /root/zpoolexport/fileb  ONLINE
root@zfs:~# 
root@zfs:~# 
root@zfs:~# 
root@zfs:~# zpool import -d zpoolexport/ otus
root@zfs:~# 
root@zfs:~#
root@zfs:~# zpool status otus
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
config:

	NAME                         STATE     READ WRITE CKSUM
	otus                         ONLINE       0     0     0
	  mirror-0                   ONLINE       0     0     0
	    /root/zpoolexport/filea  ONLINE       0     0     0
	    /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors                                                  
```
</details>
 <details>
<summary> Определение настроек пула: </summary>

```
root@zfs:~# zfs get all otus
NAME  PROPERTY              VALUE                      SOURCE
otus  type                  filesystem                 -
otus  creation              Пт мая 15  9:00 2020  -
otus  used                  2.04M                      -
otus  available             350M                       -
otus  referenced            24K                        -
otus  compressratio         1.00x                      -
otus  mounted               yes                        -
otus  quota                 none                       default
otus  reservation           none                       default
otus  recordsize            128K                       local
otus  mountpoint            /otus                      default
otus  sharenfs              off                        default
otus  checksum              sha256                     local
otus  compression           zle                        local
otus  atime                 on                         default
otus  devices               on                         default
otus  exec                  on                         default
otus  setuid                on                         default
otus  readonly              off                        default
otus  zoned                 off                        default
otus  snapdir               hidden                     default
otus  aclmode               discard                    default
otus  aclinherit            restricted                 default
otus  createtxg             1                          -
otus  canmount              on                         default
otus  xattr                 on                         default
otus  copies                1                          default
otus  version               5                          -
otus  utf8only              off                        -
otus  normalization         none                       -
otus  casesensitivity       sensitive                  -
otus  vscan                 off                        default
otus  nbmand                off                        default
otus  sharesmb              off                        default
otus  refquota              none                       default
otus  refreservation        none                       default
otus  guid                  14592242904030363272       -
otus  primarycache          all                        default
otus  secondarycache        all                        default
otus  usedbysnapshots       0B                         -
otus  usedbydataset         24K                        -
otus  usedbychildren        2.01M                      -
otus  usedbyrefreservation  0B                         -
otus  logbias               latency                    default
otus  objsetid              54                         -
otus  dedup                 off                        default
otus  mlslabel              none                       default
otus  sync                  standard                   default
otus  dnodesize             legacy                     default
otus  refcompressratio      1.00x                      -
otus  written               24K                        -
otus  logicalused           1020K                      -
otus  logicalreferenced     12K                        -
otus  volmode               default                    default
otus  filesystem_limit      none                       default
otus  snapshot_limit        none                       default
otus  filesystem_count      none                       default
otus  snapshot_count        none                       default
otus  snapdev               hidden                     default
otus  acltype               off                        default
otus  context               none                       default
otus  fscontext             none                       default
otus  defcontext            none                       default
otus  rootcontext           none                       default
otus  relatime              on                         default
otus  redundant_metadata    all                        default
otus  overlay               on                         default
otus  encryption            off                        default
otus  keylocation           none                       default
otus  keyformat             none                       default
otus  pbkdf2iters           0                          default
otus  special_small_blocks  0                          default
otus  prefetch              all                        default
root@zfs:~# 
root@zfs:~# 
root@zfs:~# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
root@zfs:~# 
root@zfs:~# 
root@zfs:~# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
root@zfs:~# 
root@zfs:~#           
root@zfs:~# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
root@zfs:~# 
root@zfs:~# 
root@zfs:~# zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
root@zfs:~# 
root@zfs:~# 
root@zfs:~# zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local                                             
```
</details>
4. Работа со снапшотами
 <details>
<summary> Восстановление файловой системы из снапшота: </summary>

```
root@zfs:~# wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
[1] 32652
root@zfs:~# 
Вывод перенаправляется в «wget-log».

[1]+  Завершён        wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI
root@zfs:~# 
root@zfs:~# 
root@zfs:~# zfs receive otus/test@today < otus_task2.file                     -                                                    
```
</details>
 <details>
<summary> Поиска  файла и просмотр его содержимого: </summary>

```
root@zfs:~# 
root@zfs:~# 
root@zfs:~# find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
root@zfs:~# 
root@zfs:~# 
root@zfs:~# cat /otus/test/task1/file_mess/secret_message 
https://otus.ru/lessons/linux-hl/                                                   
```
</details>