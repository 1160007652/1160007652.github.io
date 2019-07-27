---
title: okcoin-cli 辅助
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-12-09 13:17:34
tags:
    - cli
    - npm
    - iconfont
copyright:
password:
---

### [okIcon](https://github.com/1160007652/okIcon)

[okIcon](https://github.com/1160007652/okIcon)是一个自动获取阿里iconfont的一个助手，只需要第一次使用安装是初始化，后续一个命令自动下载释放。

#### 使用的模块

fs node自带 文件库
[fs-extra](https://www.npmjs.com/package/fs-extra) 第三方文件库
[commander](https://www.npmjs.com/package/commander) 命令行指令解决方案模块
[inquirer](https://www.npmjs.com/package/inquirer) 命令行交互模块
[request](https://www.npmjs.com/package/request) 网络访问库
[log-symbols](https://www.npmjs.com/package/log-symbols) 命令行输出图标
[chalk](https://www.npmjs.com/package/chalk) 命令行文字颜色
[cross-spawn](https://www.npmjs.com/package/cross-spawn) 跨平台shell执行
[walk](https://www.npmjs.com/package/walk) 目录、文件遍历库

<!-- more -->

### 制作okcoin

#### 接收命令行参数

如： npm init -y ,这里 init -y 就是执行命令是传入的参数，每个参数都是空格隔开的。
如下代码，我定义了4个指令， 分别是 -v -i -d --help , 对应的指令对应相应的函数去执行不同的功能。-v是查看版本、-i初始化、-d下载是否文件、- -help 查看帮助。

```js
program.version('1.0.0', '-v, --version')
    .option('-i, --init', '初始化配置文件')
    .option('-d, --download', '下载iconfont文件')
    .description('自动下载IconFont助手');

program.on('--help', function(){
    console.log('\n-------------------------------------\n');
    console.log('先使用 okIcon -i 进行初始化\n');
    console.log("后续iconfont的改动，使用 okIcon -d 进行下载\n");
});
    
program.parse(process.argv);
```

#### 交互式操作

如果执行了指令 **-i** ,那么就会触发inquirer方法。提示用户输入对应的信息。做初始化配置作用！

```js
if(program.init){
    inquirer.prompt([{
        type: 'input',
        name: 'iconAddr',
        message: '请输入iconfont的项目下载地址:\n'
    },
    {
        type: 'input',
        name: 'iconCookies',
        message: '请输入iconfont的Cookies:\n'
    },
    {
        type: 'input',
        name: 'iconPath',
        message: '请输入保存iconfont的绝对路径:\n'
    }]).then((answers) => {
        const {iconAddr,iconCookies,iconPath} = answers;
        if(setIconInfo(iconAddr,iconCookies,iconPath)){
            console.log(logSymbols.success, chalk.green('设置完毕，开始下载'));
            getIconfont();
        } else {
            console.log(logSymbols.error, chalk.red('设置失败,再试一次'));
        }
    });
}
```

#### ruquest携带自定义cookie

封装request请求为同步方法，至于怎么传入cookie,都放到hearders字断中即可。完了使用pipe管道释放iconfont字节集数据到对应的目录即可。

```js
function requestSync(url, cookies) {
    return new Promise(function(resolve, reject){
        let options = {
            url: url,
            headers: {
            'User-Agent': `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36`,
            Cookie: cookies
            }
        };

        request(options,function(e,r,b){
            if(e){
                reject(e);
            }else{
                resolve(b);
            }
        }).pipe(fs.createWriteStream(`${__dirname}/../iconfont.zip`));
    });
}
```

#### 文件遍历封装

其实也不能说是封装，主要是说下这个walker库。
walk.walk(); 要传入遍历查询的根目录。返回一个 walker执行对象。
剩下的只需要 walker.on('事件',（root, fileStats, next); 这个模块还是设计比较舒服的。有koa 洋葱模型的思想。fileStatus 是遍历处文件的一系列信息，next()执行下一个。
```js
function moveIconfont(iconPath){
    const walker = walk.walk(`${__dirname}/../temp`);
    const iconFiles = ['iconfont.css', 'iconfont.eot', 'iconfont.js', 'iconfont.svg', 'iconfont.ttf', 'iconfont.woff'];
    walker.on("file", function(root, fileStats, next){
        if (iconFiles.includes(fileStats.name)){
            fse.moveSync(`${root}/${fileStats.name}`, `${iconPath}/${fileStats.name}`, { overwrite: true });
        }
        next();
    });

    walker.on("errors", function (root, nodeStatsArray, next) {
        next();
    });
    
    walker.on("end", function () {
        console.log(logSymbols.success, chalk.green("下载完毕"));
        fse.removeSync(`${__dirname}/../temp`);
        fse.removeSync(`${__dirname}/../iconfont.zip`);
    });
}
```

### 源码

okIcon[GitHub](https://github.com/1160007652/okIcon)

```js
#!/usr/bin/env node

const fs = require('fs');
const fse = require('fs-extra');
const program = require('commander');
const inquirer = require('inquirer');
const request = require('request');
const logSymbols = require('log-symbols');
const chalk = require('chalk');
const spawn = require('cross-spawn');
const walk = require('walk');

const configPath = `${__dirname}/../config.ini`;

program.version('1.0.0', '-v, --version')
    .option('-i, --init', '初始化配置文件')
    .option('-d, --download', '下载iconfont文件')
    .description('自动下载IconFont助手');

program.on('--help', function(){
    console.log('\n-------------------------------------\n');
    console.log('先使用 okIcon -i 进行初始化\n');
    console.log("后续iconfont的改动，使用 okIcon -d 进行下载\n");
});
    
program.parse(process.argv);

if(program.init){
    inquirer.prompt([{
        type: 'input',
        name: 'iconAddr',
        message: '请输入iconfont的项目下载地址:\n'
    },
    {
        type: 'input',
        name: 'iconCookies',
        message: '请输入iconfont的Cookies:\n'
    },
    {
        type: 'input',
        name: 'iconPath',
        message: '请输入保存iconfont的绝对路径:\n'
    }]).then((answers) => {
        const {iconAddr,iconCookies,iconPath} = answers;
        if(setIconInfo(iconAddr,iconCookies,iconPath)){
            console.log(logSymbols.success, chalk.green('设置完毕，开始下载'));
            getIconfont();
        } else {
            console.log(logSymbols.error, chalk.red('设置失败,再试一次'));
        }
    });
}

if(program.download){
    getIconfont();
}


function getIconfont() {
    
    // iconfont 下载配置文件
    let configData = null;

    //创建存放 iconfont 解压数据的临时目录
    fse.ensureDirSync(`${__dirname}/../temp`);

    // 判断文件是否存在
    try {
       if(fs.existsSync(configPath) && fs.existsSync(`${__dirname}/../temp`)){
            try {
                configData = JSON.parse(fs.readFileSync(configPath));
            } catch (error) {
                console.log(logSymbols.warning, chalk.yellow('读取配置文件失败'));
            }
       } else {
            console.log(logSymbols.info, "还未初始化,请执行", chalk.blue(' okIcon -i'));
            console.log("");
            return;
       }
    } catch (error) {
        console.log(logSymbols.info, "还未初始化,请执行", chalk.blue(' okIcon -i'));
        return;
    }

    // 同步从阿里获取 iconfont
    requestSync(configData.iconAddr, configData.iconCookies).then(()=>{

        // 解压iconfont configData.iconPath
        spawn.sync('tar', ['zxf', `${__dirname}/../iconfont.zip`, '-C', `${__dirname}/../temp`], { stdio: 'inherit' });
        
        moveIconfont(configData.iconPath);
    });
    
}

function setIconInfo(iconAddr,iconCookies,iconPath){
    const source = { iconAddr, iconCookies, iconPath }
    try {
        const s = fs.writeFileSync(configPath, JSON.stringify(source));
        return true;
    } catch (error) {
        return false;
    }  
}

function requestSync(url, cookies) {
    return new Promise(function(resolve, reject){
        let options = {
            url: url,
            headers: {
            'User-Agent': `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36`,
            Cookie: cookies
            }
        };

        request(options,function(e,r,b){
            if(e){
                reject(e);
            }else{
                resolve(b);
            }
        }).pipe(fs.createWriteStream(`${__dirname}/../iconfont.zip`));
    });
}

function moveIconfont(iconPath){
    const walker = walk.walk(`${__dirname}/../temp`);
    const iconFiles = ['iconfont.css', 'iconfont.eot', 'iconfont.js', 'iconfont.svg', 'iconfont.ttf', 'iconfont.woff'];
    walker.on("file", function(root, fileStats, next){
        if (iconFiles.includes(fileStats.name)){
            fse.moveSync(`${root}/${fileStats.name}`, `${iconPath}/${fileStats.name}`, { overwrite: true });
        }
        next();
    });

    walker.on("errors", function (root, nodeStatsArray, next) {
        next();
    });
    
    walker.on("end", function () {
        console.log(logSymbols.success, chalk.green("下载完毕"));
        fse.removeSync(`${__dirname}/../temp`);
        fse.removeSync(`${__dirname}/../iconfont.zip`);
    });
}
```