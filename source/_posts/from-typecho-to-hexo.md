---
title: 从 Typecho 到 Hexo
date: 2017-05-29 16:42
categories: 瞎折腾
thumbnail: https://ooo.0o0.ooo/2017/05/29/592bf4db4ae24.png
---

今天我把博客迁移到了Hexo，原博客不会被删除，但也不会更新。新的地址： [https://hexo.trumeet.top](https://hexo.trumeet.top)

# 备份数据

首先，使用压缩文件的方式备份站点网站目录，再用 PhpMyAdmin 备份数据库，这里不多说

# 安装 Hexo

安装起来比较简单，但我也踩了一些坑：

## Permission Denied

我按照官方文档，在 `root` 用户安装，到最后一步却提示 `Permission Denied` ？！ 我无奈建了一个用户，安装成功了。



# 安装 Material 主题

安装我一直用的 [Material主题](https://material.viosey.com/)，按照官方文档安装，~~踩坑~~：

## Unhandled rejection TypeError

到最后一步 `hexo g` 的时候出的错，对这方面一点不懂的我一脸懵逼（

到最后发现是没有设定语言的锅，在主题 `_config.yml` 中按照文档设定语言即可



# 迁移文章

这个我也找了一些脚本，都~~不会用~~，就自己一点点迁移过去的，反正 Typecho 和 Hexo 都资瓷Markdown嘛



#  

没遇到更多坑了，这些坑已经够受了（XDDD

