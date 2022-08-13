# 使用zram创建swap分区
以下是一个简单的脚本用来创建zram swap设备，注意，这个脚本假设你还没有使用zram，并提供了启动的systemd配置:
1. `/usr/local/bin/zramswap-on`:
```shell
#!/bin/bash
# Disable zswap
echo 0 > /sys/module/zswap/parameters/enabled

# Load zram module
modprobe zram

# use zstd compression
echo zstd > /sys/block/zram0/comp_algorithm

# echo 512M > /sys/block/zram0/disksize
echo 2G > /sys/block/zram0/disksize

mkswap /dev/zram0

# Priority can have values between -1 and 32767
swapon /dev/zram0 -p 32767
```
2. `/usr/local/bin/zramswap-off`:
```shell
#!/bin/bash
swapoff /dev/zram0
echo 0 > /sys/class/zram-control/hot_remove

# Not required, but creating a blank uninitalzed drive
# after removing one may be desired
cat /sys/class/zram-control/hot_add
```
3. `/etc/systemd/system/create-zram-swap.service`
```shell
[Unit]
Description=Configures zram swap device
After=local-fs.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/zramswap-on
ExecStop=/usr/local/bin/zramswap-off
RemainAfterExit=yes

[Install]
WantedBy = multi-user.target
```
然后执行服务配置加载和激活:
```shell
sudo chmod +x /usr/local/bin/zramswap-on
sudo chmod +x /usr/local/bin/zramswap-off
sudo systemctl daemon-reload
sudo systemctl enable --now create-zram-swap.service
```
*参考*：[https://cloud-atlas.readthedocs.io/zh_CN/latest/linux/redhat_linux/kernel/zram.html](https://cloud-atlas.readthedocs.io/zh_CN/latest/linux/redhat_linux/kernel/zram.html)

