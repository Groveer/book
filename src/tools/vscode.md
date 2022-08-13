# vscode 使用小技巧

## vscode中go开发环境配置

### 安装插件

首先`go`插件必须要安装上，安装完成后打开go文件，会提示安装`gopls`，但是此时不一定可以安装，因为是国内环境，懂得都懂。

### 安装gopls和go-outline

正确的安装方式是，首先配置go的代理：
```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

然后就可以简单的使用`go install`进行命令进行安装啦：
```
go install golang.org/x/tools/gopls@latest
go install github.com/ramya-rao-a/go-outline@latest
```

最后，重新打开vscode,是不是发现智能提示已经有了。

### 配置调试工具

在打开的go文件中按`F5`调试，会提示安装dlv-dap工具，点击`install`进行安装，安装完成后，再次按`F5`会有一行错误提示：
```
go.mod file not found in current directory or any parent directory; see 'go help modules'
```

此时，在终端中输入：
```
go env -w GO111MODULE=auto
```

然后再次按`F5`启动调试，发现程序可以正常运行了！

## vscode 配置 plantuml

1. 安装 plantuml 和 graphviz
debian:
```shell
sudo apt install plantuml graphviz
```
archlinux
```shell
sudo pacman -S plantuml graphviz
```
3. vscode 安装 plantuml 插件

## VSCodium和Code-OSS使用vscode扩展

1. 找到product.json文件：
```
debian: /usr/share/codium/resources/app/product.json
archlinux: /usr/lib/code/product.json
```
2. 修改"extensionsGallery"字段：
```json
"extensionsGallery":{
    "serviceUrl":"https://marketplace.visualstudio.com/_apis/public/gallery",
    "cacheUrl":"https://vscode.blob.core.windows.net/gallery/index",
    "itemUrl": "https://marketplace.visualstudio.com/items"
}
```
3. 重启code即可

## vscode使用gitee同步配置

1. 在网页上登陆gitee后，点击右上角“发布代码片段”
2. 随便填写信息，如“vscode clound sync”，点击发布
3. 发布后跳转的浏览器地址最后一段就是gistID,若遗忘，则在个人主页后添加`/codes`就可以进入代码片段管理页面了
4. 点击头像，进入设置页面，找到私人令牌，创建私人令牌
5. 安全起见，建议只勾选`user_info`和`gists`权限
6. 提交后会弹出消息框，务必记录该私人令牌，因为它只会出现一次，如我的就是：
```
ae82d00e89e159112b1bacce8dd3c8a8
```
7. 打开vscode，按下`Ctrl+Shift+P`，输入`settings.json`打开设置，并填入以下内容：
```
"gitee.gist":"将双引号里面的内容替换成前面获取到的gistID",
"gitee.access_token":"将双引号里面的内容替换成你的私人令牌"
```
8. 配置好后，可以使用`upload setting`命令上传设置，`download setting`命令下载配置


