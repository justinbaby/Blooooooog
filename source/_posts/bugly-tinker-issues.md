---
title: Bugly热更新填坑记录
tags:
- Bugly
- 热更新
- Tinker
- Issues
- Android开发
categories: Android开发
thumbnail: https://raw.githubusercontent.com/Tencent/tinker/master/assets/tinker.png
---

这段时间用Bugly的热更新功能，按照官方文档做了之后，踩了不少坑，记录一下，也方便大家不要跳进去。

#  未匹配到可应用补丁包的App版本，请确认补丁包的基线版本是否已发布

这个确认自己配置对了之后，**仔细读文档**，需要上传 **build/outputs/patch** 里的补丁包，而不是 **build/outputs/tinkerpatch** 里面的！！

# TinkerID和发布多个补丁

这里解释以下Bugly的TinkerID是如何设置的，我们只需要了解如何发布一个全量版本，然后发布补丁，然后再次发布补丁，然后再发布全量版本...

Tinker ID 必须全局唯一，不管是两个版本之间，两个补丁之间还是补丁和版本之间。

## 全量版本

如我们的App要发布 `2.0` 版，versionCode是`2`。

* Tinker ID 需要设置为 `2-base`。 2 是 versionCode，这是为了和以前的全量版进行区分。后面的 base 是指这是一个全量版。
* 然后编译。找到 `bakApk` 目录名，填写到gradle配置中。
* 正常发布版本。

## 发布第一个补丁

我们在发布 `2.0` 之后，需要发布一个补丁来修复Bug。

* 正常修改代码
* Tinker ID 改为 `2-patch-0` ，2 还是 versionCode，`patch` 为了和全量版区分，`0` 指的是第0个补丁。（这个Tinker ID只要全局唯一即可，你也可以用git commit号之类的，都行）
* 执行 `tinker-support`里面的编译任务，**`tinker`里面的执行完后不会在 `build/outputs/patch` 生成补丁！！**
* 打开后台，正常发布补丁。

## 发布第二个补丁

后来又有一个Bug需要用补丁修复，我们还需要发布一个补丁。

* 正常修改代码
* Tinker ID 改为 `2-patch-1`。 **不发布新的全量版，`bakApk目录`始终保持不变。**
* 同上进行发布即可。

-----

更多的坑，还在慢慢爬..