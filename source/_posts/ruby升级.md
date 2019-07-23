---
title: ruby升级
date: 2018-06-21 18:28:29
tags: [OSX,ruby]
categories: OSX
---

一、使用RVM(Ruby Version Manager)版本管理器来升级ruby，RVM包含了Ruby的版本管理和Gem库管理(gemset)。（以下命令都在终端中进行，因为基本都是命令行）   

- RVM安装  

    `$ curl -L get.rvm.io | bash -s stable `

之后就是等待一段时间之后，就可以安装成功了，使用以下命令来验证 

 ```
     $ source ~/.bashrc 

     $ source ~/.bash_profile 
```

- 测试是否安装正常  

    `$ rvm -v `

如果出现rvm（版本号）就算是安装RVM成功了。 

二、使用RVM升级Ruby 

- 查看当前ruby版本 
   
    ` $ ruby -v `

这一步会显示出来当前ruby的版本 

- 列出已知ruby的版本 

    `$ rvm list known `

- 安装ruby 指定版本 

    `$ rvm install 2.2.4 `