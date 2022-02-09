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

对于Travis CI而言，它要求用户在他们需要进行自动部署的仓库的根目录下有一个`.travis.yml`文件来作为自动部署时运行的脚本文件。这个文件本质上说就是一个结构化的shell脚本。它分为三个阶段，环境配置，安装（install），以及搭建（script）。分别用来配置环境，安装部署所需的工具和软件，以及部署应用。而对于安装和搭建阶段，它们又各自拥有一个安装之前（before install）和搭建之前（before script）的阶段，起到进一步对环境进行配置（如修改文件，清理缓存等）的作用。所以我们的代码基本结构如下：

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

##### GitHub Personal Access Token的加密与配置方案

现在代码主体部分已经写好了，不过接下来的问题就是如何在无法输入密码的情况下如何通过GitHub的安全审查。第一时间想到的是GitHub提供的Personal Access Token，但是把Token以明文形式直接写在`.travis.yml`里面实在是过于危险，因为这个Token的权限很高。经过查询，发现而Travis CI是支持对Token进行哈希加密的，而且步骤也很简单。先获取GitHub Personal Access Token；然后本地安装Travis命令行工具进行加密并将加密后的Token保存至`.travis.yml`文件中；最后配置文件在相应位置替换这个Token即可。具体操作如下：
1. 前往Github右上角`+`号选项卡进入`Settings`页面，在左侧选择`Developer settings`后再在左侧点选`Personal Access Token`，然后在右侧面板点击`Generate new token`来新建一个Token。`Note`可以随便起。你可以简单描述一下这个Token干的事，比如`Travis CI deployment`之类。`Select scopes`中勾选`repo`和`admin:repo_hook`两类功能即可，因为这两类功能就能让我们获得读写我们的GitHub仓库的权限。安全起见，请不要勾选我们不需要的功能。需要注意的是，创建完的Token只有第一次可见，之后再访问就无法看见（只能看见他的名称），因此要在一个安全的地方保存好这个值。
2. 在GitHub上创建完并获得Token之后，接着需要在本地安装Travis命令行工具来对你的Token进行加密。这个工具需要`Ruby`环境，`macOS`用户如果没有的话上可以使用`Homebrew`来安装，这里我不做更多展开。具体命令如下：
{% codeblock lang:sh %}
# 假设你已经安装好了ruby和gem包管理工具
# 安装 Travis CI 命令行工具
$ gem install travis

# 登录Travis CI
$ travis login #你需要输入你登录Travis CI的账号密码，如果你使用GitHub登录的那么这个登录信息就是你的GitHub的登录信息

