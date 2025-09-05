root@otus-nfss:~# cat /etc/default/grub
# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=15
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX=""

root@otus-nfss:~# update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-153-generic
Found initrd image: /boot/initrd.img-5.15.0-153-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done

Меню загрузчика:

<img width="663" height="506" alt="image" src="https://github.com/user-attachments/assets/d6c04bbf-6ab1-4d88-92a5-498d67da5a40" />

Добавление init=/bin/bash

<img width="999" height="739" alt="image" src="https://github.com/user-attachments/assets/a499d27e-48c7-4702-9eef-51010acddf57" />

Загрузка в режиме rw

<img width="1007" height="748" alt="image" src="https://github.com/user-attachments/assets/60413483-83c6-409c-bac7-17a7f5b17669" />

проверка изменения файла

<img width="999" height="772" alt="image" src="https://github.com/user-attachments/assets/f3da0669-e975-463a-8a73-4cecbc3f1fad" />

Загрузка в режиме recovery

<img width="1111" height="857" alt="image" src="https://github.com/user-attachments/assets/647a20a8-a8e6-4faa-9724-a2ee2fc95c89" />

Смена пароля root

<img width="1217" height="1075" alt="image" src="https://github.com/user-attachments/assets/1558d59f-0517-4321-8a01-8438a6c1185b" />

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
LVM
root@otus-nfss:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <23.00g 11.50g

root@otus-nfss:~# vgrename ubuntu-vg ubuntu-otus
  Volume group "ubuntu-vg" successfully renamed to "ubuntu-otus"
root@otus-nfss:~# vim /boot/grub/grub.cfg
root@otus-nfss:~# vgs
  VG          #PV #LV #SN Attr   VSize   VFree
  ubuntu-otus   1   1   0 wz--n- <23.00g 11.50g
