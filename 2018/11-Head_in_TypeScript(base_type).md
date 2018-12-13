### 写在前面的话：

-   迫于业务升级，开始使用 typescript，这里来了解一下 typescript 的基本类型和泛型的使用。现在 typescript 已经 3.1 版本了，非常成熟了。

## typeScript 基础类型

-   下面只介绍一些区别于 JavaScript 的特殊类型

### Tuple 元组

-   元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。比如，你可以定义一对值分别为 string 和 number 类型的元组。

```javascript
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
```

### enum 枚举

-   enum 类型是对 JavaScript 标准数据类型的一个补充。像 C#等其他语言一样，使用枚举类型可以为一组数值赋予友好的名字。

```javascript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

-   默认情况下，从 0 开始为元素编号。 你也可以手动的指定成员的数值。 例如，我们将上面的例子改成从 1 开始编号：

```javascript
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
```

### Any

-   有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。 这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 any 类型来标记这些变量：

```javascript
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

```javascript
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

### Void

-   这玩意一般用在 func 上，刚刚和 Any 相反

```javascript
function warnUser(): void {
    console.log("This is my warning message");
}

//声明一个void类型的变量没有什么大用，因为你只能为它赋予undefined和null：
let unusable: void = undefined;
```

### Null 和 Undefined

-   TypeScript 里，undefined 和 null 两者各自有自己的类型分别叫做 undefined 和 null。 和 void 相似，它们的本身的类型用处不是很大：

```javascript
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

### Never

-   never 类型表示的是那些永不存在的值的类型。 例如， never 类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 never 类型，当它们被永不为真的类型保护所约束时。

```javascript
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}

// 推断的返回值类型为never
function fail() {
    return error("Something failed");
}

// 返回never的函数必须存在无法达到的终点
function infiniteLoop(): never {
    while (true) {}
}
```

## typescript 泛型

### 先来谈谈使用场景

-   模拟一个场景，当我们要使用一个服务器提供的不同数据，我们需要先建立一个中间件来进行处理(验证，容错，纠正)，再进行使用。使用 JavaScript 来写 。

### JavaScript

```javascript
// 模拟服务，提供不同的数据。这里模拟了一个字符串和一个数值
var service = {
    getStringValue: function() {
        return "a string value";
    },
    getNumberValue: function() {
        return 20;
    }
};

// 处理数据的中间件。这里用 log 来模拟处理，直接返回数据当作处理后的数据
function middleware(value) {
    console.log(value);
    return value;
}

// JS 中对于类型并不关心，所以这里没什么问题
var sValue = middleware(service.getStringValue());
var nValue = middleware(service.getNumberValue());
```

### typeScript

-   服务器返回的类型要进行一定的限制

```javascript
const service = {
    getStringValue(): string {
        return "a string value";
    },

    getNumberValue(): number {
        return 20;
    }
};
```

-   为了保证在对 sValue 和 nValue 的后续操作中类型检查有效，它们也会有类型(如果 middleware 类型定义得当，可以推导，这里我们先显示定义其类型)

```javascript
const sValue: string = middleware(service.getStringValue());
const nValue: number = middleware(service.getNumberValue());
```

> 换成 `typescript` 的时候我们的中间件 `middleware` ，需要返回正确的 `string`和`number`类型。我们需要怎么做啦？

### 方法一，特殊类型 any

```javascript
function middleware(value: any): any {
    console.log(value);
    return value;
}
```

> 这个方法可以让最后的类型检测通过，但是使用 `any` 的话，致使 `middleware` 就没有什么用了。

### 方法二，多个`middleware`

```javascript
function middleware1(value: string): string { ... }
function middleware2(value: number): number { ... }
```

-   使用 typescript 的重载(overload)

```javascript
function middleware(value: string): string;
function middleware(value: number): number;
function middleware(value: any): any {
    // 实现一样没有严格的类型检查
}
```

> 但是类型过多的话这种方法也不好使。

### 正解： 使用 typescript 泛型(Generic)

-   先简单的来说一下什么是泛型？
-   就是表示一个类型的变量，用他来代替某个实际的类型用于编程。

```javascript
function middleware<T>(value: T): T {
    console.log(value);
    return value;
}
```

> `middleware` 后面紧接的 `<T>` 表示声明一个表示类型的变量，`Value: T` 表示声明参数是 `T` 类型的，后面的 `: T` 表示返回值也是 `T` 类型的。那么在调用 `middlewre(getStringValue())` 的时候，由于参数推导出来是 `string` 类型，所以这个时候 T 代表了 `string`，因此此时 `middleware` 的返回类型也就是 `string`；而对于`middleware(getNumberValue())` 调用来说，这里的 `T` 表示了 `number`。

-   如果你使用 `vscode` 的话，我们默认你已经安装的支持 typescript 的环境。可以看到我们在推导类型和返回值类型的时候，`vscode` 会提示你对应的 `string` 和 `number`的类型。

## 参考

-   https://ts.xcatliu.com/basics/primitive-data-types.html
-   https://segmentfault.com/a/1190000010774159
