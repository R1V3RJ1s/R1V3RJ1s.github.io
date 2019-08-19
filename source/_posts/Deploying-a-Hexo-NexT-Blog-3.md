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

打开自己的<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>并将`math`类的`enable`和`per_page`字段都填为`true`。`engine`填为`mathjax`，然后`cdn`处填为`https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML`即可。`per_page`字段为`true`的话若要开启MathJax模式则所有博文YAML头必须有`mathjax: true`字段。

#### LeanCloud阅读计数插件

#### Valine评论插件

和LeanCloud阅读计数插件类似，你需要在LeanCloud主面板中再创建一个`开发版`的应用，然后在`设置`面板的`安全域名`选项处的`Web 安全域名`栏目下填写自己的个人主页地址。协议和域名需要完全一致。注意不要把`https`打成`http`。保存后点击`应用 Key`选项，记录下你的`App ID`和`App Key`，打开<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>，在`valine`类的`enable`字段填为`true`，`appid`和`appkey`字段填你刚才记录下的`App ID`和`App Key`。然后`visitor`字段填`false`，不然在有阅读计数插件的环境下可能会导致bug。然后就可以重新部署源代码测试是否成功开启。

#### Google Analytics

读者分析插件我选的是Google Analytics，你需要注册一个Google Analytics的应用，然后再Google Analytics的`管理`面板上的`媒体资源`选项卡的`媒体资源设置`里把`媒体资源名称`改成你自己的博客名，`默认网址`输入为你的博客地址，注意`http`选项要改成`https`。另外`数据视图`选项卡的`数据视图设置`里的`http`选项可能也需要改成`https`。然后回到`媒体资源设置`选项，记录下你的`跟踪 ID`，打开自己的<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>并将`google_analytics`类里的`tracking_id`字段里的值改为自己的`跟踪 ID`号码，格式应该是UA-XXXXXXXXX-X。`localhost_ignored`字段填为`true`。重新部署后打开个人网站主页，点击右键选择`显示网站源代码`（以Chrome浏览器为例），并在源代码页面寻找`gtag`关键字，如果有的话应该就说明插件已经成功开启。

#### 可能遇到的问题

* <!-- more -->不能起到分割的作用，你需要检查你的分割字段写的是<!-- more -->（more左右各有一个空格）还是<!--more-->（more左右无空格）
* 开启了mathjax插件但是公式不显示，可能是cdn有问题，常见问题是前缀没有加`https://`，或者是网址过期。只需要把cdn里的地址复制到浏览器中打开看是否有显示就可以判断是否是cdn地址出错。
* 已经开启了Google Analytics插件，并打开个人博客页面，但是Google Analytics实时用户没有显示。可能是浏览器开启了防跟踪插件，比如DuckDuckGo。也可能是默认网址填写错误。比如网站后多加了`/`或者是`http`选项没有改成`https`。Chrome用户可以安装Google Tag Assistant插件进行Debug。
* 对于`.yml`文件而言，处于同级的字段缩进量需要完全一致。<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>和<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>都用双空格作为缩进，所以当有无法理解的问题发生时检查一下缩进量也许会有意想不到的事情发生。
