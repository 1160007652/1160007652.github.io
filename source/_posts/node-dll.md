---
title: node-dll 调用
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-08-26 08:56:42
tags:
    - node
    - dll
copyright: node
password:
---

### DLL介绍

DLL(Dynamic Link Library)文件为动态链接库文件，又称"应用程序拓展"，是软件文件类型。在Windows中，许多应用程序并不是一个完整的可执行文件，它们被分割成一些相对独立的动态链接库，即DLL文件，放置于系统中。当我们执行某一个程序时，相应的DLL文件就会被调用。一个应用程序可使用多个DLL文件，一个DLL文件也可能被不同的应用程序使用，这样的DLL文件被称为共享DLL文件。

### Node 怎么调用DLL

使用 node-ffi 模块，非常灵活的node中去调用dll中暴露的方法。

<!-- more -->

### 安装node-ffi模块

> [Node-gyp](https://github.com/nodejs/node-gyp)
> https://github.com/nodejs/node-gyp

> [Node-ffi](https://github.com/node-ffi/node-ffi)
> https://github.com/node-ffi/node-ffi

> 安装命令  npm install ffi

** 在安装node-ffi模块前，必须保证node-gyp 安装成功，并且node-gyp可以使用 **

### 安装node-ffi 遇到的坑

如果安装成功了，就看看我是怎么解决这个坑的吧。

不知道是怎么回事，每次安装ffi模块，都会导致 node-gyp build  V8类型出错。

** 解决方法：**
直接克隆node-ffi仓库到本地，把node-ffi 放到全局的 node-modules中。

``` bash
 git clone git://github.com/node-ffi/node-ffi.git 
```

再进入CMD命令窗口中，执行** npm install node-ffi –g **,即可成功安装。
安装完后，全局的 node-modules中会出现 快捷方式的 ffi文件夹。
需要创建一个ffi文件夹，并把快捷方式的 ffi文件夹内容剪切到 新的 ffi文件夹中，也就是去掉快捷方式即可。

### 开发DLL

> 开发DLL工具，我使用的是VS2015。

** 步骤：b**

开发DLL工具，我使用的是VS2015。
打开** VS2015 ** -> ** 文件 ** -> ** 新建 ** -> ** 项目 ** -> ** 选择Win32控制台应用程序 **

{% asset_img node-dll-1.png 创建项目 %}

{% asset_img node-dll-2.png 应用程序向导 %}

选择 ** 下一步 **

{% asset_img node-dll-3.png 应用程序向导 %}

选择 ** 控制台应用程序 空项目 ** 最后点击 ** 完成 **

{% asset_img node-dll-4.png 应用程序向导 %}

右击** nodeFile ** 项目名称，选择 ** 添加 ** -> ** 新建项 **

{% asset_img node-dll-5.png 应用程序向导 %}

选择 ** 头文件 **，名称我写的 ** fileChange.h **点击添加

同理，相同的步骤创建C++文件（.cpp），名称fileChange.cpp 点击添加

在源文件 ** fileChange.cpp ** 中编写代码：

``` c++
// fileChange.cpp 

#include "fileChange.h"
	
	//同步函数，求和方法
int add(int i, int j) {
		return i + j;
}
	//异步函数，求和方法
int addSync(int i, int j, void (*callfuct)(int a, int b)) {
		int sum = i + j;
		callfuct(sum, j);
		return 0;
```

在头文件 ** fileChange.h ** 中编写代码：

``` c++
//  fileChange.h

#pragma once
extern "C" __declspec(dllexport) int add(int i, int j);
extern "C" __declspec(dllexport) int addSync(int i, int j, void (*callfuct)(int a, int b));
```
在这里必须使用，** extern "C" ** 让编译器使用C解析方法去导出DLL，不然node-ffi模块识别不了。

### 生成DLL 

因为我的电脑环境是64位，node也是64位，所以我导出的dll也需要是64位，不然node-ffi也是调用不成功。

** 导出方法： **

{% asset_img node-dll-6.png 导出方法 %}

导出的dll文件在在项目目录下的 \x64\Debug 中，即可找到 nodeFile.dll动态链接库文件。

{% asset_img node-dll-7.png 应用程序向导 %}

### node使用DLL

``` javascript
//导入ffi模块
var ffi = require('ffi');
//使用ffi.Library加载dll, 第一个参赛是库文件路径，第二个参数是JSON格式，用于定义使用的dll方法。
//int 表示整形，pointer 表示 指针地址，也可以使用 int * 表示。
var libm = ffi.Library('./../dll/nodeFlie', {
  'add': [ 'int', [ 'int','int' ] ],
  'addSync': ['int', ['int','int','pointer']]
});

//调用方法
const sum = libm.add(1,5);
console.log(sum);

//定义 回调函数
let addCallback = ffi.Callback('void', ['int', 'int'], (a, b) => {
  console.log(a,b);
});

//执行回调的 dll方法
libm.addSync(2,3,addCallback);

执行结果：

PS E:\nodejsproject\node-fileChangeAttr\src> node .\index.js
6
5 3
```