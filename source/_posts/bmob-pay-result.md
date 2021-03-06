---
title: Bmob SDK导致应用出现广告始末
date: 2017-06-07
categories: 大新闻
thumbnail: https://ooo.0o0.ooo/2017/05/29/592bf381a33b6.png
---

# 最初的事件

相信大家已经阅读了这篇文章：[关于 Bmob SDK导致应用出现后台广告的调查](https://hexo.trumeet.top/2017/05/28/bmob-seo/)

事情就是这样发生的。随后我将这篇文章转载到了[知乎](https://zhuanlan.zhihu.com/p/27161542)，[掘金](https://juejin.im/entry/592eb4c50ce4630057aa8cf7/detail)也进行了转载。



# 官方人员的回应

文章发布3天后，Bmob 官方认为这件事严重影响了其声誉，并认为 **他们的SDK并没有相关问题** ，并 **联系我希望查清楚此事**。

首先，他们在文章评论和酷安回复我要求取得联系：

![小小琪是Bmob官方人员](https://ooo.0o0.ooo/2017/06/07/59375ead4882f.png)



# 配合调查

我通过邮件与Bmob进行联系， **配合他们调查，并提供了相关抓包、APK等结果**。 与此同时，Bmob说这也可能是运营商劫持等导致。

![配合调查部分联系截图](https://ooo.0o0.ooo/2017/06/07/59375fe8669b2.png)



# 调查结果

过了几天之后，Bmob公布了调查结果：[http://www.bmob.cn/repository/detail/3690](http://www.bmob.cn/repository/detail/3690) ，总结原因如下：

* 接入的Bmob相关SDK并非正版，而是经过二次打包、写入恶意代码的版本。首先，要确定sdk是从官网而非其它平台下载；其次，也不要直接沿用其它平台(如github、csdn等)的开源项目进行开发；最后，即使是在官方下载sdk时，也要确保下载的文件未经劫持(如路由器劫持、浏览器劫持、甚至是系统劫持等)，这一点需要Bmob在SDK下载页面公布每个官方SDK的md5/sha1，以便开发者下载后进行核验。


* 由于Bmob审核流程的不严谨，可能导致某个版本的SDK中包含恶意广告代码。Bmob将加强内部监督和管理，规范产品上线审核流程；Bmob技术团队将尽量不使用、少使用其它团队开发的库(包括开源)。功能性的部件全部由Bmob开发人员造轮子；上线前除了功能性测试外，还要进行完整的上述过程的检测。


* 广告投放者有意避开了上述测试环境进行广告的投放，例如避开比目公司所在地区进行投放、避开某些机型进行投放、检测到有被抓包可能时停止投放等。上述检测过程已经考虑到了这种可能性，使用VPS全局代理，利用多个国家、国内多个省份的ip进行检测，使用多种机型进行检测等，但并不能完全排除这种可能；为避免广告选择在手机未连接Wifi时投放(一般情况下，手机用自身GPRS上网则很难利用PC进行抓包，利用tcpdump抓包则不难)，事后我们人工抽样了其中约10款应用，完全模拟使用GPRS上网的正常用户来使用应用，其中一半通过观察流量来判断，另一半通过tcpdump检测，均未发现异常，但由于时间有限样本较小，也只能排除一定可能性。


* 广告投放者在看到开发者反馈该类现象后，关停了广告投放的行为。即使行为不存在，代码还是存在的，Bmob工作人员将在空余时间继续对有疑问的App进行代码审查，如果有类似情况的用户也请主动联系官方客服提供帮助。
* 广告投放具有限制或地域性。这种可能性最有可能是通过网络劫持达成，例如应用为了呈现H5页面用到了浏览器，网络劫持可以让浏览器做他们想做的事情。

-----

至此，由于并没有在Bmob那里找到明确结论，他们指出了可能的原因（如劫持等），所以此事暂时了结。

**特别强调：若转载本文，请注明出处。如有异议，欢迎及时联系我。有文笔疏忽指出，望谅解、指出，谢谢支持**