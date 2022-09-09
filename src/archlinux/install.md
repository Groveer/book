# Arch Linux(MBR and UEFI)安装

确认主板系统是UEFI,这里使用GPT分区格式，关于究竟该使用MBR还是GPT请参考[这里](https://wiki.archlinux.org/title/Partitioning_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E9%80%89%E6%8B%A9_GPT_%E8%BF%98%E6%98%AF_MBR)。

## 下载Arch Linux镜像

[下载地址](https://www.archlinux.org/download/)<br>
向下滚动到China下载，网易的节点就很快。

## 验证镜像完整性

`md5 archlinux-2021.05.01-x86_64.iso`<br>
将输出和下载页面提供的md5值对比一下，看看是否一致，不一致则不要继续安装，换个节点重新下载，直到一致为止。

## 将镜像写入U盘

* **Linux/Unix: dd**
1. 用`lsblk`找到U盘并确保没有挂载，使用`umount /dev/sdb1`卸载
2. 使用dd命令：`sudo dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress`
* **MacOS: dd**
1. `sudo su - root` 输入开机密码，获取root权限。
2. 查一下iso镜像在哪，pwd得到路径，我的在<font color=green>/Users/guo/Desktop/archlinux-2021.05.01-x86_64.iso</font>
3. 查看U盘挂载点：`df -h`，我的是<font color=green>/dev/disk2</font>
4. 卸载U盘再装：`diskutil unmountDisk /dev/disk2`
5. `dd if=/Users/guo/Desktop/archlinux-2021.05.01-x86_64.iso of=/dev/disk2 bs=1m`
* **Windows: [USBWriter](https://sourceforge.net/projects/usbwriter/)**

## 从U盘启动Arch live环境

在`BIOS`中设置启动磁盘为刚刚写入镜像的U盘，一般来说开机按`F12`即可选择启动盘，如果不行，则需`F2`或`DEL`进入BIOS修改U盘为优先启动，注意装完系统后将U盘启动改回后面去。<br>
进入U盘的启动引导程序后，选择第一项即可。

## 连接internet

* 查看连接：`ip link`
* 连接

    对于有线网络，安装镜像启动的时候，会默认启动dhcpcd，如果没有启动，可以手动启动：`dhcpcd`</br>
    对于无线网络，官方推荐使用[iwctl](https://wiki.archlinux.org/title/Iwd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#iwctl)

## 磁盘分区

* 查看磁盘设备`lsblk -f`
* 新建分区表：由于我的本子有两块硬盘，一块固态，一块机械，所以我将固态盘作为系统分区，机械盘作为数据分区，即根、boot、swap分区在固态盘上，home分区在机械盘上。<br>
    `fdisk /dev/sda`<br>
    1. 输入`g`，新建GPT分区表
    2. 输入`w`，保存修改，这个操作会抹掉磁盘所有数据，慎重执行！
    3. 对`/dev/sdb`做同样处理。
* 分区创建<br>
    1扇区 = 512字节<br>
    `fdisk /dev/sda`<br>
  1. 新建 EFI System 分区（非UEFI引导可以省略此步）<br>
     1. 输入`n`
     2. 选择分区区号，直接`Enter`，使用默认值，fdisk 会自动递增分区号
     3. 分区开始扇区号，直接`Enter`，使用默认值
     4. 分区结束扇区号，输入`+512M`（推荐大小）
     5. 输入`t`修改刚刚创建的分区类型
     6. 选择分区号，直接`Enter`， 使用默认值，fdisk 会自动选择刚刚新建的分区
     7. 输入`1`，使用 EFI System 类型
  2. 新建 Linux root (x86-64) 分区
     1. 输入`n`
     2. 选择分区区号，直接`Enter`，使用默认值，fdisk 会自动递增分区号
     3. 分区开始扇区号，直接`Enter`，使用默认值
     4. 分区结束扇区号，这里要考虑预留给 swap 分区空间，计算公式：root 分区结束扇区号 = 磁盘结束扇区号 - 分配给 swap 分区的空间 (GB) * 1024 * 1024 * 1024 / 512，输入后`Enter`
     5. 输入`t`修改刚刚创建的分区类型
     6. 选择分区号，直接`Enter`， 使用默认值，fdisk 会自动选择刚刚新建的分区
     7. 输入`23`，使用 Linux root (x86-64) 类型
  3. 新建 Linux swap 分区（使用[交换文件](https://wiki.archlinux.org/title/Swap_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E4%BA%A4%E6%8D%A2%E6%96%87%E4%BB%B6)可以省略此步）
     1. 输入`n`
     2. 选择分区区号，直接`Enter`，使用默认值，fdisk 会自动递增分区号
     3. 分区开始扇区号，直接 Enter，使用默认值
     4. 分区结束扇区号，比如`+8G`，这里直接使用默认值，剩下的空间均给 swap 分区
     5. 输入`t`修改刚刚创建的分区类型
     6. 选择分区号，直接`Enter`， 使用默认值，fdisk 会自动选择刚刚新建的分区
     7. 输入`19`，使用 Linux swap 类型
  4. 保存新建的分区
     1. 输入`w`</br>

* 机械盘进行分区：`fdisk /dev/sdb`<br>
  1. 新建 Linux root (x86-64) 分区
     1. 输入`n`
     2. 选择分区区号，直接`Enter`，使用默认值，fdisk 会自动递增分区号
     3. 分区开始扇区号，直接`Enter`，使用默认值
     4. 分区结束扇区号，直接`Enter`，使用默认值，使用全部空间
     5. 输入`t`修改刚刚创建的分区类型
     6. 选择分区号，直接`Enter`， 使用默认值，fdisk 会自动选择刚刚新建的分区
     7. 输入`23`，使用 Linux root (x86-64) 类型
  2. 保存新建的分区
     1. 输入`w`<br>

## 磁盘格式化

* 格式化 EFI System 分区：`mkfs.fat -F32 /dev/sda1`<br>
    如果格式化失败，可能是磁盘设备存在 Device Mapper，一般来说使用了lvm，会存在这种情况
    * 显示 dm 状态：`dmsetup status`
    * 删除 dm：`dmsetup remove <dev-id>`
* 格式化 Linux root 分区：`mkfs.ext4 /dev/sda2`、`mkfs.ext4 /dev/sdb1`
* 格式化 Linux swap 分区：`mkswap /dev/sda3` && `swapon /dev/sda3`

## 挂载文件系统

在这里需要将除 swap 分区外，其他分区都挂上去，注意先后顺序不能错<br>
```shell
mount /dev/sda2 /mnt
mkdir /mnt/boot
mkdir /mnt/home
mount /dev/sda1 /mnt/boot
mount /dev/sdb1 /mnt/home
```
交换分区启用:<br>
```shell
mkswap /dev/sda3
swapon /dev/sda3
```

## 配置 pacman mirror

一般来说，不需要进行修改，如果发现下载速度很慢，可以修改其中顺序或加入[中国节点](https://wiki.archlinux.org/title/Mirrors_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E4%B8%AD%E5%9B%BD)的地址<br>
```
vim /etc/pacman.d/mirrorlist`
```
或直接使用`reflector`对源速度进行排序：
```
reflector --country China --protocol http --protocol https --latest 5 --save /etc/pacman.d/mirrorlist
```

## 安装 Arch 和 Package Group

`pacstrap /mnt base base-devel linux linux-firmware`

## 生成 fstab 文件

`genfstab -U /mnt >> /mnt/etc/fstab`

## 切换至安装好的 Arch

`arch-chroot /mnt`

## 安装必备工具

`pacman -S vim networkmanager sudo`

## 本地化

* 修改 /etc/locale.gen，取消注释下面这两行配置：<br>
    ```shell
    en_US.UTF-8 UTF-8
    zh_CN.UTF-8 UTF-8
    ```
* 生成 locale 信息：`locale-gen`
* 创建 /etc/locale.conf 并写入：`LANG=en_US.UTF-8`

## 网络配置

* 修改 hostname，创建 /etc/hostname 并写入：`Arch`
* 配置 hosts，编辑 /etc/hosts
  ```shell
  127.0.0.1 localhost
  ::1       localhost
  127.0.1.1 Arch.localdomain
  ```

## 修改 root 密码

`passwd`

## 安装 Microcode

* AMD CPU：`pacman -S amd-ucode`
* Intel CPU：`pacman -S intel-ucode`

## 安装 GRUB

MBR引导：
```shell
pacman -S grub
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

UEFI引导：
```shell
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```

## 重新启动

```shell
exit    # 退出 chroot 环境，或按 Ctrl+D
reboot
```

## 重启后的设置

使用root账户登录后：
* 开启时间自动同步
  ```shell
  timedatectl set-ntp true
  timedatectl set-timezone Asia/Shanghai
  ```
* 新建用户
  ```shell
  useradd -m <username>
  ```
