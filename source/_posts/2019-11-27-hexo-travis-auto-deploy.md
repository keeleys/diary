---
title: hexo通过travis免费自动构建到github的page上
date: 2019-11-27 15:41:14
tags: github
---

## 前言
hexo是个本地构建md生成静态页面的工具，今天我讲下搭配自动部署的方法。   
我感觉这个travis还是不错的，相较于jenkins需要自己搭建服务，还需要公网接受hook，这个完全免费，配置也简单

## 步骤
1. github建立一个项目，关联你的hexo源代码，先不用提交代码
2. 在github添加[travis](https://github.com/marketplace/travis-ci)服务
3. 去[Applications settings](https://github.com/settings/installations)设置下授权自动构建的仓库，这里可以选前面步骤1的项目。
4. 生成一个[github的token](https://github.com/settings/tokens),复制好生成的token字符串哦。、
5. 然后回到你的[Travis CI](https://travis-ci.com/)网站,在`My Repositories`里面找到你授权的项目，点击进入项目详情，然后在右边的`more action`里面选择`setting`,在页面中间部分的`Environment Variables`里面添加一个环境变量`GH_TOKEN`,值就是你前面github生成的 `token`，然后点击add添加成功。
6. 前面几步就弄完了项目关联了，现在我们在hexo的源代码根目录添加`.travis.yml`文件，配置下让travis知道我们提交代码之后触发什么命令,内容如下，这段配置的含义就是让关联的仓库的master分支提交代码之后，执行下指定script，然后将`local-dir`目录推到`gh-pages`分支
```yml [.travis.yml]
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```
