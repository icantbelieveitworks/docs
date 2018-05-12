В linux kernel появилась новая уязвимость CVE-2016-5195 (<a href="https://habr.com/company/pentestit/blog/313276/">подробное описание</a>)<br/>
Попробуем воспроизвести. Далее с помощью KernelCare (без перезагрузки сервера) => пропатчить ядро.<br/>


Создаем файл, записываем текст. Владелец `root`, права `0404`
```
# echo this is not a test > foo
# chmod 0404 foo
# ls -lah foo
-r-----r-- 1 root root 19 Oct 26 02:23 foo
```

Создаем пользователя, заходим под ним.
```
# adduser --disabled-login test
# su test
```

Пытаемся перезаписать файл.
```
# echo m00000000000000000 > foo
bash: foo: Permission denied
```

Скачиваем Proof-of-Concept эксплоит.
```
wget https://raw.githubusercontent.com/dirtycow/dirtycow.github.io/master/dirtyc0w.c
```

Компилируем и применяем.
```
# gcc -pthread dirtyc0w.c -o dirtyc0w
# ./dirtyc0w foo m00000000000000000
mmap 7fb2d8d05000

madvise 0

procselfmem 1800000000
```

Файл перезаписан.
```
# cat foo
m00000000000000000

# ls -lah foo
-r-----r-- 1 root root 19 Oct 26 02:23 foo
```

<hr/>

Тестируем KernelCare. На <a href="https://www.cloudlinux.com/all-products/product-overview/kernelcare">официальном сайте</a>- получаем trial лицензию на 30 дней.

```
## centOS
# rpm -i https://downloads.kernelcare.com/kernelcare-latest.x86_64.rpm
## Debian
# wget http://patches.kernelcare.com/kernelcare-latest.deb
# dpkg -i kernelcare-latest.deb
# kcarectl  --register x
```

После установки - автоматически устанавливается апдейт.
```
# kcarectl --info
kpatch-state: patch is applied
kpatch-for: Linux version 3.16.0-4-amd64 (debian-kernel@lists.debian.org) (gcc version 4.8.4 (Debian 4.8.4-1) ) #1 SMP Debian 3.16.7-ckt25-2+deb8u3 (2016-07-02)
kpatch-build-time: Fri Oct 21 02:05:59 2016
kpatch-description: 3;3.16.36-1+deb8u2

# kcarectl --update
Kernel is safe

# dmesg|grep 'kcare'
[2467632.682121] kcare: registered device with node 10:59
[2467637.409636] kcare: allocated 94448 bytes for patch at ffffc90007624000
[2467637.409650] kcare: verifying patch...
[2467637.409651] kcare: verified successfully
[2467637.409651] kcare: allocating memory in module space...
[2467637.409663] kcare: allocated 94448 bytes at ffffffffa0590000
[2467637.409671] kcare: 1037 relocations to fixup...
[2467637.409675] kcare: fixed 1037 relocations
[2467637.409675] kcare: jumping to ffffffffa0598b80
```

Пробуем повторить. Запускаем Proof-of-Concept эксплоит.
```
# echo this is not a test > foo
# su test
# cat foo 
this is not a test

# echo m00000000000000000 > foo 
bash: foo: Permission denied

# ./dirtyc0w foo m00000000000000000
mmap 7f8ccbb21000

madvise 0

procselfmem 1800000000

# cat foo
this is not a test
```

Отлично, ядро пропатчено без перезагрузки сервера, уязвимость не работает.
