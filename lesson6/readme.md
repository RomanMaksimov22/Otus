root@otus-nfss:~# apt install nfs-kernel-server

root@otus-nfss:~# ss -tnplu | grep 2049
tcp   LISTEN 0      64                   0.0.0.0:2049       0.0.0.0:*
tcp   LISTEN 0      64                      [::]:2049          [::]:*
root@otus-nfss:~# ss -tnplu | grep 111
udp   UNCONN 0      0                    0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=1450,fd=5),("systemd",pid=1,fd=74))
udp   UNCONN 0      0                       [::]:111           [::]:*    users:(("rpcbind",pid=1450,fd=7),("systemd",pid=1,fd=84))
tcp   LISTEN 0      4096                 0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=1450,fd=4),("systemd",pid=1,fd=73))
tcp   LISTEN 0      4096                    [::]:111           [::]:*    users:(("rpcbind",pid=1450,fd=6),("systemd",pid=1,fd=83))

root@otus-nfss:~# mkdir -p /srv/share/upload
root@otus-nfss:~# chown -R nobody:nogroup /srv/share/
root@otus-nfss:~# chmod 0777 /srv/share/upload

root@otus-nfss:~# cat << EOF > /etc/exports
/srv/share 192.168.56.103/32(rw,sync,root_squash)
EOF

root@otus-nfss:~# exportfs -r
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "192.168.56.103/32:/srv/share".
  Assuming default behaviour ('no_subtree_check').
  NOTE: this default has changed since nfs-utils version 1.0.x

root@otus-nfss:~# exportfs -s
/srv/share  192.168.56.103/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
root@otus-nfss:~#
 
++++++++++++++++++++++++++++++++++

root@otus-nfsc:~# apt install nfs-common

root@otus-nfsc:~# echo "192.168.56.102:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab

root@otus-nfsc:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-ItQ8GC3MQ3ZVkT00o7ZobsnS16XT8Q2k2Dk1xMeTTZ1RhfcHMzhhHca2aU513PEl / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/14272a21-331a-4a0a-a870-63a9e247ca22 /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
192.168.56.102:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0

root@otus-nfsc:~# systemctl daemon-reload
root@otus-nfsc:~# systemctl restart remote-fs.target

root@otus-nfsc:~# mount | grep mnt
nsfs on /run/snapd/ns/lxd.mnt type nsfs (rw)
systemd-1 on /mnt type autofs (rw,relatime,fd=49,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=31195)

root@otus-nfsc:~# cd /mnt
root@otus-nfsc:/mnt# mount | grep mnt
nsfs on /run/snapd/ns/lxd.mnt type nsfs (rw)
systemd-1 on /mnt type autofs (rw,relatime,fd=49,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=31195)
192.168.56.102:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.56.102,mountvers=3,mountport=34154,mountproto=udp,local_lock=none,addr=192.168.56.102)

+++++++

root@otus-nfsc:/mnt# cd /mnt/upload/
root@otus-nfsc:/mnt/upload# touch 1.file
root@otus-nfsc:/mnt/upload# ls
1.file

root@otus-nfss:~# ls /srv/share/upload/
1.file
