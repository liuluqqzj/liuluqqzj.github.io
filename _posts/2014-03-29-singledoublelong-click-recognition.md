---
layout: post
title: "Single/double/long click recognition"
description: ""
category: nRF51822 Development
tags: [nRF51822]
---
{% include JB/setup %}


题记：实现按键的单击这一单一功能可能会比较简单，但是要求识别多种操作就需要相对复杂的判断。对此，在大量占用CPU宝贵资源的提前下，本文中介绍了如何使用定时器中断及引脚中断的相互配合来实现按键响应的处理。
实现原理：1、可能大家常用的按键方式是拉低引脚电平来触发一次按键，在本文中采用的却是拉高引脚电平来触发一次按键。2、而在按键的过程中会出现“抖动”的情况，即按键按下的前几十毫秒引脚的电平跳变很厉害，采用延时10~20ms的方式再次检测引脚电平可避免抖动的影响。3、一次有效按键识别之后在其下降沿依次判断三种情况：a)这次按键是否超过2s，是则判为长按，结束。 b)否则，开启0.5s计时，0.5s内是否再次出现有效按键，是则判为双击，结束。c)否则，0.5s后超时，判为一次单击，结束。
具体实现如下：
	
	1、 a)引脚初始化配置为输入下拉，上升沿中断。b)定时器配置为计时模式，同时将匹配寄存器的值根据当前滴答配置为CC[0] = 10ms， CC[3] = 0.5s，CC[1]用来计时；关闭定时器中断及定时器禁止。c）按键有效标识初始化为0，用来记录是否此次按键有效。此标识是长按、短按、双击成立的共同前提条件。
	
	2、 引脚识别一个上升沿跳变触发引脚中断：在中断函数中进入上升沿处理区域，清除引脚中断位，关闭引脚中断并配置为下降沿；开启定时器10ms计时。
	
	3、 定时器10ms匹配中断：关闭定时器中断，开启引脚中断。a)判断此时是否是高电平以及按键有效标识为0，是，则按键有效标识置1。b)判断此时是否是高电平以及按键有效标识为1,初次肯定不成立，后续处理见5。
	
	4、 引脚识别一个下降沿跳变触发引脚中断：在中断函数中进入下降沿处理区域，清除引脚中断位，配置引脚中断为上升沿；a)捕获此时的定时器计时到CC[1]，是否大于2s并且按键有效标识为1，是，则判为长按，定时器中断关闭，定时器关闭，清除CC[1]，有效按钮标识清零。b）否a)，判断有效按钮标识是否为1，是，开启定时器0.5s计时。
	
	5、 若0。5s内再次发生1、2、的情况，则依然进入3，此时按键有效标识已置1，则3中进入b处理: 执行到这里说明再次的按键有效在0.5s之内发生，判断为双击，定时器中断关闭，定时器关闭，清除CC[1]，有效按钮标识清零。
	
	6、 0.5s计时中断触发：此时表明0.5s内未发生5，则视为超时未检测到二次按键，判为单击，定时器中断关闭，定时器关闭，清除CC[1]，有效按钮标识清零。
 

 代码如下：
	
	#include "button.h"
	#include <stdbool.h>
	#include <stdint.h>
	#include "nrf.h"
	#include "nrf_gpio.h"
	#include "xprintf.h"

	#define nRF_gpio_INT_pin         6     // 引脚(8)计时中断检测

	static uint8_t button_tmp = 0;
	// 0: nothing happened, 1:single-click, 2:double-click, 3:long-click 
	static uint8_t BUTTON_STATE = 0;

	//NRF_TIMER_Type * p_timer; //NRF_TIMER0/1/2
	#define p_timer             NRF_TIMER0

	// 中断事件处理函数
	void GPIOTE_IRQHandler()
	{
		 // Event causing the interrupt must be cleared.
		if((NRF_GPIOTE->EVENTS_IN[0] == 1) && (NRF_GPIOTE->INTENSET & GPIOTE_INTENSET_IN0_Msk) && (NRF_GPIOTE->CONFIG[0] &  (GPIOTE_CONFIG_POLARITY_HiToLo << GPIOTE_CONFIG_POLARITY_Pos) ) ) //上升沿
		{
			NRF_GPIOTE->INTENSET  = GPIOTE_INTENSET_IN0_Disabled << GPIOTE_INTENSET_IN0_Pos;     // 关GPIO引脚中断
			NRF_GPIOTE->EVENTS_IN[0] = 0;
			NRF_GPIOTE->CONFIG[0] =  (GPIOTE_CONFIG_POLARITY_LoToHi << GPIOTE_CONFIG_POLARITY_Pos)
							   | (nRF_gpio_INT_pin << GPIOTE_CONFIG_PSEL_Pos)  
							   | (GPIOTE_CONFIG_MODE_Event << GPIOTE_CONFIG_MODE_Pos);
				   
			p_timer->TASKS_CLEAR = 1;
			p_timer->INTENSET       = TIMER_INTENSET_COMPARE0_Enabled << TIMER_INTENSET_COMPARE0_Pos;   // 开通延时中断1
			p_timer->TASKS_START = 1;
			//xprintf("High INT.\r\n");
		}
		// Event causing the interrupt must be cleared.
		if((NRF_GPIOTE->EVENTS_IN[0] == 1) && (NRF_GPIOTE->INTENSET & GPIOTE_INTENSET_IN0_Msk) && (NRF_GPIOTE->CONFIG[0] &  (GPIOTE_CONFIG_POLARITY_LoToHi << GPIOTE_CONFIG_POLARITY_Pos) ) )  // 下降沿
			{
				//xprintf("Low INT.\r\n");
			   // NRF_GPIOTE->INTENSET  = GPIOTE_INTENSET_IN0_Disabled << GPIOTE_INTENSET_IN0_Pos;     // 关GPIO引脚中断
				NRF_GPIOTE->EVENTS_IN[0] = 0;
				NRF_GPIOTE->CONFIG[0] =  (GPIOTE_CONFIG_POLARITY_HiToLo << GPIOTE_CONFIG_POLARITY_Pos)
							   | (nRF_gpio_INT_pin << GPIOTE_CONFIG_PSEL_Pos)  
							   | (GPIOTE_CONFIG_MODE_Event << GPIOTE_CONFIG_MODE_Pos);
				
				p_timer->TASKS_CAPTURE[1] = 1;  // 获取当前的定时器计时
				if( (p_timer->CC[1] >= 28846) && (button_tmp == 1) )   //  获得一次按键并且是长按
				{
					BUTTON_STATE = 3;
					button_tmp  = 0;
					p_timer->TASKS_STOP = 1;
					p_timer->TASKS_CLEAR = 1;
					p_timer->INTENSET = TIMER_INTENSET_COMPARE0_Disabled << TIMER_INTENSET_COMPARE0_Pos; // 关定时器延时中断0
					p_timer->INTENSET = TIMER_INTENSET_COMPARE3_Disabled << TIMER_INTENSET_COMPARE3_Pos; // 关延时中断
					xprintf("Long click.\r\n");
				}
				else
				{
					if(button_tmp == 1)    //  获得一次按键，但不是长按，开启1s 双击计时
					{
					   // p_timer->TASKS_CLEAR = 1;
						//xprintf("1s jishi.\r\n");
						p_timer->INTENSET      = TIMER_INTENSET_COMPARE3_Enabled << TIMER_INTENSET_COMPARE3_Pos;   // 开延时中断2
					}
				}
			  //NRF_GPIOTE->INTENSET  = GPIOTE_INTENSET_IN0_Set << GPIOTE_INTENSET_IN0_Pos;     // 开GPIO引脚中断
			}
	}
	void TIMER0_IRQHandler()
	{
		if((p_timer->EVENTS_COMPARE[0] ==1) && (p_timer->INTENSET & TIMER_INTENSET_COMPARE0_Msk))     // 仅用来去抖延时
		{
			p_timer->EVENTS_COMPARE[0] = 0;
			p_timer->INTENSET = TIMER_INTENSET_COMPARE0_Disabled << TIMER_INTENSET_COMPARE0_Pos; // 关定时器延时中断0
			//p_timer->TASKS_CLEAR = 1;
			
			NRF_GPIOTE->INTENSET  = GPIOTE_INTENSET_IN0_Enabled << GPIOTE_INTENSET_IN0_Pos;     // 开GPIO引脚中断
			
			if((nrf_gpio_pin_read(8) & 1) && (button_tmp == 0))     // 一次有效高电平
			{
				button_tmp = 1;
			   // xprintf("one ok.\r\n");
			}
			else 
			{
				if( (nrf_gpio_pin_read(8) & 1) && (button_tmp == 1))     // 检测到两次有效高电平 结束
				{
					p_timer->TASKS_STOP = 1;
					p_timer->TASKS_CLEAR = 1;

					p_timer->INTENSET = TIMER_INTENSET_COMPARE3_Enabled << TIMER_INTENSET_COMPARE3_Disabled; // 关延时中断2
					BUTTON_STATE = 2;
					button_tmp = 0;
					xprintf("Double click.\r\n");
				}
			}
			//xprintf("10ms.\r\n");
		}
		if( (p_timer->EVENTS_COMPARE[3] ==1) && (p_timer->INTENSET & TIMER_INTENSET_COMPARE3_Msk) && (nrf_gpio_pin_read(8) == 0 )&& (button_tmp == 1))  // 1s延时中断到达，只检测到一次
		 {
			 p_timer->EVENTS_COMPARE[3] = 0;
			 p_timer->TASKS_STOP  = 1;
			 p_timer->TASKS_CLEAR = 1;
			 p_timer->INTENSET = TIMER_INTENSET_COMPARE0_Disabled << TIMER_INTENSET_COMPARE0_Pos; // 关定时器延时中断0
			 p_timer->INTENSET = TIMER_INTENSET_COMPARE3_Disabled << TIMER_INTENSET_COMPARE3_Pos; // 关延时中断
			 BUTTON_STATE = 1;
			 button_tmp = 0;
			 xprintf("Single click.\r\n");
		 }
	}

	// gpio中断配置
	void nRF_gpio_INT_config()
	{
	  nrf_gpio_cfg_input(nRF_gpio_INT_pin, NRF_GPIO_PIN_PULLUP);
		// Enable interrupt:
	  NVIC_EnableIRQ(GPIOTE_IRQn);
	   // 1,翻转 2,8引脚 3,事件模式
	  NRF_GPIOTE->CONFIG[0] =  (GPIOTE_CONFIG_POLARITY_HiToLo << GPIOTE_CONFIG_POLARITY_Pos)
							   | (nRF_gpio_INT_pin << GPIOTE_CONFIG_PSEL_Pos)  
							   | (GPIOTE_CONFIG_MODE_Event << GPIOTE_CONFIG_MODE_Pos);
		//使能中断
	  NRF_GPIOTE->INTENSET  = GPIOTE_INTENSET_IN0_Set << GPIOTE_INTENSET_IN0_Pos;
	}


	void button_start()
	{
		// Start 16 MHz crystal oscillator.
		NRF_CLOCK->EVENTS_HFCLKSTARTED  = 0;    // HFCLK oscillator state = 0.
		NRF_CLOCK->TASKS_HFCLKSTART     = 1;    // Start HFCLK clock source

		// Wait for the external oscillator to start up.
		while (NRF_CLOCK->EVENTS_HFCLKSTARTED == 0) 
		{
			// Do nothing.
		}
		p_timer->MODE           = TIMER_MODE_MODE_Timer;        // Set the timer in Timer Mode.
		p_timer->PRESCALER      = 9;                            // Prescaler 9 produces 31250 Hz timer frequency => 1 tick = 32 us.
		p_timer->BITMODE        = TIMER_BITMODE_BITMODE_32Bit;  // 16 bit mode.
		/* Bit 16 : Enable interrupt on COMPARE[0] */
		NVIC_EnableIRQ(TIMER0_IRQn);
		
		p_timer->TASKS_CLEAR    = 1;                            // clear the task first to be usable for later.
		
		//
		p_timer->CC[0]          = 288;    // 15ms延时
		p_timer->CC[1]          = 0;
		p_timer->CC[2]          = 0;
		p_timer->CC[3]          = 9615;    // 0.5s延时
		
		// p_timer->TASKS_START    = 1;                    // Start timer.

		
	}