# 加密 Personal Access Token
$ travis encrypt -r R1V3RJ1s/r1v3rj1s.github.io GITHUB_TOKEN=XXX # -r后的参数是你的GitHub个人主页仓库的名字（用户名/仓库名）
{% endcodeblock %}
接着你会获得一串很长的以`secure="...="`为格式的密文，将这行完整地复制到你的`.travis.yml`文件中，Travis CI启动部署的时候会使用它自己与之相匹配的私钥来读取这个密文并将你加密的`GITHUB_TOKEN`部分作为环境变量，你可以看成一个加密的`export GITHUB_TOKEN=XXX`命令。`.travis.yml`中的具体脚本如下：
{% codeblock lang:sh %}
env:
  global:
    - GITHUB_REPO: github.com/R1V3RJ1s/r1v3rj1s.github.io
    - secure: "...="
{% endcodeblock %}
3. 最后一步是告诉Travis CI如何使用`GITHUB_REPO`和`GITHUB_TOKEN`这两个环境变量。在本系列文章的[第二篇](https://r1v3rj1s.github.io/2019/08/15/Deploying-a-Hexo-NexT-Blog-2/)中我们知道我们是通过<span style="background-color:#4fa7f0"><font color="white">&nbsp;站点配置文件&nbsp;</font></span>底下的`Deployment`类的`deploy`子类来告诉Hexo部署位置的，那么我们只需要把`deploy`子类下的`repo`属性从GitHub的HTTPS地址改为通过API Token访问的地址，我们就可以获得修改的权限了。而我们发现GitHub的HTTPS地址服从`https://${GITHUB_REPO}.git`的格式，而GitHub的Token Access地址服从`https://${GITHUB_TOKEN}@${GITHUB_REPO}.git`的格式。因此，我们只需要通过shell的`sed`命令，利用正则表达式来临时修改并暂存我们的文件，就能达到我们的目的。`.travis.yml`中的具体脚本如下：
{% codeblock lang:sh %}
before_install:
  # Basic config
  - git config --global user.name "r1v3rj1s"
  - git config --global user.email "djieastgo@yahoo.com"
  - sed -i "s#${GITHUB_REPO}#${GITHUB_TOKEN}@${GITHUB_REPO}#g" _config.yml # `-i`标记告诉`sed`命令需要将修改结果保存在文件中，`s`和`g`是开始和结束的标记，`#`号作为字符串分隔符从而避免与地址分隔符`/`相混淆。
{% endcodeblock %}

另外，Travis CI也支持通过SSH方式来向GitHub部署代码，具体可以参考[这篇博文](https://www.zhuwenlong.com/blog/article/5c24b6f2895e3a0fb4072a5c)。

##### LeanCloud相关插件信息的加密与配置方案

到这一步，如果我们的博客没有什么别的需要额外配置的插件，那么我们的自动部署脚本就已经编写完成并且应该能够成功运行了。但是由于我们还启动了LeanCloud的计数和评论插件，以及计数插件的安全补丁，因此为了不在自动部署过程中输入账号密码信息，同时又不在配置文件中明文储存该信息，我们还需要额外配置一下。本质而言并没有什么区别，因为Travis CI支持通过多`secure`字段进行多环境变量加密传递的方式，所以我们把我们需要加密的信息统统`travis encrypt`一遍再复制到`.travis.yml`文件中即可。然后配置过程也类似，在原来的配置文件中给你需要修改的字段分别设置一个唯一标识码（比如你可以用一个特别的字符串，或者一个简单一点的哈希算法），然后在自动部署过程中使用`sed`命令识别，修改并暂存配置文件即可。我以LeanCloud的计数插件的App ID为例来说明一遍具体步骤，其他字段修改的流程完全一样，不再赘述：

{% codeblock lang:sh %}
# 加密 LeanCloud Counter App ID
$ travis encrypt -r R1V3RJ1s/r1v3rj1s.github.io LEANCLOUD_VISITORCOUNTER_ID=XXX
{% endcodeblock %}

然后`.travis_yml`中的脚本如下：

{% codeblock lang:sh %}
before_install:
  # Basic config
  - git config --global user.name "r1v3rj1s"
  - git config --global user.email "djieastgo@yahoo.com"
  - sed -i "s#${GITHUB_REPO}#${GITHUB_TOKEN}@${GITHUB_REPO}#g" _config.yml
  # 修改站点配置文件LeanCloud Counter Security里的LeanCloud Counter App ID的信息
  - sed -i "s#${LEANCLOUD_VISITORCOUNTER_TMP_ID}#${LEANCLOUD_VISITORCOUNTER_ID}#g" _config.yml
  # 修改主题配置配置文件里LeanCloud Counter的App ID的信息
  - sed -i "s#${LEANCLOUD_VISITORCOUNTER_TMP_ID}#${LEANCLOUD_VISITORCOUNTER_ID}#g" themes/next/_config.yml

env:
  global:
    - GITHUB_REPO: github.com/R1V3RJ1s/r1v3rj1s.github.io
    - LEANCLOUD_VISITORCOUNTER_TMP_ID = XXX
    - secure: "...=" # GITHUB_TOKEN=XXX
    - secure: "...=" # LEANCLOUD_VISITORCOUNTER_ID=XXX
{% endcodeblock %}

最后，Travis CI在网站中也提供了一个设置预输入环境变量的页面，然后在启动自动部署运行脚本时所有在该页面中进行预输入的环境变量都会以非明文的形式显示在部署日志里。所以最后我把凡是可以明文写在`.travis_yml`中的环境变量统统都放到了预输入页面里，然后加密之后的环境变量依旧写在`env`的`global`子类下。所以最后的`.travis.yml`文件会看上去缺了一些环境变量。该设置预输入环境变量的页面在`Dashboard`页面里仓库信息的`Setting`选项中的`Environment Variables`栏里。设置好变量名（`NAME`），值（`VALUE`）以及该变量使用的分支（`BRANCH`）之后，把`DISPLAY VALUE IN BUILD LOG`这个选项关掉即可。该选项应该是默认关闭的。

#### commit历史恢复问题解决方案

到现在为止，我们的Travis CI应该已经能够正常执行`.travis.yml`脚本，将我们的源代码渲染为网页文件并将之部署到master分支和GitHub Pages上了。但如果你查看master的commit历史你可能能会发现一个问题：无论我们更新多少次，代码commit历史永远只有两条。这就很麻烦了。一方面来说这个commit的历史也是博客构建的历史，如果永远只显示最新的一次，总归是显得有些失落，因为过去的积累都显示不出来；另一方面而言commit历史丢失也可能造成潜在的问题，比如分支混淆造成源码丢失时，这个问题会导致整个代码提交改动历史也都随之丢失，甚至整个博客也会毁于一旦。因此解决这个问题就显得颇为重要了。经过查询，这个问题的原因在于`hexo-deploy-git`插件在master分支没有之前deploy生成的`.deploy_git`目录情况下动生成了一个`.deploy_git`文件夹，并将 public 复制到这个目录下再进行push，所以每次master分支都会获得一个新的`.deploy_git`目录，而commit历史又是靠`.deploy_git`来记录的，所以会丢失。这个问题相对比较好解决，只需要在每次构建的时候都`clone`一下`.deploy_git`目录即可。`.travis.yml`中的具体脚本如下：

{% codeblock lang:sh %}
before_install:
    - git clone --branch=master {your_blog_repo_git_url} .deploy_git
{% endcodeblock %}

#### 博文更新时间恢复问题解决方案

除了上面的问题之外，Travis CI的自动部署还会导致另一个问题：所有博文的更新时间总是最后一次部署的时间。这个原因是Hexo内的更改时间显示的是markdown文件在系统中的最后修改时间， 而Travis CI 在构建的时候，总是会重新clone repo，这就造成了所有文件的最后修改时间都是最新的clone时间。解决方案是我们可以采用git的最后commit时间来替代，这样子就能恢复文件的修改时间了。但这个问题网上的解决方案一般是只适用于ASCII字符的文件名，而对于非英文用户稍微有点麻烦，是因为有时候我们会用non-ASCII字符（比如中文）来起文件名。但[这篇博文](https://wafer.li/Hexo/%E8%A7%A3%E5%86%B3%20Travis%20CI%20%E6%80%BB%E6%98%AF%E6%9B%B4%E6%96%B0%E6%97%A7%E5%8D%9A%E5%AE%A2%E7%9A%84%E9%97%AE%E9%A2%98/)给出了当文件名含non-ASCII字符时的解决方案。`.travis.yml`中的具体脚本如下，注意**双引号是必须的**：

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
node_js: 13


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
  - sed -i "s#${GITHUB_REPO}#${GITHUB_TOKEN}@${GITHUB_REPO}#g" _config.yml

  # Restore last modified time
  - "git ls-files -z | while read -d '' path; do touch -d \"$(git log -1 --format=\"@%ct\" \"$path\")\" \"$path\"; done"

  # Deploy history
  - git clone --branch=master --single-branch https://${GITHUB_REPO}.git .deploy_git


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

* [使用 Travis CI 自动更新 GitHub Pages](https://notes.iissnan.com/2016/publishing-github-pages-with-travis-ci/)
* [解决 Travis CI 总是更新旧博客的问题](https://wafer.li/Hexo/%E8%A7%A3%E5%86%B3%20Travis%20CI%20%E6%80%BB%E6%98%AF%E6%9B%B4%E6%96%B0%E6%97%A7%E5%8D%9A%E5%AE%A2%E7%9A%84%E9%97%AE%E9%A2%98/)
* [使用Travis-ci自动SSH部署代码](https://www.zhuwenlong.com/blog/article/5c24b6f2895e3a0fb4072a5c)

