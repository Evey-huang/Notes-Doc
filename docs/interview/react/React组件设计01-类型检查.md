---
title: React组件设计01-类型检查
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# React组件设计01-类型检查

静态类型检查对于当今前端项目越来越不可或缺，尤其是大型项目。它可以：

- 在开发时就避免许多类型问题，减少低级错误
- 通过类型智能提示，可以提高编码的效率
- 有利于书写自描述的代码（类型即文档）
- 方便代码重构（配合IDE可以自动重构）

JavaScript的类型检查器主要有`TypeScript`和`Flow`，我只用过TypeScript，TypeScript很强大，可以避免很多坑，有更好的生态（例如第三方库类型声明），VSCode内置支持。所以本章基于TypeScript(v3.3)对React组件进行类型检查声明。

>React 组件类型检查依赖于`@types/react`和`@types/react-dom`

## 1. 函数组件

- 使用`ComponentNameProps`形式命名Props类型，并导出
- 优先使用`FC`类型来声明函数组件

`FC`是`FunctionComponent`的简写，`这个类型定义了默认的props(如children)以及一些静态属性(如defaultProps)`

```js
// 声明Props类型
export interface MyComponentProps {
  className?: string;
  style?: React.CSSProperties;
}

export const MyComponent: React.FC<MyComponentProps> = props => {
  return <div>hello react</div>
}
```

使用普通函数来进行组件声明：

```js
export interface MyComponentProps {
  className?: string;
  style?: React.CSSProperties;
  // 手动声明children
  children?: React.ReactNode
}
  
export function MyComponent(props: MyComponentProps) {
  return <div>hello react</div>
}
```



- 不要直接使用`export default`导出组件

这种方式导出的组件在`React Inspector`查看时会显示为`Unknown`

不建议：

```js
export default (props: {}) => {
  return <div>hello react</div>
}
```

如果非要使用，则使用`命名 function`定义：

```js
export default function Foo(props: {}) {
  return <div>hello react</div>
}
```

- 默认props声明

例如：

```js
export interface HelloProps {
  name?: string; // 可选属性
}
  
// 利用对象默认属性值语法
export const Hello: FC<HelloProps> = ({name = 'Evey'}) => <div>hello {name}</div>
```

- 泛型函数组件

泛型在列表型或容器型的组件中比较常用，直接使用`FC`无法满足需求:

```js
import React from 'react';

export interface ListProps<T> {
  visible: boolean;
  list: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
}

export function List<T>(props: ListProps<T>) {
  return <div />
}
  
function Test() {
  return (
    <List 
      list={[1,2,3]} 
      renderItem={i => {
       // 自动推断i为number类型
      }}
    />
  )
}
```

- 子组件声明

使用`Parent.Child`形式的JSX可以让节点父子关系更加直观，例如：

```js
import React, { PropsWithChildren } from 'react';

export interface LayoutProps {}
export interface LayoutHeaderProps {} // 采用ParentChildProps形式命名
export interface LayoutFooterProps {}

export function Layout(props: PropsWithChildren<LayoutProps>) {
  return <div className="layout">{props.children}</div>;
}

// 作为父组件的属性
Layout.Header = (props: PropsWithChildren<LayoutHeaderProps>) => {
  return <div className="header">{props.children}</div>;
};

Layout.Footer = (props: PropsWithChildren<LayoutFooterProps>) => {
  return <div className="footer">{props.children}</div>;
};

// Test
<Layout>
  <Layout.Header>header</Layout.Header>
  <Layout.Footer>footer</Layout.Footer>
</Layout>;
```

- Forwarding Refs

`Forwarding Refs`在16.3新增，用于转发ref，适用于HOC和函数组件

