<h1 align="center">Installing Arch Linux with full-disk encryption</h1>
<p align="center">
  <img src="https://img.shields.io/badge/MAINTAINED-YES-green?style=for-the-badge">
  <img src="https://img.shields.io/badge/LICENSE-MIT-blue?style=for-the-badge">
  <img src="https://img.shields.io/github/issues/RDHCode/sunrise?style=for-the-badge">
</p>

<h5 align="center">"This is a guid on how to install arch linux with full disk encryption"</h1>

## Installation Steps
1. Choose a keyboard layout :
    ```loadkeys
    loadkeys ( keyboard layout )
    ```
2. Connect to wifi ( Only if you use wifi on your system ) :
    ```iwctl
    station ( network interface ) connect ( SSID ) password ( password )
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
    mount -t btrfs /dev/mapper/archlinux /mnt
    cd /mnt
    btrfs subvolume create root /mnt
    btrfs subvolume create root
    btrfs subvolume create home
    btrfs subvolume create swap
    (Optional: For timeshift) btrfs create snapshots
    
