Проверить время на сервере можно с помощью команды.
```
date
```
Debian/ Ubuntu.
```
dpkg-reconfigure tzdata
```
CentOS => смотрим часовой пояс, выводим список доступных, меняем часовой пояс.
```
# ls -l /etc/localtime
lrwxrwxrwx. 1 root root 38 May 27 08:11 /etc/localtime -> ../usr/share/zoneinfo/America/New_York
# timedatectl list-timezones
# timedatectl set-timezone Europe/Moscow
```
