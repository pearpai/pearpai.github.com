---
layout:     post
title:      "mac下安装Jekyll"
date:       2016-08-14 11:06:00
author:     "Pearpai"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - 开发环境
    - Blog
---

需要环境支持  
Ruby，Mac自带，如果没有请安装  
(这里推荐用RVM安装Ruby,这样就不需要命令上加sudo)  

## 安装RubyGems  
RubyGems是Ruby第三方插件管理器  
下载RubyGems到本地后，在终端输入如下代码  

```Shell
#检查gem版本
gem -v

#更新Gem(提示权限)
gem update --system
```

附上官方安装教程：https://rubygems.org/pages/download
`gem update --system`。这一步如果出现404错误。参考：https://ruby.taobao.org/ 来解决


## 安装jekyll
用Gem安装jekyll

```Shell
#安装jekyll(提示权限)
$ gem install jekyll

#安装成功之后，查看版本号
$ jekyll -v
```

## 使用jekyll
jekyll安装成功之后，可以在终端上执行 jekyll 命令来使用了，有两种方法，一种是自己新建一个jekyll博客，另外一种是使用现成的博客。

我比较懒，当然是直接使用现成的博客了。

我使用的主题：http://enml.github.io/site

下载主题，在终端中使用命令cd到该主题根目录下；

```Shell
#博客生成，默认生成再_site目录下，当然也可以在配置文件中自定义
jekyll build

#开启jekyll本地预览
jekyll server
```

不能访问请检查_config.yml配置文件是否需要修改

遇到的坑：

较老版本使用 jekyll --server
执行 jekyll build 命令报错

>ERROR: YOUR SITE COULD NOT BE BUILT:
       Missing dependency: rdiscount

解决：rdiscount是 Jekyll依赖的一个包，可以通过安装这个包来解决。

```Shell
安装discount
$ gem install rdiscount
```

如果缺少其他包，同理使用 gem install 解决

上传GitHub

再_post中放入md文件，文件格式必须遵从YEAR-MONTH-DAY-title.md。
上传至GitHub后，我们就可以在线查看博客了。

>补充：  
关于创建jekyll博客教程：http://www.jekyll.org/ ,需要翻墙  
各种主题下载：http://jekyllthemes.org/
