---
layout: post
category : tool
tagline: "项目无Bug之日即是下线之时"
title: 使用Sublime+Clojure搭建一个脚本环境
tags : [sublime, clojure, 工具]
---
* 利用Sublime的跨平台及强大的文本编辑能力， 加上Clojure继承Lisp家族的表达能力，搭建一个very geek风格的工作脚本环境so easy    
   
{{ site.excerpt_separator }}

* 下载最新的clojure代码
  > https://github.com/clojure/clojure.git

* 编译
  > mvn package 

* 在Sublime中 点击Tools/Build System/New Build System...，在弹出的文件中,输入

    {% gist 9f548704f1ce4afbbeec %}
    把脚本保存到 ~/AppData/Roaming/Sublime Text 3/Packages/User 目录下   
    然后即可在 Tools/Build System 菜单下看到新增的脚本构建脚本   

* 在sublime 中输入
   
   	(println "Hello, Clojure!!!")   

* Ctrl + B 编译运行, 一切顺利下, 会输出   

   ![hello clojure]({{ site.img_root }}/hello_clojure.gif)
