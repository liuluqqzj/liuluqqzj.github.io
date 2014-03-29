---
layout: post
title: "PPI function in nRF51822"
description: ""
category: nRF51822 development
tags: []
---
{% include JB/setup %}


51822内部的PPI总线可以让一个事件的发生触发另外一个任务的执行，比如如下的这种配置：
	
	/** @brief Function for initializing the PPI peripheral.
	*/
	static void ppi_init(void)
	{
	// Configure PPI channel 0 to stop Timer 0 counter at TIMER1 COMPARE[0] match, which is every even number of seconds.
	NRF_PPI->CH[0].EEP = (uint32_t)(&NRF_TIMER1->EVENTS_COMPARE[0]);
	NRF_PPI->CH[0].TEP = (uint32_t)(&NRF_TIMER0->TASKS_STOP);
	// Configure PPI channel 1 to start timer0 counter at TIMER2 COMPARE[0] match, which is every odd number of seconds.
	NRF_PPI->CH[1].EEP = (uint32_t)(&NRF_TIMER2->EVENTS_COMPARE[0]);
	NRF_PPI->CH[1].TEP = (uint32_t)(&NRF_TIMER0->TASKS_START);
	// Enable only PPI channels 0 and 1.
	NRF_PPI->CHEN = (PPI_CHEN_CH0_Enabled << PPI_CHEN_CH0_Pos) | (PPI_CHEN_CH1_Enabled << PPI_CHEN_CH1_Pos);
TIMER1产生一个事件匹配中断，产出一个脉冲，高电平为1，则会触发TIMER0的TASK_STOP被置1，TIMER0停止工作。我们还可以将所触发的任务设为gpio输出，这样就可以一个事件脉冲输出，作为一个中断信号。