# LAB-19
## Стенд ссетевой лабораторией
### Цели
- Научится менять базовые сетевые настройки в Linux-based системах.

### Описание домашнего задания
1. Скачать и развернуть Vagrant-стенд https://github.com/erlong15/otus-linux/tree/network
2. Построить следующую сетевую архитектуру:

Сеть office1

	- 192.168.2.0/26      - dev
	- 192.168.2.64/26     - test servers
	- 192.168.2.128/26    - managers
	- 192.168.2.192/26    - office hardware

Сеть office2

	- 192.168.1.0/25      - dev
	- 192.168.1.128/26    - test servers
	- 192.168.1.192/26    - office hardware

Сеть central

	- 192.168.0.0/28     - directors
	- 192.168.0.32/28    - office hardware
	- 192.168.0.64/26    - wifi

![pic.jpg](pic.jpg)

Итого должны получиться следующие сервера:

	- inetRouter
	- centralRouter
	- office1Router
	- office2Router
	- centralServer
	- office1Server
	- office2Server

Задание состоит из 2-х частей: теоретической и практической.
В теоретической части требуется: 
- Найти свободные подсети
- Посчитать количество узлов в каждой подсети, включая свободные
- Указать Broadcast-адрес для каждой подсети
- Проверить, нет ли ошибок при разбиении

В практической части требуется: 

	- Соединить офисы в сеть согласно логической схеме и настроить роутинг
	- Интернет-трафик со всех серверов должен ходить через inetRouter
	- Все сервера должны видеть друг друга (должен проходить ping)
	- У всех новых серверов отключить дефолт на NAT (eth0), который vagrant поднимает для связи
	- Добавить дополнительные сетевые интерфейсы, если потребуется

### Комментарии
1. С помощью Vagrant и Ansible разварачиваются и настраиваются семь ВМ
2. Проверка работы скрипта:    
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