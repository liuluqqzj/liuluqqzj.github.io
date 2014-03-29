---
layout: post
title: "RTC clock initialization"
description: ""
category: nRF51822 development
tags: [RTC]
---
{% include JB/setup %}


在51822中，RTC的时钟为32.768kHZ, 它来源于LFCLK,手册上的描述为：The RTC will run off a low-frequency clock (LFCLK) running at 32.768 kHz. This clock may be either a RC oscillator or a crystal oscillator. The COUNTER resolution will therefore be 30.517 μs。而51822外部还接有一个32.768kHZ的时钟，所以在实际的选择当中，我们配置LFCLK为External 32KiHz crystal。
LFCLK为所有的低频率模块提供时钟源，所以一次性配置好以后就不要轻易的改变，以免引起其它模块的不正常。其初始化配置如下：
	
	NRF_CLOCK->LFCLKSRC = (CLOCK_LFCLKSRC_SRC_Xtal << CLOCK_LFCLKSRC_SRC_Pos);
	NRF_CLOCK->EVENTS_LFCLKSTARTED = 0;
	NRF_CLOCK->TASKS_LFCLKSTART = 1;
	while (NRF_CLOCK->EVENTS_LFCLKSTARTED == 0)
	{
	//Do nothing.
	}
对于 NRF_CLOCK->EVENTS_LFCLKSTARTED  这个标志位来说，当LFCLK时钟源开启时，它自动置为1，但是当你关闭LFCLK时，你需要手动将其置为0，这同其它的任何中断中需要将中断标志位清零是类似的道理。
        此外RTC结构体中还提到一个功能，就是：Configures event enable routing to PPI for each RTC event。其含义为将内部的中断引出到外部接口（gpio）,意思就是：当内部的RTC发生一个中断时，所对应的gpio引脚可能产生一个脉冲或者电平的跳变。