---
title: WebAssembly 初体验
date: 2020-01-19
tags:
categories: FE
---


## WebAssembly 是什么

>WebAssembly 是一种新的编码方式，可以在现代的网络浏览器中运行 － 它是一种低级的类汇编语言，具有紧凑的二进制格式，可以接近原生的性能运行，并为诸如C / C ++等语言提供一个编译目标，以便它们可以在Web上运行。

个人理解：有了 WebAssembly 浏览器可以执行通过其他语言编译而来的二进制代码，并接近原生性能。



## Hello world

详细步骤参考： [https://webassembly.org/getting-started/developers-guide/](https://webassembly.org/getting-started/developers-guide/)

简单总结一下步骤：

1. 下载相关工具
```
$ git clone https://github.com/emscripten-core/emsdk.git
$ cd emsdk
$ ./emsdk install latest
$ ./emsdk activate latest
```

2. 设置环境变量: `source ./emsdk_env.sh --build=Release`

3. 编码
`hello.c :`
```c
#include <stdio.h>

int main (int argc, char ** argv) {
  printf("Hello world!\n");
}
```

4. 编译: `emcc hello.c -o hello.html`
5. 起一个 WebServer: `emrun --no_browser --port 8080 .`
6. 打开 `localhost:8080` 查看
