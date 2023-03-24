---
title: for 循环与 forEach 的区别 ?
date: 2023/02/08
tags: 面试,问答
categories: 每日一问
description:  一个用来等待和发送消息和事件的程序结构。
keywords: js、for、forEach
cover: /img/md/js.jpg
---

# 知识点
1. for 循环可以使用 break 跳出循环，但 forEach 不能。
2. for 循环可以控制循环起点（i初始化的数字决定循环的起点）， forEach 只能默认从索引 0 开始。
3. for 循环过程中支持修改索引（修改 i ），但 forEach 做不到（底层控制 index 自增，无法左右它）。
