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

在这里我只会写我自己调整过的设置，更多主题方面的设置可以参考[NexT主题曾经的官方网站](https://theme-next.iissnan.com/)和[NexT现在的官方网站](https://theme-next.org/)。

#### 主题风格配置

NexT提供四个不同的风格可选，在<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>的`Schemes`类下将你想开启的风格前的`#`号删除然后把原来的风格前的#号加上即可。比如开启`Mist`风格：

{% codeblock lang:sh %}
# Schemes
#scheme: Muse
scheme: Mist
#scheme: Pisces
#scheme: Gemini
{% endcodeblock %}

注意删除`#`号时不可将空格一起删除，不然会出现错误。

#### 基本个人信息和站点信息配置

由于在本系列文章的[第二篇](https://r1v3rj1s.github.io/2019/08/15/Deploying-a-Hexo-NexT-Blog-2/)中我们已经设置站点主题为`NexT`，所以对于<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>而言，现在只需要填写`title`（标题），`description`（简介或座右铭），`author`（作者），`language`（语言），`url`（网站链接）字段。前三个字段可以随心所欲，`language`的话英文用户填`en`，简体中文用户填写`zh-CN`，繁体中文用户填写`zh-TW`或`zh-hk`。`url`字段则直接复制个人主页网址地址即可，比如本博客链接的`url`就是`https://r1v3rj1s.github.io`。
然后我们配置头像和媒体信息。找到一张你想用的头像图片（不要太大，像素500x500以下，.gif最好，其他常见图片格式也可）并存于个人博客源代码根目录/source/image文件夹下（若source文件夹下不存在image文件夹那么手动创建一个即可）。然后打开<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>并找到`avatar`类，然后在`url`字段中复制刚刚下载的头像图片地址即可（比如`r1v3rj1s.github.io/source/image/avatar.gif`）。接着找到`social`类，然后在你想添加的媒体信息前删除#号并将后面的链接修改为你自己的媒体账号地址即可。

#### 开启标签和分类

1. 打开<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>，将标签（tags）和分类（catagories）前面的`#`号去掉：
{% codeblock lang:sh %}
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
{% endcodeblock %}
2. 然后在个人网站源代码仓库根目录下输入以下命令：
{% codeblock lang:sh %}
$ hexo new page "tags"
$ hexo new page "catagories"
{% endcodeblock %}
3. 去`source`文件夹确认是否生成了`tags`和`categories`子文件夹，并且子文件夹中各自有`index.md`文件
4. 编辑`tags`子文件夹的`index.md`文件的YAML标签信息，内容如下：
{% codeblock lang:sh %}
---
title: Tags
type: "tags"
layout: "tags"
comments: false
---
{% endcodeblock %}
5. 编辑`categories`子文件夹的`index.md`文件的YAML标签信息，内容如下：
{% codeblock lang:sh %}
---
title: Categories
type: "categories"
layout: "categories"
comments: false
---
{% endcodeblock %}
重新部署问题即可解决。

#### 版权信息

找到`creative_commons`类，做如下设置即可开启版权尾注：

{% codeblock lang:sh %}
creative_commons:
  license: by-nc-sa
  sidebar: false
  post: true
  language: zh-CN
{% endcodeblock %}

之后写博文的时候如果需要添加版权尾注在YAML标签添加`copyright: true`字段即可。[这篇文章](https://segmentfault.com/a/1190000009544924)提供了更多自定义版权的方式。

#### MathJax

打开自己的<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>并将`math`类的`enable`和`per_page`字段都填为`true`。`engine`填为`mathjax`，然后`cdn`处填为`https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML`即可。`per_page`字段为`true`的话若要开启MathJax模式则所有博文YAML标签必须有`mathjax: true`字段。

#### LeanCloud阅读计数插件

接着我们想要做的是开启

#### Valine评论插件

和LeanCloud阅读计数插件类似，你需要在LeanCloud主面板中再创建一个`开发版`的应用，然后在`设置`面板的`安全域名`选项处的`Web 安全域名`栏目下填写自己的个人主页地址。协议和域名需要完全一致。注意不要把`https`打成`http`。保存后点击`应用 Key`选项，记录下你的`App ID`和`App Key`，打开<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>，在`valine`类的`enable`字段填为`true`，`appid`和`appkey`字段填你刚才记录下的`App ID`和`App Key`。然后`visitor`字段填`false`，不然在有阅读计数插件的环境下可能会导致bug。然后就可以重新部署源代码测试是否成功开启。

#### Google Analytics

读者分析插件我选的是Google Analytics，你需要注册一个Google Analytics的应用，然后再Google Analytics的`管理`面板上的`媒体资源`选项卡的`媒体资源设置`里把`媒体资源名称`改成你自己的博客名，`默认网址`输入为你的博客地址，注意`http`选项要改成`https`。另外`数据视图`选项卡的`数据视图设置`里的`http`选项可能也需要改成`https`。然后回到`媒体资源设置`选项，记录下你的`跟踪 ID`，打开自己的<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>并将`google_analytics`类里的`tracking_id`字段里的值改为自己的`跟踪 ID`号码，格式应该是UA-XXXXXXXXX-X。`localhost_ignored`字段填为`true`。重新部署后打开个人网站主页，点击右键选择`显示网站源代码`（以Chrome浏览器为例），并在源代码页面寻找`gtag`关键字，如果有的话应该就说明插件已经成功开启。

#### 可能遇到的问题

* `<!-- more -->`不能起到分割的作用，你需要检查你的分割字段写的是`<!-- more -->`（more左右各有一个空格）还是`<!--more-->`（more左右无空格）。
* 开启了mathjax插件但是公式不显示，可能是cdn有问题，常见问题是前缀没有加`https://`，或者是网址过期。只需要把cdn里的地址复制到浏览器中打开看是否有显示就可以判断是否是cdn地址出错。
* 已经开启了Google Analytics插件，并打开个人博客页面，但是Google Analytics实时用户没有显示。可能是浏览器开启了防跟踪插件，比如DuckDuckGo。也可能是默认网址填写错误。比如网站后多加了`/`或者是`http`选项没有改成`https`。Chrome用户可以安装Google Tag Assistant插件进行Debug。
* 对于`.yml`文件而言，处于同级的字段缩进量需要完全一致。<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>和<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>都用双空格作为缩进，所以当有无法理解的问题发生时检查一下缩进量也许会有意想不到的事情发生。
