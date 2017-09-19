---
title: 记一次宕机
date: 2017-05-30 18:11
tags: 
- Issues
- 瞎折腾
categories: 瞎折腾
thumbnail: https://ooo.0o0.ooo/2017/05/30/592d5d883d6aa.png
---

# 起因

事情是这样的。我准备把位于英国机房的VPS换到日本去，随后按照网上说法，先 `Snapshot`， 新建实例后恢复。

我备份快照之后，想新建一个实例，但是好像只能有一个？？~~（全是洋文没仔细读）~~ 随后就 `Destroy` 了现在的，去建了个新的VPS，然后恢复。之后改了DNS A记录，主页能够正常访问，一切似乎进展顺利...

# Root密码忘记了

为了安全性，我一直用的是默认生成的很复杂的一串 Root密码，但是在恢复Snapshot后 `后台显示的Root密码仍然是创建实例时生成的，但是VPS里面的密码已经恢复为Snapshot里的了`！ 我也没有备份密码，糟了（

去查了相关资料 [1](https://www.vultr.com/docs/boot-into-single-user-mode-reset-root-password) [2](http://vultr.wang/tag/vultr%E4%BF%AE%E6%94%B9root%E5%AF%86%E7%A0%81/) 后，解决了这个问题：

* 进入 [Vultr后台](my.vultr.com)， 选择你的服务器。
* 点击右上角 `View Console` 按钮
* 等待进入登录Console界面，点击右上角发送 `Ctrl+Alt+Delete` 按钮，随后使劲按 `Esc`，直到显示蓝色的 `grub` 界面
* 不要按 `↑或↓等光标键`，保证选择的项目是 `默认的（通常是第一项）`。因为不是默认修改后导致无法启动（Trumeet深受其害，连系统都引导不进，最后通过控制台关机再开才进去的，控制台重启都不行，非常危险！）
* 随后按下 `e` 键，进入编辑
* 找到 `linux /boot/` 开头的一行（我的机器是Debian，`quite`结尾），一直按 `→` 滚动到这行末尾，添加 `  init="/bin/bash"`到末尾。（**注意空格！**）
* 然后按下 `Ctrl+X` 或 `F10` 进入系统。
* 这时，惊异地发现，进入Root终端了！！
* 然后输入 `mount -rw -o remount /` 挂载分区，否则后面会无法修改密码。
* 最后，输入 `passwd` 设定 Root 密码吧，一定牢记哦~ （推荐安装 `sudo`，Debian默认不带）



# 无法启动MySQL

我这方面真心渣，所以一定说SQL问题一下子吓到了，我的SQL、PHP、Nginx都是用lnmp装的，一点不懂..

根据 `https://iwww.me/240.html` 的解决方案，我将 `/etc/mysql/my.conf ` 删掉后再重启就解决了，这方面不懂，也就不多阐述了（