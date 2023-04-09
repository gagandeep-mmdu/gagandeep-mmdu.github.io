## Arch Linux Installation with full disk encryption
### Written guide for [video posted on my Youtube channel](https://www.youtube.com/watch?v=g3axA7KUMqM).

#### Note that if you follow these notes to setup encrypted swap then you won't be able to hibernate your machine as this type of swap won't support doing that. 


#### If you want hibernation support consider using [swapfiles](https://wiki.archlinux.org/index.php/Dm-crypt/Swap_encryption#Using_a_swap_file) or [LVM on LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Swap_encryption#LVM_on_LUKS) or follow [this wiki](https://wiki.archlinux.org/index.php/Dm-crypt/Swap_encryption#mkinitcpio_hook).

##### Please note that you *MUST* ensure that you have made sure that your drive(s) are [prepared](https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation) for encryption.
First check if you are booted using UEFI (Legacy bios work too [bootloader installation may be different] )

    ls /sys/firmware/efi


Also make sure that you are connected to the internet

    ping archlinux.org    

If you are on wired the connection will be setup automatically ,for wireless use wifi-menu or iwctl

Now Partition the disk (use gdisk if using GPT OR fdisk if using MBR), I'm using GPT here.

    gdisk /dev/sdX
Where X denotes the drive where you want to do the installation e.g. a,b,c....
Partition your disk(s) according to your requirements


Format the partition you want to use as ESP (required for UEFI as bootloader will be installed there) as FAT32

    mkfs.vfat /dev/sdXy
where X is your device letter and y is your partition number.

Now lets encrypt our drive(s)/partition(s)

    cryptsetup -y -v -s 512 -h sha512 --use-urandom luksFormat --type luks2 /dev/sdXy
where X is your device letter and y is your partition number.

Note that you may not need all these parameters, if unsure read the [Arch Wiki](https://wiki.archlinux.org/index.php/Dm-crypt#Device_encryption) or go with the defaults (they are pretty good)

    cryptsetup -y -v /dev/sdXy (using the same command with default options)


Make sure you set a *STRONG PASSWORD*, without that encryption will be pointless.

Repeat the same step for your seperate home partition (if using)

    cryptsetup -y -v -s 512 -h sha512 --use-urandom luksFormat --type luks2 /dev/sdXy
where X is your device letter and y is your partition number.


Now decrypt the partition(s) and mount the same

    cryptsetup open /dev/sdXy name_your_device    "example cryptroot"


Format the drive with filesystem of your choice
 
    mkfs.ext4 /dev/mapper/name_your_device

Make a directory to mount the partitions at (e.g. /mnt)

    mkdir /mnt

Now mount the partition to the directory

    mount /dev/mapper/name_your_device /mnt

Make directory for boot 

    mkdir /mnt/boot
    Note that if using Grub you may need to mount ESP at /boot/EFI do accordingly

mount the ESP

    mount /dev/sdXy /mnt/boot

Now just run the pacstrap script with directory with mounted drive(s) as target

    pacstrap /mnt base 

Now wait as pacman downloads and installs the packages.

Now chroot to your new system 

    arch-chroot /mnt

Now we will generate a keyfile to use with our home partition

    dd bs=1024 count=4 if=/dev/urandom of=/root/keycrypthome
    
Now change the permissions of the file so that only root can read it
    chmod 0400 /root/keycrypthome

Note that you can use external usb devices for storing the keyfile too, but make sure the device is encrypted or there is some other kind of placeholder on present

### DON'T KEEP YOUR KEYFILE ON AN UNENCRYPTED DRIVE

Now add the keyfile to the crypt partition

    cryptsetup luksAddKey /dev/sdXy /root/keycrypthome

Exit from the chroot environment and mount the partition

    cryptsetup open /dev/sdXy name_your_device

Format the partition with a filesystem of your choice

    mkfs.ext4 /dev/mapper/name_your_device

Mount the partition 

    mount /dev/mapper/name_your_device /mnt/home

`arch-chroot` again

now setup our encrypted swap 

    mkfs.ext4 -L name_your_swap /dev/sdXy 1M

Now edit the entries in `/etc/crypttab`

    swap	LABEL=name_your_swap	/dev/urandom	swap,cipher=aes-xts-plain64:sha512,size=512
    "Note that you should add cipher of your choice OR let the defaults be

Add entry for home partition too

    name_your_device	UUID='your /dev/mapper/name_your_device uuid goes here'	/path/to/your/key

Exit out of the chroot environment

    exit

Generate the fstab entries

    genfstab -U /mnt >> /mnt/etc/fstab

Add your swap to the `/etc/fstab`

    /dev/mapper/swap	none	swap	defaults	0	0

Now, set your timezone, sync your hardware clock, set your locale, add users, set passwords, etc

Now lets edit the `/etc/mkinitcpio.conf` to detect and unlock our encrypted drive(s)

    add encrypt HOOK in your HOOKS section
    HOOKS=(..... keyboard encrypt .......)

Now generate new initramfs

    mkinitcpio -p linux

Now lets install the bootloader (systemd-boot)

    bootctl install

Edit `/boot/loader/loader.conf` and add your default entries there

    default arch   	"or any one that you want"
    timeout 5

Edit `/boot/loader/entries/arch.conf` and add your parameters

    title Arch Encrypted
    linux /vmlinuz-linux
    initrd /initramfs-linux.img
    options rw cryptdevice=UUID=uuid of your disk partition i.e. /dev/sdXy:name_your_device root=/dev/mapper/name_your_device

Setup your network manager tool (i.e. NetworkManager, systemd-networkd, connman, etc)

Now just reboot.

The System should ask you for your password at boot time.

Enjoy your freshly installed Encrypted arch linux install.