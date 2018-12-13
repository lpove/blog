## 写在最前面

-   为了在 react 中更好的使用 ts，进行一下讨论
-   怎么合理的再 react 中使用 ts 的一些特性让代码更加健壮

> 讨论几个问题，react 组件的声明？react 高阶组件的声明和使用？class 组件中 props 和 state 的使用？...

### 在 react 中使用 ts 的几点原则和变化

-   所有用到`jsx`语法的文件都需要以`tsx`后缀命名
-   使用组件声明时的`Component<P, S>`泛型参数声明，来代替 PropTypes！
-   全局变量或者自定义的 window 对象属性，统一在项目根下的`global.d.ts`中进行声明定义
-   对于项目中常用到的接口数据对象，在`types/`目录下定义好其结构化类型声明

### 声明 React 组件

-   react 中的组件从定义方式上来说，分为类组件和函数式组件。

-   类组件的声明

```javascript
class App extends Component<IProps, IState> {
    static defaultProps = {
        // ...
    }

    readonly state = {
        // ...
    };
    // 小技巧：如果state很复杂不想一个个都初始化，可以结合类型断言初始化state为空对象或者只包含少数必须的值的对象：  readonly state = {} as IState;
}
```

-   [ts 断言参考资料](https://ts.xcatliu.com/basics/type-assertion.html)

> 需要特别强调的是，如果用到了`state`，除了在声明组件时通过泛型参数传递其`state`结构，还需要在初始化`state`时声明为 `readonly`

这是因为我们使用 `class properties` 语法对`state`做初始化时，会覆盖掉`Component<P, S>`中对`state`的`readonly`标识。

### 函数式组件的声明

```javascript
// SFC: stateless function components
// v16.7起，由于hooks的加入，函数式组件也可以使用state，所以这个命名不准确。新的react声明文件里，也定义了React.FC类型^_^
const List: React.SFC<IProps> = props => null;
```

### class 组件都要指明 props 和 state 类型吗？

-   `是的`。只要在组件内部使用了`props`和`state`，就需要在声明组件时指明其类型。
-   但是，你可能发现了，只要我们初始化了`state`，貌似即使没有声明 state 的类型，也可以正常调用以及`setState`。没错，实际情况确实是这样的，但是这样子做其实是让组件丢失了对`state`的访问和类型检查！

```javascript
// bad one
class App extends Component {
    state = {
        a: 1,
        b: 2
    }

    componentDidMount() {
        this.state.a // ok: 1

        // 假如通过setState设置并不存在的c，TS无法检查到。
        this.setState({
            c: 3
        })；

        this.setState(true)； // ???
    }
    // ...
}

// React Component
class Component<P, S> {
        constructor(props: Readonly<P>);
        setState<K extends keyof S>(
            state: ((prevState: Readonly<S>, props: Readonly<P>) => (Pick<S, K> | S | null)) | (Pick<S, K> | S | null),
            callback?: () => void
        ): void;
        forceUpdate(callBack?: () => void): void;
        render(): ReactNode;
        readonly props: Readonly<{ children?: ReactNode }> & Readonly<P>;
        state: Readonly<S>;
        context: any;
        refs: {
            [key: string]: ReactInstance
        };
    }


// interface IState{
//    a: number,
//    b: number
// }

// good one
class App extends Component<{}, { a: number, b: number }> {

    readonly state = {
        a: 1,
        b: 2
    }

    //readonly state = {} as IState,断言全部为一个值

    componentDidMount() {
        this.state.a // ok: 1

        //正确的使用了 ts 泛型指示了 state 以后就会有正确的提示
        // error: '{ c: number }' is not assignable to parameter of type '{ a: number, b: number }'
        this.setState({
            c: 3
        })；
    }
    // ...
}
```

### 使用 react 高阶组件

> [什么是 react 高阶组件？装饰器？](https://juejin.im/post/5b5146596fb9a04fac0d0dc3)

-   因为 react 中的高阶组件本质上是个高阶函数的调用，所以高阶组件的使用，我们既可以使用函数式方法调用，也可以使用装饰器。但是在 TS 中，编译器会对装饰器作用的值做签名一致性检查，而我们在高阶组件中一般都会返回新的组件，并且对被作用的组件的`props`进行修改（添加、删除）等。这些会导致签名一致性校验失败，`TS`会给出错误提示。这带来两个问题：

#### 第一，是否还能使用装饰器语法调用高阶组件？

-   这个答案也得分情况：如果这个高阶组件正确声明了其函数签名，那么应该使用函数式调用，比如 `withRouter`：

```javascript
import {RouteComponentProps} from "react-router-dom";

const App = withRouter(
    class extends Component<RouteComponentProps> {
        // ...
    }
);

// 以下调用是ok的
<App />;
```

> 如上的例子，我们在声明组件时，注解了组件的 props 是路由的`RouteComponentProps`结构类型，但是我们在调用 App 组件时，并不需要给其传递`RouteComponentProps`里说具有的`location`、`history`等值，这是因为`withRouter`这个函数自身对齐做了正确的类型声明。

#### 第二，使用装饰器语法或者没有函数类型签名的高阶组件怎么办？

---

### 如何正确的声明高阶组件？

-   就是将高阶组件注入的属性都声明可选（通过`Partial`这个映射类型），或者将其声明到额外的`injected`组件实例属性上。
    我们先看一个常见的组件声明：

```javascript
import { RouteComponentProps } from 'react-router-dom';

// 方法一
@withRouter
class App extends Component<Partial<RouteComponentProps>> {
    public componentDidMount() {
        // 这里就需要使用非空类型断言了
        this.props.history!.push('/');
    }
    // ...
});

// 方法二
@withRouter
class App extends Component<{}> {
    get injected() {
        return this.props as RouteComponentProps
    }

    public componentDidMount() {
        this.injected.history.push('/');
    }
    // ...
```

#### 如何正确的声明高阶组件？

```javascript
interface IUserCardProps {
    name: string;
    avatar: string;
    bio: string;

    isAdmin?: boolean;
}
class UserCard extends Component<IUserCardProps> {
    /* ... */
}
```

> 上面的组件要求了三个必传属性参数：name、avatar、bio，isAdmin 是可选的。加入此时我们想要声明一个高阶组件，用来给 UserCard 传递一个额外的布尔值属性 visible，我们也需要在 UserCard 中使用这个值，那么我们就需要在其 props 的类型里添加这个值：

```javascript
interface IUserCardProps {
    name: string;
    avatar: string;
    bio: string;
    visible: boolean;

    isAdmin?: boolean;
}
@withVisible
class UserCard extends Component<IUserCardProps> {
    render() {
        // 因为我们用到visible了，所以必须在IUserCardProps里声明出该属性
        return <div className={this.props.visible ? '' : 'none'}>...</div>
    }
}

function withVisiable(WrappedComponent) {
    return class extends Component {
        render() {
            return <WrappedComponent {..this.props}  visiable={true} />
        }
    }
}
```

-   但是这样一来，我们在调用 UserCard 时就会出现问题，因为 visible 这个属性被标记为了必需，所以 TS 会给出错误。这个属性是由高阶组件注入的，所以我们肯定是不能要求都再传一下的。

可能你此时想到了，把 visible 声明为可选。没错，这个确实就解决了调用组件时 visible 必传的问题。这确实是个解决问题的办法。但是就像上一个问题里提到的，这种应对办法应该是对付哪些没有类型声明或者声明不正确的高阶组件的。

所以这个就要求我们能正确的声明高阶组件：

```javascript
interface IVisible {
    visible: boolean;
}

//排除 IVisible
function withVisible<Self>(WrappedComponent: React.ComponentType<Self & IVisible>): React.ComponentType<Omit<Self, "visible">> {
    return class extends Component<Self> {
        render() {
            return <WrappedComponent {...this.props} visible={true} />;
        }
    };
}
```

如上，我们声明 withVisible 这个高阶组件时，利用泛型和类型推导，我们对高阶组件返回的新的组件以及接收的参数组件的 props 都做出类型声明。

### 参考：

-   组内大佬的 wiki
