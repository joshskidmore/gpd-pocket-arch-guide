## Partition Layout

    DEVICE              FS        REC SIZE      NOTES/MNT POINT
    /dev/mmcblk0p1      vfat      100M          /boot/efi (default; do not modify)
    /dev/mmcblk0p2      vfat      16M           (default; windows buffer)
    /dev/mmcblk0p3      ntfs                    (default windows partition)
    /dev/mmcblk0p4      ext4      1G            /boot
    /dev/mmcblk0p5      swap      4G            /swap
    /dev/mmcblk0p6      ext4                    /


## Installation
Use dd (or similar) to write the latest [Arch ISO](https://www.archlinux.org/download/) to a USB stick. You will need a USB A hub with both the Arch USB stick as well as a generic USB ethernet adapter considering that you'll need internet to retrieve Hans De Goede's kernel in order for wireless to work.

    cgdisk /dev/mmcblk0

    # setup partitions
    # 4   1GiB    boot parition      # hex code 8300
    # 5   4GiB    swap parition      # hex code 8200
    # 6   100%    root parition      # hex code 8300

    # format partitions
    mkfs.ext4 /dev/mmcblk0p4
    mkfs.ext4 /dev/mmcblk0p6

    # setup swap
    mkswap /dev/mmcblk0p5
    swapon /dev/mmcblk0p5

    # mount partitions
    mount /dev/mmcblk0p6 /mnt
    mkdir /mnt/boot
    mount /dev/mmcblk0p4 /mnt/boot
    mkdir /mnt/boot/efi
    mount /dev/mmcblk0p1 /mnt/boot/efi

    # install/pacstrap
    pacstrap /mnt base base-devel grub-efi-x86_64 bash bash-completion vim git efibootmgr dialog wpa_supplicant

    # fstab
    genfstab -pU /mnt >> /mnt/etc/fstab

    # enter the new system
    arch-chroot /mnt /bin/bash

    # set hostname
    echo MYHOSTNAME > /etc/hostname


    # add repo with hans de goede's kernel (and other pocket-specific utils)
    vim /etc/pacman.conf

      # append the following to the end:
      [gpd-pocket]
      SigLevel = Optional TrustAll
      Server = https://github.com/njkli/$repo/raw/master/repo

    # pacman sync; install hans de goede's kernel
    pacman -Sy
    pacman -S linux-jwrdegoede linux-jwrdegoede-docs linux-jwrdegoede-headers

    # remove standard kernel package
    pacman -R linux

    # fix rotatation (for terminal only)
    vim /etc/default/grub

    # change GRUB_CMDLINE_LINUX_DEFAULT to read:
    GRUB_CMDLINE_LINUX_DEFAULT="fbcon=rotate:1 quiet"

    # set locale
    echo LANG=en_US.UTF-8 >> /etc/locale.conf
    echo LANGUAGE=en_US >> /etc/locale.conf
    echo LC_ALL=C >> /etc/locale.conf

    vim /etc/locale.gen
      # uncomment en_US.UTF-8 UTF-8
    locale-gen

    # set root password
    passwd

    # create user
    useradd -m -g users -G wheel,storage,power -s /bin/bash MYUSERNAME
    passwd MYUSERNAME

    # allow wheel group sudoers access
    visudo
      # uncomment: %wheel ALL=(ALL) NOPASSWD: ALL

    # regenerate initrd
    mkinitcpio -p linux-jwrdegoede

    # setup grub
    grub-install
    grub-mkconfig -o /boot/grub/grub.cfg

    # unmount partitions and restart (remove usb stick)
    exit      # exit arch-chroot
    umount -R /mnt
    reboot


## Setup
Upon reboot, you should be seeing a small, but properly rotated login. Login using MYUSERNAME.

### Enable dhcpcd
By default, Arch disables dhcpcd. Enabling will allow your USB ethernet adapter to receive an IP and continue working as it did before.

    sudo systemctl enable dhcpcd.service
    sudo systemctl start dhcpcd.service


### Install yaourt + Enable AUR/community Packages

    sudo vim /etc/pacman.conf

      # append/uncomment these lines:
      [multilib]
      Include = /etc/pacman.d/mirrorlists

      [archlinuxfr]
      SigLevel = Never
      Server = http://repo.archlinux.fr/$arch

    # sync
    sudo pacman -Sy

    # install yaourt
    sudo pacman -S yaourt


### Fix Tiny Ass Terminal Font

    # install terminus font
    yaourt -S terminus-font

    # create vconsole.conf file which uses larger terminus font
    echo "FONT=ter-v32n" > /etc/vconsole.conf

    # reboot and enjoy a readable terminal


### Time

    # timezones
    sudo rm /etc/localtime
    sudo ln -s /usr/share/zoneinfo/US/Eastern /etc/localtime
    sudo tzselect    # Choose Americas -> United States -> Eastern

    # install, enable and start ntpd (keeps time synchronized)
    yaourt -S ntp
    sudo systemctl enable ntpd.service
    sudo systemctl start ntpd.service


### Networking

    # write broadcom brcm4356 config file
    sudo curl https://raw.githubusercontent.com/joshskidmore/gpd-pocket-arch-guide/master/files/lib/firmware/brcm/brcmfmac4356-pcie.txt -o /lib/firmware/brcm/brcmfmac4356-pcie.txt

    # remove, re-modprobe brcmfmac module (now with config!)
    sudo rmmod brcmfmac; sudo modprobe brcmfmac

    # install networkmanager (note: will install gtk/ui packages)
    yaourt -S networkmanager network-manager-applet nm-connection-editor

    # enable and start networkmanager
    sudo systemctl enable NetworkManager
    sudo systemctl start NetworkManager

    # connect to a wireless network using nmtui
    sudo nmtui
      # Activate a Connection -> (choose wi-fi connection) -> Activate -> Back -> Quit

    # should be able to disconnect usb hub and/or usb ethernet because wireless is working!


### Xorg + XFCE4

    # install xorg + utils
    yaourt -S xorg-server xorg-xev xorg-xinit xorg-xkill xorg-xmodmap xorg-xprop xorg-xrandr xorg-xrdb xorg-xset

    # use pocket-specific xorg configs
    sudo curl https://raw.githubusercontent.com/joshskidmore/gpd-pocket-arch-guide/master/files/etc/X11/xorg.conf.d/20-intel.conf -o /etc/X11/xorg.conf.d/20-intel.conf
    sudo curl https://raw.githubusercontent.com/joshskidmore/gpd-pocket-arch-guide/master/files/etc/X11/xorg.conf.d/30-monitor.conf -o /etc/X11/xorg.conf.d/30-monitor.conf
    sudo curl https://raw.githubusercontent.com/joshskidmore/gpd-pocket-arch-guide/master/files/etc/X11/xorg.conf.d/99-touchscreen.conf -o /etc/X11/xorg.conf.d/99-touchscreen.conf

    # install intel video drivers
    yaourt -S xf86-video-intel

    # install xfce4
    yaourt -S xfce4 xfce4-goodies
      # select all the default package options

    # enable xfce4 start in ~/.xinitrc
    echo "exec startxfce4" >> ~/.xinitrc



## Sources Used in this Document
  * [arch linux's GPD Pocket device wiki](https://wiki.archlinux.org/index.php/GPD_Pocket)
  * [cawilliamson's excellent gpdpocket github repo](https://github.com/cawilliamson/ansible-gpdpocket)
  * [gilbertw1's original arch linux setup guide](https://gist.github.com/gilbertw1/81ef4b0bcf3ddefa4b18)
