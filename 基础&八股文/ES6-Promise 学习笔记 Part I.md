# Promise 学习笔记 Part I

> 目的：为了防止回调地狱

## 规范	

### 核心Promises/ A + 规范

1. 一个 Promise 必须处于 3种状态之一： ***请求态(pending)***、***完成态(fulfilled)***、***拒绝态(rejected)*** 。
2. 当 Promise 处于 ***请求态(pending)*** 时，可以转为另外两种状态。
3. 当 Promise 处于 ***完成态(fulfilled)*** 时，不能转为另外两种状态。 

---

## 实例方法  

### .then()

```javascript
// 回调方法
function request(callback){
  $.ajax({
    success:function(){
      callback();
    }
  });
}

// promise
function request(){
  return new Promise(function(resolve,reject){
    setTimeout(resolve(111),1000);
  });
}

// 决议
request().then(function(res1){
  console.log(res); // 111
  return 222;
}).then(function(res2){
  console.log(res2) // 222
})
```

- Promise 决议的时候，第一个 then 的回调函数里参数 res 是 Promise 决议的值，即 resolve 的参数
- 后续 then 的 res 是上一个 then 的返回值，即 return 的值

```javascript
// 特殊情况-1
function myPromise(){
  return new Promise(function(resolve,reject){
    setTimeout(function(){
      resolve(1);
    },1000);
  });
}
myPromise().then(function(res1){
  console.log(res1); // 1
  return new Promise(function(resolve,reject){
    setTimeout(function(){
      resolve(2);
    });
  },1000);
}).then(function(res2){
  console.log(res2); // 2
});
```

- 如果 then 中返回一个新的 Promise ，那么下一个 then 回调函数的参数 res 返回上一个 then 中返回的新 Promise 决议的值，即 resolve 的参数

```javascript
// 接上面
myPromise().then(function(res1){
  console.log(res1); // 1
  var newPromise = new Promise(function(resolve,reject){
    setTimeout(function(){
      resolve(2);
    },1000);
  });
  return newPromise.then(function(res2){
    return 3;
  });
}).then(function(res2){
  console.log(res2); // 3
});
```

### .catch()

> 解决 try-catch 捕获不到异步编程中的错误

```javascript
// 接上面
myPromise().then(function(res1){
  console.log(res1); // 1
}).then(function(res2){
  throw Error(1);
  return 2;
}).then(function(res3){
  console.log(res3); // 不打印
  var newPromise = new Promise(function(resolve,reject){
    throw Error(2);
  }); // 异步中套异步报错，外层 catch 捕获不到
  return 3;
}).catch(function(error){
	console.log(error); // 1
  return 4;
}).then(function(res4){
	console.log(res4); // 4 catch 后会接着往下走
})
```

### .all() & .race()

> ***.all()*** : 多个Promise都决议后的api，返回一个 Promise;
>
> ***.race()*** : 只获取第一个决议的 Promise 的决议值

```javascript
var promise1 = new Promise(function(resolve,reject){
  resolve(1);
});
var promise2 = new Promise(function(resolve,reject){
  resolve(2);
});
var promise3 = new Promise(function(resolve,reject){
  resolve(3);
});
var promise4 = Promise.all([promise1,promise2,promise3]);
promise4.then(function(res){
  console.log(res); // [1,2,3]
});
var promise5 = Promise.race([promise1,promise2,promise3]);
promise5.then(function(res){
  console.log(res); // 1
});
```

## 静态方法

```javascript
Promise.resolve(); // 返回一个已决议的 Promise 对象，从而可以继续使用 then 的链式方法调用
Promise.reject(); // 同上
Promise.all(); // 参数为一个 promise 数组，等数组中的 promise 均决议后，生成并返回一个新的 promise
Promise.race(); // 参数同上，等数组中第一个 promise 决议后，生成并返回一个新的 promise
```

