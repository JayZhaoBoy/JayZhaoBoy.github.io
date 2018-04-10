---
title: JavaScript闭包学习及引申思考的总结
catalog: true
date: 2018-04-10 15:37:14
subtitle:
header-img: "/img/header_img/code.jpg"
tags:
- JavaScript
---

JavaScript闭包学习及引申思考的总结
====

最近准备入坑 RN，所以系统学习了 js 基础。闭包又是一个比较 js 中比较重要的概念。当然是要了解一下的，然后也发现总结了一些比较有意思的事情。

看了一些资料，这里总结一下，知乎上有人总结了闭包的几个关键词：
* 函数
* 自由变量
* 环境  
形成条件是：词法作用域和函数当做值传递。
用阮一峰老师的话总结来说js中的闭包就是能访问其他函数内部变量的函数。详细可参考  阮一峰[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

所以闭包是一个函数，一个特殊一点的函数，上例子（例子均来自阮一峰博客）


```
function f1(){

　　　　var n=999;

　　　　function f2(){
　　　　　　alert(n); 
　　　　}

　　　　return f2;

　　}

　　var result=f1();

　　result(); // 999
```
我们通过 f2 函数访问到了 f1 中的私有变量，f2 函数就是一个闭包。
其实在这里的时候我还是自认为明白了「闭包」的概念。内部可以访问外部变量，反之则不行，这种「链式作用域」的结构与 java 不是一样的嘛，只不过前面加 var 就是局部的，不加就是全局的。然而看到下面的几个例子～
****

第一个例子：

```
function f1(){

　　　　var n=999;

　　　　nAdd=function(){n+=1}

　　　　function f2(){
　　　　　　alert(n);
　　　　}

　　　　return f2;

　　}

　　var result=f1();

　　result(); // 999

　　nAdd();

　　result(); // 1000
```
我最开始以java的逻辑思维去思考：无论你是否 nAdd了，你最后调用 result 函数的时候，不就相当于调用了 f1 函数吗， 所以 n 的值不都是被初始化为 999 了吗，当然我知道我的想法肯定是不对的，所以 debug 一步步的走了一下。恍然大悟，顿时有一种，如果说 java 面向对象万物皆对象的话， Js真的是万物皆变量，尤其是函数变量作为值传递。rensult = f1（）；确实是相当于进行了实力化操作，但因为 f1 函数返回的结果是 一个函数 f2，用java的话来说：就相当于 result 的引用指向了函数 f2.  变量 result 本事就是一个 function 类型的变量，result（）就等于 直接调用 f2（）。在 JS 中，就函数可以作为值传递这一项来说，真的是灵活极了。
****

还有就是最后两个例子：

```
　　var name = "The Window";

　　var object = {
　　　　name : "My Object",

　　　　getNameFunc : function(){
　　　　　　return function(){
　　　　　　　　return this.name;
　　　　　　};

　　　　}

　　};

　　alert(object.getNameFunc()());
```

```
　var name = "The Window";

　　var object = {
　　　　name : "My Object",

　　　　getNameFunc : function(){
　　　　　　var that = this;
　　　　　　return function(){
　　　　　　　　return that.name;
　　　　　　};

　　　　}

　　};

　　alert(object.getNameFunc()());
```
结果当然是：
第一个例子： The window
第二个例子： My Object

第二个例子不难理解。第一个例子我当时第一时间确实还是想错了，但是看到结果在回头想一下也就明白了。这里我们可以结合 Java 的知识来理解一下，其实在这里面不考虑全局变量的时候，局部变量的链式作用域就相当于 java 中的继承关系一样。而且 java 也是支持闭包的，而且在 java 中处处都是闭包，只是无法明确感知到。还有还记得 java 中的匿名内部类引用外部变量为什么要用 final 修饰吗？具体可参考知乎上的第一条回答[https://www.zhihu.com/question/21395848](https://www.zhihu.com/question/21395848)

看完之后回到例子上来，如果要想理清楚这个问题，就要明白 this 的调用环境。首先我们要知道：
* js中匿名函数以及没有 var 修饰的变量是去全局的
* 这个全局的 object 就是 window


所以我们稍微修改一下例子一：

```
　　var name = "The Window";

　　var object = {
　　　　name : "My Object",

　　　　getNameFunc : function(){
　　　　　　return function(){
　　　　　　　　return Window.this.name; //只是修改了这一行
　　　　　　};

　　　　}

　　};

　　alert(object.getNameFunc()());
```
这样就十分清楚了。

```
　return function(){
　　　　　　　　return Window.this.name; //只是修改了这一行
　　　　　　};
```
匿名函数的所属环境是全局的，也就是 window 的，所以 this 并不是 object.this 而是 window.this。