```js
/*****************************
 * MyModal.tsx
 ****************************/
import React, { useState, useImperativeHandle, FC, useRef, useCallback } from 'react';

export interface MyModalProps {
  title?: React.ReactNode;
  onOk?: () => void;
  onCancel?: () => void;
}

/**
 * 暴露的方法, 适用`{ComponentName}Methods`形式命名
 */
export interface MyModalMethods {
  show(): void;
}

export const MyModal = React.forwardRef<MyModalMethods, MyModalProps>((props, ref) => {
  const [visible, setVisible] = useState();

  // 初始化ref暴露的方法
  useImperativeHandle(ref, () => ({
    show: () => setVisible(true),
  }));

  return <Modal visible={visible}>...</Modal>;
});

/*******************
 * Test.tsx
 *******************/
const Test: FC<{}> = props => {
  // 引用
  const modal = useRef<MyModalMethods | null>(null);
  const confirm = useCallback(() => {
    if (modal.current) {
      modal.current.show();
    }
  }, []);

  const handleOk = useCallback(() => {}, []);

  return (
    <div>
      <button onClick={confirm}>show</button>
      <MyModal ref={modal} onOk={handleOk} />
    </div>
  );
};

```

## 2. 类组件

- 继承`Component`或`PureComponent`

```js
import React from 'react';

/**
 * 首先导出Props声明, 同样是{ComponentName}Props形式命名
 */
export interface CounterProps {
  defaultCount: number; // 可选props, 不需要?修饰
}

/**
 * 组件状态, 不需要暴露
 */
interface State {
  count: number;
}

/**
 * 类注释
 * 继承React.Component, 并声明Props和State类型
 */
export class Counter extends React.Component<CounterProps, State> {
  /**
   * 默认参数
   */
  public static defaultProps = {
    defaultCount: 0,
  }; // 编译时默认public， 可以省略

  /**
   * 初始化State
   */
  public state = {
    count: this.props.defaultCount,
  };

  /**
   * 声明周期方法
   */
  public componentDidMount() {}
  /**
   * 建议靠近componentDidMount, 资源消费和资源释放靠近在一起, 方便review
   */
  public componentWillUnmount() {}

  public componentDidCatch() {}

  public componentDidUpdate(prevProps: CounterProps, prevState: State) {}

  /**
   * 渲染函数
   */
  public render() {
    return (
      <div>
        {this.state.count}
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.decrement}>Decrement</button>
      </div>
    );
  }

  /**
   * ① 组件私有方法, 不暴露
   * ② 使用类实例属性+箭头函数形式绑定this
   */
  private increment = () => {
    this.setState(({ count }) => ({ count: count + 1 }));
  };

  private decrement = () => {
    this.setState(({ count }) => ({ count: count - 1 }));
  };
}
```



- 使用`static defaultProps`定义默认props

在defaultProps中定义的props可以不需要`?`可选操作符修饰，如上代码

- 子组件声明

类组件可以使用静态属性形式声明子组件

```js
export class Layout extends React.Component<LayoutProps> {
  public static Header = Header;
  public static Footer = Footer;

  public render() {
    return <div className="layout">{this.props.children}</div>;
  }
}

```



- 泛型

```js
export class List<T> extends React.Component<ListProps<T>> {
  public render() {}
}
```



## 3. 高阶组件

在React Hooks出来之前，`高阶组件`是React的一个重要逻辑复用方式，但是HOC容易造成嵌套地狱，对TypeScript类型化也不友好，新项目还是建议使用`React Hooks`，一个简单HOC：

```js
/**
 * 抽取出通用的高阶组件类型
 */
type HOC<InjectedProps, OwnProps = {}> = <P>(Component: React.ComponentType<P & InjectedProps>) => React.ComponentType<P & OwnProps>

  /**
 * 声明注入的Props
 */
export interface ThemeProps {
  primary: string;
  secondary: string;
}

export const withTheme: HOC<ThemeProps> = Component => props => {
   // 假设theme从context中获取
  const fakeTheme: ThemeProps = {
    primary: 'red',
    secondary: 'blue'
  }
  return <Component {...fakeTheme} {...props} />
}
```

使用HOC还有一些痛点：

- 无法完美使用ref
  - 在`React.forwardRef`发布之前，有一些库会使用innerRef或者wrapperRef转发给封装的组件的ref
  - 无法推断ref引用组件的类型，需要显示声明
- HOC类型报错很难理解

## 4. Render Props

React的props(包括children)并没有限定类型，它可以是一个函数，于是就有了render props，这是和HOC一样常见的模式

