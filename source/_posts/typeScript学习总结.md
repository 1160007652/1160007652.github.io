---
title: typeScript
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-08-25 16:27:17
tags:
copyright:
password:
---

### 安装

> npm  i  -g  typescript

### 开发辅助

#### 编译 ts为js文件
``` bash
tsc xxxx.ts
```
#### 在vscode中，自动编译ts

``` bash
tsc --init   // 生成配置文件

out.dir :''  // 修改tsconfig.js字段， js生成目录

选择 -> 任务 -> 运行任务 -> 监视ts
```
### 变量

- string 			字符串
- boolean		    布尔
- number		    数值
- Array | [] 	    数组
- [ 类型]           元组
- enum			    枚举

<!-- more -->

-  any			    任意类型
- undefined	        未知-未定义
- null				空
- never 			不会出现的值，包含 null 和 nudefined

### 特性

- 类的 抽象 接口 多态 泛型
- 函数 接口 泛型 传参 可省略参数  匿名函数 定义

> 如果有面向对象的后台开发语言基础 如：Java 理解起来更快

### 简明代码

#### 变量的定义

##### string字符串类型

``` javascript
var str:string = "你好";

```
##### boolean布尔类型

``` javascript

var flag:boolean= true;

```
##### Number数值类型

``` javascript

var num:number = 123;

```
##### Array数组类型

``` javascript

var arr:number[]=[11,22,33]
var arr1:Array<number>=[11,22,33];

```
##### []元组类型

``` javascript

let arr2:[number,string]=[123,'234'];

```
##### enum枚举类型

``` javascript

enum COLOR {
    blue=123, red=456
}
// 如果 没有给枚举定义的变量 复制，就是返回下标。否则返回值
let myColor:COLOR = COLOR.blue;
console.log(myColor)

```
##### any任意类型

``` javascript

var numx:any = 123;
numx='222';

```
##### null-undefined类型
> 其它数据类型的子类型

``` javascript

var nums:number|undefined;
console.log(nums);
nums=12;
console.log(nums);

```
##### void类型
> 没有任何类型 , 一般 用于 方法返回

``` javascript

function setAge():void {
    console.log(1);
}
setAge();
function setAges():number {
    console.log(1);
    return 1211;
}
setAge();
console.log(setAges());

```
##### never类型
> 包括null和 undefined ，表示不会出现的值

``` javascript

var a:never;
a=(()=>{
    throw new Error('错误');
})();

```

#### 函数的定义

##### 匿名函数 

``` javascript
let func1 = function():string{
    return '123';
}

function func2():boolean{
    return true;
}

```
##### 传参 

``` javascript

function getinfo(name:string, age: number):string{
    return `我是${name},年龄${age}`;
}

console.log(getinfo('李三',20));

```
##### 无返回值

``` javascript

function func3():void{
    console.log('没返回值');
}
 
```
##### 可选参数 
>  如 使用?修饰 如 age?:number

``` javascript

function getinfos(name:string, age?:number):string{
    if(age){
        return `我是${name},年龄${age}`;
    }else{
        return `我是${name},年龄保密`;
    }
}
console.log(getinfos('李四'));
console.log(getinfos('李四',42));

```
##### 默认参数 
>  如 age:number=18

``` javascript

function getAge(age:number=18):number{
    return age;
}
console.log(`我的年龄是${getAge()}`);
console.log(`我的年龄是${getAge(33)}`);

```
##### 剩余参数 

> 如 ...arrAy

``` javascript

function sum(...result:number[]):number{
    let sum:number = 0;
    result.forEach(ele => {
        sum+=ele;
    });
    return sum;
}
console.log(sum(1,2,3,4));

```
##### 函数重载 

> extends super

``` javascript

function getName(name:string):string;
function getName(age:number):number;
function getName(str:any):any{
    if(typeof str === 'string'){
        return '我叫'+str;
    } else {
        return '我的年龄是'+str;
    }
}
console.log(getName('王老五'));
console.log(getName(23));

```
#### 类的定义 

> extends super

``` javascript

// 类的定义
class Person{
    name:string; //属性 省略了public 关键词
    private salary:number=10000;
    protected children:number = 2;
    constructor(name:string){ // 构造函数， 实例化的时候触发的方法
        this.name = name;
    }
    run():void{
        console.log(this.name,'工资：'+this.salary,'有'+this.children+'个孩子');
    }
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name = name;
    }
}
const person:Person = new Person('小明');
person.run();
person.setName('李四');
console.log('我的名字是：'+person.getName());


```
##### 类的继承 

> extends super

``` javascript

class WoMan extends Person{
    constructor(){
        super('女人');
    }
}
const woMan:Person = new WoMan();
woMan.run();
woMan.setName('女人们');
console.log(woMan.getName());


```
##### 类的修饰符

> public:     公有      在声明的类里面、继承声明的子类、调用类的实力化对象都可以访问
> protected:  保护类型   在声明的类里面、继承声明的子类可以访问，调用类的实力化对象没法访问
> private:    私有       在声明的类里面可以访问，继承声明的子类可以访问，调用类的实力化对象不可以访问

``` javascript
    
class Man extends Person{
    private smoke:boolean=true;
    constructor(){
        super('男人');
    }
    run():void{
        console.log(`这个${this.name} ${this.smoke ? '吸烟' : '不吸烟'}`);

        // console.log(`这个${this.name}的工资${this.salary}`);
        // -- 提示 private修饰的salary 属性 只能在 Person 声明属性的类中访问

        console.log(`这个${this.name}有${this.children}个孩子`);
    }
}
const man:Person = new Man();
man.run();
man.setName('男人们');
// console.log(`这个${man.name}有${man.children}个孩子`);
// -- 提示 protected修饰的 children 属性 只能在 声明属性的类及子类内部访问
console.log(man.getName());


```
##### 静态方法及属性

