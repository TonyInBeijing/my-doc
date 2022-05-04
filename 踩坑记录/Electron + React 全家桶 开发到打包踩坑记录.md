# Electron + React 全家桶 开发到打包踩坑记录

> 最近做了一个 electron 相关的项目，搬砖过程中遇到了很多坑，写一篇博客记录一下
>
> 项目地址:  https://github.com/TonyInBeijing/electron-react.git

##  项目依赖

- create-react-app v5.0.0
- electron-builder v22.14.13
- react v17.0.2
- react-dom v17.0.2
- react-router-dom v5.3.0
- electron v14.1.1
- @electron/remote v2.0.1
- antd v4.18.6

## 创建项目

1. 使用 create-react-app 创建一个 react 项目

   ```bash
   create-react-app electron-react
   ```

2. 进入项目根目录,安装路由管理库 react-router-dom

   ```bash
   cd electron-react
   npm install react-router-dom
   ```

3. 安装 electron 和 @electron/remote

   ```bash
   # electron 在 14.0.0 版本后强制要求使用 @electron/remote
   npm install electron @electron/remote --save-dev
   ```

4. 在项目根目录中创建一个 main.js 作为 electron 启动文件

```javascript
/**
 * @description electron 启动文件
 */

// 引入electron并创建一个Browserwindow
const { app, BrowserWindow } = require('electron');

// 保持window对象的全局引用,避免JavaScript对象被垃圾回收时,窗口被自动关闭.
let mainWindow

function createWindow() {
    // 初始化electron- remote
    require("@electron/remote/main").initialize();

    //创建浏览器窗口,宽高自定义
    mainWindow = new BrowserWindow({
        width: 768,
        height: 1366,
        webPreferences: {
            // 给渲染进程注入node模块
            nodeIntegration: true,
            // 语境隔离：设置为false时允许渲染进程访问window上的变量
            // https://runebook.dev/zh-CN/docs/electron/tutorial/context-isolation 
            contextIsolation: false,
            enableRemoteModule: true,
        }
    });
    /* 
     * 加载应用-----  electron-quick-start中默认的加载入口
     */
    //创建每一个窗口后，都要调用
    require("@electron/remote/main").enable(mainWindow.webContents)

    // 开发
    mainWindow.loadURL("http://localhost:3000/");

    // 关闭window时触发下列事件.
    mainWindow.on('closed', function () {
        mainWindow = null
    })
}

// 当 Electron 完成初始化并准备创建浏览器窗口时调用此方法
app.on('ready', createWindow)

```

5. 修改 package.json 文件

   ```json
   {
     ...
     "homepage": ".", // 将绝对路径转换成相对路径，否则启动 electron 后找不到静态资源文件
     "main": "main.js", // 定义 electron 启动文件路径
     "scripts": {
       ...
       "electron" : "electron ." // 配置 electron 启动命令
       ...
     },
     ...
   }
   ```
   



## 启动项目

```bash
# 1.启动 react 
npm start

# 2.启动 electron
npm run electron
```

![image-20220210212227182](https://tva1.sinaimg.cn/large/008i3skNly1gz8qemqzkcj30u013babj.jpg)

出现该窗口后表明项目已经启动成功！恭喜你已经成功上手了 electron + react 项目

## 安装第三方库

> 在我们日常开发中，通常会安装其他第三方库进行配合开发，此处安装 ant-design 组件库，也是我个人特别喜欢的一个桌面端组件库，具体使用方法详见ant-design 官网: https://ant.design/index-cn

```bash
# 安装 ant-design
npm install antd --save
```

然后在项目根目录的 index.js 中引入 ant-design 的样式

```javascript
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import "antd/dist/antd.css"; // 此处引入 antd 样式
import App from './App';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

为了测试 antd 引入是否成功，我们修改 App.js 文件，将原来的 Learn React 字符串更换成 antd 的 button 组件

```jsx
// App.js
import logo from './logo.svg';
import './App.css';
import { Button } from "antd"; // 引入 antd 的 Button 组件

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          {/* 使用 Button 组件并设置属性 */}
          <Button type="primary" size="large">Learn React</Button>
        </a>
      </header>
    </div>
  );
}

export default App;

```

修改完成后保存，然后转到 electron 应用界面，可以发现原来的字符串已经变成了 antd 的 Button 组件，说明 antd 引入成功

![image-20220210215626302](https://tva1.sinaimg.cn/large/008i3skNly1gz8rdzvpjzj30u013bta6.jpg)

继续～我们在项目中使用了 react-router-dom 来管理我们的路由，先在 src 路径下新建两个文件夹 routers 和 views，分别用来存放我们的路由配置文件和页面组件

![image-20220210222700894](https://tva1.sinaimg.cn/large/008i3skNly1gz8s9tmd1wj30dw07gjri.jpg)

```jsx
/**
 * @description Home 组件
 */

