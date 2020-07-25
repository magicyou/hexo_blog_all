---
title: jQ的链式操作是怎么实现的
date: 2017-08-14 12:13:02
updated: 2017-08-15 12:13:04
tags: [jQuery]
categories: [前端]
---
天天用jQuery，“$('id').find('span').parents('.box-wrapper').html('test')” 链式操作如此方便是怎么实现的？

<!--more-->

#### 1. 实现原理：
这里使用原生js来模拟这样的效果：

```
let fun = {
    fun1: function() {
        console.log("fun1");
        return this;
    },

    fun2: function() {
        console.log("fun2");
        return this;
    },
     
    fun3: function() {
        console.log("fun3");
        return this;
    }
}

fun.fun1().fun2().fun3();
```
这样就可以连续的输出字符串的fun1、fun2、fun3 了，原因是在每个方法后面加了一个return this。

如果没有加上return this语句的话，那么执行完一个函数之后，会默认返回undefined，这个是js内部自己隐式添加的。返回undefined的时候，再调用另一个方法肯定就会报错，因为undefined是没有方法的。

#### 2. 为什么要返回this？
因为在一个对象里面，this指向的是对象本身，而我们连续调用方法的时候，这些方法都是在对象内部定义的，所以this是可以访问到这些方法

例子：

```
function jQ(selector){
  this.elem = document.querySelectorAll(selector)[0];
}
jQ.prototype = {
   html: function(strHtml) {
    let elem = this.elem;
    elem.innerHTML = strHtml;
    return this;
   },
   on: function(type, func) {
    let elem = this.elem;
    elem.addEventListener(type, func)
    return this
   }
};

window.$ = function(selector) {
  return new jQ(selector);
}
console.log($('#test_demo'));
$('#test_demo').html('<a>111</a>').on('click', function() {
  console.log('click');
})
```






