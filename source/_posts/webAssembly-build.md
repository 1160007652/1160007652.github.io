---
title: webAssembly
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-08-26 09:37:54
tags:
    - webAssembly
copyright:
password:
---

## What WebAssembly ?

- WebAssembly简称WASM，是一种以安全有效的方式运行便携式程序的新技术，主要针对Web平台。与ASM.js类似，WASM的目标是低级别的抽象，适合作为更高级别程序的中间表示 - 即WebAssembly代码旨在由编译器生成而不是由人类编写。在W3C社区组包括来自最大的网络浏览器的公司，包括谷歌，微软，苹果和Mozilla做这件事，而令人兴奋的代表。

- WebAssembly是一种新的适合于编译到Web的，可移植的，大小和加载时间高效的格式，是一种新的字节码格式。它的缩写是”.wasm”，.wasm 为文件名后缀，是一种新的底层安全的“二进制”语法。它被定义为“精简、加载时间短的格式和执行模型”，并且被设计为Web 多编程语言目标文件格式。 
这意味着浏览器端的性能会得到极大提升，它也使得我们能够实现一个底层构建模块的集合.

<!-- more -->

## webAssembly的优势

webassembly相较于asm.js的优势主要是涉及到性能方面。根据WebAssembly FAQ的描述：在移动设备上，对于很大的代码库，asm.js仅仅解析就需要花费20-40秒，而实验显示WebAssembly的加载速度比asm.js快了20倍，这主要是因为相比解析 asm.js 代码，JavaScript 引擎破译二进制格式的速度要快得多。

主流的浏览器目前均支持webAssembly。

- Safari 支持 WebAssembly的第一个版本是 11
- Edge 支持 WebAssembly的第一个版本是 16
- Firefox 支持 WebAssembly的第一个版本是 52
- chrome 支持 WebAssembly的第一个版本是 57

使用WebAssembly，我们可以在浏览器中运行一些高性能、低级别的编程语言，可用它将大型的C和C++代码库比如游戏、物理引擎甚至是桌面应用程序导入Web平台。

## 开发前准备工作

### 操作系统

> MAC系统

### 安装工具

> git 、node 、python 、cmake、emsdk

#### git

已经默认装

#### node



#### python

官网：https://www.python.org
下载地址：https://www.python.org/ftp/python/3.7.0/python-3.7.0-macosx10.9.pkg

#### cmake
 
 官网：http://www.cmake.org
 下载地址：https://cmake.org/files/v3.12/cmake-3.12.1-Darwin-x86_64.dmg

 为了可以在命令行中使用，需要执行一下命令。
 sudo "/Applications/CMake.app/Contents/bin/cmake-gui" --install

#### emsdk

Emscripten 的底层是 LLVM 编译器，理论上任何可以生成 LLVM IR（Intermediate Representation）的语言，都可以编译生成 asm.js。 但是实际上，Emscripten 几乎只用于将 C / C++ 代码编译生成 asm.js。

> ** C/C++ ** ⇒ ** LLVM ** ==> ** LLVM IR ** ⇒ ** Emscripten ** ⇒ ** asm.js **

``` bash
git clone https://github.com/juj/emsdk.git

cd emsdk

./emsdk update
./emsdk install latest
./emsdk activate latest

source ./emsdk_env.sh

```

### 浏览器支持

> Chrome: 打开 chrome://flags/#enable-webassembly，选择 enable。
> Firefox: 打开 about:config 将 javascript.options.wasm 设置为 true。

## 开始编程

### 编写C代码

创建** example.c ** 文件，并写入如下代码

``` C 
// example.c

float add(float a,float b){
return a+b;
}

```

### 编译出wasm

** bash 执行命令： **

``` bash 
emcc -s EXPORTED_FUNCTIONS="['_add]" example.c -s WASM=1 -s SIDE_MODULE=1 -s BINARYEN_ASYNC_COMPILATION=0 -o example.wasm 
```

### 调用wasm

> js调用wasm中导出的module

** loader模块编写 **

``` javascript
function loadWebAssembly(path, imports = {}) {
  return fetch(path)
    .then(response => response.arrayBuffer())
    .then(buffer => WebAssembly.compile(buffer))
    .then(module => {
      //针对wasm所需的import处理，importObject
      imports.env = imports.env || {}
      imports.env = imports.env || {}
      imports.env.DYNAMICTOP_PTR = imports.env.DYNAMICTOP_PTR || 0;
      imports.env.tempDoublePtr = imports.env.tempDoublePtr || 0;
      imports.env.ABORT = imports.env.ABORT || 0;
      imports.global = imports.global || { NaN: 1, Infinity: 2 };
      imports.env.abortStackOverflow = imports.env.abortStackOverflow || new Function();
      imports.env.nullFunc_X = imports.env.nullFunc_X || new Function();
      // 开辟内存空间
      imports.env.memoryBase = imports.env.memoryBase || 0
      if (!imports.env.memory) {
        imports.env.memory = new WebAssembly.Memory({ initial: 256 })
      }
      // 创建变量映射表
      imports.env.tableBase = imports.env.tableBase || 0
      if (!imports.env.table) {
        imports.env.table = new WebAssembly.Table({ initial: 2, element: 'anyfunc' })
      }
      // 创建 WebAssembly 实例
      return new WebAssembly.instantiate(module, imports)
    })
}
```

### html片段

