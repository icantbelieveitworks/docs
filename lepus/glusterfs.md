GlusterFS - это распределённая, параллельная, линейно масштабируемая файловая система.<br/>
В тестировании участвуют два сервера (Debian 8), на каждый из них => установим GlusterFS.
```
apt-get install glusterfs-server
```

Редактируем `/etc/hosts` на первом сервере (на каждом сервере).
```
149.202.127.90   server1.gfs     server1
149.202.127.89   server2.gfs     server2
```

Ниже приведены => часто используемые команды.
```
glusterfsd --version
gluster peer probe...
gluster peer status
gluster volume create testvol...
gluster volume info
gluster volume start testvol
gluster volume stop testvol
gluster volume delete testvol
gluster volume set testvol auth.allow 192.168.0.2 # разрешить доступ по IP
```

Монтирование через `/etc/fstab`
```
server1.gfs:/testvol /mnt/glusterfs glusterfs defaults,_netdev 0 0
```

Правила iptables (на оба сервера).
```
iptables -N ALLOWED
iptables -A INPUT -j ALLOWED
iptables -A ALLOWED -i lo -j ACCEPT
iptables -A ALLOWED -s 149.202.127.90 -j ACCEPT
iptables -A ALLOWED -s 149.202.127.89 -j ACCEPT
iptables -A INPUT -p tcp --dport 111 -j DROP
iptables -A INPUT -p tcp --dport 2049 -j DROP
iptables -A INPUT -p tcp --dport 24007:24011 -j DROP
iptables -A INPUT -p tcp --dport 49152:59153 -j DROP
iptables -A INPUT -p tcp --dport 38465:38467 -j DROP
ip6tables -A INPUT -p tcp -j DROP
```

<hr/>

Шифрование. На первом сервере создаем ключи.
```
openssl genrsa -out /etc/ssl/glusterfs.key 1024
openssl req -new -x509 -key /etc/ssl/glusterfs.key -subj /CN=Anyone -out /etc/ssl/glusterfs.pem
cp /etc/ssl/glusterfs.pem /etc/ssl/glusterfs.ca
```

Копируем ключи на второй сервер.
```
scp /etc/ssl/glusterfs.ca root@149.202.127.89:/etc/ssl/
scp /etc/ssl/glusterfs.key root@149.202.127.89:/etc/ssl/
scp /etc/ssl/glusterfs.pem root@149.202.127.89:/etc/ssl/
```

На первом и на втором выполняем.
```
echo '' > /var/lib/glusterd/secure-access
gluster volume set testvol client.ssl on
gluster volume set testvol server.ssl on
/etc/init.d/glusterfs-server restart
```

Проверяем.
```
# cat /var/log/glusterfs/glustershd.log | grep 'SSL support is ENABLED'
[2016-06-03 18:14:56.381777] I [socket.c:3578:socket_init] 0-testvol-client-1: SSL support is ENABLED
[2016-06-03 18:14:56.388131] I [socket.c:3578:socket_init] 0-testvol-client-0: SSL support is ENABLED
```

<hr/>

Файловый сервер. На первом сервере выполняем команды.
```
gluster volume create testvol server1.gfs:/data force
gluster volume start testvol
gluster volume set testvol auth.allow 149.202.127.89
```

На втором сервере монтируем.
```
mount -t glusterfs server1.gfs:/testvol /mnt
```

<hr/>

Репликация. На первом сервере выполняем команды.
```
gluster peer probe 149.202.127.89
gluster peer status
gluster volume create testvol replica 2 transport tcp 149.202.127.90:/data 149.202.127.89:/data force
gluster volume start testvol
gluster volume set testvol auth.allow 149.202.127.89,149.202.127.90
```

На первом сервере монтируем.
```
mount -t glusterfs server1.gfs:/testvol /mnt
```

И на втором.
```
mount -t glusterfs server2.gfs:/testvol /mnt
```

<hr/>

Striped. На первом сервере выполняем команды.
```
gluster peer probe 149.202.127.89
gluster peer status
gluster volume create testvol stripe 2 transport tcp 149.202.127.90:/data 149.202.127.89:/data force
gluster volume start testvol
gluster volume set testvol auth.allow 149.202.127.89,149.202.127.90
```

На первом сервере монтируем.
```
mount -t glusterfs server1.gfs:/testvol /mnt
```

И на втором.
```
mount -t glusterfs server2.gfs:/testvol /mnt
```
