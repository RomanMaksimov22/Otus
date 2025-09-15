#!/bin/bash
LOCK_FILE="/root/nginx_monitor.lock"
FROM_EMAIL=myemail@dom.ru
To_EMAIL=myemail@dom.ru

if [ -e "$LOCK_FILE" ]; then
    exit 1
fi

#1. 
echo "first" | tee -a report.txt
awk '{print $1}' /root/access.log | sort | uniq -c | sort -n | tail -n 10 | tee -a report.txt
#2.
echo "second" | tee -a report.txt
awk '{print $7}' /root/access.log | sort | uniq -c | sort -n | tail -n 10 | tee -a report.txt
#3.
echo "third" | tee -a report.txt
awk '{print $9,$1,$7}' /root/access.log | uniq -c | sort -n | grep -v "200" | tee -a report.txt
#4.
echo "fourth" | tee -a report.txt
awk '{print $9}' /root/access.log | sort | uniq -c | sort -n  | tee -a report.txt
cat Результат выполнения | mailx -r "$FROM_EMAIL" -a report.txt -s "SUBJECT" "$To_EMAIL" 

rm -f "$LOCK_FILE"
