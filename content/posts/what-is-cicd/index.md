+++
date = '2026-03-08T11:10:05+08:00'
draft = false
title = 'What Is Cicd'
+++


`CI/CD`这个概念，很早就出现了，我自己在工作中也会用到，我觉得这是互联网人必备的一个技能。

相信很多读者，在工作中也会用到。今天我用通俗易懂地语言给大家讲解一下这个概念，并通过自己做的项目实战案例，分享一下如何从0到1使用这个工具。

# 1、ci/cd简史

ci/cd 是两个词：ci，`Continuous Integration（持续集成）`；cd，`Continuous Delivery（持续交付）或 Continuous Deployment（持续部署）`。

## 1.1 ci

`ci`这个概念，早在上世纪九十年代（1990年）就已经提出来了（那个时候还没有git）。

提出这个观念的主要目的，就是希望解决开发过程中，合并代码的成本问题——过去程序员做开发，几个月才会合一次代码。

你敢想象吗？几个月合一次代码，这里面的代码冲突、代码bug得有多恐怖。

所以，就有人提出了`持续集成`的说法：我们要频繁合并代码，不要等几个月再合并。

再后来，`git`的工具出现，又进一步优化了集成代码的体验。

## 1.2 cd

在集成代码的问题解决之后，聪明的人类就开始思考，`部署`的流程是不是也可以简化一下呢？我们提交完代码，是不是就可以顺便部署在测试环境里跑一下呢？

于是， `cd` 就出现了。它的主要用途，就是自动化部署。

过去云服务还没有出现的时候，大家都用的物理机，装个系统、统一环境十分复杂，所以，要想做到自动化部署难度比较大。

后来，云服务出现了，配置服务器便捷很多，装系统、配置环境也十分方便，鼠标点一点就可以了。

因此，`cd` 也逐渐地普及开来。例如，`GitLab CI/CD`，`蓝盾`,`Jenkins`,`GitHub Actions` 等工具都已经具备了 ci 和 cd 这些功能。


总的来说，`ci`解决了代码集成的痛苦，`cd` 解决了软件发布的痛苦。


# 2、cicd 使用

下面通过一个实战案例，讲解一下 cicd 的使用。

我先介绍一下项目背景：

* 一个博客项目
* 项目部署在云服务器

假设没有cicd工具，每次更新博客的时候，我需要把博客的网页代码，手动上传到服务器上，如下图所示，效率十分低下。

下面，我带大家借助 GitHub Action 这个 cicd 工具，解决一下这个问题。

## 2.1 打通git和你的服务器

简单梳理一下，目前总共有三个端——本地电脑，git平台，远端服务器。然后，本地电脑可以上传代码到git，本地电脑可以访问服务器，这些都没有问题。

但是，git无法访问这台服务器。所以，需要打通git和服务器之间的链路。

在 cmd 下执行命令`ssh-keygen -t ed25519 -C "git_actions" -f C:\Users\Gao\.ssh\git_actions`

一路回车，就会生成两个密钥对，如下所示：



打开博客的git仓库，找到如下配置页面。


然后，把 `git_actions` 文件里面的内容，完整地粘贴进来。点击 add secret 保存。如下图所示。

然后，用同样的方式把服务器的用户名和ip都填上。



git平台会把这些填好的变量名称全部统一为大写，记住这些大写的变量名，一会会用到。


然后，找到`git_actions.hub`文件，把里面的内容(也就是公钥)copy出来。

登录到服务器上，在 `root` 用户的 `~/.ssh` 目录下，输入 `echo "你复制的公钥内容" >> ~/.ssh/authorized_keys`，如下图所示：


最后，配置一下权限，输入`chmod 700 ~/.ssh` 和 `chmod 600 ~/.ssh/authorized_keys` 指令。

## 2.2 创建工作流

在本地的项目目录下，创建一个`.github/workflows/deploy.yml`这样一个文件。



然后，填写内容如下：

```
#cicd任务名称
name: Deploy Hugo Blog

#仓库对应的推送分支
on:
  push:
    branches: [main]

jobs:
  deploy:
  # 操作系统
    runs-on: ubuntu-latest
    steps:

    #运行插件拉取代码
      - uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

    #运行插件下载hugo
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.157.0'
          extended: true
    
    #执行命令
      - name: Build
        run: hugo --minify

    #运行插件部署到服务器
      - name: Deploy to Server
        uses: burnett01/rsync-deployments@6.0.0
        with:
          switches: -avzr --delete
          path: public/
          remote_path: /opt/1panel/www/sites/gaomian.org/index/
          remote_host: ${{ secrets.HOST }}
          remote_user: ${{ secrets.USER }}
          remote_key: ${{ secrets.BLOG }}
```

deploy.yml 本质上是一个工作脚本，目的是告诉git：你接下要做什么。git就会按照你写的这个“剧本”，按照顺序依次执行。















