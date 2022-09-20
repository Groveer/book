# lazygit：强大而好玩的 git 终端界面工具

git 是一个强大和好用的工具，可以让一切都基于 git 来做，用过都说好（先赞美一下git :-)）。

但是 git 又是一个相对不好入门的东西，因为 git 的一些和精华都是基于命令行的。虽然有层出不穷的 git gui 客户端，但是基本都是对 git 基本命令的包装，也没有哪一个是值得可取地方。今天介绍一款简单好用的 git 终端工具。

lazygit 是一个用于 git 命令行的简单终端 ui，使用 go 语言编写，用到了 gocui 库，目的是在命令行提供 git 的图形界面。

## 安装

安装非常简单，因为 lazygit 被编译为独立的二进制文件，所以只要获取其二进制即可运行。

lazygit 代码托管在 github，可以到 [lazygit-release](https://github.com/jesseduffield/lazygit/releases) 中获取最新的版本，也可以在各大发行版中使用包管理进行安装。

## 使用

lazygit 使用非常简单，cd 到 git 仓库目录，然后执行：
```shell
lazygit
```
当然，也许将 lazygit 起一个别名是一个更佳的选择：
```shell
echo "alias lg='lazygit'" >> ~/.zshrc
```
或者任何一个正在使用的 bash `rc` 文件。

既然是客户端工具，那就需要了解一下其界面：
![lazygit ui](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/760a854e5f7346899361ed73c801e00b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

界面整体被分为两大部分，左侧是5个状态窗口，右侧为每个状态的详细内容窗口，左侧切换焦点，右侧会显示详细内容。可以使用tab、数字1～5或者左右箭头来切换左侧窗口的焦点状态，使用 j k 或上下箭头对窗口内容进行选取。

lazygit 所有的功能都支持快捷键，并且快捷键较为复杂，不用担心，只需要记住一个万能快捷键：`x`。它用于显示当前可操作的菜单，并且会显示相应的快捷键。由于每个状态窗口的快捷键均会有所不同，所以这里不会一一列举。
