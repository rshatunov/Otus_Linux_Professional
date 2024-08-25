# LAB-18
## Резервное копирование
### Цели
- Научиться настраивать резервное копирование с помощью утилиты Borg

### Описание домашнего задания
1. Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. (Студент самостоятельно настраивает Vagrant)
2. Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:
	- директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB; (Студент самостоятельно настраивает)
	- репозиторий для резервных копий должен быть зашифрован ключом или паролем - на усмотрение студента;
	- имя бэкапа должно содержать информацию о времени снятия бекапа;
	- глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;
	- резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;
	- написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на усмотрение студента;
	- настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.

### Комментарии
1. С помощью Vagrant и Ansible разварачиваются и настраиваются две ВМ (server и client)
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