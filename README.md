# **Инструкция по выполнению домашнего задания**

### **Настраиваем сервер NFS** 

Заходим на сервер: 192.168.90.60

**`root@nfss:/etc#`** `apt install nfs-kernel-server`

Настройки сервера находятся в файле `/etc/nfs.conf` 

Проверяем наличие слушающих портов `2049/udp, 2049/tcp,111/udp, 111/tcp` (не все они будут использоваться далее,  но их наличие сигнализирует о том, что необходимые сервисы готовы принимать внешние подключения):  
**`root@nfss:/etc#`** `ss -tnplu` 

Создаём и настраиваем директорию, которая будет экспортирована в будущем   
**`root@nfss:/etc#`** `mkdir -p /srv/share/upload`   
**`root@nfss:/etc#`** `chown -R nobody:nogroup /srv/share`   
**`root@nfss:/etc#`** `chmod 0777 /srv/share/upload` 

Cоздаём в файле `/etc/exports` структуру, которая позволит экспортировать ранее созданную директорию:  
**`root@nfss:/etc#`** `cat << EOF > /etc/exports`   
`/srv/share 192.168.90.89/23(rw,sync,root_squash)`  
`EOF`

Экспортируем ранее созданную директорию:  
**`root@nfss:/etc#`** `exportfs -r` 

Проверяем экспортированную директорию следующей командой  
**`root@nfss:/etc#`** `exportfs -s` 

Вывод должен быть аналогичен этому: 

**`root@nfss:/etc#`** `exportfs -s`   
`/srv/share  192.168.90.89/23(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)`

### **Настраиваем клиент NFS** 

Заходим на сервер 192.168.90.89

Дальнейшие действия выполняются от имени пользователя имеющего повышенные привилегии, разрешающие описанные действия.   
Установим пакет с NFS-клиентом  
**`root@nfsc:~#`** `sudo apt install nfs-common`

Добавляем в `/etc/fstab` строку   
**`root@nfsc:~#`** `echo "192.168.90.60:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab`

и выполняем команды:  
**`root@nfsc:~#`** `systemctl daemon-reload`   
**`root@nfsc:~#`** `systemctl restart remote-fs.target` 
**`root@nfsc:~#`** `mount /mnt/` 

Заходим в директорию `/mnt/` и проверяем успешность монтирования:  
**`root@nfsc:~#`** `mount | grep mnt` 

При успехе вывод должен примерно соответствовать этому:  
**`root@dz-7:~#`** `mount | grep mnt`   
`systemd-1 on /mnt type autofs (rw,relatime,fd=71,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=30269)\n192.168.90.60:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.90.60,mountvers=3,mountport=52172,mountproto=udp,local_lock=none,addr=192.168.90.60)`

### **Проверка работоспособности** 

* Заходим на сервер.   
* Заходим в каталог `/srv/share/upload`.  
* Создаём тестовый файл `touch check_file`.  
* Заходим на клиент.  
* Заходим в каталог `/mnt/upload`.   
* Проверяем наличие ранее созданного файла.  
* Создаём тестовый файл `touch client_file`.   
* Проверяем, что файл успешно создан.

Предварительно проверяем клиент: 

1. перезагружаем клиент
   root@dz-7-1:/mnt/upload# reboot  
1. заходим на клиент
1. заходим в каталог 
   root@dz-7-1:~# cd /mnt/upload/ 
1. проверяем наличие ранее созданных файлов.
   root@dz-7-1:/mnt/upload# ll
drwxrwxrwx 2 nobody nogroup 4096 Jul 31 10:58 ./
drwxr-xr-x 3 nobody nogroup 4096 Jul 31 08:15 ../
-rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 check_file
-rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 client_file

Проверяем сервер: 

1. заходим на сервер в отдельном окне терминала;  
1. перезагружаем сервер;  
   root@dz-7:~# reboot  
1. заходим на сервер;  
1. проверяем наличие файлов в каталоге `/srv/share/upload/`;  
   root@dz-7:~# ll /srv/share/upload/
   drwxrwxrwx 2 nobody nogroup 4096 Jul 31 10:58 ./
   drwxr-xr-x 3 nobody nogroup 4096 Jul 31 08:15 ../
   -rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 check_file
   -rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 client_file
1. проверяем экспорты `exportfs -s`;  
   root@dz-7:~# exportfs -s
   /srv/share  192.168.90.89/23(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
   
Проверяем клиент: 

1. возвращаемся на клиент;  
1. перезагружаем клиент;  
   root@dz-7-1:/mnt/upload# reboot
1. заходим на клиент;  
1. проверяем работу RPC `showmount -a 192.168.90.60`;  
   root@dz-7-1:/mnt/upload# showmount -a 192.168.90.60
   All mount points on 192.168.90.60:
   192.168.90.89:/srv/share
1. заходим в каталог `/mnt/upload`;  
1. проверяем статус монтирования `mount | grep mnt`;
   root@dz-7-1:/mnt/upload# mount | grep mnt
   systemd-1 on /mnt type autofs (rw,relatime,fd=54,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=3990)
   192.168.90.60:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.90.60,mountvers=3,mountport=52172,mountproto=udp,local_lock=none,addr=192.168.90.60)

1. проверяем наличие ранее созданных файлов; 
   root@dz-7-1:/mnt/upload# ls -al /mnt/upload/
   drwxrwxrwx 2 nobody nogroup 4096 Jul 31 10:58 .
   drwxr-xr-x 3 nobody nogroup 4096 Jul 31 08:15 ..
   -rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 check_file
   -rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 client_file

1. создаём тестовый файл `touch final_check`;  
   root@dz-7-1:/mnt/upload# touch final_check
1. проверяем, что файл успешно создан.
   root@dz-7-1:/mnt/upload# ls -al /mnt/upload/
   drwxrwxrwx 2 nobody nogroup 4096 Jul 31 13:15 .
   drwxr-xr-x 3 nobody nogroup 4096 Jul 31 08:15 ..
   -rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 check_file
   -rw-r--r-- 1 nobody nogroup    0 Jul 31 11:04 client_file
   -rw-r--r-- 1 nobody nogroup    0 Jul 31 13:15 final_check



