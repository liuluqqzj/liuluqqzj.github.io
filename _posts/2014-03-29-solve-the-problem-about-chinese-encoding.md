---
layout: post
title: "solve the problem about Chinese encoding"
description: ""
category: Git/Ruby Basic
tags: []
---
{% include JB/setup %}


##解决中文编码问题：error: invalid byte sequence in GBK.

在文件路径：D:\Documents\Ruby200-x64\lib\ruby\gems\2.0.0\gems\jekyll-1.5.0\lib\jekyll 的文件：convertible.rb 。 在大概38行有这样一段代码：
	
	begin
		self.content = File.read_with_options(File.join(base, name),:encoding=>"utf-8")  // 替换成utf-8编码
		                                     # merged_file_read_opts(opts))              // 此处注释掉了
保存，再编译，应该就没问题了。

###注意：
	本文采用Notepad++编辑。具体编码设定如下：
	1）将编码改为：以UTF-8无BOM格式编码。
	2）视图中->显示符号->显示所有符号，可以看到当前的换行符编码。
	3）编辑->档案格式转换->转换为UNIX格式。
采用这种方式基本上能够解决中文编码出错的问题，本文就是这样处理的。