const Home = () => (<>This is Home</>);

export default Home;
```

```jsx
/**
 * @description About 组件
 */

const About = () => (<>This is About</>);

export default About;
```

```jsx
/**
 * @description 路由配置文件
 */

import { lazy, Suspense } from "react"; // 使用 React.lazy 进行代码分割
import { HashRouter, Link, Route, Switch } from "react-router-dom";
// 引入 Home 和 About 组件
const Home = lazy(() => import("../views/Home"));
const About = lazy(() => import("../views/About"));

const Router = () => (
    <HashRouter>
        <Suspense fallback={<div>Error !!!</div>}>
            <div>
                <Link to="/">Home</Link>|
                <Link to="/about">About</Link>
            </div>
            <div>
                <Switch>
                    <Route path="/" exact component={Home}></Route>
                    <Route path="/about" component={About}></Route>
                </Switch>
            </div>
        </Suspense>
    </HashRouter>
);

export default Router;
```

> 这里使用 HashRouter 而不使用 BrowserRouter 是因为 BrowserRouter 同时使用 lazy 懒加载时，使用 electron- packager 打包后容易找不到某些 css 和 js文件
>
> 本文使用 electron-builder 进行打包，从而可以避开这个坑

然后修改一下 App.js ，引入我们配置好的路由

```jsx
import logo from './logo.svg';
import './App.css';

import Router from './routers/Router'; // 引入路由配置文件

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <Router />
      </header>
    </div>
  );
}

export default App;

```

![image-20220210223311267](https://tva1.sinaimg.cn/large/008i3skNly1gz8sg8i704j30u013b402.jpg)

点击 Home 和 About 链接下面文字可以切换说明路由配置成功

## 使用 electron-builder 打包项目

1. 安装 electron-builder

   ```bash
   # 先安装 electron-builder，这里我选择全局安装
   sudo npm install electron-builder -g
   ```

   

2. 修改 main.js 文件

   ```javascript
   const { app, BrowserWindow } = require('electron');
   const path = require("path"); // 此处引入 path
   let mainWindow
   
   function createWindow() {
       require("@electron/remote/main").initialize();
       mainWindow = new BrowserWindow({
           width: 768,
           height: 1366,
           webPreferences: {
               nodeIntegration: true,
               contextIsolation: false,
               enableRemoteModule: true,
           }
       });
     
       require("@electron/remote/main").enable(mainWindow.webContents)
   
       // 开发
       // mainWindow.loadURL("http://localhost:3000/");
   
       // 发布: 打开 react 项目打包后的 index.html 文件
       mainWindow.loadURL(path.join("file://", __dirname, "./build/index.html"));
   
       // 关闭window时触发下列事件.
       mainWindow.on('closed', function () {
           mainWindow = null
       })
   }
   
   // 当 Electron 完成初始化并准备创建浏览器窗口时调用此方法
   app.on('ready', createWindow)
   ```

3. 修改 package.json 文件

   ```json
   {
     ...
     "author": {
       "name": "YOUR",
       "email": "YOUR_EMAIL"
     }, // 声明 author 打包需要
     ...
     "build": {
       "appId": "electron-react",
       "productName": "electron-react",
       "files": [
         "build/**/*",
         "node_modules/**/*",
         "public/**/*",
         "src/**/*",
         "main.js"
       ], // 此处要配置 files 字段，因为 electron-builder 默认不会打包 build 目录
       "extends": null
     }
   }
   ```

4. 在项目根目录下运行以下命令

   ```bash
   # 先进行 react 项目打包
   npm run build
   
   # 然后运行 electron-builder 打包
   
   sudo electron-builder
   ```




## 最后一坑（想不到吧！～）

> electron-builder 运行完成后，在项目目录下多出一个 dist 文件夹，里面有安装文件。当你欢天喜地安装完成双击打开程序时会报找不到 @electron/remote/main 模块的错，这时候我们依然需要修改一下 package.json

```json
{
  ...
  "dependencies": {
    "@electron/remote": "^2.0.1", // 手动添加这一行
    "@testing-library/jest-dom": "^5.16.2",
    "@testing-library/react": "^12.1.2",
    "@testing-library/user-event": "^13.5.0",
    "antd": "^4.18.6",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-router-dom": "^5.3.0",
    "react-scripts": "5.0.0",
    "web-vitals": "^2.1.4"
  },
}
```

然后删除掉 build 和 dist 文件夹，重复打包过程

```bash
# 先删除之前打包文件
rm -rf build
sudo rm -rf dist

# 重新打包
npm run build
sudo electron-builder

```

![image-20220211002420483](https://tva1.sinaimg.cn/large/008i3skNly1gz8vnwmnsxj30u013b402.jpg)

## 成功运行！撒花🎉～