---
title: Hexo远程部署博客到vps
date: 2017-09-22 16:46:12
tags: Hexo Hexo远程部署博客到vps
thumbnail: http://mg.soupingguo.com/attchment2/ArticleImg/600/100/148/886/100400957.png 
---

### 一、什么是Hexo

  Hexo是一个博客的构建工具 [传送门](https://hexo.io/zh-cn/docs/index.html) 其原理将md的文件打包成静态网页。

### 二、搭建

  搭建分两部分，一个是在本地搭建Hexo的环境，另一个是在我们的vps上搭建服务器环境。（如果你是用github page实现自动部署，用不到我的服务器配置）

#### a、本地环境
  
  首先安装Hexo,安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：

  [nodejs](https://nodejs.org/en/)

  [git](https://git-scm.com/)

  安装过程自行百度，这个两个工具安装完之后我们在本地安装Hexo

     $ npm install -g hexo-cli
 
  接下来我们要做的就是配置我们的git，将用户名和邮箱改成自己的即可

    $ git config --global user.name "John Doe"
    $ git config --global user.email johndoe@example.com
    
  因为Hexo需要将我们的代码上传到我们自己的vps所以需要先登录服务器，这里我们只要配置成用公匙登录即可，首先在本地先生成，生成之前先确定是否已经生成过

    $ cd ~/.ssh
    $ ls
    authorized_keys2  id_dsa       known_hosts
    config            id_dsa.pub

  如果没有看到id_rsa.pub证明还没有生成过，执行ssh-keygen，遇到提示回车确认就好

    $ ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
    Created directory '/home/schacon/.ssh'.
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/schacon/.ssh/id_rsa.
    Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
    The key fingerprint is:
    d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local

  验证一下是否成功，如果看到下面的内容证明你成功了

    $ cat ~/.ssh/id_rsa.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
    GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
    Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
    t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
    mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
    NrRFi9wrf+M7Q== schacon@mylaptop.local

  我们来验证一下是否可用

    $ ssh 用户名@vps地址

  如果没有提示输入密码，可以直接登录服务器，说明我们的公匙生效了，我们可以进行下一步了，现在本地初始化一个 Hexo的目录，等待安装完之后我们的本地环境就配置完成了。

    $ hexo init Hexo

#### b、配置服务器环境

  上面我们已经讲了本地环境如何配置，本地环境的配置相对服务器来说要简单一点所以我会重点的说一下服务器的配置方法，我们先来讲一下Hexo是如何实现自动部署的，简单来说是通过git的[hooks](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)来实现的,而我们这里主要用到post-receive的方法，下面来具体说一下如何实现的，首先在服务上建立一个git的仓库和一个运行目录

    $ mkdir blog.git  // 建立git仓库
    $ mkdir website   //  建立网站的运行目录
    $ cd blog.git && git init --bare // 初始化仓库
    $ cd .. && cd website && git clone ../blog.git // 初始化运行目录
    $ vi ../blog.git/hooks/post-receive  // 编写钩子函数

  在post-receive里面输入下面的内容

    #!/bin/sh
    echo "开始部署"
    unset GIT_DIR
    cd /home/work/website/blog # 路径改成自己额就好
    git pull origin master
    echo "部署成功"

  接下来我们要在服务器上安装nginx[传送门](http://seanlook.com/2015/05/17/nginx-install-and-config/)跟着这个安装就好，这里不做详述，安装好之后要修改一下nginx的配置

    $ vi /etc/nginx/conf.d/default.conf

  更改如下内容

       location / {
          root   /home/work/website/blog;
          index  index.html;
        }

  保存之后重启nginx

    systemctl restart nginx.service
    
### 三、运行

  在运行之前我们要先把本地的Hexo的配置文件补充一下进入_config.yml文件更改如下内容

    deploy:
      type: git
      repo: 用户名@vps地址: 服务器上git仓库的路径 // root@123.123.13.12: /home/work/blog.git
      branch: master

  保存之后运行下面的命令
      
      $ hexo new "first blog"
      $ hexo g
      $ hexo d

  如果输出 "开始部署" "部署成功" 等字样说明我们的部署成功了




  

  
