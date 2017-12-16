---
title: 实现listView单选及复用的坑
catalog: true
date: 2017-12-16 13:58:46
subtitle:
header-img: "img/header_img/code-bg2.jpg"
tags:
- listView
- 复用
---

大多数人看到标题有listView，肯定顿时心生鄙视：“这都什么年代了，还在用listView”。咳咳，大兄弟们先消消气。咱都知道谁的项目还没有点‘历史遗留问题’嘛，而且其实在功能不复杂界面也比较简单的时候，listView还是很好用的嘛～。那咱么就开始说正事：

这个listView的使用场景呢就是实现一个点击单选的功能，当然了假如需求是单选按钮加textView的形式，可以使用ArrayAdapter及listView的默认单选布局，使用方法如下：
