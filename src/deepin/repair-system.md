# Deepin 系统修复方法

## 简介
Deepin系统是一个基于debian的操作系统（Deepin为其专业版），最近发现很多小伙伴在升级时可能导致系统崩溃，无法进入的情况，这里提供一个通用的修复方法。一般升级出现的问题都是系统核心模块没有正常升级导致(通常是systemd);这里的修复方法是通过live系统对Deepin本身的文件系统进行挂载，然后chroot进去进行再次更新来进行修复。

## 术语说明
Deepin：Deepin操作系统，基于debian实现，属于一种linux发行版。

live：live系统，通过iso制作的启动盘就是live系统，任何启动盘都可以。

chroot：这是一个命令，目的是切换根目录，将某个目录当作根目录来识别。

## 关于磁盘挂载的知识
在linux系统中，分区的挂载不像windows中那样分为C盘、D盘，在linux中，所有分区都是以挂载点的方式挂到指定的目录中，而且必须有一个根分区，其挂载点就是`/`。

如：有这样一个分区方案，用户在安装时，将`/dev/sda`磁盘分了两个分区进行安装，其中一个是根分区，另一个是`home`分区，什么意思呢，就是有一个分区的路径是`/dev/sda1`，叫做第一个分区，其挂载点是`/`，还有一个分区的路径是`/dev/sda2`，叫做第二个分区，其挂载点是`/home`，这样，用户在使用时，系统的数据全部在第一个分区中，而用户的数据就全部在第二个分区中。对于操作系统来说，其并不关心`/home`是哪个分区，它只要有这个路径就行，如果没有第二个分区，它就会使用第一个分区的空间，不管有没有第二个分区，只有有一个根分区，并且其根文件系统在其中，就可以运行linux操作系统。

现在我们知道，分区的挂载最重要的是挂载点，那么如何进行挂载？可以使用`mount`与`unmount`命令进行挂载和卸载，如上方所言，需要将`/dev/sda1`挂载到`/mnt`目录，那么可以使用`sudo mount /dev/sda1 /mnt`即可。那如何知道自己的分区是`/dev/sda1`？使用`lsblk -f`查询一下即可，该命令会输出所有的分区信息。
## 操作步骤
1. 首先需要准备一个启动盘，任何版本都可以，启动盘的制作这里不再赘述。
2. 将启动盘插入电脑，然后以U盘启动进入安装界面。
3. 按`ctrl+alt+f2`切换tty,等待片刻。
4. 这一步很重要，需要首先知道根分区是哪个分区，可以通过`lsblk -f`命令查看当前电脑里有哪些分区，一般`ext4`格式为Deepin的根分区，当然，可能不止一个`ext4`分区，可以凭借记忆挂载根分区，如果真的忘记了也不要紧，可以把每个ext4分区挂载一遍，然后查看一下就清楚了。
5. 查到根分区后就可以查看`fstab`的内容，这个文件是指定Deepin系统所有分区的挂载方式，任何分区方案都在这里写的明明白白。如当前电脑中`/dev/sda2`是Deepin的根分区，那么可以使用`sudo mount /dev/sda2 /mnt`进行挂载，然后`cat /mnt/etc/fstab`查看`fstab`的内容。
6. 如我的Deepin分区方案是这样，`/dev/sda1`是根分区，`/dev/sda2`是home分区，那么，可以这样挂载：
```
sudo mount /dev/sda1 /mnt
sudo mount /dev/sda2 /mnt/home
```
7. 分区挂载好了，还需要将设备文件绑定过去，分别按下面顺序进行挂载：
```
sudo mount --bind /sys /mnt/sys
sudo mount --bind /proc /mnt/proc
sudo mount --bind /dev /mnt/dev
sudo mount --bind /dev/pts /mnt/dev/pts
```
8. 分区和设备文件都挂载好了，可以chroot进入Deepin的根进行继续更新Deepin了，`sudo chroot /mnt`，这时会发现终端变得不一样了，说明已经成功进行chroot了，然后执行一下:
```
apt update
apt dist-upgrade
update-grub
```
当执行`apt update`可能会出现无法获取仓库源的情况，这是由于live系统未激活导致，但是一般会出现`dpkg --configure -a`的提示，此时使用该命令再次更新即可。
9. 至此，Deepin系统已经成功进行再次更新，然后按`ctrl+d`退出chroot环境，最后使用`sudo reboot`重启系统进行Deepin就可以查看是否正常了
