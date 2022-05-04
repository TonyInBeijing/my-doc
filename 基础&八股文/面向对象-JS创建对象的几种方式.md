# JS创建对象的几种方式



## 1.工厂模式

```javascript
function objectFactory(name,age){
  var obj = {};
  obj.name = name;
  obj.age = age;
  return obj;
}

var obj = objectFactory("obj",20);

// 缺陷：无法判断对象类型
```

## 2.构造函数模式

```javascript
function Person(name,age){
  this.name = name;
  this.age = age;
  this.say = function(){
    console.log("Hello");
  };
}

var person1 = new Person("Tony",20);
var person2 = new Person("Mary",21);

// 缺陷: 每次创建一次新对象，构造函数上的方法都会被重新创建一遍，造成浪费
console.log(person1.say === person2.say); // false
```

## 3.原型模式

```javascript
function Person(){}
Person.prototype = {
  name: "Tony",
  age: 20,
  say: function(){
    console.log("Hello");
	}
};
var person1 = new Person();
var person2 = new Person();
console.log(person1.name === person2.name); // true
console.log(person1.age === person2.age); // true
console.log(person1.say === person2.say); // true
// 缺陷:所有属性都定义在prototype上，创建实例时无法给定特殊的初始值
```

## 4.构造函数+原型模式

```javascript
function Person(name,age){
  this.name = name;
  this.age = age;
}
Person.prototype = {
  say: function(){
    console.log("Hello");
	}
};

var person1 = new Person("Tony",20);
var person2 = new Person("Mary",21);
console.log(person1.name,person1.age); // Tony 20
console.log(person2.name,person2.age); // Mary 21
console.log(person1.say === person2.say); // true

// 当前此种方式为最广泛认可度最高的创建对象的方式，实例本身不同的属性都是在构造函数中定义的，实例共用的属性和方法都是在原型上定义的
```

