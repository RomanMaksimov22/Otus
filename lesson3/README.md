Создал PV и VG (vg_root) и LV lv_root. 
Примонтировал в /mnt
root@otuslinux2:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 38.8M  1 loop /snap/snapd/21759
loop1                       7:1    0   87M  1 loop /snap/lxd/29351
loop2                       7:2    0 89.4M  1 loop /snap/lxd/31333
loop3                       7:3    0 63.7M  1 loop /snap/core20/2496
loop5                       7:5    0 44.4M  1 loop /snap/snapd/23771
loop6                       7:6    0 63.8M  1 loop /snap/core20/2501
sda                         8:0    0   15G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 13.2G  0 part
  └─ubuntu--vg-ubuntu--lv 253:2    0   10G  0 lvm  /
sdb                         8:16   0   10G  0 disk
└─vg_root-lv_root         253:0    0   10G  0 lvm  /mnt
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom
sr1                        11:1    1 1024M  0 rom

Скопировал содержимое / в /mnt (rsync -avxHAX --progress / /mnt/)
Меняем grub
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u

root@otuslinux2:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 63.8M  1 loop /snap/core20/2501
loop1                       7:1    0   87M  1 loop /snap/lxd/29351
loop2                       7:2    0 63.7M  1 loop /snap/core20/2496
loop3                       7:3    0 89.4M  1 loop /snap/lxd/31333
loop4                       7:4    0 44.4M  1 loop /snap/snapd/23771
loop5                       7:5    0 38.8M  1 loop /snap/snapd/21759
sda                         8:0    0   15G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 13.2G  0 part
  └─ubuntu--vg-ubuntu--lv 253:1    0   10G  0 lvm
sdb                         8:16   0   10G  0 disk
└─vg_root-lv_root         253:0    0   10G  0 lvm  /
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom
sr1                        11:1    1 1024M  0 ro

root@otuslinux2:~# lvremove /dev/ubuntu-vg/ubuntu-lv
Do you really want to remove and DISCARD active logical volume ubuntu-vg/ubuntu-lv? [y/n]: y
  Logical volume "ubuntu-lv" successfully removed

root@otuslinux2:~# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0               7:0    0 63.8M  1 loop /snap/core20/2501
loop1               7:1    0   87M  1 loop /snap/lxd/29351
loop2               7:2    0 63.7M  1 loop /snap/core20/2496
loop3               7:3    0 89.4M  1 loop /snap/lxd/31333
loop4               7:4    0 44.4M  1 loop /snap/snapd/23771
loop5               7:5    0 38.8M  1 loop /snap/snapd/21759
sda                 8:0    0   15G  0 disk
├─sda1              8:1    0    1M  0 part
├─sda2              8:2    0  1.8G  0 part /boot
└─sda3              8:3    0 13.2G  0 part
sdb                 8:16   0   10G  0 disk
└─vg_root-lv_root 253:0    0   10G  0 lvm  /
sdc                 8:32   0    2G  0 disk
sdd                 8:48   0    1G  0 disk
sde                 8:64   0    1G  0 disk
sr0                11:0    1 1024M  0 rom
sr1                11:1    1 1024M  0 rom

root@otuslinux2:~# lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg
WARNING: ext4 signature detected on /dev/ubuntu-vg/ubuntu-lv at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/ubuntu-vg/ubuntu-lv.
  Logical volume "ubuntu-lv" created.

root@otuslinux2:~# lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg
  Logical Volume "ubuntu-lv" already exists in volume group "ubuntu-vg"

root@otuslinux2:~# mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv
  mke2fs 1.46.5 (30-Dec-2021)
  Creating filesystem with 2097152 4k blocks and 524288 inodes
  Filesystem UUID: b89a9c7a-0ee4-4401-8f5f-1975ef432368
  Superblock backups stored on blocks:
          32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632
  
  Allocating group tables: done
  Writing inode tables: done
  Creating journal (16384 blocks): done
  Writing superblocks and filesystem accounting information: done

root@otuslinux2:~#  mount /dev/ubuntu-vg/ubuntu-lv /mnt
root@otuslinux2:~# rsync -avxHAX --progress / /mnt/

root@otuslinux2:~# for i in /proc/ /sys/ /dev/ /run/ /boot/; \
 do mount --bind $i /mnt/$i; done
root@otuslinux2:~# chroot /mnt/
root@otuslinux2:/# grub-mkconfig -o /boot/grub/grub.cfg
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-134-generic
Found initrd image: /boot/initrd.img-5.15.0-134-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done

root@otuslinux2:/# update-initramfs -u


#### Выделяем том под /var

root@otuslinux2:/# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.

  root@otuslinux2:/# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created
root@otuslinux2:/# lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.
root@otuslinux2:/# mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 243712 4k blocks and 60928 inodes
Filesystem UUID: c382faa2-fff0-48ff-a708-45cf6fb4d3c6
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@otuslinux2:/# mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.46.5 (30-Dec-2021)
/dev/vg_var/lv_var contains a ext4 file system
        created on Mon Apr 14 13:09:13 2025
Proceed anyway? (y,N) y
Creating filesystem with 243712 4k blocks and 60928 inodes
Filesystem UUID: d4850b62-a183-46f0-abff-6f1b8ed72688
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@otuslinux2:/# cp -aR /var/* /mnt/
root@otuslinux2:/# mount /dev/vg_var/lv_var /var
root@otuslinux2:/# echo "`blkid | grep var: | awk '{print $2}'` \
 /var ext4 defaults 0 0" >> /etc/fstab
