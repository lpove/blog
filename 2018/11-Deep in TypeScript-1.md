## 写在最前面

-   最近学习 ts，公司中大佬做了一次关于 ts 的分享，收获颇丰，自己总结了一下笔记，下面给大家分享一下。
-   第一篇主要是分享几个关于 ts 类型的几个细节的问题。

## ts 类型

-   基本类型
    -   boolean、number、string、symbol、object、null、undefined
    -   enum
    -   array、tuple
    -   any、void、never、unknown

## 思考问题？

-   1、Object 和 object 的区别？
-   2、String 和 string 的区别？
-   3、null 和 undefined 的区别？
-   3、any、void、never、unknown 的区别？

### 1.1 String 和 string 的区别？

-   思考下面的代码

```javascript
"123" === new String("123"); // ??
// false
typeof new String("123"); // ??
// object

//b 包装类型
let a = "123",
    b = new String("123");
a.name = b.name = "string";
console.log(a.name, b.name); // ??
```

> 上面的 a.name 会打印 undefined,b.name 可以正常的打印出 'string'

### 区分原始类型和包装类型（wrapper object）

-   而 TS 中的 string、boolean、number 等声明类型，则是指原始的基本数据类型。

```javascript
let str: string = new String(123); // error: Type 'String' is not assignable to type 'string'.
let str: string = '123'; // ok
let str: String = '123'; // ok

let a = new String(1234);
let: str: typeof '1234'; // str: string
let str: typeof a; //str: String
```

### 2.1 Object 和 object 的区别？

-   在 js 中，Boolean、String、Number 为基本包装类型的构造函数（类）。在 TS 中，他们在类型声明上下文中，则指代接口定义：

```javascript
interface String {
    /** Returns a string representation of a string. */
    toString(): string;

    /**
      * Returns the character at the specified index.
      * @param pos The zero-based index of the desired character.
      */
    charAt(pos: number): string;

    /**
      * Returns the Unicode value of the character at the specified location.
      * @param index The zero-based index of the desired character. If there is no character at the specified index, NaN is returned.
      */
    charCodeAt(index: number): number;


    ...
}

```

> 即，在 TS 中，new String(...) 这里的 String 是指对于 interface String 的类实现（class implements）。

-   而在类型声明上下文中，String 则指的就是这个 interface String。

-   2.1.1 Object 其实也是一个接口声明，可以去看看 ts 源代码的 Object 是怎么声明的。任意类型都是 Object

```javascript
let num: Object = 1; // ok
let str: Object = "a"; // ok
let obj: Object = {foo: 123}; // ok
let nul: Object = null; // error
let undef: Object = undefined; // error

obj.foo; // error: Property 'foo' does not exist on type '{}'
obj.toString(); // ok
```

> 事实上，声明为 Object 可以赋予除了 null、undefined 以外的任意值！此时访问这些声明的变量，都可以访问 Object 接口所定义的几个基本方法。

-   2.1.2 {}:而空的花括号{}类型，则和 Object 很类似，同样可以接受任意类型的值。它是指空对象类型。声明为{}类型，则没有任何成员变量可以访问（连 Object.prototype.toString 等方法都不可以）

```javascript
let num: {} = 1; // ok
let str: {} = "a"; // ok
let obj: {} = {foo: 1234}; // ok
let nul: {} = null; // error
let undef: {} = undefined; // error

obj.foo; // error: Property 'foo' does not exist on type '{}'
obj.toString(); // error: Property 'toString' does not exist on type '{}'
```

-   2.1.3 object : object 类型，则单纯指代非 string、number、boolean、symbol、null、undefined 的其他类型！与{}类似，同样没有任何成员属性或方法可以访问！

```javascript
// 这么赋值
let num: object = 1; // error
let str: object = "a"; // error
let obj: object = {foo: 1234}; // ok
let nul: object = null; // error
let undef: object = undefined; // error

obj.foo; // error: Property 'foo' does not exist on type 'object'
obj.toString(); // error: Property 'toString' does not exist on type 'object'
```

-   类型的检测的宽泛度：**类型限制范围上：any > {} ~ Object > object**

> 总结: 表示基本对象类型时，应当总是使用 object 类型，或者使用接口定义结构化对象。

### 3.1 null 和 undefined 的区别？

-   ts 中也有 null 和 undefined 类型，声明为这两种类型的值，也只能赋予同名值：

```
let a: null = null;
let b: undefined = undefined;
```

-   在默认情况下，null 和 undefined 这两个值（不是指类型！！）被当作其他类型的子类型，即可以赋予任意其他类型声明的变量。但是在开启了 --strictNullChecks 编译选项后，他们则只能被赋予 void 类型，或者各自的同名类型。

```
let a: void = null;
let b: void = undefined;
```

### 4.1 any、void、never、unknown 的区别

-   4.1.1 any ts 检测弱，兼容性问题解决方案。
-   成员访问无限制

```
let user: any = {};

user.name // ok
```

> 如以上例子中，user 被声明为 any 类型，即使其没有 name 这个属性，tsc 也不会对其进行检查。所以 any 可以用来指代哪些由外部传入、服务端返回等黑盒子结构的数据！！

-   事实上，任意未明确声明类型并切无法推导出类型的值都默认为 any 类型。

```
let a; // a: any
a = 1;
let a = 1; //a: number
```

-   4.1.2 void

void 应当仅仅用于函数声明，即没有明确返回值的函数，应该被声明为 void 类型。将 void 用户变量声明，则只能为其赋予 null 或 undefined。

-   4.1.3 never

never 用于函数返回值时，表示函数有抛出异常，没有正常执行到底。用于变量声明，无法为其赋予任何值！

never 是所有类型的子类型并且可以赋值给所有类型。
没有类型是 never 的子类型或能赋值给 never（never 类型本身除外）。

在函数表达式或箭头函数没有返回类型注解时，如果函数没有 return 语句，或者只有 never 类型表达式的 return 语句，并且如果函数是不可执行到终点的（例如通过控制流分析决定的），则推断函数的返回类型是 never。

在有明确 never 返回类型注解的函数中，所有 return 语句（如果有的话）必须有 never 类型的表达式并且函数的终点必须是不可执行的。

-   never 还可以用于映射类型中表达式，用于删除/过滤当前类型

```
let a: string | never; // a: string
type Exclude<T, K> = T extends K ? never: T;
```

-   4.1.4 unknown

> unknown 相对于 any，任意类型都可以赋值给 unknow，但是不可对其进行任何访问操作（仅仅为类型安全，any 操作访问也安全）

```
let a: any = 123;
a.toFixed(); //ok
let a: unknown = 123;
a.toFixed(); // error
```

## 未完待续...
