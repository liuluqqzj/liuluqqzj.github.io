---
layout: post
title: "ble sports data synchronous"
description: ""
category: nRF51822 development
tags: []
---
{% include JB/setup %}






###### It really take me several days to solve this problem. now I record the bug I meet in my program.


1. Transmission protocol error. We identified a transport protocol, however, I forget the sports data flag which led to a transmit error.

2.  My data structure is too complex that made me very hard to handle the data. As we know, the bracelet just storage the data, handling the data should delivered to the phone. So I just need to sign a data structure to storage the data as simple as I can.

3. The process of and computing. It's a small problem, but reflects my careless in treatment of details. I wrote '&' as "&&" and spent half a day to discover it. How frustrating it is!

4. The process of array's boundary. e.g. *offset point to the data's address I gonna to fetch, get_pedo_data_length() is the amount of the data.The data's length I get is ( get_pedo_data_length() - *offset) but not (*offset < get_pedo_data_length() - *offset +1). In addtion, the determine condition should be *offset ( *offset < get_pedo_data_length() ), not ( *offset <=get_pedo_data_length() )

5. write "pedo_data[pedo_ctrl.diff_pointer[i] + count]" as "pedo_data[pedo_ctrl.diff_pointer[i + count] ]" and  "(time_diff >> 0/(time_diff >> 8 )/...)" as "(time_diff >> i*8 )"...Oh, shit...
