# 使用systemd实现自动登陆
1. 修改文件`/etc/systemd/system/getty.target.wants/getty@tty1.service`
2. 追加`-a/--autologin USERNAME`到该行：`ExecStart=-/sbin/agetty --noclear %I $TERM`
3. 修改后：`ExecStart=-/sbin/agetty -a USERNAME %I $TERM`
4. 可能还会删除`-o '-p -- \\u'`（当前Arch安装中存在），因为这将启动登录名，USERNAME但仍要求输入密码。
5. 重新启动后，您将自动登录。

## 防止systemd更新导致配置被重置
1. 查看软链到哪个配置文件：`ls -l /etc/systemd/system/getty.target.wants/getty@tty1.service`
2. 结果为：`/usr/lib/systemd/system/getty@.service`
3. 修改文件属性：`sudo chattr +i /usr/lib/systemd/system/getty@.service`
