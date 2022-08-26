# Arch Linux安装后配置

## 配置源

### 添加`archlinuxcn`仓库
1. 添加文件：`/etc/pacman.d/archlinuxcn-mirrorlist`</br>
添加内容：`Server = https://repo.archlinuxcn.org/$arch`</br>
更多内容：参考[国内仓库](https://github.com/archlinuxcn/mirrorlist-repo/blob/master/archlinuxcn-mirrorlist)</br>
2. 修改文件：`/etc/pacman.conf`</br>
取消注释 `#Color`</br>
添加内容：
```
[archlinuxcn]
Include = /etc/pacman.d/archlinux-mirrorlist
```
3. 安装 GPG key：
```shell
sudo pacman -Syu
sudo pacman -S archlinuxcn-keyring
```
若安装失败，可以参考[GnuPG-2.1 与 pacman 密钥环](https://www.archlinuxcn.org/gnupg-2-1-and-the-pacman-keyring/)重新生成密钥环。
### 配置[aur](https://wiki.archlinux.org/title/Arch_User_Repository_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
```shell
sudo pacman -S yay
```
以后就可以使用`yay`代替`pacman`了，语法完全一致，无需提前`sudo`，并且可访问`aur`仓库。
### 安装扩展固件
为了让电脑充分发挥性能，在配置`aur`仓库后，应安装扩展固件：
```shell
yay -S mkinitcpio-firmware
```

## 配置控制台登录图形桌面

### 配置X11环境
1. 安装[xorg](https://wiki.archlinux.org/title/Xorg_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))：
```shell
yay -S xorg-server xorg-xinit
```
2. 修改`～/.xinitrc`文件，并添加内容，挑选自己的桌面环境添加：
```
exec startplasma-x11       (kde-x11)
exec startdde              (dde)
exec gnome-session         (gnome)
exec openbox-session       (openbox)
exec startxfce4            (xfce)
exec mate-session          (mate)
```
3. 修改`~/.bash_profile`(bash)或者`～/.zprofile`（zsh）文件，并添加内容：
```
if [[ -z $DISPLAY && $(tty) == /dev/tty1 ]]; then
  exec startx
fi
```

### 配置Wayland环境
wayland本身只是个协议，并不提供图形环境，因此需要安装混合器或直接安装有内置混合器的桌面环境
1. KDE桌面环境
```shell
yay -S plasma-desktop plasma-wayland-session
startplasma-wayland
```
2、Gnome桌面环境
```
yay -S gnome-shell
gnome-shell --wayland
```

其他桌面环境自行搜索：[wayland](https://wiki.archlinux.org/title/Wayland)

注意：某些应用程序是支持wayland协议，但是默认却走的xwayland，导致显示效果很不理想，可以设置环境变量或配置文件使其走wayland协议
1. Firefox：
环境变量：
```
MOZ_ENABLE_WAYLAND=1
```
2. Chromium：
在`~/.config/chromium-flags.conf`中，添加以下几行:
```
--enable-features=UseOzonePlatform
--ozone-platform=wayland
--enable-features=WebRTCPipeWireCapturer
```
3. qt程序：
需要额外安装：
```shell
yay -S qt5-wayland qt6-wayland
```
然后每次启动程序添加参数：`-platform wayland`
或直接设置`QT_QPA_PLATFORM=wayland`，这样就不必每次添加参数


## 安装[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)及插件

1. 安装必备工具：
```shell
yay -S git wget
```
2. 安装`oh-my-zsh`：
创建文件`install.sh`并写入以下[链接](https://gitee.com/mirrors/oh-my-zsh/blob/master/tools/install.sh
)中的内容<br>
修改以下内容：
```shell
# Default settings
ZSH=${ZSH:-~/.oh-my-zsh}
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
BRANCH=${BRANCH:-master}
```
改为：
```
REPO=${REPO:-mirrors/oh-my-zsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}
```
添加可执行权限：
```shell
chmod +x install.sh
```
执行脚本进行安装：
```shell
./install.sh
```
3. 安装历史记录插件和语法检查插件
```shell
cd ~/.oh-my-zsh/plugins
git clone https://github.com/zsh-users/zsh-autosuggestions.git
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
```
4. 修改`～/.zshrc`文件，找到`plugins=(git)`，改为：
```
plugins=(
	git
	sudo
	zsh-syntax-highlighting
	zsh-autosuggestions
)
```
5. 使插件生效：
```shell
source ~/.zshrc
```

## 安装输入法

1. 安装[fcitx5](https://wiki.archlinux.org/title/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))：
```shell
yay -S fcitx5 fcitx5-chinese-addons fcitx5-gtk fcitx5-configtool
```
2. 配置环境变量`/etc/environment`：
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
INPUT_METHOD=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

## 配置开发环境
### python
1. 下载pip
```shell
yay -S python-pip
```
2. 设置pip源
```shell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
### golang
1. 安装[go](https://wiki.archlinux.org/title/Go_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))：
```shell
yay -S go
```
2. 选择一个go工作目录，以`～/Develop/go`为例，新建三个目录：
```shell
mkdir -p ~/Develop/go/{src,bin,pkg}
```
3. 配置环境变量`~/.bash_profile`，zsh环境为`~/.zprofile`：
```shell
export GOROOT=/usr/lib/go
export GOPATH=~/Develop/go		   # 这两行你需要
export GOBIN=~/Develop/go/bin		# 修改为自己的
export PATH=$PATH:$GOROOT/bin:$GOBIN
```
```shell
source ~/bash_profile
```
4. 配置GOPROXY
```shell
go env -w GOPROXY=https://goproxy.io,direct
```

## 安装KDE桌面环境

1. 最小化安装：
```shell
yay -S plasma-desktop
yay -S plasma-nm plasma-pa kscreen bluedevil powerdevil kwalletmanager konsole
```
`plasma-desktop`：基本的KDE桌面环境<br>
`plasma-nm`：网络管理模块，右下角的网络连接以及系统设置中的网络设置<br>
`plasma-pa`：声音模块，右下角的声音设置以及系统设置中的音频设置<br>
`kscreen`：屏幕管理模块，系统设置中的显示器配置<br>
`bluedevil`：蓝牙模块，启用需`systemctl enable bluetooth`<br>
`powerdevil`：电源管理模块，系统设置中的电源管理<br>
`kwalletmanager`：KDE 钱包管理，一般用来禁用电子钱包<br>
`konsole`：KDE 的仿真终端<br>
2. 基本安装：<br>
基本安装并不代表最小化安装，基本安装会附带很多程序，如桌面小工具、discover等
```shell
yay -S plasma
```
3. 安装KDE应用：
注意，这仅仅安装应用程序，并不会安装 Plasma 桌面。
```shell
yay -S kde-applications
```
