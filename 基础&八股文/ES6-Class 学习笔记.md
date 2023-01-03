<!--
 * @Author: TonyInBeijing
 * @Date: 2022-11-06 23:14:55
 * @LastEditors: TonyInBeijing
 * @LastEditTime: 2022-11-07 23:50:30
 * @FilePath: \my-doc\基础&八股文\ES6-Class 学习笔记.md
 * @Description: ES6-Class 学习笔记
 * 
-->
# ES6-Class 学习笔记

> 部分资料来源: [ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/class)

## 什么是类？

*“类是创造对象的模板”*

以上这句话基本上适用于所有面向对象的语言，但是貌似之前 JavaScript 中没有像 Java 或者 C# 语言中的 Class，所以 ES6 中新增了一个 Class 关键字。

ES6 中的 Class 可以理解为一个新增的语法糖，使得 JavaScript 创建实例对象从通过构造函数创建转变成通过类创建，更加清晰明了。

## ES5 中的类
在 Class 关键字之前，我们要声明“一类”对象通常使用构造函数模式
```javascript
// TODO: 声明
function Person(name,age){
    this.name = name;
    this.age = age;
};

Person.prototype.sayHello = function(){
    console.log(`Hi, I'm ${this.name}, I'm ${this.age} years old!`);
};

// TODO: 实例化
var tony = new Person("Tony",18);
tony.sayHello(); // Hi, I'm Tony, I'm 18 years old!

```
这种声明方式由于大家用的时间都不短了，所以我个人其实并没有觉得多么奇怪，不过相信有很多同学跟我一样被原型和原型链搞得头皮发麻，所以一看到这个 xxx.prototype 就产生了抵触心理，所以官方很贴心地推出了 Class 
## ES6 中的类
使用 Class 来声明一个类就非常的舒服~
```javascript
// TODO: 声明
class Person{
    constructor(name,age){
        this.name = name;
        this.age = age;
    }

    sayHello(){
        console.log(`Hi, I'm ${this.name}, I'm ${this.age} years old!`);
    }
}

// TODO: 实例化（与上面相同）
var tony = new Person("Tony",18);
tony.sayHello(); // Hi, I'm Tony, I'm 18 years old!
``` 
可以看到，Class 方法和构造函数方法差异主要存在于类的声明部分，实例化部分完全相同，既然之前说 Class 只是一个语法糖，那么后面主要研究 Class 中各部分分别对应构造函数中的哪些

这时候就要把万能的 Babel 请出来了~


## 部分实现