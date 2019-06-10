---
title: Webassembly-Rust
date: 2019-06-10 07:54:44
categories: 编程
tags: webassembly, rust
---

上一篇文章有简单介绍Webassembly格式, 以及利用在线转化工具把C代码编辑到Webassembly. 这一篇文章我们继续探讨如何将Rust语言源代码编译到Webassembly并在谷歌浏览器中运行。Rust是火狐公司开发出来的一款静态编译型语言, Rust语言的详细介绍可以看[官网](https://www.rust-lang.org/).

之前例子在实际项目中的应用还要很多工作要做，比如如何在C中调用浏览器环境中的API，JS引擎的数据结构如何传递到C中等等，相较于之前介绍C语言编译到Webassembly并在浏览器运行的简单尝试，Rust社区一直积极的推动Webassembly于web端实际应用，并开发出一系列工具与类库。

## 安装 Rust 环境

### 安装Rust
前往 [Install Rust](https://www.rust-lang.org/install.html) 页面并跟随指示安装 Rust。这里会安装一个名为 “rustup” 的工具，这个工具能让你管理多个不同版本的 Rust。默认情况下，它会安装用于惯常Rust开发的 stable 版本 Rust Release。Rustup 会安装 Rust 的编译器 rustc、Rust 的包管理工具 cargo、Rust 的标准库 rust-std 以及一些有用的文档 rust-docs。

### 安装wasm-pack
要将Rust模块构建为前端npm包，我们需要一个额外工具 wasm-pack。它会帮助我们把我们的代码编译成 WebAssembly 并制造出正确的 npm 包。使用下面的命令可以下载并安装它：
```txt
$ cargo install wasm-pack
```

## 构建我们的 WebAssembly npm 包
创建一个新的 Rust 包。打开你用来存放你私人项目的目录，做这些事：
```txt
$ cargo new --lib wasm-test
     Created library `wasm-test` project
```
这里会在名为 wasm-test 的子目录里创建一个新的库，里面有下一步之前你所需要的一切：
```txt
+-- Cargo.toml
+-- src
    +-- lib.rs
```
这里有一个 Cargo.toml 文件，这是我们配置构建的方式。如果你用过 npm 的 package.json，你应该会感到很熟悉。Cargo 的用法和它们类似。
接下来，Cargo 在 src/lib.rs 生成了一些 Rust 代码：

### 来写点 Rust 代码
在 src/lib.rs 写一些代码替换掉原来的：
```rust
extern crate wasm_bindgen; // 申明外部依赖库

use wasm_bindgen::prelude::*; // 导入外部依赖库

/* 
在 #[] 中的内容叫做 "属性"，并以某种方式改变下面的语句。在这种情况下，下面的语句是一个 extern，
它将告诉 Rust 我们想调用一些外部定义的函数。这个属性 wasm-bindgen 告诉我们如何找到这些函数
 */
#[wasm_bindgen]
extern { 
    pub fn alert(s: &str);
}

/* 
我们又看到了 #[wasm_bindgen] 属性。在这里，它并非定义一个 extern 块，而是 fn，
这代表我们希望能够在 JavaScript 中使用这个 Rust 函数。 这和 extern 正相反：我们并非引入函数，而是要把函数给外部世界使用。
 */
#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("Hello, {}!", name));
}
```

### 把我们的代码编译到 WebAssembly
为了能够正确的编译我们的代码，首先我们需要配置 Cargo.toml。打开这个文件，将内容改为如下所示:
```txt
[package]
name = "wasm-test"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
description = "A sample project with wasm-pack"
license = "MIT/Apache-2.0"
repository = "https://github.com/yourgithubusername/wasm-test"

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```
最重要的是添加底下的部分。第一个部分 — [lib] — 告诉 Rust 为我们的包建立一个 cdylib
[dependencies] 部分。在这里我们告诉 Cargo 我们需要依赖哪个版本的 wasm-bindgen

### 构建NPM包
Rust源代码完成，依赖配置也设置好了，准备构建NPM前端模块！
```txt
$ wasm-pack build
```
wasm-pack build 会进行如下几个步骤：
1. 把Rust代码编译为 WebAssembly.
2. 在生成的 WebAssembly文件执行 wasm-bindgen , 生成一个npm能识别JS模块，这个模块包含刚生成WebAssembly文件.
3. 生成一个 pkg 文件夹，并把生成的JS模块与WebAssembly文件移入.
4. 读取 Cargo.toml 并生成对应的 package.json.
5. 拷贝 README.md（如果有的话）到 pkg文件夹.


## 在网站上使用刚生成的包

新建一个前端文件夹site，并运行：
```txt
$ npm init -y
```
编译package.json
```json
{
  "scripts": {
    "serve": "webpack-dev-server"
  },
  "devDependencies": {
    "webpack": "^4.25.1",
    "webpack-cli": "^3.1.2",
    "webpack-dev-server": "^3.1.10"
  }
}
```
新建一个webpack.config.js文件，内容如下：
```js
const path = require('path');
module.exports = {
  entry: "./index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.js",
  },
  mode: "development"
};
```
新建一个index.html文件，内容如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>hello-wasm example</title>
  </head>
  <body>
    <script src="./index.js"></script>
  </body>
</html>
```
新建一个index.js文件，内容如下：
```js
const js = import("./pkg/wasm-test.js");
js.then(js => {
  js.greet("WebAssembly");
});
```
打开控制台，运行：
```cmd
$ npm install
$ npm run serve
```
打开谷歌浏览器，访问http://localhost:8080 应该会看到一个alert弹窗。