---
layout: post
title: "Android Tips"
description: "关于Handler,Looper,Thread,Runnable"
category: 
tags: [android]
---
{% include JB/setup %}

关于Handler Looper Thread Runnable
=============================

之前对Handler Looper Thread Runnable这几个对象一直不太明白，今天在StackOverflow
上突然下就明白了

. Handler: 是和消息队列关联起来的一个桥梁，自动和创建该Handler的线程消息队列
    关联起来。可通过post和send将Runnable和Message发送到MessageQueue处理。可以
    响应processMessage自己处理消息，因此也可以作为两个线程间通信的桥梁

. Looper: 就相当于Windows下的消息循环，默认Thread没有Looper

. Thread: 线程，Handler不会新起线程

. Runnable: 相当于定义执行动作的地方
