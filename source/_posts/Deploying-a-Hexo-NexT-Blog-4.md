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
published: false
---

本系列文章的主要目的是介绍本博客（Hexo静态博客框架+NexT主题）是怎么配置，搭建与部署在GitHub Pages上的，以及介绍相关的过程中有哪些可能出现的问题以及其解决方案，希望后来人能够少踩坑。本文是本系列的第四篇，主要内容是介绍本博客如何利用Travis CI进行自动部署。

<!-- more -->

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




