---
title: Webassembly-Introduce
date: 2019-05-26 17:32:59
categories: 编程
tags: webassembly
---

我们知道 JavaScript 是一门脚本语言，具有动态类型和灵活的表达力，我们知道脚本语言通常要解释运行，这也将要消耗一些性能开销，于是 Google 在 2009 年在 V8 中引入了 JIT(Just in time compiling) 技术，把 JavaScript 运行时的性能推到了顶峰，同年利用 V8 引擎诞生了 Node.js，打开了使用 JavaScript 写后端应用的大门。对于目前写网络应用而言 JavaScript 已经足够用了，加上 Google V8 引擎能帮我们解决掉大部分问题。但是当我们把 JavaScript 应用到诸如 3D 游戏、虚拟现实、增强现实、计算机视觉、图像/视频编辑以及大量的要求原生性能的其他领域的时候，就遇到了性能问题，尤其是移动平台进一步放大了这些性能瓶颈。

而 WebAssembly 的出现就是为了解决这个问题，它是一门低级的类汇编语言，可以运行在现代网络浏览器中的新型代码并且提供新的性能特性和效果。它设计的目的不是为了手写代码，而是为了诸如 C、C++ 和 Rust 等低级源语言提供一个高效的编译目标以便它们能够在网络上运行。对于网络平台而言，这具有巨大的意义——这为客户端 app 提供了一种在网络平台以接近本地速度的方式运行多种语言编写代码的方式；在这之前，客户端 app 是不可能做到的。

## 文件格式与生成

WebAssembly 有二进制 (.wasm) 和文本 (.wast) 两种格式，并且两种格式可以互享转换，在传输和运行时使用二进制格式，而文本格式是为了阅读和开发调试所使用。

以 C 为例，首先把 C 程序编译为 WebAssembly 格式，本地编译可以查看 编译 [C/C++ 为 WebAssembly](https://developer.mozilla.org/zh-CN/docs/WebAssembly/C_to_wasm) 文章进行相关软件的安装，为了方便可以直接使用这款在线工具进行转换 [WasmFiddle](https://wasdk.github.io/WasmFiddle/)

我们先来个简单的，在 main 中返回一个数字，在 add 方法中求和:
```c
int main() { 
  return 42;
}

int add(int a, int b) {
  return a + b;
}
```

转换过后的 .wast 文本格式为，我们可以从这个文本看出分别 export 了 main 与 add 方法
```
(module
 (table 0 anyfunc)
 (memory $0 1)
 (export "memory" (memory $0))
 (export "main" (func $main))
 (export "add" (func $add))
 (func $main (; 0 ;) (result i32)
  (i32.const 42)
 )
 (func $add (; 1 ;) (param $0 i32) (param $1 i32) (result i32)
  (i32.add
   (get_local $1)
   (get_local $0)
  )
 )
)
```

## 导入并初始化

使用 Fetch 获取模块并初始化调用:
```js
fetch('module.wasm')
  .then(response => response.arrayBuffer())
  .then(bytes => WebAssembly.instantiate(bytes))
  .then(results => {
    console.log(typeof results.instance.exports.main); // function
    console.log(results.instance.exports.main()); // 42
    console.log(results.instance.exports.add(1, 2)); // 3
  })
```

使用 XMLHttpRequest 获取模块并初始化调用:
```js
const request = new XMLHttpRequest();
request.open('GET', 'module.wasm');
request.responseType = 'arraybuffer';

request.onload = function() {
  var bytes = request.response;
  WebAssembly
  .instantiate(bytes)
  .then(results => {
    console.log(typeof results.instance.exports.main);
    console.log(results.instance.exports.main());
    console.log(results.instance.exports.add(1, 2));
  });
};
request.send();
```

可以畅想下以后 WebAssembly 普及的话，到时我们用的 React、Vue、Angular 可能就是用 C/C++ 或者其他底层语言写的。