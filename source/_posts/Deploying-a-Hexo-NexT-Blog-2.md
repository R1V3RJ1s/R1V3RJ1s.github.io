---
title: "用Hexo在GitHub Pages上搭建个人Blog（二）: 前置要求和初始化配置"
copyright: true
date: 2019-08-15 12:17:21
tags: 
  - Git
  - Hexo
  - NexT
category:
  - Blog
published: true
---

本系列文章的主要目的是介绍本博客（Hexo静态博客框架+NexT主题）是怎么配置，搭建与部署在GitHub Pages上的，以及介绍相关的过程中有哪些可能出现的问题以及其解决方案，希望后来人能够少踩坑。本文是本系列的第二篇，主要内容是介绍本博客的前置要求和初始化配置。

<!-- more -->

#### 配置Git 

此处我以最基本的HTTPS操作为例，想要追求更高安全性的朋友可以采取SSH操作模式，相关教程很多，此不赘述。
1. 注册一个GitHub账号
2. 点选GitHub网站右上角的`+`号选项卡里的`New Repository`选项，创建一个名为`本人GitHub用户名.github.io`的新仓库，此名即为你的GitHub Pages的链接地址。注意该仓库一定要选择为公开(`Public`)，不然个人页面会显示404错误。`Initialize this repository with a README`选项打钩，`Add .gitignore`和`Add a license`用默认的`None`即可。举例：我的GitHub用户名是R1V3RJ1S,我的仓库名即为`R1V3RJ1S.github.io`
3. 在本地终端中设置你的git个人信息
{% codeblock lang:sh %}
# 注意双引号是必须的
$ git config --global user.name "GitHub用户名"
$ git config --global user.email "GitHub注册邮箱"
# 可以通过 git config --get 来验证我们的个人新设置是否正确,注意此处返回值则无双引号
$ git config --get user.name # GitHub用户名
$ git config --get user.email # GitHub注册邮箱
{% endcodeblock %}
4. 由于Hexo存在将源文件额外渲染成HTML网页文件的步骤，而GitHub Pages在使用默认地址的情况下只接受来自部署在master分支上的页面，因此此处有三个解决方案可选：
    * GitHub上的个人主页仓库只保留一个master分支用于存储渲染后的网页文件，Hexo生成的源文件，NexT主题文件，以及博客原稿等另存在本地。此方案优点在于配置简单，分支管理容易，而缺点在于源文件缺乏版本控制系统的管理，而由于修改的又主要都各式源文件，一旦出错，在缺乏版本控制系统帮助的情况下撤销原操作会是一个很麻烦的问题。而且这个方案做不到利用持续集成/持续部署（CI/CD）工具进行自动部署。
    * GitHub上创建两个仓库，个人主页仓库用于存储渲染后的网页文件，个人主页源码仓库用于存储各式源文件，两个仓库都只存有master分支。这样本地仓库文件夹分别存网页文件和源文件。此方案优点同样在于配置简单，而且分支管理容易，还可以做到利用CI/CD工具进行自动部署。此方案的缺点在于要多创建一个仓库，两个仓库都要管理，不仅麻烦，还容易混淆。
    * GitHub上的个人主页仓库创建两个分支：master分支用于存储渲染后的网页文件，pg-source分支用于同步存储各式源文件。本地仓库文件夹中储存且只储存Hexo生成的源文件，NexT主题文件，以及博客原稿等，不存储渲染后的网页文件。此方案优点在于只存在一个仓库，虽然有两个分支，但是实际操作上如果配置得当，基本只用在pg-source分支上进行操作，从而将混淆分支，造成源码丢失或者污染的的可能性降至最低。并且这个方案也可以做到利用CI/CD工具进行自动部署。而此方案缺点在于一旦分支混淆造成源码丢失，整个代码提交改动历史都有可能丢失（此问题会在后续文章提到，并提出解决方案），造成操作回撤困难。本人最后选择了这个方案，因为在利用CI/CD工具进行自动部署的情况下，此方案是最高效的。因此接下来我会用**此方案**作为例子。
5. 点选个人主页仓库页面仓库内文件信息右上`Clone or download`选项，复制内含链接后在本地终端运行`git clone`命令将仓库内容复制到本地，此时本地会得到一个以个人网站地址为名的文件夹。进入此文件夹，创建，进入并上传pg-source分支。
{% codeblock lang:sh %}
# 此链接格式应为https://github.com/本人GitHub用户名/GitHub个人博客地址.git
$ git clone 刚复制的链接
$ cd 个人主页仓库文件夹 # 进入刚才下载的文件夹
$ git checkout -b pg-source # 创建并进入pg-source分支
$ git branch # 确认我们现在在pg-source分支，如果运行正确你会看到两行信息，分别是master和pg-source，其中pg-source前有星号（* pg-source），代表我们现在处在pg-source分支
$ git push --set-upstream origin pg-source # 确认并上传pg-source分支
{% endcodeblock %}
刷新你的GitHub个人主页仓库页面，在仓库内文件信息左上Branch选项卡内应该此时能看到多了一个pg-source分支。

#### 初始化Hexo

