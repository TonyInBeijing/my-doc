# Javascript 的预解析机制

> 部分参考自：https://www.cnblogs.com/shytong/p/5100426.html

最近福报太多，又是一个撸码的周末，博客更新有点慢。

虽然没什么人看但是当初立下的flag还是得尽量实现～

今天沉浸在“快乐”的福报中突然想到一个好玩的问题：

```javascript
// 这两种声明函数的方式到底有什么区别
var func = function(){};
function func(){};
```

由于一直使用 const 声明箭头函数 const func = () => {} ，很少用 function func(){} 这种函数声明了，今天就彻底搞懂它！

## 首先来个栗子🌰

```javascript
function test1(){
  var func = function(){
    console.log(1);
  }
  function func(){
    console.log(2);
  }
  func();
}
test1(); // 1
```

这看起来好像就不太符合常理，我后定义了 function func(){} ，为什么执行了上面方法里的语句呢？

## 变量提升

关于变量提升的详细情况，那可复杂多了，简单来说就是 js 要在 node 或者浏览器中运行，首先需要一个  js 引擎，这个引擎用来分析并执行 js 代码，在这个过程中，变量的声明会被提升到函数作用域的顶部并赋一个初始值 `undefined`，这就是我理解的变量提升，这个过程是在代码执行阶段之前的

但是`函数声明`和`函数表达式`的优先级是不一样的，函数声明的优先级要高于函数表达式声明的，所以上面的例子实际执行起来应该是这样的

```javascript
function test1(){
  function func(){
    console.log(2);
  }
  var func = undefined;
  func = function(){
    console.log(1);
  }
  func();
}
test1(); // 1
```

## 又一个栗子🌰

```javascript
function test2(target){
  console.log(target); // Function target
  var target = 1; 
  function target(){};
  console.log(target); // 1
};

test2(0);

function test3(target){
  console.log(target); // 0
  var target = 1;
};
test3(0);
```

？？？这又是怎么回事呢？好像在 test3 中变量又不提升了？

以上面的test2为例，代码执行过程是这样的

```javascript
// 此时加入了形参
function test2(target){
  var target = undefined; // 1.将target变量提升上来
  target = 0; // 2.实参是0
  target = function(){}; // 3.将函数提升上来
  console.log(target); // 这里打印出 Function target
  target = 1; // 原 var target = 1 声明语句
  console.log(target); // 这里打印出 1
};
test2(0);
```

根据上面的两个栗子，可以总结出来预解析的过程大概是

1. 函数被执行时，创建一个活动的对象（简称 active object，AO）
2. 找到函数的形参和 var 声明（const 和 let请听下回分解）挂到上面的对象上 （AO[key]=undefined）
3. 接收实参，然后赋值函数（AO["形参"]=实参，如果后面有函数声明跟形参同名就覆盖）
4. 分析函数声明（这里如果有跟函数同名的形参或 var 声明的变量，函数将覆盖，否则会整体提升上来声明）

## 最终总结

```javascript
function finalTest(target){
  console.log("target-1::",target);
  var target = 1;
  function target(){
    console.log("function-1");
  };
  console.log("target-2::",target);
  var target = function(){
    console.log("function-2");
  };
  console.log("target-3::",target);
  function target(){
    console.log("function-3::",target);
  }
	console.log("target-5::",target);
};
finalTest(0); // 稍微有点乱... 结果是什么呢？
```

