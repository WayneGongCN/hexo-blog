---
title:  LED 限流电阻计算
date: 2018-07-03
tags:
categories: other
---


LED 发光二极管限流电阻阻值计算。

## 一些参数
- `PM` 允许功耗，超过会烧毁
- `IFM` 最大正向电流，超过会烧毁
- `VRM` 最大反向电压，超过会击穿
- `IF` 正向导通电流，一般为 `0.6 * IFM`
- `VF` 正向导通电压，一般为 `IF = 20mA` 时取得，`1.4V - 3V`不同颜色电压不同

LED 正常工作不应超过 `PM`、`IFM`、`VRM` 这三个极限参数。

## 限流电阻计算
![原理图](https://raw.githubusercontent.com/zowiegong/hexo-blog/master/source/images/20190722121759.png)

假设 LED1 `VF=3V`、`IF=20mA`，电源电压为 `Vcc=5V`。

则 `R1` 上分得电压 `Ur1 = Vcc - VF = 5 - 3 = 2V`

要是使得 `LED1` 上的电流为 `20mA`，根据 `R = U / I` 

则 `R1` 的电阻值应该为 `R1 = Ur1 / IF = 2 / 0.02 = 100Ω`

## 计算公式

得到 LED 限流电阻计算公式为：`R = (UV - VF) / IF`

- `R` 限流电阻阻值
- `UV` 电源电压
- `VF` LED 正向导通电压
- `IF` LED 正向导通电流

## 其他
[LED限流电阻在线计算](http://www.dianziaihaozhe.com/gongju/LED_xianliudianzu/)

原理图由 [EasyEDA](https://easyeda.com) 绘制
