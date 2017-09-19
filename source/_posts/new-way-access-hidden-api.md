---
title: 访问隐藏 API 的另一条路
date: 2017-09-12 16:50
categories: Android开发
tags: 
  - 隐藏 API
  - Android SDK
---
# 前言
现在，博主使用 hidden-api 有这几种途径：
* 反射
* 编译好的 framework jar （如 GitHub android-hidden-api 项目）
  这里面介绍一下最近学到的另一种方式。

# Android SDK
先观察一下（反编译）Android SDK（android.jar），我们可以看到 **所有的 构造器、方法** 的实现均被替换成了`throw new RuntimeException ("Stub!");` 的 “有名无实” 实现，而且在运行的时候87并没有出现 “Stub” Crash。这是怎么回事呢——

与设备上同名的方法和构造器（如 `android.os.Environment#getExternalStorage` 不知道拼写对不对 逃跑），会在真机运行时被 **替换为本机实现**，所以也就 “正常地调用了”。

有一次我下载个一加Launcher，打开之后 Crash 了，Message就是 Stub，也就说明它调用了一加ROM的一些 API。这些接口在我的原生上面不存在，所以就不会替换为本地实现，从而触发 Stub。

## 那么 SDK 是做什么用的
SDK 只是用于 **通过编译**，而且 framework jar 是使用 **provided** 方式引用的，不会被打包进 apk

## 为什么非要 Stub
这个是 [Doclava](http://androidxref.com/7.1.1_r6/xref/external/doclava/src/com/google/doclava/) 产生的..

当然，丢出异常我自己是认为 **这样就无需返回值了**，当然，理论上写其他实现也会被替换掉。

# 调用隐藏 API
言归正传，我们说说隐藏 API 的事。

## API 是 什么时候被隐藏的
众所周知，javadoc 标注了 `@hide` 的方法、类、构造器、域 等就是被隐藏的，API 无法访问，但真机上有。

但是，Android 代码中是有这些东西的，为什么开发时找不到了呢..

编译 SDK 的时候，依照上文的介绍，所有方法之类的都会被填充 Stub ，生成 android.jar， **这时，如果 hide，就不会被打包进去**，也就无法访问了。（尚未仔细研究，个人推测分析）与其说是 “无法访问”，不如说是 “没有添加进 SDK，不存在”。

所以，如果在生成 Stub 的时候不管这些可恶的 hide，也就会照常生成这些隐藏 API。

## 模仿 SDK 制作 Stub
到了本文的核心部分。既然我们知道了 Android SDK 以及 hidden-api 的原理，我们就可以按需将隐藏的 API 添加到项目。

比如说，我需要访问 `android.os.Environment#UserEnvironment` 内部类，我们只需要做如下几步：

* 创建 `android.os` 包，用于覆盖 SDK
* 把 SDK 中的 `Environment` 复制进去
* 在 `Environment` 中添加 `UserEnvironment` 内部类，同时按照 AOSP 源码添加方法签名，实现随便抛出异常。

之后，编码访问 `UserEnvironment`，辣鸡鸡 idea 还会标红，这是因为它优先访问 SDK。但是实际编译中，**自建的 Environment 类会覆盖 SDK 中的类**，从而通过编译。

正是由于 **自建的 Environment 类会覆盖 SDK 中的类**，我们得以 “自己做 SDK ”  来调用这些 API，细看 黑域、Shkzuku、condom 都是这么做的。



# 文末福利

提供这两天编译 Android O 顺手（专门）弄的 带有隐藏 API 的 android.jar 一份~

> 参考： [http://www.wxtlife.com/2015/03/31/how-to-use-android-hide-methods-or-class/](http://www.wxtlife.com/2015/03/31/how-to-use-android-hide-methods-or-class/)

**不带例如 Telephony、Services 这些与 Framework 分开的 jar**

地址：[点这儿](https://github.com/Trumeet/android-hidden-api/blob/master/android-26/android.jar)

随手给 [android-hidden-api](https://github.com/anggrayudi/android-hidden-api) 提交了 [PR](http://www.wxtlife.com/2015/03/31/how-to-use-android-hide-methods-or-class/)

