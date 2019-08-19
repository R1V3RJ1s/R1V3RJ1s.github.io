---
title: "利用GitHub Pages搭建个人Blog（四）: 利用Travis CI进行自动部署"
copyright: true
date: 2019-08-15 23:30:39
tags: 
  - Git
  - Hexo
  - NexT
category:
  - Blog
published: true
---

本系列文章的主要目的是介绍本博客（Hexo静态博客框架+NexT主题）是怎么配置，搭建与部署在GitHub Pages上的，以及介绍相关的过程中有哪些可能出现的问题以及其解决方案，希望后来人能够少踩坑。本文是本系列的第四篇，主要内容是介绍本博客如何利用Travis CI进行自动部署。

<!-- more -->

到现在为止我们对博客源码（比如站点设置，主题设置，博文等）的每次更改都需要我们手动部署到GitHub Pages上。重复的次数多了就显得很麻烦，出错的几率也会变大。一旦分支混淆，还会造成远程源码丢失的问题。不过好在现在支持持续集成/持续部署（CI/CD）的工具很多，它们能够让我们把这个过程自动化。我选择的是[Travis CI](https://travis-ci.org/)，因为它简单好用，而且免费。注册Travis CI和让Travis CI为特定的GitHub仓库开启自动Build的过程非常容易，在此不再赘述。本过程的主要难点在于需要自己编写部署文档。

#### 自动部署的基本过程

#### 加密部署方案

#### commit历史恢复问题解决方案

到现在为止，我们的Travis CI应该已经能够正常执行`.travis.yml`脚本，将我们的源代码渲染为网页文件并将之部署到master分支和GitHub Pages上了。但如果你查看master的commit历史你可能能会发现一个问题：无论我们更新多少次，代码commit历史永远只有两条。这就很麻烦了。一方面来说这个commit的历史也是博客构建的历史，如果永远只显示最新的一次，总归是显得有些失落，因为过去的积累都显示不出来；另一方面而言commit历史丢失也可能造成潜在的问题，比如分支混淆造成源码丢失时，这个问题会导致整个代码提交改动历史也都随之丢失，甚至整个博客也会毁于一旦。因此解决这个问题就显得颇为重要了。经过查询，这个问题的原因在于`hexo-deploy-git`插件在master分支没有之前deploy生成的`.deploy_git`目录情况下动生成了一个`.deploy_git`文件夹，并将 public 复制到这个目录下再进行push，所以每次master分支都会获得一个新的`.deploy_git`目录，而commit历史又是靠`.deploy_git`来记录的，所以会丢失。这个问题相对比较好解决，只需要在每次构建的时候都`clone`一下`.deploy_git`目录即可。`.travis.yml`中的具体脚本如下：

{% codeblock lang:sh %}
before_install:
    - git clone --branch=master {your_blog_repo_git_url} .deploy_git
{% endcodeblock %}

#### 博文更新时间恢复问题解决方案

除了上面的问题之外，Travis CI的自动部署还会导致另一个问题：所有博文的更新时间总是最后一次部署的时间。这个原因是Hexo内的更改时间显示的是markdown文件在系统中的最后修改时间， 而Travis CI 在构建的时候，总是会重新clone repo，这就造成了所有文件的最后修改时间都是最新的clone时间。解决方案是我们可以采用git的最后commit时间来替代，这样子就能恢复文件的修改时间了。但这个问题网上的解决方案一般是只适用于ASCII字符的文件名，而对于非英文用户稍微有点麻烦，是因为有时候我们会用non-ASCII字符（比如中文）来起文件名。但[这篇博文](https://wafer.li/Hexo/%E8%A7%A3%E5%86%B3%20Travis%20CI%20%E6%80%BB%E6%98%AF%E6%9B%B4%E6%96%B0%E6%97%A7%E5%8D%9A%E5%AE%A2%E7%9A%84%E9%97%AE%E9%A2%98/)给出了当文件名含non-ASCII字符时的解决方案。`.travis.yml`中的具体脚本如下：

{% codeblock lang:sh %}
before_install:
    - "git ls-files -z | while read -d '' path; do touch -d \"$(git log -1 --format=\"@%ct\" \"$path\")\" \"$path\"; done"
{% endcodeblock %}

#### .travis.yml脚本参考

这里给出我最终测试成功的 Travis CI 脚本给大家参考：

{% codeblock lang:sh %}
language: node_js
node_js: stable


before_install:
  # Basic config
  - sed -i "s#${LEANCLOUD_CRITIQUE_TMP_ID}#${LEANCLOUD_CRITIQUE_ID}#g" themes/next/_config.yml
  - sed -i "s#${LEANCLOUD_CRITIQUE_TMP_KEY}#${LEANCLOUD_CRITIQUE_KEY}#g" themes/next/_config.yml
  - sed -i "s#${LEANCLOUD_VISITORCOUNTER_TMP_ID}#${LEANCLOUD_VISITORCOUNTER_ID}#g" themes/next/_config.yml
  - sed -i "s#${LEANCLOUD_VISITORCOUNTER_TMP_KEY}#${LEANCLOUD_VISITORCOUNTER_KEY}#g" themes/next/_config.yml
  - sed -i "s#${LEANCLOUD_VISITORCOUNTER_TMP_ID}#${LEANCLOUD_VISITORCOUNTER_ID}#g" _config.yml
  - sed -i "s#${LEANCLOUD_VISITORCOUNTER_TMP_KEY}#${LEANCLOUD_VISITORCOUNTER_KEY}#g" _config.yml
  - sed -i "s#${LEANCLOUD_COUNTERSECURE_USER_TMP_ID}#${LEANCLOUD_COUNTERSECURE_USER_ID}#g" _config.yml
  - sed -i "s#${LEANCLOUD_COUNTERSECURE_TMP_TOKEN}#${LEANCLOUD_COUNTERSECURE_TOKEN}#g" _config.yml
  - git config --global user.name "r1v3rj1s"
  - git config --global user.email "djieastgo@yahoo.com"
  - sed -i "s#${GITHUB_REF}#${GITHUB_TOKEN}@${GITHUB_REF}#g" _config.yml

  # Restore last modified time
  - "git ls-files -z | while read -d '' path; do touch -d \"$(git log -1 --format=\"@%ct\" \"$path\")\" \"$path\"; done"

  # Deploy history
  - git clone --branch=master --single-branch https://${GITHUB_REF}.git .deploy_git


install:
  - npm install babel-runtime@6
  - npm install -g hexo-cli
  - npm install hexo --save
  - npm install hexo-deployer-git --save
  - npm install hexo-leancloud-counter-security --save
  - npm install


before_script:
  - hexo clean 


script:
  - hexo g 
  - hexo d


git:
  depth: false
branches:
  only:
    - pg-source
env:
  global:
    - secure: "...="
    - secure: "...="
{% endcodeblock %}

#### 参考链接

1. [使用 Travis CI 自动更新 GitHub Pages](https://notes.iissnan.com/2016/publishing-github-pages-with-travis-ci/)
2. [解决 Travis CI 总是更新旧博客的问题](https://wafer.li/Hexo/%E8%A7%A3%E5%86%B3%20Travis%20CI%20%E6%80%BB%E6%98%AF%E6%9B%B4%E6%96%B0%E6%97%A7%E5%8D%9A%E5%AE%A2%E7%9A%84%E9%97%AE%E9%A2%98/)


