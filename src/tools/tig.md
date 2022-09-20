# 颠覆 git 命令使用体验的神器 -- tig

tig，就是一个命令行工具，日常使用中用它来取代 git 最高频的几个操作, 如 git log, git diff 以及 git blame等。

git 和 tig 的关系有点像 top 和 htop, 是一种命令行交互式操作工具 tig 的所有功能都是 git 命令行已经具备的,  tig 提供了一种直观, 方便快捷的 git 操作。

## 安装

tig 在各大发行版均有集成，使用包管理器就可以很轻松的进行安装。

如果想体验最新版的功能，还可以使用源码进行安装，详情查看其[官方文档](https://github.com/jonas/tig/blob/master/INSTALL.adoc)

## 使用

1. 在 git 项目中敲 tig, 进入 tig 界面后再敲 h (代表help) 即可进入帮助界面, 该界面列出了所有常用命令。tig 默认进入 log 界面，也叫 main 视图, 使用 j/k 或 上/下 键可以选择指定提交, 回车后, 界面的一半会展示此次 commit 详情。 此时, 上/下 键可以选择 log 中的 commit, 详情界面会跟着变化, 而 j/k 键会在 commit 详情内移动焦点, 选中 commit 中列出的文件, 回车会跳转到该文件的详情, 而使用 @ 可以按照代码块的粒度来浏览 commit 中的内容, 通过这些操作, 我们可以很容易的快速浏览log 中多个commit 中的内容, 而这一点通过 git 命令或 gui 都是很难快速方便的完成的。<br>
![快速查看 log 详情及 help](https://upload-images.jianshu.io/upload_images/185581-abaddeb4a1e9a76a.gif?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp)

2. 在使用 git 命令的过程中, 最高频的命令应该是 git status, 主要用来查看 staged changes 和 unstaged changes, 通过 tig, 可以很方便的像刚才查看 commit 那样查看 staged changes 和 unstaged changes。敲 tig 进入 log 界面后, 排在最上面的便是 staged changes 和 unstaged changes, 至此, staged changes 和 unstaged changes 就像一个 commit 一样被方便地展示出来了, 敲回车, 详情界面展示出来后敲 u 会使整个 changes 由staged changes 变为 unstaged changes, 或是由unstaged changes 变为 staged changes, 如果想要 changes 中的某一个文件改变状态, 则在详情界面选中该文件, 回车, 再敲 u ,即可使该文件由 staged 变为 unstaged, 或是由 unstaged 变为 staged, 如果你想重置某个文件的修改, 选中该文件敲 ! 即可, 再也不用使用 git reset HEAD这个命令了。<br>
![log 界面最上方可以查看未提交修改](https://upload-images.jianshu.io/upload_images/185581-3c2818551c84078b.png?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp)

3. 如果还想看 untracked files 怎么办呢? tig 提供了一种更纯粹的查看 git status 的界面, 进入 tig 后直接敲 s 即可, 选中 untracked file 或 unstaged file, 敲 u, 即可变为 staged file, 选中 staged file 敲 u 变为 unstaged file, 如果你想重置某个文件未保存的修改, 在该文件下敲 ! 即可, 如果你准备好提交了, 按下 shift + c 即可打开默认命令行编辑器来编辑 commit message, 如果在 tig 主界面按下shift + c, 将会使用 git cherry-pick 命令，敲 r 可进行分支选择，选中要激活的分支，然后通过 shift + c 即可激活某个分支，然后再次选择其他分支，然后在想拣选的 commit 上按 shift + c 即可进行 cherry-pick。<br>
![tig 的 status 界面](https://upload-images.jianshu.io/upload_images/185581-821f344829fb8bd7.png?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp)

4. tig 也可以当做命令行版的 finder 来使用, 在 log 主界面敲 t (代表 tree) 即可进入此次 commit 中所有文件列表, 在文件夹下回车可以进入文件夹, 在文件下回车可以在界面的一半展示该文件的全貌(而不是此次 commit 的修改)。<br>
![tree 界面](https://upload-images.jianshu.io/upload_images/185581-c9bdacb04e0c495f.png?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp)

5. 如果选中文件, 按 b 即可进入该文件的 blame 界面, 在 blame 中选中任意一行回车, 即可在界面的一半展示此次 commit 的所有内容, 依然可以用 j/k 控制详情内容的单行移动, 回车跳转到某文件, @按照代码块粒度滚动, 这种操作比使用 git blame 方便了许多。<br>
![blame 界面下快速查看 commit 全貌](https://upload-images.jianshu.io/upload_images/185581-d6b8e713d7362aed.png?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp)

6. 如何查看一个文件的全部提交记录? 以及快速查看某次提交的全部内容? 有了 tig, 可以轻松做到这一点, 直接 tig filename, 进入到该文件的 tig 主界面, 即可快速查看指定文件的 log 和提交内容, 你还可以选择只查看某个 commit 以及之前的提交, 只需要使用 tig commit-id filename 即可。

7. 如何查找 commit message 中带有指定文字的 commit 呢? 如果终端本身支持搜索功能, 使用终端自带的 cmd + f 即可搜索 tig 主界面中的任何文本, 那如何通过 commit-id 查找呢? tig 主界面中默认没有展示 commit-id, 使用 shift + x 即可展示 commit-id。

8. tig 也自带搜索功能, 敲 / 即可进入, 输入字符后回车, 将高亮展示所有匹配项, 敲 n 将聚焦到离当前焦点最近的下方的匹配项, 大写 N 则是上方的匹配项, 敲回车将展示详情。<br>
![tig 的原生搜索功能](https://upload-images.jianshu.io/upload_images/185581-056846efaba11607.gif?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp)

9. 在提交 commit 中常常会碰到按代码块的粒度来提交的需求, 使用原生的 git add -i 略显繁琐, 在 tig 中, 这个操作变得无比简单, 只需要在 staged changes 或 unstaged changes 使用 @ 选中代码块, 敲 u 即可改变状态, 如果你只想改变一行代码的状态, 使用 j/k 选中要改变的单行代码, 用数字键 1 代替 u 即可实现这个原本用 Git 命令行很难实现的功能。

[参考](https://www.jianshu.com/p/e4ca3030a9d5)
