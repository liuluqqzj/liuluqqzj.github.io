---
layout: post
title: "Use jekyll to manage your blog under Win7"
description: ""
category: git/ruby basic
tags: []
---
{% include JB/setup %}


若是不想折腾或者像我这样折腾了好久精疲力尽的人，可以参考本文秒速度安装jekyll.
老外搞的jekyll十足的蛋疼，使用ruby所编写，安装之前需要安装ruby，安装完ruby又是要版本管理工具RVM，又要开发环境DevKit，没准还要跑出一个python.安装过程问题不断，浪费大量时间配置环境。好了不废话了，好在老外提供了打包的解决方案，如下：
	
	1) 将一下的链接包下载下来：https://www.dropbox.com/sh/40l6mgbl1ce2kej/lF6ykQxt9d
	
	2) 解压缩到本地的  D:\PortableJekyll 目录下。
	
	3) 在Dos中执行 :   SET PATH=%PATH%;D:\PortableJekyll\ruby\bin;D:\PortableJekyll\devkit\bin;D:\PortableJekyll\git\bin;D:\PortableJekyll\Python\App;
	
	4) 运行ruby -v 及jekyll -v 看是不是安装成功了，提示：命令在github shell 下可能不能执行。OK~~讲解完毕。。。