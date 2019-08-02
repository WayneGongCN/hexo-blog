---
title: Javascript 五子棋
date: 2017-09-22
tags: javascript
categories: notes
---

## 简单的 Javascript 五子棋 Demo

---

Dome: [https://zowiegong.github.io/javascript-five-chess/](https://zowiegong.github.io/javascript-five-chess/)

Repo: [https://github.com/helloAwei/javascript-five-chess](https://github.com/helloAwei/javascript-five-chess)


### 简介

- 实现胜负判断，并给出赢棋提示。
- 界面用 DOM 实现
- 实现悔棋

前端用 [Knockout](http://knockoutjs.com/) 做双向绑定。

思路：用一个二维数组生成棋盘，对格子添加点击事件，用一个布尔值进行切换黑白轮流落子，落子后对当前棋子进行胜负判断 ...


### 生成二维数组

```javascript
function GeneratorList() {
    let array = new Array(ROW).fill(null);

    //生成二维数组
    let result = array.map((value, y) => {
        return array.map((value, x) => {
            return {
                hasChess: ko.observable(false),
                value: ko.observable(null),
                x: x,
                y: y
            }
        });
    });
    return result;
}
```
生成一个 15*15 的二维数组，`List[y][x]`能取到对应格子的状态。
初始的状态为：
```javascript
{
    hasChess: ko.observable(false),    //是否有棋子
    value: ko.observable(null),        //棋子的值（黑或白）
    x: x,                              //坐标
    y: y
}
```
这里没有单独去定义一个棋子类，直接用的字面量形式。


### 落子

```javascript
ViewModel.prototype.down = function (data) {
    if (data.hasChess()) return;
    if (this.winer()) this.rePlay();
    // change
    data.hasChess(true);
    data.value(this.isBlack() ? 'X' : 'O');

    // add history
    this.history.push({
        x: data.x,
        y: data.y
    });

    //判断输赢
    this.referee(data);

    //换手
    this.isBlack(!this.isBlack());
}
```
落子前进行判断，落子，然后添加一条历史记录，判断输赢，换手。。。


### 判断输赢的算法

```javascript
ViewModel.prototype.referee = function (item) {
    //四个方向
    let change = [[0, -1], [-1, 0], [-1, -1], [1, -1]];

    change.forEach(c => {
        //朝某个方向递归
        this.test(item, c);

        //不足五连珠朝反方向递归
        if (this.lineTotal <= 4) {
            //反方向
            let reveis = c.map(x => {
                return x * -1;
            });

            this.test(item, reveis)
            this.lineTotal = 0
        };
    })
}
```

判断胜负需要检查一个棋子的八个方向上有无五连珠的情况。
![](http://ofl97l8av.bkt.clouddn.com/17-9-22/69929859.jpg)

`change`是一个保存了四个检查方向的二维数组。
在朝某个方向检查完之后然后反向的检查，这样能够将八个方向缩减为四个。
例： 1、2、3、4 反向后对应 5、6、7、8

如果某个方向不满足五连珠的情况（代码中为<=4，因为检查不包括本身），则向反向的方向进行检查。

下面是一个递归检查的方法：
```javascript
//判断连子
ViewModel.prototype.test = function (item, change, total = 0) {
    let cx = item.x + change[0];
    let cy = item.y + change[1];

    let isInBoard = (cx >= 0 && cx <= ROW - 1) && (cy >= 0 && cy <= ROW - 1);
    let cItem = isInBoard && this.allBoxs()[cy][cx];
    let isNull = cItem !== null;

    if (isNull &&
        isInBoard &&
        item.value() === cItem.value()
    ) {
        total += 1;
        //递归
        this.test(cItem, change, total);
    } else {
        //跳出
        this.lineTotal += total;
    }

    //win
    if (this.lineTotal >= 4) {
        this.win();
    };
}
```
直到下一个棋子不是与本身同类型的棋子便跳出递归，同时将`total` 返回。

完。