1. 本地安装`Node.js`，去`Node.js`的[官方网站](https://nodejs.org)下载安装即可，但如果你喜欢的话，用`Homebrew`安装也行。
2. 终端中运行`node -v`和`npm -v`以测试Node.js和npm都已经成功安装
{% codeblock lang:sh %}
$ node -v # v10.16.1
$ npm -v # 6.10.3
{% endcodeblock %}
3. 安装Hexo命令行，创建Hexo框架源文件包，并将源文件移至本地个人主页仓库文件夹
{% codeblock lang:sh %}
$ npm install hexo-cli -g # 安装Hexo命令行
$ hexo init blog_source # 创建Hexo框架源文件包
$ mv blog_source/* 个人主页仓库文件夹 # 将源文件移至本地个人主页仓库文件夹
$ rm -rf blog_source 
{% endcodeblock %}
4. 在本地以调试模式尝试运行Hexo，如果成功的话在浏览器内输入`http://localhost:4000`应该会出现Hexo Landscape主题的博客页面
{% codeblock lang:sh %}
$ cd 个人主页仓库文件夹
$ npm install
$ hexo server --debug # 如果成功出现页面，即可按ctrl+C退出
$ hexo clean # 删除临时信息
{% endcodeblock %}

#### 初始化NexT

1. 下载NexT主题文件至主题文件夹内
{% codeblock lang:sh %}
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
{% endcodeblock %}
2. 由于我们相当于是在一个Git仓库中又拉取了一个Git仓库，因此如果此时上传我们的本地文件到远程仓库服务器会提醒你要不要把NexT主题文件夹作为submodule上传进行管理，但由于submodule本身管理比较麻烦，外加到时候做自动部署的时候还需要额外步骤，增加部署失败概率，所以我直接将NexT文件夹里所有的Git信息都删除了。这样做的优点就是可以将其和我们自己的个人网站作为一个整体进行管理，减轻负担，但缺点是到时候NexT升级的时候需要重新下载和覆盖。不过好在NexT才更了一个大版本，所以这个应该暂时不会成为一个问题。
{% codeblock lang:sh %}
$ cd themes/next
$ rm -rf .git* # 移除Git相关信息
{% endcodeblock %}

#### 初始配置文件

现在我们基本的前置软件都已经安装好了。下一步便是变更初始配置以使得我们的Hexo博客可以用NexT主题，并将之部署在我们的GitHub Pages上。如同上一篇文章所提到，我们的网站存在两类配置文件，<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>和<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>。这两个文件都以`_config.yml`为名。其中，<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>在站点根目录下（`个人主页仓库文件夹/_config.yml`），<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>则在相应主题根目录下(比如NexT的<span style="background-color:#c082ed"><font color="white">&nbsp;主题配置文件&nbsp;</font></span>地址应该是`个人主页仓库文件夹/themes/next/_config.yml`)。因此我们需要如下操作：
1. 更改<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>将默认的主题文件指向NexT。
    1.1. 打开<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>。
    1.2. 找到`Extensions`类里的`Themes`子类，将底下的`theme: 默认主题` 改为`theme: next`，保存并退出。
    1.3. 如果成功的话，此时在终端输入`hexo server --debug`进入调试模式并打开`http://localhost:4000`应该已经能显示NexT主题的博客页面了。
2. 在<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>内添加部署信息，让hexo能部署在我们的GitHub Pages上
    2.1. 打开<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>。
    2.2. 找到`Deployment`类的，在`deploy:`底下添加如下信息：
    {% codeblock lang:sh %}
    deploy:
    -
      type: git
      repo: https://github.com/本人GitHub用户名/GitHub个人博客地址.git
      branch: master
      message: "一些信息"
    {% endcodeblock %}
3. 尝试将网页部署至GitHub Pages服务器端
{% codeblock lang:sh %}
# 你的终端此时位置需要在个人主页仓库文件夹中
$ npm install hexo-deployer-git --save
$ npm install
$ hexo g 
$ hexo d
{% endcodeblock %}
如果成功的话，此时进入你的GitHub个人博客地址，应该就能看到和调试模式打开时候一样的页面了。
4. 创建.gitignore文件，将源码提交至GitHub服务器
如果一切顺利，那么恭喜，你的博客已经初步成型了，下一步是创建一个`.gitignore`文件，这个文件会告诉git命令哪些文件在提交代码和上传文件时是可以直接忽略的。
    4.1. 在个人主页仓库文件夹中创建`.gitignore`文件
    {% codeblock lang:sh %}
    # 在.gitignore文件内复制如下内容
    .DS_Store
    .deploy*/
    *.log
    public/
    db.json
    node_modules/
    {% endcodeblock %}
    4.2. 提交代码并上传文件
    {% codeblock lang:sh %}
    # 确认我们现在在pg-source分支
    $ git branch # * pg-source
    $ git add .
    $ git commit -m '一些信息'
    $ git push
    {% endcodeblock %}

#### 可能遇到的问题
* 如果在任何含`npm install`的步骤出现问题，尤其报错`Permission Denied`时可以尝试改成在管理员模式(sudo)下运行该命令。
* 如果部署之后GitHub个人博客也一直报404错误，可以看看是否将`https`打成了`http`，又或者是否没有将仓库设置成公开。