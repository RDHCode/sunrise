<h1 align="center">Installing Arch Linux with full-disk encryption</h1>
<p align="center">
  <img src="https://img.shields.io/badge/MAINTAINED-YES-green?style=for-the-badge">
  <img src="https://img.shields.io/badge/LICENSE-Apache_2.0-blue?style=for-the-badge">
  <img src="https://img.shields.io/github/issues/RDHCode/archlinux?style=for-the-badge">
</p>

<h5 align="center">"This is a guid on how to install arch linux with full disk encryption"</h1>

## Installation Steps
1. Choose a keyboard layout :
    ```loadkeys
    loadkeys ( keyboard layout )
    ```
2. Connect to wifi ( Only if you use wifi primarily on your system ) :
    ```iwctl
    iwctl
    station ( network interface ) connect ( SSID ) password ( password )
    exit
    ```
3. Partition disks ( Use lsblk if you don't know what your disk is called ) :
    ```cfdisk
    cfdisk /dev/( Disk name )
      1. Select gpt then enter
      2. Delete existing partitions
      3. Create a partition with 256M of storage and change the type to "EFI System"
      4. Create a second partition with 512M of storage
      5. Create a third partition using the amount of storage given to you by default ( Which will be the rest of your storage )
      6. Press write, type yes then enter and press quit
    ```
4. Format partitions
    ```formating
    mkfs.vfat -n "EFI System" /dev/( Disk name ) ( 1 for hhd's and ssd's. p1 for nvmes )
    mkfs.ext4 -L boot /dev/( Disk name ) 2
    mkfs.ext4 -L root /dev/ ( Disk name ) 3
    ```
5. Encryption support
    ```modprobe
    modprobe dm-crypt
    modprobe dm-mod
    ```
6. Encryption setup
    ```encryption-setup
    cryptsetup luksFormat -v -s 512 -h sha512 /dev/( Disk name ) 3
    cryptsetup open /dev/( Disk name ) 3 archlinux
    ```
7. Format encrypted partitions
    ```formatting-encrypted-partitions
    mkfs.btrfs -L root /dev/mapper/archlinux
8. Mounting & setting up the btrfs filesystem
    ```setup-btrfs
    1. Mounting
        mount -t btrfs /dev/mapper/archlinux /mnt
    2. Creating subvolumes
        cd /mnt
        btrfs subvolume create root /mnt
        btrfs subvolume create root
        btrfs subvolume create home
        btrfs subvolume create swap
        (Optional: For timeshift) btrfs create snapshots
     3. Mounting subvolumes
        cd /
        umount -R /mnt
        mkdir /mnt/home
        mkdir /mnt/swap
        (Optional: For timeshift) mkdir /mnt/snapshots
        mount -t btrfs -o subvol=root /dev/mapper/archlinux /mnt
        mount -t btrfs -o subvol=home /dev/mapper/archlinux /mnt/home
        mount -t btrfs -o subvol=swap /dev/mapper/archlinux /mnt/swap
        (Optional: For timeshift) mount -t btrfs -o subvol=snapshots /dev/mapper/archlinux /mnt/snapshots
    ```
9. Setting up GRUB
    ``` grub
    mkdir /mnt/boot
    mkdir /mnt/boot/efi
    mount /dev/( Disk Name ) /mnt/boot
    mount /dev/( Disk Name ) /mnt/boot/efi ( If the file/folder dose not exist: mkdir /mnt/boot/efi and then mount it )
    ```
10. Setting up Swap
    ``` swapfile
    cd /mnt
    touch /mnt/swap/swapfile
    chattr +C /mnt/swap/swapfile
    fallocate /mnt/swap/swapfile -l ( Swap size: Your ram size or otherwise. Examples: 1024M, 64M, 8G, )
    mkswap /mnt/swap/swapfile
    swapon /mnt/swap/swapfile
    chmod 0600 /mnt/swap/swapfile
    ```
11. Install System
    ```install-system
    pacstrap -i /mnt base base-devel linux linux-firmware grub efibootmgr networkmanager sudo nano
    ```
12. Genfstab
    ```genfstab
    genfstab -U /mnt > /mnt/etc/fstab
    ```
13. Chroot
    ```arch-chroot
    arch-chroot /mnt
    ```
14. Root password
    ```password
    passwd
    ```
15. Locale
    ```locale
    nano /etc/locale.gen ( Scroll until you get to your locale then uncomment it. Ctrl+o to save Ctrl+x to exit. )
    locale-gen
    echo LANG=( LANG ).UTF-8 ( Example: en_US.UTF-8 ) > /etc/locale.conf
    ```
16. Keyboard layout
    ```keymap
    echo KEYMAP=( Keyboard layout ) ( Example: us ) > /etc/vconsole.conf
    ```
17. Timezone
    ```timezone
    ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
    hwclock --systohc --utc
    ```
18. Hostname
    ```hostname
    echo ( Host name of choice ) > /etc/hostname
    nano /etc/hosts ( 'Your IP' localhost.localdomain 'Your hostname of choice'. Ctrl+o to save Ctrl+x to exit. ) 
    ```
19. Grub encryption configuration
    ```grub-encrypt-config
    nano /etc/default/grub ( Scroll dowm to GRUB_CMDLINE_LINUX and inside the quotes type: cryptdevice=/dev/( Disk Name ):archlinux . Ctrl+o to save Ctrl+x to exit. )
    ```
20. Mkinitcpio configuration
    ```mkinitcpio-config
    nano /etc/mkinitcpio.conf ( Scroll down to HOOKS and after block type encrypt. Ctrl+o to save Ctrl+x to exit. )
    mkinitcpio -p linux
    ```
21. Install and configure grub
    ```grub-config
    grub-install --boot-directory=/boot --efi-directory=/boot/efi /dev/( Disk Name )
    grub-mkconfig -o /boot/grub/grub.cfg
    grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg
    ```
22. Exit & Reboot
    ```exit-and-reboot
    exit
    reboot
    ```
23. Useradd
    Once you login as root user with the password you set earlier type :
    ```useradd
    useradd -m -g users -G wheel -s /bin/bash ( Username of Choice )
    passwd ( Username of Choice )
    EDITOR=nano visudo ( Scroll down to %wheel All=(ALL) ALL and uncomment it. Ctrl+o to save Ctrl+x to exit. )
    ```
24. Exit
    ```exit
    Exit ( After exiting login with the user you created )
    ```
25. Activate Network Connection
    ```network
    sudo systemctl enable NetworkManager
    sudo systemctl start NetworkManager
    (Optional: For wifi) nmcli device connect ( SSID ) password ( password )
    ```
26. Install X window system and audio
    ```install-x-window-system-and-audio
    pacman -S pulseaudio pulseaudio-alsa asla-utils xorg xorg-xinit xorg-server
    ```
27. Download a Desktop Environment or a Window Manager

    __NOTE__: I would like to point out that I have not tested all these desktops. In particular, some Login Managers may not work with a given desktop. For the                            full list of Login Managers look at the [Arch Wiki](https://wiki.archlinux.org/index.php/Display_manager) and try the one you like.
    ```de-or-wm
    XFCE:
        pacman -S xfce4 lightdm lightdm-gtk-greeter
        echo "exec startxfce4" > ~/.xinitrc
        systemctl enable lightdm
        
    Gnome:
        echo "exec gnome-session" > ~/.xinitrc
        sudo pacman -S gnome
        
    Cinnamon:
        echo "exec cinnamon-session" > ~/.xinitrc
        sudo pacman -S cinnamon mdm
        systemctl enable mdm
        
    Mate:
       echo "exec mate-session" > ~/.xinitrc
       sudo pacman -S mate lightdm lightdm-gtk-greeter
       systemctl enable lightdm
       
    Budgie:
       echo "export XDG_CURRENT_DESKTOP=Budgie:GNOME" > ~/.xinitrc
       echo "exec budgie-desktop" >> ~/.xinitrc
       sudo pacman -S budgie-desktop lightdm lightdm-gtk-greeter
       systemctl enable lightdm
       
    Openbox:
       echo "exec openbox-session" > ~/.xinitrc
       sudo pacman -S openbox lightdm lightdm-gtk-greeter
       systemctl enable lightdm
       
    i3:
       echo "exec i3"  > ~/.xinitrc
       pacman -S i3 rxvt-unicode dmenu lightdm
       systemctl enable lightdm
       
    Awesome:
       echo "exec awesome"  > ~/.xinitrc
       sudo pacman -S awesome lightdm
       systemctl enable lightdm
       
    Deepin:
       echo "exec startdde"  > ~/.xinitrc
       sudo pacman -S deepin lightdm
       systemctl enable lightdm

       Also, edit the file /etc/lightdm/lightdm.conf to have this line:

       greeter-session=lightdm-deepin-greeter
       
    LXDE:
       echo "exec startlxde"  > ~/.xinitrc
       sudo pacman -S lxdm-gtk3 lxdm
    ```
28. Reboot & Enjoy
     ```   
     reboot
    ```
___

<h4 align="center">I hope you enjoyed the guid I made though if you have had any isssues with my guid please submit it in them 
issues section.</h4>



