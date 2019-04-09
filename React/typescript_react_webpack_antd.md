---
title: 手动搭建 TypeScript+React+Webpack+Ant Design of React
date: 2019/4/3
tag:
  - TypeScript
  - React
  - Webpack
  - antd
urlname: typescript-react-webpack-antd
---

我们在创建React项目的时，大多都是通过create-react-app来直接npm到本地的已经搭建好的脚手架，直接使用就好了。但有时各种脚手架工具不能满足我们的需求时，这就需要自己手动搭建。今天我们就手动搭建TypesSript+React+Webpack+Ant Design of React。

> 注意：
1. 在开始之前，我默认已经你在使用[Node.js](https://nodejs.org)的与[NPM](https://www.npmjs.com)，请确认自己的Node环境。
2. 本文所有操作均在Linux下完成，请酌情参考。但和Windows下的PowerShell或CMD区别不大。
3. 本文将循序渐进的搭建所需要的的工具，请耐心阅读。

<!--more-->

我们开始一个全新的项目，从头开始搭建。

### 创建项目

让我们从一个创建文件夹开始。我们暂时将其命名 `proj` ，也可以将其更改为你想要的任何名称。

```bash
mkdir proj
cd proj
```

首先，我们将按以下目录结构构建项目：

```
proj/
├─ dist/
└─ src/
   └─ components/
```

我们将使用Webpack默认约定，TypeScript源代码将保存在 `src` 目录，经过TypeScript编译和Webpack打包的 `main.js` 文件保存自 `dist` 目录。所有的组件将保存在 `src/components` 目录中。

```bash
mkdir -p src/components
```

Webpack 打包时会自动生成 `dist` 目录。

## 初始化项目

现在我们将这个目录变成一个npm包。

```bash
npm init -y
```

`-y` 参数表示一系列提示直接输入yes，我们所有的都使用默认值。也可以随时返回并在`package.json`为生成的文件中更改这些内容。

###  安装依赖项

首先确保全局安装Webpack。

```bash
npm i -g webpack webpack-cli
```

Webpack是一个将代码和所有依赖项捆绑到一个 `.js` 文件中的工具。

继续安装 React 和 React-DOM 依赖，并将其声明文件作为依赖项添加到您的`package.json`文件中：

```bash
npm i -S react react-dom @types/react @types/react-dom antd moment
```

使用 `@types` 前缀去得到 React 和 React-DOM 的声明文件，通常我们当导入类似 "react" 的路径时，TypeScritp会查看`react`包内部去得到声明文件，但是不是所有的包本身都包含了声明文件，所有我们需要从 `@types/react` 中获取声明文件。

接下来我们给生产环境添加 [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader) 和 [source-map-loader](https://www.npmjs.com/package/source-map-loader) 依赖。

```bash
npm install -D typescript awesome-typescript-loader source-map-loader ts-import-plugin style-loader css-loader
```

这两个依赖可以使 TypeScript 跟 Webpack 很好的协同工作，awesome-typescript-loader 在 `tsconfig.json` 配置下可以帮助Webpack更好的编译TypeScript代码。source-map-loader 可以在我们自己创建的代码资源文件时通知 Webpack，方便我们调试TypeScript源代码时同步最终的输出文件。ts-import-plugin 是在我们使用 Antd UI 组件库时，按需加载组件代码和样式的插件。

> 本文的加载插件使用了 awesome-typescript-loader ts-import-plugin 如需使用 [ts-loader ](https://github.com/TypeStrong/ts-loader) [babel-plugin-import](https://github.com/ant-design/babel-plugin-import) 请自行配置

### 创建 TypeScript 配置文件

在项目根目录下创建 `tsconfig.json`文件，用来指定编译项目所需的文件和编译器选项。

```json
{
    "compilerOptions": {
        "outDir": "./dist/",
        "sourceMap": true,
        "noImplicitAny": true,
        "module": "ESNext",
        "moduleResolution": "node",
        "target": "es6",
        "jsx": "react"
    },
    "include": [
        "./src/**/*"
    ]
}
```

因为我们使用了 ts-import-plugin 插件，在 `tsconfig.json` 配置中 `module` 的值必须为 `esnext`，并且需要将 `moduleResolution` 的值设置为 `node`

### 创建 Webpack 配置文件

在项目根目录下创建 `webpack.config.js`文件

```js
const tsImportPluginFactory = require('ts-import-plugin')

module.exports = {
    // entry: "./src/index.tsx",  // 默认地址 不需要额外配置
    
    mode: 'development',

    resolve: {
        // 添加 '.ts' '.tsx' 作为可识别后缀
        extensions: ['.ts', '.tsx', '.js', '.json']
    },

    module: {
        rules: [
            //对所有带有'.ts'或'.tsx'扩展名的文件进行处理 
            {
                test: /\.tsx?$/,
                loader: "awesome-typescript-loader",
                options: {
                    getCustomTransformers: () => ({
                        before: [tsImportPluginFactory({
                            libraryName: 'antd',
                            libraryDirectory: 'lib',
                            style: "css"
                        })],
                    }),
                },
                exclude: /node_modules/ // 排除依赖目录
            },

            { test: /\.css$/,loader: ["style-loader", "css-loader"] },
            
            { enforce: "pre",test: /\.js$/,loader: "source-map-loader"}
        ]
    },

}
```

### 编写示例代码

让我们使用 React 编写我们的组件。首先，在 `src/components` 中创建一个文件名为`Hello.tsx`，写了以下内容：

```ts
import * as React from "react";
import { LocaleProvider, DatePicker, message } from 'antd';

// 由于 antd 组件的默认文案是英文，所以需要修改为中文
import zhCN from 'antd/lib/locale-provider/zh_CN';
import * as moment from "moment"

interface IProps { }
interface IState {
    date: moment.Moment
}

export class Hello extends React.Component<IProps, IState> {
    state: IState;

    constructor(props: IProps){
        super(props);
        this.state = { date: null }
    }

    handleChange = (date: moment.Moment) => {
        if (!date) return;
        message.info(`您选择的日期是: ${date.format('YYYY-MM-DD')}`);
        this.setState({ date });
    }
    render() {
        const { date } = this.state;
        return (
            <LocaleProvider locale={zhCN}>
                <div style={{ width: 400, margin: '100px auto' }}>
                    <DatePicker onChange={this.handleChange} />
                    <div style={{ marginTop: 20 }}>
                        当前日期：{date ? date.format('YYYY-MM-DD') : '未选择'}
                    </div>
                </div>
            </LocaleProvider>
        );
    }
}
```

接下来在 `src` 中创建 `index.tsx` ：

```ts
import * as React from "react";
import * as ReactDOM from "react-dom";

import "antd/dist/antd.css";

import { Hello } from "./components/Hello";

ReactDOM.render(<Hello />, document.getElementById('app'));
```

我们还需要一个页面来显示我们的`Hello`组件。在项目根目录中创建 `index.html`：

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello React!</title>
    </head>
    <body>
        <div id="app"></div>

        <!-- Main -->
        <script src="./dist/main.js"></script>
    </body>
</html>
```



###  End

最后只需要

```bash
webpack
```

用浏览器打开 `index.html` ，你将看到一个时间选择器。

