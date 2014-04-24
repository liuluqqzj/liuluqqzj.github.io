---
layout: post
title: "ble sports data synchronous"
description: ""
category: nRF51822 development
tags: []
---
{% include JB/setup %}

#####数据同步过程中出现了好几个错误，现在将错误记录下来：
	
1. 传输协议错误，没有确定好当前的传输格式。
2. 数据结构太过复杂，手环端应该只是负责数据的存储，并不需要太多的组织与处理。
3. 计算当中的一个粗心大意，将‘与’运算&写成了&& 。
4. 数据临界区的问题。`*offset` 指向我即将获取的数据地址，`get_pedo_data_length()`是数据总量。我得到的数据长度是`get_pedo_data_length() - *offset`，
而不是`*offset < get_pedo_data_length() - *offset + 1`. 此外读取的结束条件应该是 `*offset < get_pedo_data_length()`，而不是`*offset <= get_pedo_data_length()`。
5. 将`pedo_data[pedo_ctrl.diff_pointer[i] + count]` 写成了 `pedo_data[pedo_ctrl.diff_pointer[i + count] ]`；将`time_diff >> i `位  写成了 `time_diff >> i*8 `。