```js
import React from 'react';

export interface ThemeConsumerProps {}

export const ThemeConsumer = (props: ThemeConsumerProps) => {
  const fakeTheme = {primary: 'red', secondary: 'blue'};
  return props.children(fakeTheme)
}

// Test
<ThemeConsumer>{({primary}) => {return <div style={{color: primary}}/>}}</ThemeConsumer>
```



## 5. Context

Context提供了一种跨组件间状态共享机制

```js
import React, { FC, useContext } from 'react';

export interface Theme {
  primary: string;
  secondary: string;
}

/**
 * 声明Context的类型, 以{Name}ContextValue命名
 */
export interface ThemeContextValue {
  theme: Theme;
  onThemeChange: (theme: Theme) => void;
}

/**
 * 创建Context, 并设置默认值, 以{Name}Context命名
 */
export const ThemeContext = React.createContext<ThemeContextValue>({
  theme: {
    primary: 'red',
    secondary: 'blue',
  },
  onThemeChange: noop,
});

/**
 * Provider, 以{Name}Provider命名
 */
export const ThemeProvider: FC<{ theme: Theme; onThemeChange: (theme: Theme) => void }> = props => {
  return (
    <ThemeContext.Provider value={{ theme: props.theme, onThemeChange: props.onThemeChange }}>
      {props.children}
    </ThemeContext.Provider>
  );
};

/**
 * 暴露hooks, 以use{Name}命名
 */
export function useTheme() {
  return useContext(ThemeContext);
}

```



## 6. 其他

- 使用`handleEvent`命名事件处理器

如果存在多个相同事件处理器，则按照`handle{Type}{Event}`命名，例如handleNameChange

- 内置事件处理器的类型

`@types/react`内置了以下事件处理器的类型：

```js
type EventHandler<E extends SyntheticEvent<any>> = { bivarianceHack(event: E): void }['bivarianceHack'];
type ReactEventHandler<T = Element> = EventHandler<SyntheticEvent<T>>;
type ClipboardEventHandler<T = Element> = EventHandler<ClipboardEvent<T>>;
type CompositionEventHandler<T = Element> = EventHandler<CompositionEvent<T>>;
type DragEventHandler<T = Element> = EventHandler<DragEvent<T>>;
type FocusEventHandler<T = Element> = EventHandler<FocusEvent<T>>;
type FormEventHandler<T = Element> = EventHandler<FormEvent<T>>;
type ChangeEventHandler<T = Element> = EventHandler<ChangeEvent<T>>;
type KeyboardEventHandler<T = Element> = EventHandler<KeyboardEvent<T>>;
type MouseEventHandler<T = Element> = EventHandler<MouseEvent<T>>;
type TouchEventHandler<T = Element> = EventHandler<TouchEvent<T>>;
type PointerEventHandler<T = Element> = EventHandler<PointerEvent<T>>;
type UIEventHandler<T = Element> = EventHandler<UIEvent<T>>;
type WheelEventHandler<T = Element> = EventHandler<WheelEvent<T>>;
type AnimationEventHandler<T = Element> = EventHandler<AnimationEvent<T>>;
type TransitionEventHandler<T = Element> = EventHandler<TransitionEvent<T>>;
```

可以简洁地声明事件处理器类型：

```js
import { ChangeEventHandler } from 'react';

export const EventDemo: React.FC<{}> = props => {
  /**
   * 可以限定具体Target的类型
   */
  const handleChange = useCallback<changeEventHandler<HTMLInputElement>>(evt => {
    console.log(evt.target.value)
  }, []);
  
  return <input onChange={handleChange} />
}
```



- 为没有提供TypeScript声明文件的第三方库自定义模块声明

一般在项目根目录下放置一个`global.d.ts`，放置项目的全局声明文件（与tsconfig.json同一目录）

```js
// /global.d.ts

// 自定义模块声明
declare module 'awesome-react-component' {
  // 依赖其他模块的声明文件
  import * as React from 'react';
  export const Foo: React.FC<{ a: number; b: string }>;
}
```



- 开启strict模式

为了真正把TypeScript用起来，应该始终开启`strict`模式，避免使用`any`类型声明