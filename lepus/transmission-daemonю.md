Установка.
```
apt-get install transmission-daemon
```

Перед тем как редактировать `/etc/transmission-daemon/settings.json` - выключите `transmission`.
```
/etc/init.d/transmission-daemon stop
```

Поменяем папку.
```
# "download-dir": "/var/www/torrent",
mkdir /var/www/torrent/
chown debian-transmission:debian-transmission /var/www/torrent/
```

Поменяем логин и пароль.
```
"rpc-username": "login",
"rpc-password": "passwd",
```

Доступ к rpc `http://IP:9091`
```
"rpc-whitelist": "127.0.0.1",
"rpc-whitelist": "127.0.0.1, 127.0.0.50",
"rpc-whitelist": "*",
```

<hr/>

Nginx proxy.
```
"rpc-bind-address": "127.0.0.1",
"rpc-port": 9091,
```
```
location ~ ^/transmission {
  proxy_pass http://127.0.0.1:9091;
  proxy_pass_header X-Transmission-Session-Id;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

<hr/>

Auto upload torrents.
```
adduser --disabled-login torrent
mkdir /home/torrent/tmp
cat <<EOF >/home/torrent/upload.bash
#!/bin/bash
cd /home/torrent/tmp
wget --no-directories --content-disposition --restrict-file-names=nocontrol -e robots=off -A.torrent -r https://www.anilibria.tv/wget_torrents.php
for f in in /home/torrent/tmp/*.torrent; do
   transmission-remote --auth transmission:HuHacOmcass2 -a $f
done
rm /home/torrent/tmp/*.torrent
EOF

chmod 755 /home/torrent/upload.bash
chown torrent:torrent /home/torrent/tmp
chown torrent:torrent /home/torrent/upload.bash

cat <<EOF >/etc/cron.d/torrent
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=""
HOME=/

10 15 * * *   root   /root/scripts/video.bash >/dev/null 2>&1
EOF

/etc/init.d/crond restart
```

<hr/>

<img src="https://img.poiuty.com/img/01/0f214031262d1d2070ecb5d01b9ce101.png">
