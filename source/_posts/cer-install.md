---
title: Electron-cer 跨平台证书安装
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-11-25 17:16:23
tags:
    - react
    - electron
    - shell
copyright:
password:
---

### 简介

这是一份关于证书安装的教程，采用**electron**去实现跨平台证书安装。

### shell 命令

<!-- more -->

#### mac

列出证书
>sudo security dump-keychain /Library/Keychains/System.keychain

安装证书
>sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/cer.crt

删除证书
>sudo security delete-certificate -c "OK Group Certificate Authority"

#### linux

列出证书
>ls /usr/local/share/ca-certificates/

安装证书
>拷贝证书到【目录：/usr/local/share/ca-certificates/】
>命令： sudo cp cer.crt /usr/local/share/ca-certificates/foo.crt
>更新CA证书数据库存储：sudo update-ca-certificates
   
删除证书
>把证书从【目录：/usr/local/share/ca-certificates/】中删除 
>命令： sudo rm /usr/local/share/ca-certificates/cer.crt
>更新CA证书数据库存储：sudo update-ca-certificates --fresh

#### linux (centos 6)

列出证书
>ls /etc/pki/ca-trust/source/anchors/

安装证书
>安装ca-certificates包： yum install ca-certificates
>启用动态CA配置功能： update-ca-trust force-enable 
>将其作为新文件添加到【目录：/etc/pki/ca-trust/source/anchors/】 中
>命令：cp OK_Group_OA.crt /etc/pki/ca-trust/source/anchors/
>更新证书存储：update-ca-trust extract

删除证书
>把证书从【目录：/etc/pki/ca-trust/source/anchors/】中删除 
>命令： sudo rm /etc/pki/ca-trust/source/anchors/cer.crt
>更新CA证书数据库存储：sudo update-ca-trust extract

#### linux (centos 5)

列出证书
>列出证书功能 需要去 **ca-bundle.crt** 文件中查看。

安装证书
>把可信证书附加到【文件：/etc/pki/tls/certs/ca-bundle.crt】 中，也就是追加到文本末尾
>命令：cat OK_Group_OA.crt >>/etc/pki/tls/certs/ca-bundle.crt

删除证书
>把 OK_Group_OA.crt 的内容删除掉，即可。
>vim /etc/pki/tls/certs/ca-bundle.crt
>删除完， :wq 保存 

#### windows-10 || windows-7

列出证书
>cmd 运行 certmgr.msc

安装证书
>certutil -addstore -f "ROOT" cer.crt

删除证书
>certutil -delstore "ROOT" serial-number-hex


### 安装证书核心代码

证书类，提供了安装证书的功能，内部去判断所在的操作系统。

```js
const { app } = require('electron');
import sudo from 'sudo-prompt';
import { throwError } from 'rxjs';
const os = require('os');
const path = require('path');
const fs = require('fs-extra');

export default class InstallCer {
    platform = os.type().toLowerCase();
    cerPath = path.resolve(__dirname, '../assets/cers', 'cer.crt');
    configDir = app.getPath('userData');
    assetsDir = path.resolve(this.configDir, 'assets/')
    constructor(){
        console.log('证书路径：', this.cerPath);
        console.log('实例化成功,当前系统：', this.platform);
        console.log('用户目录：', this.configDir);
        this.init();
    }
    init(){
        fs.ensureDirSync(this.assetsDir);
        try {
            fs.copySync(this.cerPath, `${this.assetsDir}/cer.crt`);
            console.log('拷贝证书成功');
        } catch (err) {
           throw err;
        }
    }
    // mac 系统安装证书
    darwin(){
        return `security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain "${this.assetsDir}/cer.crt"`;
    }
    // windows 系统安装证书
    windows_nt(){
        return `certutil -addstore -f "ROOT" "${this.assetsDir}/cer.crt"`;
    }
    runCer(){
        const command = this[this.platform]();
        return new Promise(function(resolve,reject){
            sudo.exec(
                command,
                {
                  name: 'OK Network Assistant',
                  icns: path.resolve(__dirname, '../assets/', 'icon.icns')
                },
                function(error) {
                    if (error) {
                        reject(error)
                    } else{
                        resolve(true);
                    }
                }
            );
        })
    }
}
```

### 坑无敌

#### shell读取assar 资源
问题：

在进行安装时，出现了读取资源失败的问题。通过查阅官方文案，发现时assar中的资源不支持通过**shell管道**的形式获取资源。

解决：

在进行安装时，先吧=把资源从**assar**中拷贝到assar外的目录下。这里我吧目录放在了用户目录下，通过 **app.getPath('userData')** 该方法去获取每个操作系统的用户目录。

#### 开发与运行环境对文件的依赖

问题：

由于路径在开发与运行时都是相对的一个路径，所以在打包后，运行时无法找到icon\证书等文件。

解决：

在开发时提供一份资源目录，生成环境打包的地方再次提供一份目录，目前虽然解决。但是还是需要升级下解决方案！

#### 安装证书成功、失败、强制安装新证书

问题：
在安装了证书的用户电脑上，如果重新安装时，是无法进行安装的。

解决：
使用**electron-store**数据配置模块，去记录该电脑的操作。如果用户安装了就保存value:1，失败为2。但是如果再次发布新的版本时，安装成功的value值就需要重新改变。该值计次叠加！

```js
componentDidMount() {
    if ([undefined, 2, 1].includes(store.get('isInstallCer'))) {
      this.time = setTimeout(() => {
        this.handleSearch();
        clearTimeout(this.time);
      }, 1000);
    }else{
      this.props.history.push(routes.COUNTER)
    }
  }
```

在这，我把每次安装成功使用过的值都放入**[undefined, 2, 1]**这里。使得每次发布的新版本。用户在使用时可以安装到心得证书！