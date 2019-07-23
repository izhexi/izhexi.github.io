---
title: ruby使用vrm升级报Requirements installation failed with status
date: 2018-06-21 19:28:29
tags: [OSX,ruby]
categories: OSX
---
在进行ruby升级时，遇到报错  

`Requirements installation failed with status:1 `

查看错误信息为：
```  

Error running 'requirements_osx_brew_libs_install automake',  

showing last 15 lines of /Users/name/.rvm/log/1474512590_ruby-2.3.0/package_install_automake.log  

```

stackoverflow搜到网友提供的方法:  

问题还是没有解决。  

于是看日志是homebrew问题。于是执行  

`brew update`  

总是返回Already up-to-date。  

于是去到homebrewgit上看了下。 发现这么一句话 
```
 Update Bug 
 If Homebrew was updated on Aug 10-11th 2016 and brew update always says Already up-to-date. you need to run: 
 cd "$(brew --repo)" && git fetch && git reset --hard origin/master && brew update 

```
执行上面的命令，在进行`rvm install 2.3`还是报错。无奈下，卸载重装homebrew。  

卸载： 

 

` sudo ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)" `

再次安装： 

`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" `

在进行`rvm install 2.3`成功。 

参考文章ruby安装 http://www.jianshu.com/p/529943f6bc93 
