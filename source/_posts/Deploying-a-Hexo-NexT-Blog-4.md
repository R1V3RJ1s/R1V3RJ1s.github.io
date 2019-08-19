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

到现在为止我们对博客源码（比如站点设置，主题设置，博文等）的每次更改都需要我们手动部署到GitHub Pages上。重复的次数多了就显得很麻烦，出错的几率也会变大。一旦分支混淆，还会造成远程源码丢失的问题。于是我们希望有一个自动化部署的工具能够代替我们来完成这件事情。简单而言，我们理想中的基本流程如下：每当我们提交变动到我们的个人主页仓库的源代码分支时，这个自动化部署工具都能监控到这个变动，并运行部署脚本对我们的源代码进行渲染，然后把渲染后的网页文件部署到个人主页仓库的master分支上。而这个流程的本质其实就是一个持续集成/持续部署（CI/CD）的流程，所以我考虑到了[Travis CI](https://travis-ci.org/)，因为它简单好用，而且免费。注册Travis CI和让Travis CI为特定的GitHub仓库开启自动Build的过程非常容易，在此不再赘述。本过程的主要难点在于需要自己编写部署脚本。

#### 自动部署脚本的基本内容

对于Travis CI而言，它需要你需要进行自动部署的仓库的根目录下有一个`.travis.yml`文件来作为自动部署时运行的脚本文件。这个文件本质上说就是一个结构化的shell脚本。它分为三个阶段，环境配置，安装（install），以及搭建（script），分别用来配置环境，安装部署所需的工具和软件，以及部署应用。而对于安装和搭建阶段，它们又各自拥有一个安装之前（before install）和搭建之前（before script）的阶段，起到进一步对环境进行配置（如修改文件，清理缓存等）的作用。所以我们的代码基本结构如下：

{% codeblock lang:sh %}
# 指定需要运行的语言和版本
language: node_js
node_js: stable

# 指定需要监控代码改动的分支
branches:
  only:
    - pg-source

# 安装部署所需工具
install:
  - npm install babel-runtime@6
  - npm install -g hexo-cli
  - npm install hexo --save
  - npm install hexo-deployer-git --save
  - npm install hexo-leancloud-counter-security --save
  - npm install

# 清理缓存
before_script:
  - hexo clean 

# 搭建和部署
script:
  - hexo g 
  - hexo d

{% endcodeblock %}

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

但有些人可能会发现这个脚本在Travis CI去运行时恢复的修改时间只能到几天前，这是因为Travis CI默认只会克隆最新的50次commit历史记录，而我们的脚本需要完整的git历史记录才能正确的恢复文件的修改时间，所以我们还需要取消Travis CI的默认`depth`参数，从而让它克隆我们完整的git仓库。`.travis.yml`中的代码如下：

{% codeblock lang:sh %}
git:
  depth: false
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


