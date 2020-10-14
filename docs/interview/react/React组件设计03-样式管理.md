---
title: React组件设计03-样式管理
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# React组件设计03-样式管理

## 1. 认识 CSS 的局限性

#### 全局性

CSS的选择器是没有隔离性的，不管是使用命名空间还是 BRM 模式组织，最终都会污染全局命名空间，尤其是大型团队合作的项目，很难确定某个特定的类或者元素是否已经赋过样式，所以大部分情况下我们都会绞尽脑汁地新创建一个类名，而不是复用已有的类型。

**解决的方向**：`生成唯一的类名；shadow dom；内联样式；Vue-scoped方案`

#### 依赖

由于 CSS 的`全局性`，所以就产生了依赖问题：一方面我们需要在组件渲染前就需要先将 CSS 加载完毕，**但是很难清晰地定义某个特定组件依赖于某段特定的 CSS 代码**；另一方面**全局性导致你的样式可能被别的组件依赖(某种程度的细节耦合)**，你不能随便修改你的样式，以免破坏其他页面或组件的样式。如果团队没有制定合适的 CSS规范，代码很快就会失控。

**解决的方向**：`使用类似 BEM 命名规范来避免组件之间的命名突破，再通过创建优于复用，组合优于继承的原则，来避免组件间样式耦合`

#### 无用代码的移除

现代浏览器已支持 CSS 无用代码检查，但对于无组织的 CSS 效果不会太大

**解决的方向**：`如果样式的依赖比较明确，则可以安全地移除无用代码`

#### 压缩

选择器和类名的压缩可以减少文件的体积，提高加载的性能。压缩类名也会减低代码的可读性，变得难以调试。

**解决的方向**：`由工具来转换或创建类名`

#### 常量共享

常规的 CSS 很难做到在样式和 JS 之间共享变量，例如自定义主题色，通常通过内联样式来部分实现这种需求

**解决的方向**：`CSS-in-js`

#### CSS 解析方式的不确定性

CSS 规则的加载顺序是很重要的，它会影响属性应用的优先级，如果按需加载 CSS，则无法确保它们的解析顺序，进而导致错误的样式应用到元素上。

**解决的方向**：`避免使用全局样式，组件样式隔离；样式加载和组件生命周期绑定`

## 2. 组件的样式管理

#### 组件的样式应该高度可定制化

组件的样式应该是可以自由定制的，开发者应该考虑组件的各种使用场景，所以一个好的组件必须暴露相关的样式定制接口，至少需要支持为顶层元素配置`className`和`style`属性：

```js
interface ButtonProps {
  className?: string;
  style?: React.CSSProperties;
}
```

#### 避免使用内联 CSS

