---
title: 绿色守护处方
date: 2017-02-02 13:50:00
tags: Android开发
---
这段时间，绿色守护推出了一个功能名叫 `处方`，简而言之，就是依靠用户自建的规则来阻止广播。
昨天，我听到消息说可以自己写处方，就兴冲冲地去做了。
我前往了**[绿色守护官方页面](greenify.github.io)**看了说明后，大概理解了一下，也就是这个样子:
* 处方必须以**`rx-`** 开头（rxs我还没了解）
* 发布处方必须以`https://greenify.github.io/<用户名>/<代码库名>`作为URL，在手机上打开后可以被跳转到绿色守护进行导入。
* 处方内容以XML方式提供
* 所谓“社区”并不是大家所熟知的论坛，而是用户通过`GitHub版本库`自己创建的处方，自己进行发布。处方传播则依赖论坛（比如说被钦点的`酷安`[滑稽]）
## 开始创建处方
话不多说，先创建版本库。我创建一个`QQ`的广播控制，所以我起名叫 `rx-qq`。
然后在项目内创建一个名为 `rx-qq.xml`，用于处方内容。
在`rx-qq.xml`填入以下内容：**这只是示例，请不要用于真实项目**
```xml
<prescription xmlns="http://greenify.github.io/schemas/prescription/v1" type="service">
  <intent-filter>
    <action name="com.weiyun.plugin.BROADCAST" />
    (这里可以往下写更多的)
  </intent-filter>
</prescription>
```
上方`<action name="com.weiyun.plugin.BROADCAST" />`代表名为 `com.weiyun.plugin.BROADCAST`的广播会被绿色守护拦截。可以在下方写下更多的`action`标签。
```xml
<prescription xmlns="http://greenify.github.io/schemas/prescription/v1" type="service">
  <intent-filter>
    <action name="com.weiyun.plugin.BROADCAST" />
    <action name="com.tencent.mobileqq.ACTION_PLUGIN_CRASH" />
    <action name="com.tencent.mobileqq.ACTION_PLUGIN_STARTUP_FAILED" />
    <action name="com.tencent.mobileqq.ACTION_PLUGIN_DIR_INFO_LOG" />
    <action name="com.tencent.mobileqq.rdm.report" />
  </intent-filter>
</prescription>
```
提交保存。接下来使用链接导入至绿色守护：`https://greenify.github.io/liangyuteng0927/rx-qq` **请将这里的用户名和项目名称替换成你自己的**
用手机打开链接，将会被跳转到绿色守护进行导入。