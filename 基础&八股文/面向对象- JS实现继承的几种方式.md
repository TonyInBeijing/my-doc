# 面向对象- JS实现继承的几种方式

## 1.原型链继承	

```javascript
// 父类
function Parent(){
  this.name = "Parent";
  this.age = 40;
  this.extraInfo = {
    job: "Coder"
  };
};
Parent.prototype.say = function(){
  console.log("I am Parent");
};
// 子类
function Child(){};
Child.prototype = new Parent();

// 缺陷：
var child1 = new Child();
var child2 = new Child();
child1.name = "Child1";
child1.extraInfo.job = "Singer";
console.log(child2.name,child2.extraInfo.job); // "Parent" "Singer"
```

>    浏览器当遇到属性访问器（"."操作符）时，会先从 child 实例上找 child1.name 和 child1.extraInfo.job，child1.name = "Child1"是一行赋值语句，直接在实例上添加一个name属性，child1.extraInfo.job 先找 child1.extraInfo, 实例上未找到，就从 child1.__ proto __ 上找,找到 child1.extraInfo.job ,赋值 Singer。
>
> 此时 child2 实例访问 extraInfo 上的 job 属性时，就也会从原型链上找，由于上面的值被 child1.extraInfo.job = "Singer" 修改，所以读到的值就是 Singer

## 2.构造函数继承

```javascript
// 父类
function Parent(){
  this.name = "Parent";
  this.age = 40;
  this.extraInfo = {
    job: "Coder"
  };
};
Parent.prototype.say = function(){
  console.log("I am Parent");
};
// 子类
function Child(){
  Parent.call(this);
};

// 缺陷
var child1 = new Child();
child1.say(); // Uncaught TypeError: child1.say is not a function
```

> 子类中调用 call 方法，实例化的时候自动将父类 Parent 上的属性挂载到子类的实例上，但是无法访问父类 prototype 上的属性和方法

## 3.原型链+构造函数的组合继承

```javascript
// 父类
function Parent(name,age){
  this.name = name;
  this.age = age;
  this.extraInfo = {
    job: "Coder"
  };
};
Parent.prototype.say = function(){
  console.log("I am Parent");
};
// 子类
function Child(){
  Parent.call(this);
};
Child.prototype = new Parent(?,?); // 缺陷：这里无法实例化带参数的父类，只能使用无参数的父类
```

## 4.寄生组合式继承(最终解决方案)

```javascript
// 父类
function Parent(name,age){
  this.name = name;
  this.age = age;
  this.extraInfo = {
    job: "Coder"
  };
};
Parent.prototype.say = function(){
  console.log("I am Parent");
};
// 子类
function Child(name,age){
  Parent.call(this,name,age);
};
Child.prototype = Object.create(Parent.prototype);
Child.prototype.childSay = function(){
	console.log("I am Child ~");
};

var parent = new Parent("Parent1", 40);
var child1 = new Child("child1", 21);
var child2 = new Child("child2", 22);
child1.name = "Child1";
child1.age = 23;
child1.extraInfo.job = "Singer";
parent.childSay(); // Uncaught TypeError: parent.sayChild is not a function
console.log(child2.name,child2.age,child2.extraInfo.job); // "child2" 22 "Coder"
child1.childSay(); // "I am Child ~"
child2.say(); // "I am Parent"
```

> 寄生组合式继承是当前主流的 es5 实现继承的方式，在子类 Child 中调用 Parent.call(this,...args) 将父类 Parent 方法挂载到子类中，同时可以满足带参数的构造函数
>
> Object.create 方法将父类 Parent.prototype 挂载到 Child.prototype.__ proto __上，这样当给 Child.prototype 上新增属性或方法时，不会污染父类 Parent.prototype，而当想访问 Parent.prototype 上的属性或方法时，又可以通过原型链访问

## 5. ES6 的 class 继承

```javascript
// 父类
class Parent{
  constructor(name,age){
    this.name = name;
  	this.age = age;
  	this.extraInfo = {
    	job: "Coder"
  	};
  };
  say(){
    console.log("I am Parent");
  }
}

// 子类
class Child extends Parent{
  constructor(name,age){
    super(name,age);
  }
  childSay(){
    console.log("I am Child ~");
  }
}

var parent = new Parent("Parent1", 40);
var child1 = new Child("child1", 21);
var child2 = new Child("child2", 22);
child1.name = "Child1";
child1.age = 23;
child1.extraInfo.job = "Singer";
parent.childSay(); // Uncaught TypeError: parent.sayChild is not a function
console.log(child2.name,child2.age,child2.extraInfo.job); // "child2" 22 "Coder"
child1.childSay(); // "I am Child ~"
child2.say(); // "I am Parent"
```

> ES6 中新增了 class 语法糖，子类继承父类定义时使用 extends 关键字，然后在 constructor 中 调用 super() 方法

