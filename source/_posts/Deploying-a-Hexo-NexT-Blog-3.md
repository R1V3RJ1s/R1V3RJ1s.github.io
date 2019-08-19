---
title: "利用GitHub Pages搭建个人Blog（三）: 个性化和功能插件配置"
copyright: true
date: 2019-08-15 23:10:17
tags: 
  - Git
  - Hexo
  - NexT
category:
  - Blog
published: true
---

本系列文章的主要目的是介绍本博客（Hexo静态博客框架+NexT主题）是怎么配置，搭建与部署在GitHub Pages上的，以及介绍相关的过程中有哪些可能出现的问题以及其解决方案，希望后来人能够少踩坑。本文是本系列的第三篇，主要内容是介绍本博客NexT主题下一些个性化和功能插件的配置。

<!-- more -->

#### 基本主题，站点配置

#### MathJax

打开自己的<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>并将`math`类的`enable`和`per_page`字段都填为`true`。`engine`填为`mathjax`，然后`cdn`处填为`https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML`即可。

#### LeanCloud阅读计数插件

#### Valine评论插件

#### Google Analytics

读者分析插件我选的是Google Analytics，你需要注册一个Google Analytics的应用，然后再Google Analytics的`管理`面板上的`媒体资源`选项卡的`媒体资源设置`里把`媒体资源名称`改成你自己的博客名，`默认网址`输入为你的博客地址，注意`http`选项要改成`https`。另外`数据视图`选项卡的`数据视图设置`里的`http`选项可能也需要改成`https`。然后回到`媒体资源设置`选项，记录下你的`跟踪 ID`，打开自己的<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>并将`google_analytics`类里的`tracking_id`字段里的值改为自己的`跟踪 ID`号码，格式应该是UA-XXXXXXXXX-X。`localhost_ignored`字段填为`true`。重新部署后打开个人网站主页，点击右键选择`显示网站源代码`（以Chrome浏览器为例），并在源代码页面寻找`gtag`关键字，如果有的话应该就说明插件已经成功开启。

#### 可能遇到的问题

* 开启了mathjax插件但是公式不显示，可能是cdn有问题，常见问题是前缀没有加`https://`，或者是网址过期。只需要把cdn里的地址复制到浏览器中打开看是否有显示就可以判断是否是cdn地址出错。
* 已经开启了Google Analytics插件，并打开个人博客页面，但是Google Analytics实时用户没有显示。可能是浏览器开启了防跟踪插件，比如DuckDuckGo。也可能是默认网址填写错误。比如网站后多加了`/`或者是`http`选项没有改成`https`。Chrome用户可以安装Google Tag Assistant插件进行Debug。

