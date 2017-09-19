---
title: 使用app_process提权
date: 2017-05-05 16:05:00
tags: Android开发
---
>要想获取到系统权限，我们可以考虑以下几种方式：
* 使用系统插件（把APK放到/system/priv-app或/system/app）
这可以拿到部分权限，如安装和卸载软件、冻结软件、强行停止等，实现部分功能。（参见 **冰箱系统插件**、**AppOps系统插件** 等）
* Xposed（可能可行）
* Root + 终端（如，需要定期强行停止一个软件，可以使用 ``am for-stop`` 代替 `ActivityManager.forceStopPackage(String)` ，但是 **速度较慢**
* 直接用 Root 调用 `app_process` ，使用打包好的 `dex` 来调用系统API，可以拿到所有权限，没有签名验证（`am`等命令就是这样实现的）
* 造个ROM（大雾

>插：为什么要用到系统权限呢？
>一些系统工具，需要使用诸如强行停止、写入系统设置、冻结软件等操作，则需要系统权限。

# app_process 是什么
> 参考：[http://www.cnblogs.com/samchen2009/p/3294713.html](http://www.cnblogs.com/samchen2009/p/3294713.html)

`app_process` 是 **`启动zygote和其他Java程序的应用程序`**，zygote是由它启动的，其他Java程序（在Android上是`.dex`）也如此。

# am
要理解 `app_process` 怎么用，我们举个最简单的例子： **[am](https://github.com/android/platform_frameworks_base/blob/master/cmds/am/am)**

我们一定经常使用 `am` 命令，它其实是一个 Shell脚本，代码很简单，就是 **`通过app_process启动am命令行Java程序并传递参数`**

>附上[am命令行Java程序](https://github.com/android/platform_frameworks_base/blob/master/cmds/am/src/com/android/commands/am/Am.java)代码，很简单的Java程序而已，解析参数，执行操作..

# 一个结束进程的 `Kill.java`
我们通过一个示例，展现如何通过 `app_process` 提权。

写一个 Kill.java，它会在启动后 `强行停止酷安（com.coolapk.market）`。
一般用于程序是无法调用 `forceStopPackage()` 的，需要 `FORCE_STOP_PACKAGE` 权限，它是系统权限。

模仿 [am命令行Java程序](https://github.com/android/platform_frameworks_base/blob/master/cmds/am/src/com/android/commands/am/Am.java)，使用简单代码结束进程：
```Java
import android.app.*;
import android.os.*;
public class Kill {
	public static void main (String[] args) {
		System.out.println("Start");
		IActivityManager am = ActivityManagerNative.getDefault();
		try {
			am.forceStopPackage("com.coolapk.market", UserHandle.USER_ALL);	
			System.out.println("Success");
		} catch (RemoteException e) {
			e.printStackTrace();
		}
	}
}
```
用`notepad++`写即可。

## 编译
我们现在需要把它编译成`.class`。
由于使用了 `Android API`，需要添加 `framework.jar` 进classpath才能通过编译，我这里用了前一段编译ROM产生的 `framework.jar`。

用Javac：`javac -source 1.7 -target 1.7 -classpath .\framework.jar .\Kill.java`，就完事了。
**注意：这里需要设置Java版本，否则后面dx会提示不支持：`-source 1.7 -target 1.7`**

# 把`.class`转换为`.dex`
先说一下为什么要转换。
>参考：[http://blog.csdn.net/u010651541/article/details/53163542](http://blog.csdn.net/u010651541/article/details/53163542)

`Android对Java虚拟机做了修改，即使用自己的dalvik虚拟机(后来的ART)。因此，.class的字节码在Dalvik虚拟机上是不能运行的`

我们需要使用 `dx` 工具进行转换，它存在于你的 `Android SDK/build-tools/SDK版本/` 中。Windows平台是 `dx.bat`。

进入编译完的目录，执行命令行进行转换：
`dx.bat --dex --output="编译保存目录\Kill.dex" Kill.class`

完成后，会生成 `Kill.dex`。

# 发到手机
这个没什么说的，`adb push Kill.dex /data/local/tmp`

# 运行
1. 进终端，adb shell
2. 模仿 `am` 那个Shell脚本，首先我们需要声明 CLASSPATH 环境变量，否则后面执行会出现 `Aborted`
`export CLASSPATH=/data/local/tmp/Kill.dex`
3. 打开酷安，执行：`app_process /data/local/tmp/Kill.dex Kill` ，随着 `Start`，`Success`，酷安就被强行停止了！

![](https://ooo.0o0.ooo/2017/05/05/590c29039284c.png)

附：只能使用Shelle或者Root用户强行停止了，其他会被拒绝（废话..
![](https://ooo.0o0.ooo/2017/05/05/590c2903b20cd.jpg)

~~这方面菜鸟，dalao勿喷~~