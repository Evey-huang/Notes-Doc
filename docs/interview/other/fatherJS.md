---
title: 使用umi/father将npm包发布到私有仓库(Nexus)
order: 1
nav:
  order: 1
  title: 面试
toc: menu

---



# 使用umi/father将npm包发布到私有仓库(Nexus)

### 1. 新建repository

- 打开nexus，登录
- Create repository
- 选择npm(hosted)
- 填写repository相关信息，Hosted选择`Allow redeploy`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghlkaoumumj31n50u0aiw.jpg)

### 2. 配置npm

#### 查看仓库地址

在Respositories列表中选择Type为hosted的npm-internal仓库，点击`copy`查看

#### 配置仓库地址

```shell
// 将npm地址设置为私有地址
npm config set registry 仓库地址

// 将npm地址设为淘宝镜像地址
npm config set registry http://registry.npm.taobao.org/
```

#### 验证配置是否正确

```shell
npm config list
```

### 3. 添加nexus权限

在Realms菜单中，将npm` Bearer Token Realm`添加到`Active`中

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghliy1hw35j31n50u0n3o.jpg)

### 4. 上传nexus

#### 新建并初始化项目

```shell
- brew install yarn // 安装yarn
- mkdir myapp
- cd myapp
- yarn create umi // 选择library 和使用 typescript
- yarn install // 安装依赖
```

#### 配置

##### 目录大纲

```js
── README.md
├── fatherrc.ts
├── package.json
├── src
|  ├── index.tsx
|  ├── assets
|  |  ├── empty.png
|  ├── components
|  |  ├── Button
|  |  |  ├── index.tsx
|  |  |  └── index.mdx
|  |  ├── EmptyPage
|  |  |  ├── index.tsx
|  |  |  ├── index.mdx
|  |  |  └── style.less
```

###### fatherrc.ts

```js
import { IBundleOptions } from 'father';

const options: IBundleOptions = {
  cjs: 'rollup',
  esm: 'rollup',
  umd: {
    name: 'test-component',
    sourcemap: true,
    minFile: true,
  },
  doc: { typescript: true },
};

export default options;

```

###### package.json

```json
{
  "name": "myapp",
  "version": "0.0.1",
  "description": "",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "authors": {
    "name": "",
    "email": ""
  },
  "repository": "/myapp",
  "scripts": {
    "dev": "father doc dev",
    "build": "father build"
  },
  "peerDependencies": {
    "react": "16.x",
    "antd": "4.5.2"
  },
  "devDependencies": {
    "father": "^2.16.0",
    "typescript": "^3.3.3"
  },
  "license": "MIT"
}

```

###### src

index.tsx

```js
export { default as Button } from './components/Button';
export { default as EmptyPage } from './components/EmptyPage';

```

component -> EmptyPage -> index.tsx

```js
import React from 'react';
import { Empty } from 'antd';
const SIMPLE_IMAGE = require('../src/assets/empty.png');
import '../EmptyPage/styles/index.less';

export interface EmptyPageProps {}

const EmptyPage: React.FC<EmptyPageProps> = function({ children, ...props }) {
  return (
    <Empty className="empty" {...props} image={SIMPLE_IMAGE} description={false}>
      {children}
    </Empty>
  );
};

export default EmptyPage;

```



#### 打包

```shell
npm run build
```

#### 添加用户

```shell
npm adduser -registry 仓库地址
```

#### 上传包

```shell
npm publish
```

### 5. 在项目中使用

```shell
// 安装
npm install myapp@0.0.1

// 使用
import {EmptyPage} from 'myapp';
<EmptyPage />
```



### 6. 遇到的问题

- Q：通过import引用的图片未被打包进bundle

  A：解决办法：通过模块化的方式引用图片路径 const xxx = require('xxx')

  

### 7. 资料

- [father.js](https://github.com/umijs/father/tree/2.x)
- [将npm包发布在私有仓库（nexus）中](https://blog.csdn.net/wjyyhhxit/article/details/103595333)
- [nexus仓库](http://mvn.sky-cloud.net:8081/)