``` html
<button class="mybutton">取随机数</button>
    <script>
        const wasm = loadWebAssembly('example.wasm');
        let myButton = document.getElementsByClassName('mybutton')[0];
        const x = 1.22;
        const y = 2.11;
        myButton.onclick = function () {
            wasm.then(module => {
                console.log('wasm-计算：' + module.exports._add(x, y));
                console.log('js-计算：' + (x + y));
            });
        }
    </script>
```

## WebAssembly组成部分

> 实际上** WebAssembly **被称为 "模块"，因为使用WebAssembly，"程序"和"库"之间没有区别 - 只有"模块"，每个模块都可以与其他模块绑定和通信，每个模块都可以具有"模块"主功能。

### 必选部分

- Type：在模块中定义的函数的函数声明和所有引入函数的函数声明。
- Function：给出模块中每个函数一个索引。
- Code：模块中每个函数的实际函数体。

### 可选部分

- Export：使函数、内存、表（tables）、全局变量等对其他 WebAssembly 或 JavaScript 可见，允许动态链接一些分开编译的组件，即 .dll 的WebAssembly 版本。
- Import：允许从其他 WebAssembly 或者 JavaScript 中导入指定的函数、内存、表或者全局变量。
- Start：当 WebAssembly 模块加载进来的时候，可以自动运行的函数（类似于 main 函数）。
- Global：声明模块的全局变量。
- Memory：定义模块用到的内存。
- Table：使得可以映射到 WebAssembly 模块以外的值，如映射到 JavaScript 的对象。这在间接函数调用时很有用。
- Data：初始化导入的或者局部内存。
- Element：初始化导入的或者局部的表。

## 遇到的坑

### 无法导入环境变量

> 在执行 source ./emsdk_env.sh 命令时失败，无法添加环境变量

思路：脚本实现不了，就不要使用提供的脚本命令，手动添加也可以。最终是要在任何窗口中都能执行命令。
手动添加环境变量，一般都是添加在当前用户区，和windows理念一样。

``` bash
PATH="/Library/Frameworks/Python.framework/Versions/3.7/bin:${PATH}"
export PATH=/Users/liuzhipan/emsdk:$PATH
export PATH=/Users/liuzhipan/emsdk/clang/e1.38.8_64bit:$PATH
export PATH=/Users/liuzhipan/emsdk/node/8.9.1_64bit/bin:$PATH
export PATH=/Users/liuzhipan/emsdk/emscripten/1.38.8:$PATH
alias python='/Library/Frameworks/Python.framework/Versions/3.7/bin/python3.7'
export EMSDK=/Users/liuzhipan/emsdk
export EM_CONFIG=/Users/liuzhipan/.emscripten
export BINARYEN_ROOT=/Users/liuzhipan/emsdk/clang/e1.38.8_64bit/binaryen
export EMSCRIPTEN=/Users/liuzhipan/emsdk/emscripten/1.38.8
export PATH=$PATH:$EMSDK:$EM_CONFIG:$BINARYEN_ROOT:$EMSCRIPTEN
```

仿照这份环境变量修改对应目录-路径即可。

> vim ~/.bash_profile         //添加上述的环境变量
> source ~/.bash_profile   // 刷新当前用户环境变量缓冲，使其生效

### 实例化module失败

>构建出来包之后，发现importObject提供的资源缺少导致实例化module失败。

思路：官网提供的大部分都是env对象

``` javascript
// 开辟内存空间
imports.env.memoryBase = imports.env.memoryBase || 0
if (!imports.env.memory) {
    imports.env.memory = new WebAssembly.Memory({ initial: 256 })
}
```

``` javascript
// 创建变量映射表
imports.env.tableBase = imports.env.tableBase || 0
if (!imports.env.table) {
    imports.env.table = new WebAssembly.Table({ initial: 2, element: 'anyfunc' })
}
```

** 问题关键 **

可是我导出来的wasm-module不止这几个。

``` bash
(import "env" "memory" (memory $env.memory 256))
(import "env" "table" (table $env.table 2 anyfunc))
(import "env" "memoryBase" (global $env.memoryBase i32))
(import "env" "tableBase" (global $env.tableBase i32))
(import "env" "DYNAMICTOP_PTR" (global $env.DYNAMICTOP_PTR i32))
(import "env" "tempDoublePtr" (global $env.tempDoublePtr i32))
(import "env" "ABORT" (global $env.ABORT i32))
(import "global" "NaN" (global $global.NaN f64))
(import "global" "Infinity" (global $global.Infinity f64))
(import "env" "abortStackOverflow" (func $env.abortStackOverflow (type $t0)))
(import "env" "nullFunc_X" (func $env.nullFunc_X (type $t0)))
(func $f2 (type $t1) (param $p0 i32) (result i32)
```

** 处理方案： **

- 所以要对多导出来的资源进行处理。
这些都是实例化module需要提供的import.加入对应的字段即可处理！

``` javascript
 imports.env.DYNAMICTOP_PTR = imports.env.DYNAMICTOP_PTR || 0;
 imports.env.tempDoublePtr = imports.env.tempDoublePtr || 0;
 imports.env.ABORT = imports.env.ABORT || 0;
 imports.global = imports.global || { NaN: 1, Infinity: 2 };
 imports.env.abortStackOverflow = imports.env.abortStackOverflow || new Function();
 imports.env.nullFunc_X = imports.env.nullFunc_X || new Function();
```
- 还有一种更简洁的。
使用wasm-gc去处理。