---
title: app_process与UID
date: 2017-07-10 19:04
tags:
- app_process
categories:
- Android开发
---

app-process 与 内部运行的Java 程序的UID是如何决定的 呢..

读不懂 C，Trumeet 需要去实践一下（



# Root

使用 Root 执行 app_process 可行，可以去看一下 Shizuku、黑域，它们是可以通过 Root 启动的（

启动后，通过 `ps` 查看UID：

```shell
...
root      4805  1     2799728 27316            0000000000 S rikka_server
...
```

可以看出是和Root一个UID

# ADB

和上面一样嘛..

```shell
...
shell     31552 1     2798140 40736          0 7fb01ba0b4 S rikka_server
...
```

和 Shell 是一个UID

# 普通应用

使用手机上的终端启动一下 Shizuku..

```shell
...
u0_a141   4916  1     65680  14856            0000000000 R rikka_server
...
```

和那个应用UID是一样的（



也就是说 app_pocess 内的Java程序是和启动者的UID一致，而不是只允许Shell/Root调用app_process（

普通应用启动，UID也是普通应用的，想获取一些系统权限照样是不行的，保障了安全（