> 私有方法不可以访问 静态的方法书静态属性
> 静态方法也不可以访问 私有的方法和私有的属性

``` javascript

class Dog {
    name:string = '哮天犬';
    static work:string = '看门';
    constructor(){}
    eat(food:string):string{
        return `${this.name}吃${food}`;
    }
    static sleep():void{
        console.log('狗睡觉');
    }
}
const dog = new Dog();
console.log(dog.eat('骨头'));
Dog.sleep();
console.log(dog.name+Dog.work);

```
##### 多态

>  多态：定义方法不去实现，让继承类去实现方法体

``` javascript

class Animal {
    name:string;
    constructor(name:string){
        this.name=name;
    }
    // 定义方法不去实现，让继承类去实现方法体
    eat(food:string):void{};
}
class Cat extends Animal{
    constructor(name:string){
        super(name);
    }
    eat(food:string):void{
        console.log(`${this.name}吃${food}`);
    }
}
const cat:Cat = new Cat('🐱猫 ');
cat.eat('老鼠');

class Snake extends Animal{
    constructor(name:string){
        super(name);
    }
    eat(food:string):void{
        console.log(`${this.name}吃${food}`);
    }
}
const snake:Snake = new Snake('🐍蛇 ');
snake.eat('青蛙');

```
##### 抽象类

> abstract 修饰
> 抽象：要求该抽象类的子类(派生类)必须实现抽象方法，反之多态却可以不实现

``` javascript

abstract class Food {
    abstract saff(food:string):boolean;
}

class Tomato extends Food{
    constructor(){
        super();
    }
    saff(food:string):boolean{
        return true;
    }
}

const tomato:Food= new Tomato();
console.log(tomato.saff('西红柿')?'西红柿是安全食物':'西红柿不是安全食物');

```
#### 接口的定义

``` javascript

interface FullName{
    firstname:string;
    lastName:string;
    secoundName?:string;
}

function getNames(name:FullName):void{
    console.log(name);
}
getNames({
    firstname: '李',
    lastName: '四'
})
getNames({
    firstname: '李',
    lastName: '四',
    secoundName: '外号-人称刀疤李四'
})

```
##### 案例 ajax 接口封装
``` javascript

interface Config{
    method: string;
    url:string;
    data?:string;
    dataType:string;
}

function ajax(config:Config):any{
    let xhr = new XMLHttpRequest();
    xhr.open(config.method,config.url, true);
    xhr.send(config.data);
    xhr.onreadystatechange=function(){
        if(xhr.readyState == 4 && xhr.status == 200){
            if(config.dataType == 'json'){
                return JSON.parse(xhr.responseText);
            }else{
                return xhr.responseText;
            }
        }
    }
}

ajax({
    method: 'GET',
    url: 'https://blog.youngzk.com',
    dataType:'text'
})

```
##### 函数类型接口
``` javascript

interface Maths {
    (a:number,b:number):number;
}

var sums:Maths = function(as:number,bs:number):number{
    return as+bs;
}

console.log(sums(1,2));

```
##### 可索引接口
> 数组 对象的约束
``` javascript

interface Arr {
    [index:number]:string;
}
var array:Arr[]=['111','222'];

interface Obj {
    [index:string]:string;
}
const array1:Obj={
    name:'张三',
    // age: 12   错误 ，只能是字符串
}


```
##### 类的类型接口

``` javascript

interface Tiger {
    name:string
    eat(food:string):void;
}

class SmallTiger implements Tiger{
    name: string;
    constructor(name:string){
        this.name=name;
    }
    eat(food:string):void{
        console.log(`${this.name}吃${food}`);
    }
}

const smallTiger:Tiger = new SmallTiger('小脑虎');
smallTiger.eat('肉');

```
##### 接口的继承

``` javascript
interface Fruits{
    has():void;
}

interface Appale extends Fruits{
    getColor():string;
}

class RedApp implements Appale{
    constructor(){

    }
    has():void{
        console.log('红苹果是水果');
    }
    getColor():string{
        return '红苹果是红色的';
    }
}

const redApp:RedApp = new RedApp();
redApp.has();
console.log(redApp.getColor());

```

#### 泛型的定义

> 泛型
> 功能，提高复用性以及对不确定的数据类型

##### 泛型函数

``` javascript

function getData<T>(value:T):T{
    return value;
}
console.log(getData<string>('wwww'));
console.log(getData<number>(12433));

```
##### 泛型类

``` javascript

class MinClass<T>{
    list:T[]=[];
    add(num:T):void{
        this.list.push(num);
    }
    toString():string{
        return this.list.join();
    }
    getFirstChild():T{
        return this.list[0];
    }
}

const min:MinClass<number> = new MinClass<number>();
min.add(1);
min.add(5);
min.add(1);
min.add(-3);
min.add(9);
min.add(0);

console.log(min.getFirstChild());
console.log(min.toString());

const min1:MinClass<string> = new MinClass<string>();
min1.add('a');
min1.add('b');
min1.add('c');
min1.add('d');

console.log(min1.getFirstChild());
console.log(min1.toString());

```
##### 泛型函数接口
``` javascript

interface ConfigFnc {
    <T>(value:T):T;
}

const getAges:ConfigFnc = function<T>(value:T):T{
    return value;
}

console.log(getAges(1));
console.log(getAges('asd'));

interface ConfigFnc1<T> {
    (value:T):T;
}

const getAges1:ConfigFnc1<number> = function<T>(value:T):T{
    return value;
}
console.log(getAges1(1));

```