---
title: Javascript 函数式编程
date: 2017-08-15
tags: javascript
categories: notes
---


## 代码的组合 Compose


### 函数组合

```javascript
var compose = function(f,g) {
  return function(x) {
    return f(g(x));
  };
};
```

`f` 和 `g` 都是函数，`x` 是在它们之间通过“**管道**”传输的值。
在 `compose` 的定义中，`g` 将先于 `f` 执行，因此就创建了一个**从右到左**的数据流，称之为“左倾”。

组合的概念直接来自于结合律。
```javascript
// 结合律（associativity）
var associative = compose(f, compose(g, h)) == compose(compose(f, g), h);
// true
```
组合的最佳实践是可重用。


### pointfree

pointfree 模式指的是：定义函数时不使用所要处理的值，只合成运算过程。
```javascript
// 非 pointfree，因为提到了数据：word
var snakeCase = function (word) {
  return word.toLowerCase().replace(/\s+/ig, '_');
};

// pointfree
var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```


### 范畴论

有着以下这些组件（component）的搜集（collection）就构成了一个范畴：
- 对象
- 态射
- 态射的组合
- identity 这个独特的态射


#### 对象

对象就是数据类型，例如 `String`、`Boolean`、`Number` 和 `Object` 等等。
通常把数据类型视作所有可能的值的一个集合（set），然后可以利用集合论（set theory）处理类型。


#### 态射

态射是标准的、普通的纯函数。


#### 态射的组合

`compose` 函数是符合结合律的，结合律是在范畴学中对任何组合都适用的一个特性。
![态射的组合1](http://ofl97l8av.bkt.clouddn.com/17-8-14/4408562.jpg)
![态射的组合2](http://ofl97l8av.bkt.clouddn.com/17-8-14/4408562.jpg)


#### identity 的态射

```javascript
var id = function(x){ return x; };

// identity
compose(id, f) == compose(f, id) == f;
// true
```


## 类型签名

```javascript
//  capitalize :: String -> String
var capitalize = function(s){
  return toUpperCase(head(s)) + toLowerCase(tail(s));
}
```
`capitalize` 接受一个 `String` 并返回了一个 `String`。


```javascript
//  match :: Regex -> String -> [String]
var match = curry(function(reg, s){
  return s.match(reg);
});
```
match 函数接受一个 `Regex` 和一个 `String`，返回一个 `[String]`。


## 函子 Functor

函子是函数式编程里面最重要的数据类型，也是基本的运算单位和功能单位。
一般约定，函子的标志就是容器具有`map`方法。该方法将容器里面的每一个值，映射到另一个容器。
```javascript
class Functor {
    constructor(x) {
        this.__value = x;
    };

    map(f) {
        return new Functor(f(this.__value));
    };
}
```

函数式编程一般约定，函子有一个 `of` 方法，用来生成新的容器。
```javascript
Functor.of = function(x) {
    return new Functor(x)
};
```

用法的示例。
```javascript
const A = Functor.of(2);
A.map(function (x) {
    return x + 2;
});
// 4

const B = Functor.of("flamethrowers");
B.map(function (s) {
    return s.toUpperCase();
})
//FLAMETHROWERS
```


### Maybe 函子

函子接受各种函数，处理容器内部的值,外部函数未必有处理空值的机制，如果传入空值，很可能就会出错。
它的 `map` 方法里面设置了空值检查。
```javascript
class Maybe extends Functor {
    map(f) {
        return this.__value ? Maybe.of(f(this.__value)) : Maybe.of(null);
    }
}
```


### Either 函子

Either 函子内部有两个值：左值（`Left`）和右值（`Right`）。
右值是正常情况下使用的值，左值是右值不存在时使用的默认值。
```javascript
class Either extends Functor {
    constructor(left, right) {
        this.left = left;
        this.right = right;
    }

    map(f) {
        return this.right ? Either.of(f(this.right)) : Either.of(f(this.left));
    }
}

Either.of = function (left, right) {
    return new Either(left, right)
}
```

提供默认值。
```javascript
Either
    .of({
        address: 'xxx'
    }, currentUser.address)
    .map(updateField);
```

代替`try...catch`。
```javascript
function parseJSON(json) {
    try {
        return Either.of(null, JSON.parse(json));
    } catch (e: Error) {
        return Either.of(e, null);
    }
}
```


### ap 函子

函子 `A` 内部的值是 `2` ，函子 `B` 内部的值是函数`addTwo`。
我们想让函子 `B` 内部的函数，可以使用函子 `A` 内部的值进行运算。这时就需要用到 `ap` 函子。
凡是部署了 `ap` 方法的函子，就是 ap 函子。
```javascript
class Ap extends Functor{
    ap(F){
        return Ap.of(this.__value(F.__value))
    }
}
```
`ap` 方法的参数不是函数，而是另一个函子。

使用
```javascript
Ap.of(addTwo).ap(Functor.of(2))
// Ap(4)


function add(x) {
  return function (y) {
    return x + y;
  };
}

Ap.of(add).ap(Maybe.of(2)).ap(Maybe.of(3));
// Ap(5)
```