root@otuslinux2:~# lvremove /dev/vg_root/lv_root
Do you really want to remove and DISCARD active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed
root@otuslinux2:~# vgremove /dev/vg_root
  Volume group "vg_root" successfully removed
root@otuslinux2:~# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.
root@otuslinux2:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   15G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 13.2G  0 part
  └─ubuntu--vg-ubuntu--lv 253:1    0    8G  0 lvm  /
sdb                         8:16   0   10G  0 disk
└─vg_root-lv_root         253:0    0   10G  0 lvm
sdc                         8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0   253:2    0    4M  0 lvm
│ └─vg_var-lv_var         253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0  253:3    0  952M  0 lvm
  └─vg_var-lv_var         253:6    0  952M  0 lvm  /var
sdd                         8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1   253:4    0    4M  0 lvm
│ └─vg_var-lv_var         253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1  253:5    0  952M  0 lvm
  └─vg_var-lv_var         253:6    0  952M  0 lvm  /var
sde                         8:64   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom
sr1                        11:1    1 1024M  0 rom

#### Выделяем том под /home
root@otuslinux2:~# lvcreate -n LogVol_Home -L 2G /dev/ubuntu-vg
  Logical volume "LogVol_Home" created.
root@otuslinux2:~# mkfs.ext4 /dev/ubuntu-vg/LogVol_Home
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: ab09a2ad-8317-4802-ab82-d023bd60a847
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@otuslinux2:~# mount /dev/ubuntu-vg/LogVol_Home /mnt/
root@otuslinux2:~# cp -aR /home/* /mnt/
root@otuslinux2:~# rm -rf /home/*
root@otuslinux2:~# umount /mnt
root@otuslinux2:~# mount /dev/ubuntu-vg/LogVol_Home /home/
root@otuslinux2:~# echo "`blkid | grep Home | awk '{print $2}'` \
 /home xfs defaults 0 0" >> /etc/fstab

 root@otuslinux2:~# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                          8:0    0   15G  0 disk
├─sda1                       8:1    0    1M  0 part
├─sda2                       8:2    0  1.8G  0 part /boot
└─sda3                       8:3    0 13.2G  0 part
  ├─ubuntu--vg-LogVol_Home 253:0    0    2G  0 lvm  /home
  └─ubuntu--vg-ubuntu--lv  253:1    0    8G  0 lvm  /
sdb                          8:16   0   10G  0 disk
sdc                          8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0    253:2    0    4M  0 lvm
│ └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0   253:3    0  952M  0 lvm
  └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
sdd                          8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1    253:4    0    4M  0 lvm
│ └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1   253:5    0  952M  0 lvm
  └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
sde                          8:64   0    1G  0 disk
sr0                         11:0    1 1024M  0 rom
sr1                         11:1    1 1024M  0 rom

 ### Snapshot

root@otuslinux2:~# touch /home/file{1..20}
root@otuslinux2:~# ls /home
file1   file11  file13  file15  file17  file19  file20  file4  file6  file8  mrs
file10  file12  file14  file16  file18  file2   file3   file5  file7  file9

root@otuslinux2:~# lvcreate -L 100MB -s -n home_snap \
 /dev/ubuntu-vg/LogVol_Home
   Logical volume "home_snap" created.

root@otuslinux2:~# ls /home
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9  lost+found  mrs

root@otuslinux2:~# umount /home
root@otuslinux2:~# lvconvert --merge /dev/ubuntu-vg/home_snap
  Merging of volume ubuntu-vg/home_snap started.
  ubuntu-vg/LogVol_Home: Merged: 100.00%
root@otuslinux2:~# mount /dev/mapper/ubuntu--vg-LogVol_Home /home

root@otuslinux2:~# ls -al /home
total 28
drwxr-xr-x  4 root root  4096 Apr 14 14:23 .
drwxr-xr-x 20 root root  4096 Mar  5 19:17 ..
-rw-r--r--  1 root root     0 Apr 14 14:23 file1
-rw-r--r--  1 root root     0 Apr 14 14:23 file10
-rw-r--r--  1 root root     0 Apr 14 14:23 file11
-rw-r--r--  1 root root     0 Apr 14 14:23 file12
-rw-r--r--  1 root root     0 Apr 14 14:23 file13
-rw-r--r--  1 root root     0 Apr 14 14:23 file14
-rw-r--r--  1 root root     0 Apr 14 14:23 file15
-rw-r--r--  1 root root     0 Apr 14 14:23 file16
-rw-r--r--  1 root root     0 Apr 14 14:23 file17
-rw-r--r--  1 root root     0 Apr 14 14:23 file18
-rw-r--r--  1 root root     0 Apr 14 14:23 file19
-rw-r--r--  1 root root     0 Apr 14 14:23 file2
-rw-r--r--  1 root root     0 Apr 14 14:23 file20
-rw-r--r--  1 root root     0 Apr 14 14:23 file3
-rw-r--r--  1 root root     0 Apr 14 14:23 file4
-rw-r--r--  1 root root     0 Apr 14 14:23 file5
-rw-r--r--  1 root root     0 Apr 14 14:23 file6
-rw-r--r--  1 root root     0 Apr 14 14:23 file7
-rw-r--r--  1 root root     0 Apr 14 14:23 file8
-rw-r--r--  1 root root     0 Apr 14 14:23 file9
drwx------  2 root root 16384 Apr 14 14:17 lost+found
drwxr-x---  4 mrs  mrs   4096 Apr 14 13:56 mrs