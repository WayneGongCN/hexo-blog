---
title: 尾调用优化
date: 2018-10-11
tags:
categories: other
---

当函数的调用层数非常多时，需要同时保存成千上百个调用记录，调用栈会消耗不少内存，甚容易导致[栈溢出](https://zh.wikipedia.org/wiki/%E6%A0%88%E6%BA%A2%E5%87%BA)。

尾调用的[调用栈](https://zh.wikipedia.org/wiki/%E5%91%BC%E5%8F%AB%E5%A0%86%E7%96%8A)则特别易于优化，从而可减少内存空间的使用，也能提高运行速度。


## 尾调用

**尾调用** (tail call) 指的是一个函数的最后一条语句是一个返回调用函数的语句。


```javascript
// 属于尾调用
function bar(data) {
    if ( a(data) ) {
        return b(data);
    }
    return c(data);
}

function foo(data) {
    a(data);
    return b(data);
}
```

```javascript
// 不属于尾调用
function foo1(data) {
    return a(data) + 1;
}

function foo2(data) {
    var ret = a(data);
    return ret;
}
```

### 调用栈

当一个函数被调用时，函数的调用位置与返回位置被保存在调用栈中，调用结束时才能带着返回值回到该位置。

在尾调用这种特殊情形中，计算机理论上可以不需要记住尾调用的位置而从被调用的函数直接带着返回值返回调用函数的返回位置。

尾调用消除即是在不改变当前调用栈（也不添加新的返回位置）的情况下跳到新函数的一种优化。


## 尾递归

在尾调用的情况下**调用自身**的特殊尾调用称为**尾递归**。


### 优化尾递归的分析

由于函数自身调用次数很多，递归层级很深，尾递归优化则使原本 O(n) 的调用栈空间只需要 O(1)。

"尾调用优化"对递归操作意义重大，ES6 中明确规定，所有 ECMAScript 的实现，都必须部署"尾调用优化"。

所以在 ES6 中，只要使用尾递归，就不会发生栈溢出，相对节省内存。

现有`recsum`函数如下：

```javascript
function recsum (num) {
    if (num === 1) return 1
    else return num + recsum(num - 1)
}
```

调用`recsum(5)`为例:

```
recsum(5)
5 + recsum(4)
5 + (4 + recsum(3))
5 + (4 + (3 + recsum(2)))
5 + (4 + (3 + (2 + recsum(1))))
5 + (4 + (3 + (2 + 1)))
5 + (4 + (3 + 3))
5 + (4 + 6)
5 + 10
15
```

堆栈从左到右，增加到一个峰值后再计算从右到左缩小。

修改以上代码，可以成为尾递归：

```javascript
function tailrecsum (num, running_total=0) {
    if (num === 0) return running_total
    else return tailrecsum(num - 1, running_total + num)
}
```

后者尾递归对内存的消耗

```
tailrecsum(5, 0) 
tailrecsum(4, 5) 
tailrecsum(3, 9)
tailrecsum(2, 12) 
tailrecsum(1, 14) 
tailrecsum(0, 15) 
15
```


### 递归函数的修改

由于要实现尾调用优化，函数的调用栈不在增加所以无法保持函数的局部变量，需要将局部变量以参数的形式传入，如上面的例子，来实现尾调用的优化。

递归本质上是一种循环操作。纯粹的函数式编程语言没有循环操作命令，所有的循环都用递归实现，这就是为什么尾递归对这些语言极其重要。

对于其他支持"尾调用优化"的语言（比如Lua，ES6），只需要知道循环可以用递归代替，而一旦使用递归，就最好使用尾递归。