# LAB-16
## Vagrant-стенд c PAM
### Цели
- Научиться создавать пользователей и добавлять им ограничения

### Уменьшить том под / до 8G
 <details>
<summary> Создание временного тома: </summary>

```
root@pam:~# useradd otusadm -m && sudo useradd otus -m
root@pam:~#
root@pam:~# echo -e "Otus2022\nOtus2022" | passwd otusadm && echo -e "Otus2022\nOtus2022" | passwd otus
New password: Retype new password: passwd: password updated successfully
New password: Retype new password: passwd: password updated successfully
root@pam:~# 
root@pam:~# groupadd -f admin
root@pam:~# 
root@pam:~# usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin	
root@pam:~#                                                   
```
</details>
