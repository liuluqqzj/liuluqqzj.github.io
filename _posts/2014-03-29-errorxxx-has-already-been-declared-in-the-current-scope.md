---
layout: post
title: error xxx  has already been declared in the current scope 
description: ""
category: C Programming
tags: []
---
{% include JB/setup %}


###问题处理：
出现这种问题的原因多半是自己的程序写的太过杂乱。各种头文件之间互相包含导致同一个文件或者函数调用重复编译报错。
	
	解决方法：在出错的头文件中添加#ifndef xxx  #define  xxx  #endif  的用法，如下所示：
	
	头文件名为：xxx.h, 头文件内容编写为：
	#ifndef  _XXX_H_
	#define _XXX_H_
	…
	…
	… 头文件代码
	… 注意：变量不要定义在.h文件中
	…
	…
	#endif
	//空一行