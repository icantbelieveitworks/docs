
Подключаем stretch-backports, устанавливаем nginx и плагин <a href="https://github.com/arut/nginx-rtmp-module">nginx-mod-rtmp</a>.

```
echo 'deb http://ftp.debian.org/debian stretch-backports main' > /etc/apt/sources.list
apt-get update & apt-get upgrade
apt-get -t stretch-backports install nginx libnginx-mod-rtmp
```

Редактируем `/etc/nginx/nginx.conf`
```
http {
  ...
	server {
		listen 80 default_server;
		listen [::]:80 default_server;
		index index.html;
		root /var/www;
		location /hls/ {
			types {
				application/vnd.apple.mpegurl m3u8;
			}
			add_header Cache-Control no-cache;
			add_header Access-Control-Allow-Origin *;
		}
	}
...
}

rtmp_auto_push on;
rtmp {
	server {
	listen 1935;
	chunk_size 4000;
		application src {
			live on;
			hls on;
			hls_path /var/www/hls;
			hls_fragment 3s;
			hls_playlist_length 5s;
		}
	}
}
```

Запускаем стрим.
```
ffmpeg -y -f x11grab -r 30 -i :0 -c:v libx264 -pix_fmt yuv420p -preset veryfast -g 60 -f flv rtmp://IP:1935/src/stream
```

<img src="https://img.poiuty.com/img/f1/d3bf6d75d8e8b873481a09c7ed0c20f1.png">
