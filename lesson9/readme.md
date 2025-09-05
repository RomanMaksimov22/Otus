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

root@otus-syastemd:~# tail -n 1000 /var/log/syslog  | grep word
Sep  5 14:32:56 otus-syastemd kernel: [    3.628545] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
Sep  5 14:49:15 otus-syastemd root: Fri Sep  5 14:49:15 UTC 2025: I found word, Master!

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

[Install]
WantedBy=multi-user.target