- style props 添加的属性不能自动增加厂商前缀，这可能会导致兼容性问题，如果添加厂商前缀又会让代码变得啰嗦
- 内联 CSS 不支持复杂的样式配置，比如伪元素、伪类、动画定义、媒体查询和媒体回退(对象不允许同名属性，例如`display: -webkit-flex; display: flex)`
- 内联样式通过 object 传入组件，内联的 object 每次渲染会重新生成，会导致组件重新渲染
- 不方便调试和阅读

**内联 CSS 适合用于设置动态且比较简单的样式属性**

#### 使用 CSS-in-js

社区上最流行的是`styled-components`，styled-components有以下特性：

- 自动生成类名，解决 CSS 的全局性和样式冲突，通过组件名来标志样式，自动生成唯一的类名，开发者不需要为元素定义类名。
- 绑定组件，解决了 CSS 的依赖问题，让组件 JSX 更加简洁
- 天生支持关键 CSS，样式和组件绑定，可以和组件一起进行代码分割和异步加载
- 自动添加厂商前缀
- 灵活的动态样式，通过 props 和全局 theme 来动态控制样式
- 提供了一些 CSS 预处理器的语法
- 支持 react-native
- 支持 stylint，编辑器高亮和智能提示
- 支持服务端渲染
- 符合分离展示组件和行为组件原则

##### 基本用法

```js
// 定义组件 props
const Title = styled.h1<{ active?: boolean}>`
  const: ${props => (props.active ? 'red' : 'gray')};
`;

// 固定或计算组件 props
const Input = styled.input.attrs({
  type: 'text',
  size: props => (props.small ? 5 : undefined),
})``;
```

##### 样式扩展

```js
const Button = styled.button`
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

// 覆盖和扩展已有的组件, 包含styled生成的组件还是自定义组件(通过className传入)
const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

```

##### mixin 机制

在 SCSS 中，mixin 是重要的 CSS 复用机制，styled-components 也可以实现：

**定义**：

```js
import { css } from 'styled-components';

// utils/styled-mixins.ts
export function truncate(width) {
  return css`
    width: ${width};
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  `;
}

```

**使用**：

```js
import { truncate } from '~/utils/styled-mixins';

const Box = styled.div`
  // 混入
  ${truncate('250px')}
  background: papayawhip;
`;

```

##### 类 SCSS 的语法

```js
const Example = styled(Component)`
  // 自动厂商前缀
  padding: 2em 1em;
  background: papayawhip;

  // 伪类
  &:hover {
    background: palevioletred;
  }

  // 提供样式优先级技巧
  &&& {
    color: palevioletred;
    font-weight: bold;
  }

  // 覆盖内联css样式
  &[style] {
    font-size: 12px !important;
    color: blue !important;
  }

  // 支持媒体查询
  @media (max-width: 600px) {
    background: tomato;

    // 嵌套规则
    &:hover {
      background: yellow;
    }
  }

  > p {
    /* descendant-selectors work as well, but are more of an escape hatch */
    text-decoration: underline;
  }

  /* Contextual selectors work as well */
  html.test & {
    display: none;
  }
`;

```

**引用其他组件**

由于 styled-components 的类名是自动生成的，所以不能直接在选择器中声明它们，但可以在模版字符串中引用其他组件：

```js
const Icon = styled.svg`
  flex: none;
  transition: fill 0.25s;
  width: 48px;
  height: 48px;

  // 引用其他组件的类名. 这个组件必须是styled-components生成或者包装的组件
  ${Link}:hover & {
    fill: rebeccapurple;
  }
`;

```

##### JS 带来的动态性

**媒体查询帮助方法**：

```js
// utils/styled.ts
const sizes = {
  giant: 1170,
  desktop: 992,
  tablet: 768,
  phone: 376,
};

export const media = Object.keys(sizes).reduce((accumulator, label) => {
  const emSize = sizes[label] / 16;
  accumulator[label] = (...args) => css`
    @media (max-width: ${emSize}em) {
      ${css(...args)}
    }
  `;
  return accumulator;
}, {});

```

**使用**：

```js
const Container = styled.div`
  color: #333;
  ${media.desktop`padding: 0 20px;`}
  ${media.tablet`padding: 0 10px;`}
  ${media.phone`padding: 0 5px;`}
`;

```

##### 绑定组件的全局样式

全局样式和组件生命周期绑定，当组件卸载时也会删除全局样式，全局样式通常用于覆盖一些第三方组件样式

```js
const GlobalStyle = createGlobalStyle`
  body {
    color: ${props => (props.whiteColor ? 'white' : 'black')};
  }
`

// Test
<React.Fragment>
  <GlobalStyle whiteColor />
  <Navigation /> {/* example of other top-level stuff */}
</React.Fragment>

```

##### 提升开发体验

可以使用`babel-plugin-styled-components`或`babel macro`来支持服务端渲染、样式压缩和提升 debug 体验，推荐使用 macro 形式，无需安装和配置 babel 插件，在 create-react-app 中已内置支持：

```js
import styled, { createGlobalStyle } from 'styled-components/macro';

const Thing = styled.div`
  color: red;
`;
```

##### styled-components的局限性

比较能想到的局限性是性能问题：

- css-in-js：需要一个 JS 运行时，会增加 js 包体积（大约 15KB）
- 相比原生 CSS 会有更多节点嵌套和计算消耗，这个对于复杂的组件树的渲染影响尤为明显
- 不能抽取为 CSS 文件

##### 一些开发规范

- 避免无意义的组件名，比如`Div`、`Span`等这类直接照搬元素名的无意义的组件命名
- 在一个文件中定义 styled-components 组件，对于比较**简单的组件**，一般会在同一个文件中定义styled-components组件就可以，对于**比较复杂的页面组件**来说，会让文件变得很臃肿，扰乱组件的主体，一般会抽取到单独的`styled.tsx`文件中

#### 通用组件库不应该耦合 CSS-in-js/CSS-module 的方案

如果是作为第三方组件库形式开发, 个人觉得不应该耦合各种 CSS-in-js/CSS-module. 不能强求你的组件库使用者耦合这些技术栈, 而且部分技术是需要构建工具支持的. 建议使用原生 CSS 或者将 SCSS/Less 这些预处理工具作为增强方案

#### 优先使用原生 CSS

对于部分极致要求性能的组件, 一般我会回退使用原生 CSS, 再配合 BEM 命名规范. 这种最简单方式, 能够满足大部分需求

#### 使用 svgr 转换 svg 图标

`svgr`可以将 svg 文件转换为 React 组件，可就是一个普通的 JS 模块，有以下优势：

- 转换为普通 JS 文件，方便代码分割和异步加载
- 相比 svg-sprite 和 iconfont 方案更容易管理
- svg 可以通过 CSS/JS 配置，可操作性更强，相比 iconfont 支持多色
- 支持 svgo 压缩

**基本用法**：

```js
import starUrl, { ReactComponent as Star } from './star.svg';
const App = () => (
  <div>
    <img src={starUrl} alt="star" />
    <Star />
  </div>
);

```



## 3. 规范

#### 促进建立统一的 UI 设计规范

- 提高团队协作效率
- 提高组件的复用率，统一的组件规范可以让组件更好管理
- 保持产品迭代过程中品牌一致性

#### CSS编写规范

- [编码规范 by @mdo](https://codeguide.bootcss.com/)bootstrap使用的规范
- [Aotu 实验室代码规范](https://guide.aotu.io/docs/css/code.html#CSS3)

#### 使用[stylint](https://stylelint.io/)进行样式规范检查

