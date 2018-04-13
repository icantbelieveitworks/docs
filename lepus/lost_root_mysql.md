Заходим на сервер по SSH, выключаем MySQL.
```
/etc/init.d/mysql stop
```
Запускаем MySQL с параметрами.
```
/usr/bin/mysqld_safe --skip-grant-tables --user=root &
```
Заходим в консоль MySQL.
```
mysql -u root
```
Меняем пароль для пользователя root.
```
UPDATE mysql.user SET Password=PASSWORD('newpassword') WHERE User='root';
FLUSH PRIVILEGES;
exit;
```
Перезагружаем MySQL.
```
/etc/init.d/mysql restart
```
