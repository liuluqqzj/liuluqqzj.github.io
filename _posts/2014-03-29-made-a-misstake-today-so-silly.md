---
layout: post
title: "Made a misstake today, so silly"
description: ""
category: nRF51822 development
tags: []
---
{% include JB/setup %}


又犯傻了。。。
Written by liulu  on 三月 19, 2014 (Edit)
今天蓝牙协议启动不了，调试了半天，发现在定时器初始化之后就出错，对着定时器初始化代码看了半天也没发现异常。还把同事叫过来一起调试，最后自己发现了一个白痴的代码：NVIC_SetPriority(GPIOTE_IRQn, APP_IRQ_PRIORITY_LOW); 代码复制过来后忘记将GPIOTE_IRQn 改为TIMER1_IRQn ….导致协议栈初始化出错。。。。。又是出在小问题的处理上….不给力啊！！