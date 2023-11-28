This wiki is another addition to the [InstallingDebianOnStarFiveVisionFiveV2](https://wiki.debian.org/InstallingDebianOn/StarFive/VisionFiveV2). I'll update this wiki to the Debian wiki once the instructions is fixed and confirmed.

# Boot Mode Settings
You can set [`00`](https://doc-en.rvspace.org/VisionFive2/Quick_Start_Guide/VisionFive2_SDK_QSG/boot_mode_settings.html) to boot from SD card or nvme at least for my testing the setting is okay.

# Update u-boot(optional)
You can follow the [wiki](https://doc-en.rvspace.org/VisionFive2/Quick_Start_Guide/VisionFive2_SDK_QSG/spl_new.html) to update u-boot if need.

# installation

```bash
# Preparing disk image

dd if=/dev/zero of=debian-sid-risc-v-vf2.img bs=1M count=4096

# Partition image with correct disk IDs
sudo sgdisk -g --clear --set-alignment=1          -g --clear --new=1:0:+16M: --new=2:0:+100M: -t 2:EF00 --new=3:0:-1M: --attributes 3:set:2 -d 1 debian-sid-risc-v-vf2.img

# Mount image in loop device
sudo losetup --partscan --find --show debian-sid-risc-v-vf2.img

##  format partitions( note repalce your loop*)
sudo mkfs.vfat /dev/loop*p2
sudo mkfs.ext4 /dev/loop*p3
sudo e2label /dev/loop*p2 boot
sudo e2label /dev/loop*p3 rootfs

# config debian sid rootfs(my loop dev is loop6)
sudo mount /dev/loop6p3 /mnt/

# setup riscv64 rootfs
sudo debootstrap --arch=riscv64  unstable /mnt https://deb.debian.org/debian

sudo systemd-nspawn -D /mnt -M debian --bind-ro=/etc/resolv.conf

## get update
$ apt-get upgrade
$ apt-get install initramfs-tools openssh-server systemd-timesyncd rsync bash-completion

# then install prebuilt kernel from here：
# https://github.com/yuzibo/vf2-linux/actions
# or you can build your kernel Debian packages
# you do not need -dbg package in general. 
apt install -f ./*.deb


# Set up basic networking
cat >>/etc/network/interfaces <<EOF
auto lo
iface lo inet loopback

auto end0
iface end0 inet dhcp
EOF


# Set root password 'vf2'
passwd

# Change hostname
echo unmatched > /etc/hostname

# config u-boot
cat <<EOF > /boot/uEnv.txt
kernel_comp_addr_r=0xb0000000
kernel_comp_size=0x10000000
EOF

# change device tree file
cd /usr/lib/linux-image-6.6.**/starfive/
ln -s jh7110-starfive-visionfive-2-v1.2a.dtb jh7110-visionfive-v2.dtb

# update extlinux.conf
$ apt-get install u-boot-menu
$ cat <<EOF >> /etc/default/u-boot
U_BOOT_PARAMETERS="rw console=tty0 console=ttyS0,115200 earlycon rootwait stmmaceth=chain_mode:1 selinux=0"
EOF
$ cat /etc/default/u-boot # double check your work
$ dpkg-reconfigure linux-image-5.15.0-starfive-g7b7b4eddd8d5
$ cat /boot/extlinux/extlinux.conf # double check your work

# modify rootfs mount dirctory, here is on SD card.
# if you hope to boot the rootfs on nvme, you *must* change it to `/dev/nvme0n1p3`
$ sed -i -e 's|root=[^ ]*|root=/dev/mmcblk1p3|' /boot/extlinux/extlinux.conf

mkdir -p /boot/efi
$ cat <<EOF > /etc/fstab
/dev/mmcblk1p2 /boot/efi vfat umask=0077 0 1
EOF

apt install openssh-server openntpd ntpdate
apt clean

# set the time immediately at startup
sed -i 's/^DAEMON_OPTS="/DAEMON_OPTS="-s /' /etc/default/openntpd

exit
```

# Clear history
```bash
sudo rm /mnt/root/.bash_history
```

# Finish and write image to sdcard
```bash
sudo umount /mnt
sudo losetup -d /dev/loop6
```

# Finish and write image to sdcard
# take care of writing to the correct sdcard-device
sudo dd if=debian-sid-risc-v-vf2.img of=/dev/sdcard-device bs=64k iflag=fullblock oflag=direct conv=fsync status=progress

# issues

## boot from nvme 

I am still struggle to work boot from nvme directly, but it does not work. You have to use SD card bootloader then use rootfs under nvme.

Example extlinux.conf on SD card:

```bash
label l0
        menu label Debian GNU/Linux trixie/sid 6.6.2-vf2
        linux /boot/vmlinuz-6.6.2-vf2
        initrd /boot/initrd.img-6.6.2-vf2
        fdtdir /usr/lib/linux-image-6.6.2-vf2/

        append root=/dev/nvme0n1p3 rw console=tty0 console=ttyS0,115200 earlycon rootwait stmmaceth=chain_mode:1 selinux=0

```

## 8GB ram Recognition
By default the device tree support 4GB, you have to compile 8GB dtb by hand(I will added it for vf2-linux also). please refer to:
https://github.com/starfive-tech/VisionFive2/issues/20#issuecomment-1374907916

Any issue please open issue or email me, thanks.

