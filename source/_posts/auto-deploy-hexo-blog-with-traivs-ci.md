---
title: 使用travis-ci自动部署hexo博客
date: 2018-08-02 00:18:57
tags:
    - 持续集成
    - github
    - travis
categories: 技术
---
最近研究了Traivs-ci，发现持续集成真是一个不错的东西，写好脚本之后，帮我省了一堆事。
现在用它来帮我持续集成我的hexo博客，现在我只需有将新的md文件push到github上，
travis会帮我处理好之后的事。


以下是我们将要实现之后写新博客的步骤也是我们要达到的目的：
* 更新博客文章内容后 commit 到 GitHub repo
* Travis CI 自动编译生成出新的静态博客文件
* 自动部署至 GitHub Pages

<!--more-->

##  前期准备

* 创建 `github repo`，本文以 [zxh326.github.io](https://github.com/zxh326/zxh326.github.io) 为例
在你本地博客目录执行以下操作
```shell
# 以下操作将会在你的仓库新建一个source防止专门来存放hexo代码，
# 建议不要将主题一起提交，或者将主题作为git submoudle来管理
# 
git init
git remote add origin git@github.com:zxh326/zxh326.github.io.git
git checkout --orphan source
git add .
git commit -m "Initial commit"
git push origin source:source
```
* 注册 `travis-ci`，并将你的仓库添加至`travis-ci`，[travis-ci](https://travis-ci.org/)

## 生成部署专用密钥 & 写配置文件

我们有两种方法使`travis-ci`有操作我们仓库的权限

* 申请一个GitHub的 Personal access tokens，配合 Travis 的环境变量配置就可以拿到 push 权限了

* 利用travis-ci生成加密的专用部署密钥

简单的说下区别:
第一种 申请的 `token`默认是可以操作仓库里所有仓库的，包括私有仓库，`github`可以以很方便的对`token`进行权限管理，
你可以选择明文放在travis与项目相关的环境变量或者通过官方工具加密此token。

第二种 是生成一对ssh密钥对，之后利用`travis` 利用 `openssl aes-256-cbc` 加密。
如果你另`同时部署到vps`的话使用这种。


### 第一种：使用github提供的Personal access tokens
前往[github token](https://github.com/settings/tokens/new)申请
注意做好权限管理

第二种的使用方法，第一种直接申请token，放到traivs的环境变量即可，跳过这步
* 首先，新生成一个 ssh 密钥对（不要嫌麻烦直接把你机器上的秘钥拿去用了，太危险）：

```shell
# 随便生成在哪都行，文件名也随意
ssh-keygen -f travis.key
```

* 安装travis的工具

```shell
# 需要有Ruby环境，mac自带，windows的话。。。。。据说加密会失败，没试过。
# 可以试试用windwos的wsl linux 子系统
sudo gem install travis
```
* 然后通过命令行登录 Travis 并加密文件：

```shell
# 交互式操作，使用 GitHub 账号密码登录
# 如果是私有项目要加个 --pro 参数
# 加密完成后会在当前目录下生成一个 travis.key.enc 文件
# 会在你的 .travis.yml 文件里自动加上用于解密的 shell 语句（重要）
travis login --auto

travis encrypt-file travis.key -add
```
⚠️ 确保 **travis.key** **没有上传**至仓库，
⚠️ 确保仓库有travis.key.enc文件，这个文件放哪都成，我是专门建了个文件夹`.travis`用来存放这些文件

## 写配置文件
编辑`.travis.yml`文件
我在这里贴出我的配置文件

```yml
language: node_js
node_js: stable

# 只监听 source 分支的改动
branches:
  only:
  - source

# 缓存依赖，节省持续集成时间
cache:
  yarn: true
  directories:
    - node_modules
    - themes

before_install:
# 解密 RSA 私钥并设置为本机 ssh 私钥，注意travis.key.enc的目录要和你的目录一样
- openssl aes-256-cbc -K $encrypted_c626fb518bf5_key -iv $encrypted_c626fb518bf5_iv -in .travis/travis.key.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
# 修改本机 ssh 配置，防止秘钥认证过程影响自动部署
- git config --global user.name "zzde"
- git config --global user.email "zhangxh1997@gmail.com"
- yarn global add hexo-cli
# 赋予自动部署脚本可执行权限
- chmod +x .travis/deploy.sh

install:
# 安装 Hexo 及其依赖
- yarn
- yarn add hexo-generator-feed
# 当 Travis 文件缓存不存在时，从 Gitee 私有仓库 clone 主题，改为你使用的主题地址
- if [ ! -d "themes/seventeen" ]; then git clone git@gitee.com:zxh326/hexo-theme-seventeen.git themes/seventeen; fi
# - git clone https://github.com/zxh326/hexo-theme-icarus.git themes/icarus

script:
# 生成静态页面
- hexo clean
- hexo generate

after_success:
# 部署到 GitHub Pages
# 我把这一部分的操作直接分离到另外的脚本里去了
- .travis/deploy.sh

# 将github添加的信任里
addons:
  ssh_known_hosts:
  - github.com
  - gitee.com

``` 

写部署文件 deploy.sh

```shell
#!/bin/bash
set -ev
export TZ='Asia/Shanghai'

# 先 clone 再 commit，避免直接 force commit
# 不然整个 branch 就总是只有一个 commit，不好看
git clone -b master git@github.com:zxh326/zxh326.github.io.git .deploy_git

cd .deploy_git
git checkout master
mv .git/ ../public/
cd ../public

git add .
git commit -m "Site updated: `date +"%Y-%m-%d %H:%M:%S"`"

git push origin master:master --force --quiet

```

## 提交试试
将所有的改动提交至github，注意确保仓库有master分支

[Gist](https://gist.github.com/zxh326/421407e3e14d78a618d69462244bbdfc.js)
<script src="https://gist.github.com/zxh326/421407e3e14d78a618d69462244bbdfc.js"></script>