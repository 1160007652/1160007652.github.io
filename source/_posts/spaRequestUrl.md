---
title: SPA单应用-请求接口URL结构设计
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2019-04-07 15:24:53
tags:
copyright:
password:
---

## 需求

在开发一个SPA应用时，必然会遇到与后台接口进行ajax的交互。为了项目结构的维护性、可读性，我们总会去做一些相应的处理。如：
- URL集中管理
- 拦截请求
- 拦截响应

在这里，我们主要去实现**URL的集中管理**方案。

<!-- more -->
## 抛出问题

> v1: 需求问题
- 如何集中管理URL
- 如何提取出请求地址中相同的内容
- 如何区分接口环境，如['开发环境','上线环境']

> v2: 需求问题
- 如何在定义URL时，只写入不同的url即可。相同内容自动拼接，无需前置人为的拼接。
- 如何根据不同的路由类型，去区分请求接口的类型。

## 技术可行性分析

> v1: 需求解决思路
- 可以使用**ES6 import export** 的方法实现 **URl集中管理**
- 可以使用“变量拼接、字符串模版解析”的方式，去提取地址中相同的内容
- 可以使用 **process.env.NODE_ENV** 自定义环境变量，识别此时的运行环境

> v2: 需求解决思路
- 可以使用**ES6 中的 proxy 实现**

## 代码实现

> v1: code

```javascript
// 联调环境接口判断
const baseUrlEnv = {
  development: '/test',
  production: '/online'
};

// 请求接口地址
const baseUrl = 'http://www.xxx.com/' + baseUrlEnv[process.env.NODE_ENV];

// 定义URL
const URLSource = {
    // 这里定义 account 主要是为了更加的细化请求地址
  account: { // 账户类型接口
    userInfo: baseUrl + '/v1/getUserInfo',
    userlist: `${baseUrl}/v1/getUserList`, 
  }
};

export default URLSource;


```

> v2: code

```javascript
// 定义URL
const URLSource = {
    // 这里定义 account 主要是为了更加的细化请求地址
  account: { // 账户类型接口
    userInfo: '/v1/{type}/getUserInfo',
    userlist: '/v1/{type}/getUserList', 
  }
};

// 联调环境接口判断
const baseUrl = {
  development: '/test',
  production: '/online'
};


// 代理监听 URL配置
const handler = {
  get(target, key) { // get 的trap 拦截get方法
    let value = target[key];
    try {
      return new Proxy(value, handler); // 使用try catch 巧妙的实现了 深层 属性代理
    } catch (err) {
      if (typeof value === 'string') {
        // 向请求地址动态绑定执行环境 如: test
        value = baseUrl[process.env.NODE_ENV] + value;

        // 替换当前浏览器的类型 通过获取路由中的第一个路径去区分
        if (value.includes('{type}')) {
          // 获取 当前浏览器 pathName 路由中的第一个类型
          let currentType = window.location.pathname.split('/').filter((item) => {
            return item;
          });
          currentType = currentType.length > 0 ? currentType[0].toLocaleLowerCase() : '';
          if (['china', 'Korea'].includes(currentType)) {
            value = value.replace('{type}', currentType);
          }
        }
      }
      return value;
    }
  },
  set(target, key) { // 阻止外部误操作，导致URL配置文件被修改，设置属性为只读属性
    try {
      return new Proxy(target[key], handler);
    } catch (err) {
      return true;
    }
  }
};

const URL = new Proxy(URLSource, handler);

export default URL;
```