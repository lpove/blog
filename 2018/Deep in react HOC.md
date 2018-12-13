## 今日猫片

![image](https://user-gold-cdn.xitu.io/2018/12/12/167a08138e50493f?w=676&h=450&f=jpeg&s=55409)

> 优化一下 blog 界面，准备每次写都给大家看一张猫片（大雾）。

## 写在前面的话

-   开始学习 react 的时候，有一个难点，高阶组件。
-   以前写过一篇不太成熟的文章，这里忙里偷闲再来详细的理解一下。
    -   [高阶组件代理模式](https://juejin.im/post/5b5146596fb9a04fac0d0dc3)
-   最出名的高阶组件就是 redux 状态管理的 connect 组件。大家可以取看一看实现的源码。
    -   [redux connect 实现源码](https://github.com/lipeishang/react-redux-connect-demo)。

## 高阶函数的基本概念

> 想要理解高阶组件，我们先来看看高阶函数的含义。

-   函数可以作为参数被传递

```javascript
setTimeout(() => {
    console.log(1);
}, 1000);
```

-   函数可以作为返回值输出

```javascript
function foo(x) {
    return function() {
        return x;
    };
}
```

-   类似于`setTimeout()`,`setInterval()`就是普通的高阶函数。

```javascript
setTimeout();
setInterval();

// ajax
$.get("/api/v1", function() {
    console.log("data");
});
```

> 上面这两个就是标准的高阶函数

## 高阶组件 high order component ，简写 HOC

-（原来以前的 HOC 就是高阶组件）

## 高阶组件理解和使用

-   和高阶函数一样，需要函数去包裹一个组件，返回的是一个组件而已

```javascript
// A.js
import React, { Component } from 'react';
function A(WrapperedComponent){
    return class test extends Component{
        return <div>
            <WrapperedComponent />
        </div>
    }
}

export default A;

// 其他组件使用的时候
// B.js
import A from './A';

class B extends Component{
    return <div>
        hello world!!
    </div>
}
export default A(B)
```

## 高阶组件的使用

-   直接包裹 HOC(WrapperedComponent)
-   使用装饰器 @HOC
    -   要 nom run eject（这里就不详细写了）
    -   安装 babel 适配文件等

### 编写高阶组件

-   实现一个普通组件
-   函数包裹这个组件

### 一个简单的高阶组件 demo

-   需求： 有三个页面 A,B,C 需要共用的一个进度条组件。
-   https://codesandbox.io/s/rj3r509ljp (这里是源码)

```bash
-- index.ts
-- /src
---- HOCprogress.tsx
---- A.tsx
---- B.tsx
---- C.tsx

```

> HOCprogress.tsx(1)

```javascript
import React, {Component} from "react";

// 然后包裹一个 function，用WrapperedComponent传入 class 的 render()中。

function HOCprogress(WrapperedComponent, value: number) {
    //先写 class
    return class hocForm extends Component {
        render() {
            return (
                <div>
                    <WrapperedComponent />
                </div>
            );
        }
    };
}

export default HOCprogress;
```

-   优化一下，添加简单的进度条

```
// HOCprogress.tsx
import React, { Component } from "react";

function HOCprogress(WrapperedComponent, value: number) {
  return class hocForm extends Component {
    render() {

      // 添加样式
      const innerStyle = {
        padding:'10px',
        width: "100%"
      };
      const percentStyle = {
        width: `${value}%`,
        height: "20px",
        background:
          "url(https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=2440333743,1684406640&fm=26&gp=0.jpg)"
      };

      return (
          <div style={innerStyle}>
            <div style={percentStyle}> {value} %</div>
            <WrapperedComponent />
          </div>
      );
    }
  };
}

export default HOCprogress;
```

-   添加 A,B,C 三个文件
    > A.tsx

```javascript
import React, {Component} from "react";
// 引入高阶函数
import HOCprogress from "./HOCprogress.tsx";

class A extends Component {
    render() {
        return <div>这是 A 组件！</div>;
    }
}
// 使用高阶组件包裹 A 组件
export default HOCprogress(A, 56);
```

> B.tsx

```javascript
import React, {Component} from "react";
import HOCprogress from "./HOCprogress.tsx";

class B extends Component {
    render() {
        return <div>这是 B 组件！</div>;
    }
}

// 我们可以使用 @HOCprogress 装饰器这样的方式来替代函数包裹这种方式具体的见我的装饰器的那篇文章。
export default HOCprogress(B, 98);

// C.tsx 同上
```

> index.ts

```
import React from "react";
import C from "./C.tsx";
import B from "./B.tsx";
import A from "./A.tsx";

class App extends React.Component {
    render(){
        <div>
            <A />
            <B />
            <C />
        </div>
    }
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

### 最后来看看效果

![image](https://user-gold-cdn.xitu.io/2018/12/11/1679c8c4d18d3e05?w=690&h=451&f=jpeg&s=21058)

## 写在最后

-   当然高阶组件有非常多的用法还可以去了解和学习，这里只是粗浅的入门了解。
-   可以在代理模式中，去
    -   操作 props
    -   抽取组件状态
    -   访问 ref
    -   包装组件

## 参考

-   装饰器 https://juejin.im/post/5b51449a6fb9a04fdf39da4f
-   imooc https://www.imooc.com/video/18257
