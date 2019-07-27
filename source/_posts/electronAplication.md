---
title: Electron 应用 （爱翻译）
date: 2018-08-14 21:24:55
tags: 
	- electron
	- 前端
copyright: true
---
{% asset_img 翻译应用.png 翻译应用 %}
### 什么是Electron
[Electron](https://link.juejin.im?target=https%3A%2F%2Felectron.atom.io) 是前端开发者去构建跨平台桌面应用的一种方案。大家熟悉的 Atom 和 VSCode 编辑器就是使用 Electron 开发的。

[Electron](https://link.juejin.im?target=https%3A%2F%2Felectron.atom.io) 是 Node.js 和 Chromium 浏览器的结合体，用 Chromium 浏览器显示出的 Web 页面作为应用的 GUI，通过 Node.js 去和操作系统交互。 当你在 Electron 应用中的一个窗口操作时，实际上是在操作一个网页。当你的操作需要通过操作系统去完成时，网页会通过 Node.js 去和操作系统交互。

<!-- more -->

[Electron](https://link.juejin.im?target=https%3A%2F%2Felectron.atom.io) 开发桌面端应用的优点有：

*   降低开发门槛，减少了学习的成本，只需掌握前端技术和 Node.js 即可，不需要为开发一个桌面应用去学习其它的编程语言。
*   由于 Chromium 浏览器和 Node.js 都是跨平台的，Electron 能做到写一份代码在不同的操作系统运行。
*   而且可以使用vue\react等框架去开发GUI，总之强大的生态系统还需要灵活的统筹能力，去组合。

### Electron的进程问题？
#### 1、 主进程 Main Process
主进程，其实就是electron执行时，首次加载的可执行代码资源。我要渲染每个GUI页面，就需要有主进程去把控资源的分配、释放。那么在Electron中怎么确认我写的那个文件是主进程文件呢。
主进程加载的文件就是：packages.json 文件中的main字段指向的文件。

```javascript
这是一份最基础的主进程代码

const { app, BrowserWindow } = require('electron')
let win
function createWindow() {
  win = new BrowserWindow({ width: 800, height: 600 })
  const indexPageURL = `file://${__dirname}/dist/index.html`;
  win.loadURL(indexPageURL);
  win.on('closed', () => {
    win = null
  })
}
app.on('ready', createWindow)
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})
```
#### 2、渲染进程Render Process
渲染进程，就是用来进行进行页面的渲染，在主进城创建好友，会调用渲染进程去渲染页面资源。渲染进程指向的资源也就是你构建好、要呈现视图的HTML前端代码资源文件。比如你使用webpack构建出来的dist文件夹里面的数据就是。
渲染进程也可以看作是浏览器的每一个页面，一个页面对应一个进程，是不相同的。渲染进程的创建需要由主进程BrowserWindow对象去创建。

#### 3、主进程与渲染进程的关系
* 主进程在创建好后，会在创建一个子进程去加载资源，资源可以是网路资源(网站地址)，也可以是本地资源(html文件)。
* 主进程在创建窗口对象后，才会通过子进程去加载资源，那么窗口创建是有相应的生命周期事件去触发的。详情见# [BrowserWindow](https://electronjs.org/docs/api/browser-window#browserwindow)

#### 4、进程通信 [主进程-渲染进程]
我们在开发网页应用的时候，需要实现跨页面通信。可以使用**localStorage** 和 **H5 postMessages**。那么到了electron中同样适用，但是还有其它的方法去实现通信。就是进程通信！
* 一种是使用[ipcMain](https://github.com/electron/electron/blob/v1.1.3/docs/api/ipc-main.md)和[ipcRenderer](https://github.com/electron/electron/blob/v1.1.3/docs/api/ipc-renderer.md)模块，在渲染进程中使用ipcRender模块向主进程发送消息，主进程中ipcMain接收消息，进行操作，如果还需要反馈，则通知渲染进程，渲染进程根据接收的内容执行相应的操作：

```javascript
主进程

const {app, BrowserWindow, ipcMain} =  require('electron');

ipcMain.on('fy', function (event, value) {
    const {net} = require('electron')
    const request = net.request('http://translate.google.cn/translate_a/single?client=gtx&dt=t&ie=UTF-8&oe=UTF-8&sl=auto&tl=en&q=' + value)
    request.on('response', (response) => {
      response.setEncoding('utf8');
      let connect = '';
      response.on('data', (chunk) => {
        connect = chunk;
      })
      response.on('end', () => {
        mainWindow
          .webContents
          .send('fymessage', connect)
      })
    })
    request.end();
  })

渲染进程

import {remote, dialog, ipcRenderer} from  'electron';

handleSearch  = (value) =>{
    ipcRenderer.send("fy",value);
    const  that  =  this;
    ipcRenderer.on("fymessage", (event, text) => {
        console.log(text);
        that.setState({
        languageText:  text.split('\",')[0].replace(/\[\[\[\"/g,'')
    });
});

}
```
* 第二种是直接在渲染进程使用[remote](http://electron.atom.io/docs/api/remote/)模块，remote 模块可以直接获取主进程中的模块。这种方式其实是第一种方式的简化。

```javascript
通过remote对象调用主进程进行dialog操作。

 remote.dialog.showErrorBox("翻译失败", "翻译内容不能为空，请重新执行");
```
{% asset_img 错误提示.png 错误提示 %}

* 第三种是主进程向渲染进程发送消息

```javascript
mainWindow：是通过 new  BrowserWindow({}) 创建出来的窗口对象。
mainWindow.webContents.send('发送标识-类型判断', '发送内容')
```
* 第四种是渲染进程之间的通信

```javascript
// 主进程
// 两个窗口互相获取对方的窗口 id, 并发送给渲染进程
win1.webContents.send('distributeIds',{
    win2Id : win2.id
});
win2.webContents.send('distributeIds',{
    win1Id : win1.id
});

// 渲染进程
// 通过 id 得到窗口
remote.BrowserWindow.fromId(win2Id).webContents.send('someMsg', 'someThing');
```
总体来说，electron开发不是很难，而是对浏览器的重新认识。
在上述的列子中，讲解了如何进程通信，也表述了 electron 的 net 模块，进行http通信。在这里我使用了谷歌的翻译接口，这个接口也是逆向分析出来的。怎么分析就不叙述了。这里只概述electron的知识体系。 

剩下就是node的各个模块操作了。在electron中调用node的模块。如http,fs,等。
