root@otuslinux:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 89.4M  1 loop /snap/lxd/31333
loop1                       7:1    0 44.4M  1 loop /snap/snapd/23771
loop2                       7:2    0 63.7M  1 loop /snap/core20/2496
loop3                       7:3    0 38.8M  1 loop /snap/snapd/21759
loop4                       7:4    0   87M  1 loop /snap/lxd/29351
loop5                       7:5    0 63.9M  1 loop /snap/core20/2318
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 11.5G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk

root@otuslinux:~# mdadm --create --verbose /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

root@otuslinux:~# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid5 sdf[5] sde[3] sdd[2] sdc[1] sdb[0]
      4186112 blocks super 1.2 level 5, 512k chunk, algorithm 2 [5/5] [UUUUU]

unused devices: <none>

root@otuslinux:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Mar 20 19:51:50 2025
        Raid Level : raid5
        Array Size : 4186112 (3.99 GiB 4.29 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Thu Mar 20 19:52:07 2025
             State : clean
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 180cd258:44cc595f:52654c55:3d7c1855
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       5       8       80        4      active sync   /dev/sdf

root@otuslinux:~# mkfs.ext4 -m 0 /dev/md0
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1046528 4k blocks and 261632 inodes
Filesystem UUID: c62fdbae-11f5-4475-ab57-77fe149d7618
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@otuslinux:~# blkid /dev/md0
/dev/md0: UUID="c62fdbae-11f5-4475-ab57-77fe149d7618" BLOCK_SIZE="4096" TYPE="ext4"

root@otuslinux:~# cat /etc/fstab
/dev/disk/by-id/dm-uuid-LVM-tqcMuApcx7NVVkwCrRO5vOx3wc0tIMZr178ubc17mUFJV45URoYz0voyhBuIHpKC / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/212ad117-af78-4aae-8677-aff61f4f02cb /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
UUID="c62fdbae-11f5-4475-ab57-77fe149d7618" /r5 ext4 defaults 0 0

root@otuslinux:~# mount -a
root@otuslinux:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              198M  1.2M  197M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   12G  6.1G  4.6G  58% /
tmpfs                              988M     0  988M   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  257M  1.6G  15% /boot
tmpfs                              198M  4.0K  198M   1% /run/user/1000
/dev/md0                           3.9G   24K  3.9G   1% /r5

root@otuslinux:~# mdadm /dev/md0 --fail /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md0

root@otuslinux:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Mar 20 19:51:50 2025
        Raid Level : raid5
        Array Size : 4186112 (3.99 GiB 4.29 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Thu Mar 20 20:03:25 2025
             State : clean, degraded
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 180cd258:44cc595f:52654c55:3d7c1855
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       -       0        0        2      removed
       3       8       64        3      active sync   /dev/sde
       5       8       80        4      active sync   /dev/sdf

       2       8       48        -      faulty   /dev/sdd

root@otuslinux:~# mdadm /dev/md0 --remove /dev/sdd
mdadm: hot removed /dev/sdd from /dev/md0

root@otuslinux:~# mdadm /dev/md0 --add /dev/sdd
mdadm: added /dev/sdd


root@otuslinux:~# gdisk -l /dev/md0
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.
Disk /dev/md0: 8372224 sectors, 4.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): D1F879A1-3C34-44A1-AE5C-760068AA03E6
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 8372190
Partitions will be aligned on 2048-sector boundaries
Total free space is 8372157 sectors (4.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
root@otuslinux:~# gdisk /dev/md0
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): ?
b       back up GPT data to a file
c       change a partition's name
d       delete a partition
i       show detailed information on a partition
l       list known partition types
n       add a new partition
o       create a new empty GUID partition table (GPT)
p       print the partition table
q       quit without saving changes
r       recovery and transformation options (experts only)
s       sort partitions
t       change a partition's type code
v       verify disk
w       write table to disk and exit
x       extra functionality (experts only)
?       print this menu

Command (? for help): p
Disk /dev/md0: 8372224 sectors, 4.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 73E2521E-1DAE-4601-B8A8-974BDF32BC0D
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 8372190
Partitions will be aligned on 2048-sector boundaries
Total free space is 8372157 sectors (4.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-8372190, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-8372190, default = 8372190) or {+-}size{KMGTP}:
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/md0: 8372224 sectors, 4.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 73E2521E-1DAE-4601-B8A8-974BDF32BC0D
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 8372190
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         8372190   4.0 GiB     8300  Linux filesystem

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/md0.
The operation has completed successfully.


root@otuslinux:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
loop1                       7:1    0 44.4M  1 loop  /snap/snapd/23771
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part  /boot
└─sda3                      8:3    0   23G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 11.5G  0 lvm   /
sdb                         8:16   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part
sdc                         8:32   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part
sdd                         8:48   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part
sde                         8:64   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part
sdf                         8:80   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part
sr0                        11:0    1 1024M  0 rom

root@otuslinux:~# fdisk /dev/md0 -l
Disk /dev/md0: 3.99 GiB, 4286578688 bytes, 8372224 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 524288 bytes / 2097152 bytes
Disklabel type: gpt
Disk identifier: 5A994985-2FA0-4ACC-B3EF-D47000CBFCB9

Device     Start     End Sectors Size Type
/dev/md0p1  2048 8372190 8370143   4G Linux filesystem




root@otuslinux:~# blkid
/dev/mapper/ubuntu--vg-ubuntu--lv: UUID="6eba7b14-344c-4128-a3c2-7a4506248026" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sda2: UUID="212ad117-af78-4aae-8677-aff61f4f02cb" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="26b84994-e2b2-406b-82db-cc5fa2481d2c"
/dev/sda3: UUID="HHBs59-BqcF-r2Mq-areV-C1ri-1FXU-Nsnr66" TYPE="LVM2_member" PARTUUID="740075a9-8c9c-4833-91ff-30ac2495fabd"
/dev/loop1: TYPE="squashfs"
/dev/sdf: UUID="180cd258-44cc-595f-5265-4c553d7c1855" UUID_SUB="f047f27e-353c-559e-96ef-dffa4b0bde8b" LABEL="otuslinux:0" TYPE="linux_raid_member"
/dev/sdd: UUID="180cd258-44cc-595f-5265-4c553d7c1855" UUID_SUB="84e41754-2558-e20c-b80e-04653ea95643" LABEL="otuslinux:0" TYPE="linux_raid_member"
/dev/sdb: UUID="180cd258-44cc-595f-5265-4c553d7c1855" UUID_SUB="3c0f6e5b-62ed-24c3-18df-592601a0a032" LABEL="otuslinux:0" TYPE="linux_raid_member"
/dev/sde: UUID="180cd258-44cc-595f-5265-4c553d7c1855" UUID_SUB="7aa77e59-d7da-75f1-9c01-5f5545bf0694" LABEL="otuslinux:0" TYPE="linux_raid_member"
/dev/sdc: UUID="180cd258-44cc-595f-5265-4c553d7c1855" UUID_SUB="6f6ec21e-0198-6f50-d0d3-c8630ec7aaa9" LABEL="otuslinux:0" TYPE="linux_raid_member"
/dev/md0p1: UUID="dbce3f8b-7ca9-4284-8fb6-46dedf2a05b4" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="b051d37e-c92a-47e0-b2cd-93e2d0475744"
/dev/sda1: PARTUUID="87348602-b1b0-42b7-aaed-4f30b2afa43e"
root@otuslinux:~# vim /etc/fstab
root@otuslinux:~# mount -a
root@otuslinux:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
loop1                       7:1    0 44.4M  1 loop  /snap/snapd/23771
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part  /boot
└─sda3                      8:3    0   23G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 11.5G  0 lvm   /
sdb                         8:16   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part  /r5
sdc                         8:32   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part  /r5
sdd                         8:48   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part  /r5
sde                         8:64   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part  /r5
sdf                         8:80   0    1G  0 disk
└─md0                       9:0    0    4G  0 raid5
  └─md0p1                 259:0    0    4G  0 part  /r5
sr0                        11:0    1 1024M  0 rom
root@otuslinux:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              198M  1.2M  197M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   12G  6.1G  4.6G  58% /
/dev/sda2                          2.0G  257M  1.6G  15% /boot
tmpfs                              198M  4.0K  198M   1% /run/user/1000
/dev/md0p1                         3.9G   24K  3.7G   1% /r5