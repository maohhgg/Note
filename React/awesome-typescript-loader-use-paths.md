---
title: awesome-typescript-loader使用paths给目录使用别名
date: 2019/4/9
tag:
  - TypeScript
  - React
  - Webpack
urlname: awesome-typescript-loader-use-paths
---

在React实际使用过程中，使用相对路径引入其他组件，当路径层次太多时。比如像这样：` ../../../../Models`，这样显示的路径就不直接明朗，所以我们就需要使用 WebPack 来配置 alias 功能。在上篇[手动搭建脚手架](../typescript_react_webpack_antd/)中讲到，我使用了 TypeScript  + awesome-typescript-loader 组合来进行开发。而使用TypeScript 时，如果需要使用alias功能，就不能直接在 `webpack.config.js` 中直接配置。

<!--more-->

### 配置

TypeScript 多了一层转译到 JavaScript 的过程，所以我们需要在转译时对文件路径别名进行配置，

第一步，需要在 `tsconfig.json` 中添加 `baseUrl` 和 `paths`：

```json
{
    "compilerOptions": {
        ...
        "baseUrl": "./src/",
        "paths": {
            "@components/*": ["components/*"],
            "@services/*": ["services/*"],
            "@routers/*": ["routers/*"],
            "@models/*": ["models/*"],
            "@pages/*": ["pages/*"],
            "@utils/*": ["utils/*"]
        }
    }
  	...
}
```

接着启用Paths插件， awesome-typescript-loader 内部已经实现了这个插件。如果使用的是 ts-loader，就需要额外安装 [sconfig-paths-webpack-plugin](<https://github.com/dividab/tsconfig-paths-webpack-plugin>) 。在 `webpack.config.js` 中使用[TsConfigPathsPlugin](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fdividab%2Ftsconfig-paths-webpack-plugin)插件来读取相关配置文件。

awesome-typescript-loader 有一个特性，插件会将`tsconfig.json`中的配置文件自动添加到Webpack中。引用的文件是一个 Interface 时，只需要在`tsconfig.js` 文件中的`paths`进行指定即可找到相对应文件。当绝对路径最终引用的文件是一个 Function 时，如果不在 WebPack 配置中指定 alias 字段，则会出现相关错误，如`Module not found: Error: Can't resolve 'directory/function'`。需要解决此问题，需要在 WebPack alias中添加相同配置的。最终配置好的 `webpack.config.js` 文件如下：

```js
const {resolve} = require('path');
const {TsConfigPathsPlugin} = require('awesome-typescript-loader');

module.exports = {
	...
    resolve: {
        // 添加 '.ts' '.tsx' 作为可识别后缀
        extensions: ['.ts', '.tsx', '.js', '.json'],
        alias: {
            "@components": resolve(__dirname, 'src/components'),
            "@services": resolve(__dirname, 'src/services'),
            "@routers": resolve(__dirname, 'src/routers'),
            "@models": resolve(__dirname, 'src/models'),
            "@pages": resolve(__dirname, 'src/pages'),
            "@utils": resolve(__dirname, 'src/utils'),
        },
        plugins: [
            new TsConfigPathsPlugin({
                configFileName: resolve(__dirname, 'tsconfig.json')
            })
        ],
    }
}
```



