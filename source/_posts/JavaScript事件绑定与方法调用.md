---
title: JavaScript事件绑定与方法调用
categories: JavaScript
abbrlink: 54209
date: 2018-10-08 16:05:45
tags:
---
# 事件绑定
1. 事件与对象绑定，而不是与引用绑定，如果引用的对象改变，事件也会随着改变。
```
<h1>h1</h1>
<button>button</button>

$('button').click(function(){
    alert('hello');
});

//remove方法会删除对象，然后重新创建，事件丢失。
$('button').remove();
$('h1').after('<button>button</button>');

//detach方法不会删除对象，所以调用after方法后也不会重新创建，事件不会丢失。
```
2. 方法调用和对象引用绑定，即动态分派，在运行时动态确定调用的方法。

