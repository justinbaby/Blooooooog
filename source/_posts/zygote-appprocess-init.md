---
title: Zygote与app_process的启动与作用
tags: 
- Zygote
- app_process
- Android启动
categoris:
- Android开发
thumbnail: https://ooo.0o0.ooo/2017/07/10/59635ec2c1b08.png
date: 2017-07-10 19:02
---

前几天看一篇文章：[http://www.cnblogs.com/samchen2009/p/3294713.html](http://www.cnblogs.com/samchen2009/p/3294713.html)，里面说 **Zygote是由app_process启动的**，然后看 GitYuan 的文章，说 **Zygote是由Init直接启动的**，当时对这两个东西没有太深入研究，困惑 Trumeet

这几天再查了查，搞明白（

> 参考文献：[http://www.judymax.com/archives/1118](http://www.judymax.com/archives/1118)

# 首先 Zygote 是什么

~~写在前面，辣鸡博客纯属个人理解，可能有误，希望不要误导到想我这样的萌新，也更希望dalao指正~~

许多文章中都写得很明白了，Zygote是Android应用、SystemServer的孵化池。

但是，它是一个类似 app_process 这样的二进制程序吗？Trumeet 觉得不是

**而是由app_process启动的Java程序，负责注册Socket、fork、初始化SystemServer等操作**。

 至于为什么叫 Zygote，只不过是名字罢了。

`/init.zygote32.rc`

```shell
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
    class main
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    writepid /dev/cpuset/foreground/tasks
```

可以看到，`service zygote ...` 是指使用 /system/bin/app_process -Xzygote..... 创建一个名叫 Zygote 的进程。`-Xzygote是告诉app_pcocess启动Zygote（定位到ZygoteInit）`



# 然后 app_process 是什么

该打 Google，这个话题查了好久都没有找到相关明确说明 ><

同样跟着 Trumeet 思路的话，这个可以被理解为 **Android上的JVM**，就是用来跑 **.class .jar** 用的，它们在 Android 叫 **.dex**，拆包APK看一下里面都有这个文件（

如果是这样 就和前面连上了，**app_process是任何Java字节码的载体**，**Zygote是一个普通的Java程序，由init调用app_process启动，最后把自己改名 Zygote 跑掉了** （