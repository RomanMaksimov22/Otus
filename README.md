    # Otus
    1  uname -a
     Linux otuslinux 5.15.0-134-generic #145-Ubuntu SMP Wed Feb 12 20:08:39 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
    2  mkdir kernel
    3  cd kernel/
    4  wget https://kernel.ubuntu.com/mainline/v6.0.5/amd64/linux-headers-6.0.5-060005-generic_6.0.5-060005.202210261142_amd64.deb
    5  wget https://kernel.ubuntu.com/mainline/v6.0.5/amd64/linux-headers-6.0.5-060005_6.0.5-060005.202210261142_all.deb
    6  wget https://kernel.ubuntu.com/mainline/v6.0.5/amd64/linux-image-unsigned-6.0.5-060005-generic_6.0.5-060005.202210261142_amd64.deb
    7  wget https://kernel.ubuntu.com/mainline/v6.0.5/amd64/linux-modules-6.0.5-060005-generic_6.0.5-060005.202210261142_amd64.deb
    8  dpkg -i *.deb
    9  s -al /boot
    10  ls -al /boot
    11  update-grub
    12  grub-set-default 0
    13  reboot
    14  uname -a
    Linux otuslinux 6.0.5-060005-generic #202210261142 SMP PREEMPT_DYNAMIC Wed Oct 26 11:54:33 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
