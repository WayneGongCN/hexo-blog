---
title: 表单配置化思考
date: 2019-07-13
categories: thinking
keywords:
  - 表单配置化
  - 表格配置化
  - vue
  - element
  - form
  - table
  - Configuration
---



> 基于 Vue、Element，对表单配置化的 [一个实现 ![conf-table-form-lib](https://img.shields.io/github/stars/maggie-wayne/conf-table-form-lib.svg?style=social)](https://github.com/maggie-wayne/conf-table-form-lib)



在偏后台系统的界面中，大多数页面都是表单、表格，这类页面占据了大多数开发时间。

业务逻辑也通常集中在表单、表格上，字段多、改动频繁、维护成本高。

如果能把这一部分独立出来，代码会更加条理清晰、可维护，也许能减少一些工作量？



## 组件化 Vs 配置化

如何把表单的渲染与表单的业务逻辑从页面中剥离出来？

让页面代码容器化，专注与获取、处理数据承载组件以及做一些与业务无关的事情，尽量减少后期需求变更对这一部分代码的修改。




### ~~组件化~~

一份表单往往对应一个具体的业务场景，如果将带有业务逻辑的表单封装成一个个表单组件，复用程度很低。

要将一个表单组件用于多个类似的业务场景。
比如表单组件 A 用于：`Create User`、`Update User`、`User Detail`。

因为组件 A 要兼容三个业务场景，内部将会充斥着各种 `if...else`。

后期需求一旦变更只能硬着头皮往后继续 `if...else` 。甚至一处修改 N 处 Bug。

这种程度的代码复用往往不如直接硬编码来得实在。




### 配置化

为什么不喜欢业务逻辑？

业务逻辑难抽象、难复用、修改频繁，本质上就是 `if...else` 。

与其想方设法让不同业务逻辑挤在同一个组件，不如让他们一个萝卜一个坑。

用一份配置文件描述表单与逻辑，然后通过某个公共组件渲染。

这样下来，一个业务场景对应一个表单，对应一份配置文件，使用同一个组件渲染。

配置文件该如何定义才能足够灵活？渲染组件如何应对各种配置？
