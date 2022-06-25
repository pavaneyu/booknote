---
title: MQ消息队列
date: 2022年2月20日15:44:48
tags: []
categories: 学习笔记
---



# 解耦 异步  削峰

[视频](https://www.bilibili.com/video/BV15Z4y1x7tE)

1.系统之间不产生依赖关系,增加和删除系统不会影响其他系统代码

<img src="https://s2.loli.net/2022/02/16/Jt6UwFTRLhxqXfk.png" alt="image-20220216202439238" style="zoom:80%;" />

2.不需要请求者等待,等结果处理完再返回结果

<img src="https://s2.loli.net/2022/02/16/BTZgFlOz7VSvXj4.png" alt="image-20220216211043584" style="zoom:80%;" />



3.流量缓冲

![image-20220216211135531](https://s2.loli.net/2022/02/16/ZVw8BJiGtIhqaWn.png)

缺点,  系统复杂度提高   重复消费问题?  消息丢失?消息顺序?













