root@otus-syastemd:~# vim /etc/default/watchlog
root@otus-syastemd:~# cat /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log

root@otus-syastemd:~# vim /opt/watchlog.sh
root@otus-syastemd:~# cat /opt/watchlog.sh
#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

root@otus-syastemd:~# chmod +x /opt/watchlog.sh

root@otus-syastemd:~# vim /opt/watchlog.sh
root@otus-syastemd:~# cat /opt/watchlog.sh
#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

root@otus-syastemd:~# chmod +x /opt/watchlog.sh
root@otus-syastemd:~# vim /etc/systemd/system/watchlog.service
root@otus-syastemd:~# cat /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

root@otus-syastemd:~# vim /etc/systemd/system/watchlog.timer
root@otus-syastemd:~# cat /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target

root@otus-syastemd:~# tail -n 1000 /var/log/syslog  | grep word
Sep  5 14:32:56 otus-syastemd kernel: [    3.628545] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
Sep  5 14:49:15 otus-syastemd root: Fri Sep  5 14:49:15 UTC 2025: I found word, Master!

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

root@otus-syastemd:~# apt install spawn-fcgi php php-cgi php-cli  apache2 libapache2-mod-fcgid -y
mkdir /etc/spawn-fcgi
vim /etc/spawn-fcgi/fcgi.conf
cat /etc/spawn-fcgi/fcgi.conf
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"


vim /etc/systemd/system/spawn-fcgi.service
cat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target

systemctl start spawn-fcgi.service
systemctl status spawn-fcgi.service
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2025-09-07 16:44:57 UTC; 1s ago
   Main PID: 1395 (php-cgi)
      Tasks: 33 (limit: 3425)
     Memory: 18.8M
        CPU: 56ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─1395 /usr/bin/php-cgi
             ├─1396 /usr/bin/php-cgi
             ├─1397 /usr/bin/php-cgi
             ├─1398 /usr/bin/php-cgi
             ├─1399 /usr/bin/php-cgi
             ├─1400 /usr/bin/php-cgi
             ├─1401 /usr/bin/php-cgi
             ├─1402 /usr/bin/php-cgi
             ├─1403 /usr/bin/php-cgi
             ├─1404 /usr/bin/php-cgi
             ├─1405 /usr/bin/php-cgi
             ├─1406 /usr/bin/php-cgi
             ├─1407 /usr/bin/php-cgi
             ├─1408 /usr/bin/php-cgi
             ├─1409 /usr/bin/php-cgi
             ├─1410 /usr/bin/php-cgi
             ├─1411 /usr/bin/php-cgi
             ├─1412 /usr/bin/php-cgi
             ├─1413 /usr/bin/php-cgi
             ├─1414 /usr/bin/php-cgi
             ├─1415 /usr/bin/php-cgi
             ├─1416 /usr/bin/php-cgi
             ├─1417 /usr/bin/php-cgi
             ├─1418 /usr/bin/php-cgi
             ├─1419 /usr/bin/php-cgi
             ├─1420 /usr/bin/php-cgi
             ├─1421 /usr/bin/php-cgi
             ├─1422 /usr/bin/php-cgi
             ├─1423 /usr/bin/php-cgi
             ├─1424 /usr/bin/php-cgi
             ├─1425 /usr/bin/php-cgi
             ├─1426 /usr/bin/php-cgi
             └─1427 /usr/bin/php-cgi
Sep 07 16:44:57 otus-syastemd systemd[1]: Started Spawn-fcgi startup service by Otus.

apt install nginx
vim /etc/systemd/system/nginx@.service
cat /etc/systemd/system/nginx@.service
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target


cp /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf
cp /etc/nginx/nginx.conf /etc/nginx/nginx-second.conf

cat /etc/nginx/nginx-first.conf
user www-data;
worker_processes auto;
include /etc/nginx/modules-enabled/*.conf;
pid /run/nginx-first.pid;  # Added missing semicolon

events {
    worker_connections 768;
    # multi_accept on;
}

http {
    ##
....
 server {
        listen localhost:9002;
    }
....
    include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/sites-enabled/*;
cat /etc/nginx/nginx-second.conf
#Тоже самое что и в первом, только порт 9002

systemctl start nginx@first
systemctl start nginx@second

root@otus-syastemd:~# systemctl status nginx@first
● nginx@first.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2025-09-07 16:28:55 UTC; 7min ago
       Docs: man:nginx(8)
    Process: 1134 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-first.conf -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1135 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 1136 (nginx)
      Tasks: 3 (limit: 3425)
     Memory: 3.3M
        CPU: 44ms
     CGroup: /system.slice/system-nginx.slice/nginx@first.service
             ├─1136 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on;"
             ├─1137 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">
             └─1138 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">

Sep 07 16:28:55 otus-syastemd systemd[1]: Starting A high performance web server and a reverse proxy server...
Sep 07 16:28:55 otus-syastemd systemd[1]: Started A high performance web server and a reverse proxy server.

root@otus-syastemd:~# systemctl status nginx@second
● nginx@second.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2025-09-07 16:27:48 UTC; 9min ago
       Docs: man:nginx(8)
    Process: 1079 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-second.conf -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1080 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 1081 (nginx)
      Tasks: 3 (limit: 3425)
     Memory: 3.3M
        CPU: 48ms
     CGroup: /system.slice/system-nginx.slice/nginx@second.service
             ├─1081 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;"
             ├─1082 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">
             └─1083 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">

Sep 07 16:27:48 otus-syastemd systemd[1]: Starting A high performance web server and a reverse proxy server...
Sep 07 16:27:48 otus-syastemd systemd[1]: Started A high performance web server and a reverse proxy server.


root@otus-syastemd:~# ss -tnulp | grep nginx
tcp   LISTEN 0      511                127.0.0.1:9002      0.0.0.0:*    users:(("nginx",pid=1138,fd=6),("nginx",pid=1137,fd=6),("nginx",pid=1136,fd=6))      
tcp   LISTEN 0      511                  0.0.0.0:9001      0.0.0.0:*    users:(("nginx",pid=1083,fd=6),("nginx",pid=1082,fd=6),("nginx",pid=1081,fd=6))      
tcp   LISTEN 0      511                127.0.0.1:8080      0.0.0.0:*    users:(("nginx",pid=730,fd=6),("nginx",pid=729,fd=6),("nginx",pid=712,fd=6))  



