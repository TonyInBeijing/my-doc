# NestJS 补课（一）@这个符号到底是个什么东西

> 在拖更了两个月之后，我决定重新拾起《NestJS 逃课流攻略》这个栏目，但是一直逃课总会遇到难缠的 boss，所以新开一个档，给自己补一补课，如有不正之处烦请大伙批评指出。


## @func() 这到底是一个什么东西

### 以下是 [TC39](https://github.com/tc39/proposal-decorators) 的描述：**Decorators are functions called on classes, class elements, or other JavaScript syntax forms during definition.**

#### 在我的理解里，装饰器就是用来装饰类和类的方法的函数，它们在函数声明的时候被调用。

[caniuse](https://caniuse.com/) 网站表示很遗憾现在我们暂时 can't use 😭

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5qbeec5u6j21fb0u0grm.jpg)

所以我只能先搞一个 playground 拜托一波万能的 [Babel](https://babeljs.io/) 了🙏

```shell
# 先建一个空文件夹
mkdir playground
cd playground
# npm 初始化一波
npm init 
# 安装 Babel
npm install --save-dev @babel/core @babel/cli @babel/preset-env
# 新建一个 babel.config.json 配置文件
touch babel.config.json

```

```json
// babel.config.json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1"
        },
        "useBuiltIns": "usage",
        "corejs": "3.6.5"
      }
    ]
  ]
}
```

然后接着写一波代码

```shell
# 新建一个 src 目录，然后新建一个 decorator.js
mkdir src
cd src
touch decorator.js
```

``` javascript
// playground/src/decorator.js
const sayName = () => { 
    console.log("Hello,my name is Tony"); 
};

class Person{
    constructor(name){
        this.name = name;
    }
    @sayName()
    say(){
        console.log(this.name);
    }
}; 

new Person("Jack").say(); // ???
```

这样我们有了一个极度简单版的decorator了，现在需要运行一波Babel了

举个🌰

```javascript
const sayName = () => console.log("Hello,my name is Tony");

class Person{
  @sayName()
  say(){
    console.log("I want to say something");
  }
};

new Person().say();
```





