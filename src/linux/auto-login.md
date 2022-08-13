# 使用systemd实现自动登陆
1. 修改文件`/etc/systemd/system/getty.target.wants/getty@tty1.service`
2. 追加`-a/--autologin USERNAME`到该行：`ExecStart=-/sbin/agetty --noclear %I $TERM`
3. 修改后：`ExecStart=-/sbin/agetty -a USERNAME %I $TERM`
4. 可能还会删除`-o '-p -- \\u'`（当前Arch安装中存在），因为这将启动登录名，USERNAME但仍要求输入密码。
5. 重新启动后，您将自动登录。

