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
3. Определение алгоритма с наилучшим сжатием
















 <details>
<summary> Настройка алгоритмов сжатия для файловой системы: </summary>

```
root@zfs:~# zfs set compression=lzjb otus1
root@zfs:~# zfs set compression=lz4 otus2
root@zfs:~# zfs set compression=gzip-9 otus3
root@zfs:~# zfs set compression=zle otus4                                                    
```
</details>