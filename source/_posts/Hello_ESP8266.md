---
title: ESP8266 初体验
date: 2020-01-19
tags:
categories: other
---

## ESP8266 是什么？

> ESP8266 是一款由上海乐鑫信息科技开发的可以作为微控制器使用的成本极低且具有完整TCP/IP协议栈的Wi-Fi IoT控制芯片。

简单来说就是这块芯片能连 WiFi 、能编程控制 GPIO。


### ESP8266 开发板是什么？

但是单独对一块芯片进行开发调试会比较麻烦，需要连接串口、电源等等。
市面上有许多 ESP8266 的开发版，使用开发版我们只需要通过 USB 接口连接开发版，就可以方便的进行开发、烧录、调试。

我这里使用的是 NodeMcu 开发板。


### NodeMcu 是什么？

> NodeMCU 是一个开源的物联网平台。 它使用Lua脚本语言编程。NodeMCU 包含了可以运行在 esp8266 Wi-Fi SoC芯片之上的固件,以及基于ESP-12模块的硬件。

简单来说 NodeMcu 基于是 ESP8266 硬件（开发板），并提供一套固件让我们可以使用 Lua 脚本进行开发。

不过我虽然使用的是 NodeMcu 开发板，但使用的是 MicroPython 进行开发。


### MicroPython 是什么？

> MicroPython，是Python 3编程语言的一个完整软件实现，用C语言编写，被优化于运行在微控制器之上。

和 NodeMcu 一样，MicroPython 是跑在 ESP8266 上的一个“操作系统”，我们可以通过 python 语言对硬件进行开发。


## Hello MicroPython

由于 ESP8266 本身并没有安装 “MicroPython 操作系统”，首先我们得进行固件的烧录（安装操作系统）。

详细的步骤参考 MicroPython 的 [**文档**](http://docs.micropython.org/en/latest/esp8266/quickref.html)

简单总结一下步骤：
1. 安装串口驱动
2. 安装烧录工具 `pip install esptool`
3. 下载 ESP8266 [固件](http://micropython.org/download#esp8266)
4. 抹掉自带固件 `esptool.py --port /dev/ttyUSB0 erase_flash` (`/dev/ttyUSB0` 为串口硬件)
5. 烧录固件 `esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 固件目录`
6. 通过串口进入 REPL 环境
