---
title: Lineage OS 红米3 中国电信无法使用 2G/3G 的爬坑记录
date: 2017-05-20 06:44:00
tags: 
- 搞手机
- Lineage
- ROM
- Issues
categories: 搞手机
thumbnail: https://www.lineageos.org/images/logo.png
---
我的手机刷了类CM ROM之后就无法使用 2G（打电话/短信）、3G了，只能用 LTE上网。
-----
设备是 **红米3 高配 双卡**，卡1插电信。
症状：拨打电话/切换2G 显示为无服务

# 解决方案
> 该方案来自贴吧

* 修改 build.prop：将 `ro.telephony.default_network=9,9` 改为 `ro.telephony.default_network=10,9`。如果您没有 `ro.telephony.default_network` 常量，或其当前值不是 `9,9`，请停止操作，说明该方案可能不适用于您的设备。 
* 备份您的数据（除SD卡以外），包含应用备份和分区备份。
* 进入 TWRP，选择 WIPE，然后滑动。这将清除 /data 不清除 /data/media （SD卡），用于恢复出厂设置。
* 重启您的设备，随后即可正常使用通话等功能了！

# 原因
暂时未知，继续爬坑中

# 系统更新后如何操作
在您更新您的 ROM 之后，您需要尽量在重启前再次按照上面的方法修改 `build.prop`。因为更新会覆盖 `build.prop`。不建议您使用 `addon.d` 脚本备份恢复 `build.prop`。
