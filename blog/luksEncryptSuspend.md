# Safely hibernating while using disk encryption

Previously, I had setup an encrypted swap space while doing [my Arch linux installation](arch%20linux%20full%20disk%20encryption%installation%guide.html). It worked well for my purpose. 

Although it worked well I faced a small issue, it took a long time to boot. During boot time the partition was encrypted on the fly and mounted,that added a few extra seconds to the boot time. 

Also, that does not support suspend to disk feature which I was aware of while setting it up and considered to be a non issue. 

While I still rarely suspend to disk or hibernate, chances have come up where hibernating the system would have been convenient  and long boot time not leave me slightly annoyed.

As I did not encrypt my `/boot` partition, using initramfs to open a luks encrypted swap may prove to be a [security risk](https://wiki.archlinux.org/index.php?title=Talk:Dm-crypt&oldid=255742#Suspend_to_disk_instructions_are_insecure).

### Swapfile to the rescue

Using a swapfile proved to be the best option for me. It can be setup on an encrypted blockdevice's partition.

First create a new swapfile

` dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress `

This will create a 2GB file `/swapfile`, you can adjust the size and location of the file according to your needs.

Now set permissions and make the file into a swap

`chmod 600 /swapfile`

`mkswap /swapfile`

Now remove any previous swap 

`swapoff -a`

Now find the swap file offset

`filefrag -v /swapfile | awk '{ if($1=="0:"){print substr($4, 1, length($4)-2)} }'`

Now you just have to set boot parameters `resume` and `resume_offset` in your systemd-boot entry

```
 title Arch Encrypted
 linux /vmlinuz-linux
 initrd /initramfs-linux.img
 options rw cryptdevice=UUID=uuid of your disk partition(i.e. /dev/sdXy):name_of_your_device root=/dev/mapper/name_your_device resume=/dev/mapper/name_of_your_device resume_offset=(above output)
```

Add your swapfile to `/etc/fstab` 

`/swapfile none    swap    defaults 0 0`

Also add `resume` hook to `/etc/mkinitcpio.conf` file

Don't forget to generate new initramfs using `mkinitcpio -p ...`

Now you are all set, reboot and everything should work as expected.