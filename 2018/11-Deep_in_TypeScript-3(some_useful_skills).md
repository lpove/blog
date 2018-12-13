## 一些常用的小技巧

-   平时写代码的时候查询资料积累的一些小技巧

## 基本风格使用

```javascript
1、使用箭头函数代替匿名函数表达式。
2、只要需要的时候才把箭头函数的参数括起来。比如，(x) => x + x 是错误的，下面是正确的做法：
    x => x + x
    (x,y) => x + y
    <T>(x: T, y: T) => x === y

3、总是使用 {} 把循环体和条件语句括起来。
小括号里开始不要有空白。逗号，冒号，分号后要有一个空格。比如：
    for (let i = 0, n = str.length; i < 10; i++) { }
    if (x < 10) { }

4、function f(x: number, y: string): void { }
    每个变量声明语句只声明一个变量 。比如：使用 let x = 1; var y = 2; 而不是 let x = 1, y = 2;）。
5、如果函数没有返回值，最好使用 void
```

## class 的使用

-   在 TypeScript 中，我们可以通过 Class 关键字来定义一个类：

```javascript
class Greeter {
    static cname: string = "Greeter"; // 静态属性
    greeting: string; // 成员属行

    constructor(message: string) {
        // 构造函数 - 执行初始化操作
        this.greeting = message;
    }

    static getClassName() {
        // 静态方法
        return "Class name is Greeter";
    }

    greet() {
        // 成员方法
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

## 处理 json 和字符串

```javascript
let person = "{"name":"Sam","Age":"30"}";

const jsonParse: ((key: string, value: any) => any) | undefined = undefined;
let objectConverted = JSON.parse(textValue, jsonParse);
```

## 转换数字

-   基本的 JavaScript 的操作

```javascript
var n = +"1"; // the unary + converts to number
var b = !!"2"; // the !! converts truthy to true, and falsy to false
var s = "" + 3; // the ""+ converts to string via toString()

var str = "54";

var num = +str; //easy way by using + operator
var num = parseInt(str); //by using the parseInt operation
```

-   在 typescript 中推荐使用`Number`

```javascript
Number("1234"); // 1234
Number("9BX9"); // NaN

// 同等的字符串转换用
String(123); // '123'
```
