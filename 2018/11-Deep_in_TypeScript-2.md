## 写在最前面

-   刚开始写 typescript 遇到的问题和简单的解决方案。

## Q&A

### 1、是否所有变量都需要做类型注解？

-   这个分情况，原则上来说，我们希望能对所有的值都做类型注解。

-   对于 TS 编译器来说，如果声明变量时没有做类型注解，那么 TS 会根据赋值自动推导出变量类型。这一点大多数情况下很完美，很方便，但是有一些列外：

-   后面赋值不同类型的值
    当你后面需要重新对该变量赋值其他类型时，那么 TS 会给出错误，因为与 TS 初始推导出的类型不一致了。

```javascript
let a = 1;
a = "string"; // error!

let obj = {
    name: "John"
};
obj.age = 20; // error!
obj.name = 250; // error!

// better
let a: string | number = 1;
a = "string"; // ok

let obj: {
    name: string | number,
    age?: number
} = {
    name: "John"
};
obj.age = 20; // ok
obj.name = 250; // ok
```

> 初始赋值时不是明确的值
> 即该值当前 TS 并不知道其类型，比如来自于后端接口返回的值、其他为明确声明类型的函数返回等。即没有初始化或者 TS 无法根据初始化值推导出类型，则会默认为 any 类型。

### 2、在对接口的时候我们定义接口的参数值

-   其实这种情况下不会报错，但是这样子会丢失类型检查和代码提示功能。所以这种情况，最好可以添加类型声明和注解。

```
// normal
http.get('/api')
    .then(resp => {
        let data = resp; // data: any
    });


// good one
interface IResponse {
    code: number;
    data: IUser;
}


http.get('/api')
    .then(resp => {
        let data: IResponse = resp; // data: any

    });
```

> 如果是上面两种情况，则需要提前定义好类型，并添加类型注解。否则，我们对于是否添加类型注解，鼓励，但不强求。因为大多数情况，我们在初始化赋值时 TS 就能很好的帮助我们自动确认好类型，并且通过 `typeof` 也可以获取该值的类型。一举两得！

### 3、巧用 `type` 定义类型

```javascript

// good one
let user = {
    name: 'Lucy',
    age: 20
}
type User = typeof user;


// deprecate
type User = {
    name: string;
    age: number;
};
let user: User = {
    name: 'Lucy',
    age: 20'
}
```

> 如上第一种写法，我们在声明 user 变量时，即得到了值，又获得了类型！反之，第二种写法就有点啰嗦了。

### 4、使用 TS 改写当前代码遇到各种错误问题？

-   **对象属性不存在错误:**:
    这种情况一般在于，该对象值 TS 知道其有明确类型（不是 any，如果是 any 就不会报错了），但是当前要访问的属性不存在与其已知类型结构。这种情况分两种办法解决： - 如果能修改该值的类型声明，那么`添加上缺损值的属性即可`； - 否则，使用 `// @ts-ignore` 注释，或者使用类型断言，强制为 any 类型：(this.props as any).notExists

-   **类型不明确的错误:**
    即一个值的类型可能被注解为联合类型，那么在直接访问时，TS 无法确定当前值到底属于哪个精确的类型，所以会报告错误。这种情况有以下解决拌饭：

        - 使用类型保护(type guards)
        - 使用类型断言
        - 使用 // @ts-ignore 注释

> 应该优先考虑类型保护，因为类型保护本质上就是增加代码逻辑，帮助 TS 理解确定当前类型，所以代码也会更健壮。而后两种办法，除非明确知道此时该值就是确定的类型，否则即使通过了 TS 编译器，在代码执行阶段，依然有可能出错！

-   **值可能不存在的或为 undefined 的错误:**

    -   这种情况其实是上面提到的类型不明确错误的一种，一般发生在可选属性或者可选参数时。解决这些情况，最简单的就是使用非空类型断言（前提是确认该值确实是非空）：

    -   非空类型断言的形式是在值后面添加半角感叹号：

```javascript
someVar!.toString();
```
