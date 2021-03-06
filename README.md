# build.labriqueinter.net

## Build on olimex board

### Why ?

To build Labriqueinter.net directly with [yunohost](https://yunohost.org/) we
cannot use debootstrap with qemu-arm because mysql-server-5.5 is buggy and the
installation failed.

The best solution compiling the kernel and perform a debootstrap on an olimex
board. This process take much time but it the best solution to build
labriqueinter.net entirely with scripts.

## How ?

On your fresh olimex installation (build with
[sunxi-debian](https://github.com/bleuchtang/sunxi-debian) for instance) you
can perform the following steps:

### Retrieve scripts

```shell
apt-get install git
git clone https://github.com/labriqueinternet/build.labriqueinter.net.git /opt/build.labriqueinter.net
```

### Prepare Debian for compilation and debootstrap

```shell
cd /opt/build.labriqueinter.net && bash init.sh
```

### Build labriqueinter.net rootfs and u-boot

```shell
cd /opt/build.labriqueinter.net && bash build_labriqueinternet_lime.sh
```

### Create image file

Partitioning on loop device seems not work on my board. So we should retrieve
the tarball on your local machine and create device.

On your computer:

```shell
git clone https://github.com/bleuchtang/sunxi-debian.git && cd sunxi-debian
scp root@myolimex:/srv/olinux/labriqueinternet_"$(date '+%d-%m-%Y')".tar .
scp root@myolimex:/srv/olinux/sunxi/u-boot/u-boot-sunxi-with-spl.bin .
sudo bash olinux/create_device.sh -d img -s 1400 -t labriqueinternet_"$(date '+%d-%m-%Y')".img -b ./labriqueinternet_"$(date '+%d-%m-%Y')".tar -u ./u-boot-sunxi-with-spl.bin
```

### Compress the image

```shell
img=labriqueinternet_"$(date '+%d-%m-%Y')".img
loop=$(sudo losetup -f)
sudo losetup -o 1048576 $loop $img
sudo mount $loop /mnt
sudo sfill -z -l -l -f /mnt
sudo umount /mnt
sudo losetup -d $loop
tar cfJ $img.tar.xz $img
```

### Copy the image on your SD card

A cool howto is available on the [Yunohost web site](https://yunohost.org/#/copy_image_en)

But on our case, and if you are on a Linux machine, it's quite easy:

```shell
sudo dd if=labriqueinternet_"$(date '+%d-%m-%Y')".img of=/dev/mmcblk0
```

**Note**: Do not forget to change `/dev/mmcblk0` with the appropriate value.

### Connect to your board and the yunohost postinstall

#### With http

Find the IP of your box and connect with your browser on https://mybox.

#### With ssh

Find the IP of your box and ssh on it! (password: olinux):

```shell
ssh root@mybox
```

Change the root password and do the postinstall:

```shell
yunohost tools postinstall
```

## Build with cross compilation and cross debootstrap

/!\ Warning: with this method you cannot perform a debootstrap with yunohost
directly.

See [sunxi-debian](https://github.com/bleuchtang/sunxi-debian)
