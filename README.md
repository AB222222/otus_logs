# otus_logs

в вагранте поднимаем 2 машины web и log
на web поднимаем nginx
на log настраиваем центральный лог сервер на любой системе на выбор
- journald
- rsyslog

(звёздочку я не делал, поэтому elk на log не ставил)

настраиваем аудит следящий за изменением конфигов нжинкса

все критичные логи с web должны собираться и локально и удаленно
все логи с nginx должны уходить на удаленный сервер (локально только критичные)
логи аудита должны также уходить на удаленную систему


на web поднимаем nginx

все критичные логи с web должны собираться и локально и удаленно


___________________________________________-

Логи на web пишутся и удалённо, и локально.

Локально (при выключенной опции crit в конфиге nginx -    error_log /var/log/nginx/error.log crit;)

```
[root@web nginx]# cat error.log
2021/02/06 16:52:40 [error] 4901#0: *4 open() "/usr/share/nginx/html/sdvsvsdv" failed (2: No such file or directory), client: ::1, server: _, request: "GET /sdvsvsdv HTTP/1.1", host: "localhost"
[root@web nginx]# mc
```

___________________________________________




Логи на log складываются сюда:

```
[root@log ~]# cd /mnt/logging
[root@log logging]# ll
total 0
drwxr-xr-x. 2 root root 205 Feb  6 16:57 192.168.10.22
[root@log logging]# cd 192.168.10.22/
[root@log 192.168.10.22]# ll
total 40
-rw-------. 1 root root   62 Feb  6 16:42 audisp-remote.log
-rw-------. 1 root root  506 Feb  6 16:52 nginx_access.log
-rw-------. 1 root root  696 Feb  6 16:52 nginx_error.log
-rw-------. 1 root root  134 Feb  6 16:57 ntpd.log
-rw-------. 1 root root  942 Feb  6 16:54 polkitd.log
-rw-------. 1 root root  596 Feb  6 16:57 sshd.log
-rw-------. 1 root root 1172 Feb  6 16:57 sudo.log
-rw-------. 1 root root  177 Feb  6 16:57 systemd-logind.log
-rw-------. 1 root root  725 Feb  6 16:57 systemd.log
-rw-------. 1 root root  138 Feb  6 16:48 yum.log
[root@log 192.168.10.22]#


[root@log 192.168.10.22]# cat audisp-remote.log
Feb  6 16:42:08 web audisp-remote: Connected to 192.168.10.23
[root@log 192.168.10.22]# cat nginx_access.log
Feb  6 16:47:01 web nginx_access: ::1 - - [06/Feb/2021:16:47:01 +0300] "GET / HTTP/1.1" 200 4833 "-" "curl/7.29.0" "-"
Feb  6 16:47:13 web nginx_access: ::1 - - [06/Feb/2021:16:47:13 +0300] "GET /nonexistedfile HTTP/1.1" 404 3650 "-" "curl/7.29.0" "-"
Feb  6 16:51:06 web nginx_access: ::1 - - [06/Feb/2021:16:51:06 +0300] "GET /sdvsvsdv HTTP/1.1" 404 3650 "-" "curl/7.29.0" "-"
Feb  6 16:52:40 web nginx_access: ::1 - - [06/Feb/2021:16:52:40 +0300] "GET /sdvsvsdv HTTP/1.1" 404 3650 "-" "curl/7.29.0" "-"
[root@log 192.168.10.22]# cat nginx_error.log
Feb  6 16:47:13 web nginx_error: 2021/02/06 16:47:13 [error] 4648#0: *2 open() "/usr/share/nginx/html/nonexistedfile" failed (2: No such file or directory), client: ::1, server: _, request: "GET /nonexi>
Feb  6 16:51:06 web nginx_error: 2021/02/06 16:51:06 [error] 4647#0: *3 open() "/usr/share/nginx/html/sdvsvsdv" failed (2: No such file or directory), client: ::1, server: _, request: "GET /sdvsvsdv HTT>
Feb  6 16:52:40 web nginx_error: 2021/02/06 16:52:40 [error] 4901#0: *4 open() "/usr/share/nginx/html/sdvsvsdv" failed (2: No such file or directory), client: ::1, server: _, request: "GET /sdvsvsdv HTT>
[root@log 192.168.10.22]#



[root@log 192.168.10.22]# ausearch -ts today -i | grep nginx
node=web type=USER_CMD msg=audit(02/06/21 16:46:01.492:1570) : pid=4741 uid=vagrant auid=vagrant ses=4 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=cat /var/log/>
node=web type=USER_CMD msg=audit(02/06/21 16:46:20.492:1576) : pid=4744 uid=vagrant auid=vagrant ses=4 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=cat /var/log/>
node=web type=USER_CMD msg=audit(02/06/21 16:47:19.238:1582) : pid=4751 uid=vagrant auid=vagrant ses=4 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=cat /var/log/>
node=web type=PATH msg=audit(02/06/21 16:50:36.603:1593) : item=0 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=CWD msg=audit(02/06/21 16:50:36.603:1593) :  cwd=/etc/nginx
node=web type=PATH msg=audit(02/06/21 16:50:36.603:1594) : item=0 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=CWD msg=audit(02/06/21 16:50:36.603:1594) :  cwd=/etc/nginx
node=web type=PATH msg=audit(02/06/21 16:50:36.609:1595) : item=1 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=PATH msg=audit(02/06/21 16:50:36.609:1595) : item=0 name=/etc/nginx/ inode=33669437 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PA>
node=web type=CWD msg=audit(02/06/21 16:50:36.609:1595) :  cwd=/etc/nginx
node=web type=PATH msg=audit(02/06/21 16:53:57.439:1596) : item=0 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=CWD msg=audit(02/06/21 16:53:57.439:1596) :  cwd=/etc/nginx
node=web type=PATH msg=audit(02/06/21 16:53:57.439:1597) : item=0 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=CWD msg=audit(02/06/21 16:53:57.439:1597) :  cwd=/etc/nginx
node=web type=PATH msg=audit(02/06/21 16:53:57.439:1598) : item=1 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=PATH msg=audit(02/06/21 16:53:57.439:1598) : item=0 name=/etc/nginx/ inode=33669437 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PA>
node=web type=CWD msg=audit(02/06/21 16:53:57.439:1598) :  cwd=/etc/nginx
node=web type=PROCTITLE msg=audit(02/06/21 17:08:41.646:1645) : proctitle=nano /etc/nginx/nginx.conf
node=web type=PATH msg=audit(02/06/21 17:08:41.646:1645) : item=1 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=PATH msg=audit(02/06/21 17:08:41.646:1645) : item=0 name=/etc/nginx/ inode=33669437 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PA>
node=web type=PROCTITLE msg=audit(02/06/21 17:08:50.313:1646) : proctitle=nano /etc/nginx/nginx.conf
node=web type=PATH msg=audit(02/06/21 17:08:50.313:1646) : item=1 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=PATH msg=audit(02/06/21 17:08:50.313:1646) : item=0 name=/etc/nginx/ inode=33669437 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PA>
node=web type=PROCTITLE msg=audit(02/06/21 17:08:53.977:1647) : proctitle=nano /etc/nginx/nginx.conf
node=web type=PATH msg=audit(02/06/21 17:08:53.977:1647) : item=1 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=PATH msg=audit(02/06/21 17:08:53.977:1647) : item=0 name=/etc/nginx/ inode=33669437 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PA>
node=web type=PROCTITLE msg=audit(02/06/21 17:09:03.673:1648) : proctitle=nano /etc/nginx/nginx.conf
node=web type=PATH msg=audit(02/06/21 17:09:03.673:1648) : item=1 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=PATH msg=audit(02/06/21 17:09:03.673:1648) : item=0 name=/etc/nginx/ inode=33669437 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PA>
node=web type=PROCTITLE msg=audit(02/06/21 17:09:08.522:1649) : proctitle=nano /etc/nginx/nginx.conf
node=web type=PATH msg=audit(02/06/21 17:09:08.522:1649) : item=1 name=/etc/nginx/nginx.conf inode=67567914 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0>
node=web type=PATH msg=audit(02/06/21 17:09:08.522:1649) : item=0 name=/etc/nginx/ inode=33669437 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PA>
[root@log 192.168.10.22]#
```

