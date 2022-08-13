# vnc服务搭建方法及使用

## 简介
在日常测试时，需要在服务器上测试图形化的桌面系统功能。为了更方便开展测试工作使用远程桌面操作是非常恰当的选择。

## 服务器上安装x11vnc
### 首先要安装`x11vnc`包，debian系使用下面命令进行安装：
```
sudo apt install x11vnc
```
archlinux可以使用pacman或yay进行安装：
```
sudo pacman -S x11vnc
```

### 设置并存储vnc，这里设置为`deepin`，需要什么密码只需要把`deepin`改为对应的字符串就行。
```
sudo x11vnc -storepasswd deepin /etc/x11vnc.pass
```

### 启动 x11vnc服务端
```
sudo x11vnc -display :0 -auth /var/run/lightdm/root/:0 -forever -bg -o /var/log/x11vnc.log -rfbauth /etc/x11vnc.pass -shared -noxdamage -xrandr "resize" -rfbport 5900
```

## 客户端安装tigervnc-viewer
debian系安装：
```
sudo apt-get install tigervnc-viewer -y
```
archlinux安装：
```
sudo pacman -S tigervnc
```
选择VNC查看器，直接输入服务端的IP，输入前面服务端设置的连接密码进行链接

