---
title: flutter
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-10-13 12:39:11
tags:
    - flutter
categories: true 
copyright:
password:
---

## Flutter

> [Flutter](https://flutter.io/) 是谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。 Flutter可以与现有的代码一起工作。在全世界，Flutter正在被越来越多的开发者和组织使用，并且 [Flutter](https://flutter.io/) 是完全免费、开源的。

```
中文官网:
https://flutterchina.club/setup-macos/
https://flutter-io.cn/

第三方库   可以理解为npm
https://pub.dartlang.org/flutter

环境搭建\入门\填坑指南
https://blog.csdn.net/hekaiyou/article/details/52874796
https://www.jianshu.com/p/399c01657920
       
国内Flutter论坛
http://flutter-dev.cn/

Dart\Flutter 扩展插件
https://dartcode.org/releases/v2-19/

Flutter入门实例
https://juejin.im/post/5b31d776e51d455e2b5ab253

Widget组件介绍
https://juejin.im/post/5bab35ff5188255c3272c228

GitHub-Flutter聚集地
https://github.com/xitu/awesome-flutter
```
<!-- more -->

### 介绍

** 概念 **
1. Flutter是一个移动应用程序的软件开发工具包（SDK），具有以下特征：

    > 跨平台应用的框架，没有使用WebView或者系统平台自带的控件，使用自身的高性能渲染引擎自绘
    > 简化版的浏览器，最大限度在android和ios上统一UI，包括业务逻辑和用户体验
    > 开发语言使用dart，结合C, C++, 和Skia（2D渲染引擎）构建
    > 支持hot reload，包含着完整的控件和工具链
    
    > 一切皆控件，控件是每个Flutter应用程序的基本构建块，与分离视图、控制器、布局和其他属性的框架不同，Flutter具有一致的统一对象模型：控件。一个控件可以定义：结构元素（比如按钮或菜单）、风格元素（比如字体或颜色方案）、布局的方面（比如填充）、一些业务逻辑等
    
    > 与React理念相同，都是组合大于继承，控件本身通常由许多小型、单用途的控件组成，结合起来产生强大的效果，类的层次结构是扁平的，以最大化可能的组合数量
    
    > 强化版的WebView，框架仅提供一个View层，大部分功能要依赖原生
    > 目前只能够运行大部分Dart代码（不能引入dart:mirrors或dart:html库）

** 优势 **
1. 宏观上：

    > Flutter能够提供优美的UI和流畅的使用体验
    > Flutter降低了开发App的门槛，加速移动应用的开发速度，并且能够降低同时开发Android和iOS应用的成本和复杂度
    > Flutter能够轻松做出原型并且能够保持相当高还原度

2. 微观上：

    > 高效率，用一套代码库就能开发iOS和Android应用
    > 使用新型的、表现力强的语言和声明式的方法，用更少的代码来做更多的事情
    > 可以在应用程序运行时更改代码并重新加载查看效果，也就是热重新加载
    > 修复崩溃时可以从应用程序停止的位置继续调试
    > 创建美观、高度定制的用户体验
    > Flutter框架内置了一组丰富的质感设计控件
    > 实现定制、美观、品牌驱动的设计，而不受OEM控件集的限制
    > 深度优化，移动优先的2D渲染引擎而且对文本支持非常出色
    > react风格的框架
    > 支持单元和集成测试的API
    > 支持与系统平台和第三方SDK交互的插件API
    > 支持Windows，Mac和Linux的Headless test runner

### 运行机制

Flutter 应用运行在一个用 C++ 写的引擎中，Flutter 应用可以看做是一个游戏 App，代码都是在引擎中运行。

** Android **

引擎的C或C++代码是由Android NDK编译的，而框架的主要代码和应用的代码由Dart compiler编译成native code执行的。

对于Android应用来说，Flutter框架在引擎中实现了一个继承于SurfaceView的 FlutterView。用户所看到的UI都是在这个SurfaceView中显示。如果要和原生平台功能交互，则可以在Activity中使用FlutterView，并通过Flutter提供的消息API和原生平台收发消息。

** ios **

引擎的C或C++代码是由LLVM编译的，而所有Dart的代码会被AOT编译成native code，整个APP运行时使用的是机器指令（并不是拦截器）。

### 架构

** 层次描述 **
{% asset_img FlutterFramework.png Flutter架构 %}

Flutter的框架分为Framework和Engine两层，应用是基于Framework层开发的，Framework负责渲染中的Build，Layout，Paint，生成Layer等环节。Engine层是C++实现的渲染引擎，负责把Framework生成的Layer组合，生成纹理，然后通过Open GL接口向GPU提交渲染数据。

** 图形管道 **
{% asset_img FlutterFramework1.png Flutter运行 %}

** 渲染管道 **
{% asset_img flutterRun.png Flutter运行 %}

当需要更新UI的时候，Framework通知Engine，Engine会等到下个Vsync信号到达的时候，会通知Framework，然后Framework会进行animations, build，layout，compositing，paint，最后生成layer提交给Engine。Engine会把layer进行组合，生成纹理，最后通过Open Gl接口提交数据给GPU， GPU经过处理后在显示器上面显示。

当应用调用setState后，经过Framework一连串的调用后，最终调用scheduleFrame来通知Engine需要更新UI，Engine就会在下个vSync到达的时候通过调用_drawFrame来通知Framework，然后Framework就会通过BuildOwner进行Build和PipelineOwner进行Layout，Paint，最后把生成Layer，组合成Scene提交给Engine。接下来我们从代码中分析一下，这些环节具体是怎么样实现的。首先从Engine回调Framework的入口开始。

在Flutter应用开发中,无状态的widget是通过StatelessWidget的build方法构建UI，有状态的widget是通过State的build方法构建UI。现在具体分析一下从setState调用后到调用自定义State的build的流程是怎样的（现在只分析有状态的widget渲染过程）。

在Flutter中应用中，是使用支持layout的widget来实现布局的，支持layout的wiget有Container，Padding，Align等等，强大又简易。在渲染流程中，在widget build后会进入layout环节，下面具体分析一下layout的实现，layout入口是flushLayout。

### 应用场景

Flutter 支持ios android Tv 嵌入式系列开发

### 了解** Widget **组件

在Flutter中，我们平时自定义的widget，一般都是继承自StatefulWidget或StatelessWidget（并不是只有这两种），这两种widget也是目前最常用的两种。如果一个控件自身状态不会去改变，创建了就直接显示，不会有色值、大小或者其他属性的变化，这种widget一般都是继承自StatelessWidget，常见的有Container、ScrollView等。如果一个控件需要动态的去改变或者相应一些状态，例如点击态、色值、内容区域等，那么一般都是继承自StatefulWidget，常见的有CheckBox、AppBar、TabBar等。其实单纯的从名字也可以看出这两种widget的区别，这两种widget都是继承自Widget类。

Widget类在Flutter中是非常重要的，继承自Widget类的有PreferredSizeWidget、ProxyWidget、RenderObjectWidget、StatefulWidget、StatelessWidget。我们日常使用的绝大部分widget都是继承自Widget类，


### 生命周期

** 控件生命周期 **
{% asset_img flutterWidgetBuild.png Flutter运行 %}

** 状态生命周期 **
{% asset_img FlutterState.png Flutter运行 %}

** 生命周期有四种状态： **

created：当State对象被创建时候，State.initState方法会被调用；

initialized：当State对象被创建，但还没有准备构建时，State.didChangeDependencies在这个时候会被调用；

ready：State对象已经准备好了构建，State.dispose没有被调用的时候；

defunct：State.dispose被调用后，State对象不能够被构建。

** 完整生命周期如下： **

1. 创建一个State对象时，会调用StatefulWidget.createState；

2. 和一个BuildContext相关联，可以认为被加载了（mounted）；

3. 调用initState；

4. 调用didChangeDependencies；

5. 经过上述步骤，State对象被完全的初始化了，调用build；

6. 如果有需要，会调用didUpdateWidget；

7. 如果处在开发模式，热加载会调用reassemble；

8. 如果它的子树（subtree）包含需要被移除的State对象，会调用deactivate；

9. 调用dispose,State对象以后都不会被构建；

10. 当调用了dispose,State对象处于未加载（unmounted），已经被dispose的State对象没有办法被重新加载（remount）。

### 环境搭建

1. 克隆Flutter 仓库到本地

> git clone -b master https://github.com/flutter/flutter.git

2. 配置环境变量

```bash
export PUB_HOSTED_URL=https://pub.flutter-io.cn //国内用户需要设置
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn //国内用户需要设置
export FLUTTER_HOME= // 设置Flutter SDK 目录
export PATH=$FLUTTER_HOM/bin:$PATH
```
3. 执行命令检测Flutter依赖， 需要科学上网（这样会自动下载）

> flutter doctor

