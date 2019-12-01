---
title: 04 编写你的第一个TypeScript程序
date: 2019-12-01 11:48:40
tags: TypeScript
---

> 参考代码地址：https://github.com/geektime-geekbang/typescript-in-action

## 一、预装软件
- 1) Node.js
- 2) VSCode

## 
- 1)新建文件夹：`ts_in_action`;然后用VSCode打开该文件夹
- 2）ts_in_action目录下运行：

<!-- more -->

```
# 创建package.json
npm init -y
# 安装typescript
npm i typescript -g
# 创建typescript配置项(tsconfig.json)
tsc --init
```
- 3)ts_in_action目录下新建src，src中新建index.ts
  - `tsc index.ts`
  - 也可以借助ts官网的playground來获得编译后的代码

## 二、配置构建工具
- 1)先安装三个包
```
npm i webpack webpack-cli webpack-dev-server -D
```

- 2)webpack配置文件
> 配置时，要区分开发环境和生产环境（两个环境的功能是不一样的，需要做不同的事情）。为了工程可维护性，将配置文件分为开发环境配置、生产环境配置、公共配置，最后通过插件来合并

- 3)创建build目录，用于存放所有的webpack配置文件
    - 配置文件为：
        - `webpack.base.config.js`：公共环境的配置
            - 先指定入口文件配置：`entry: './src/index.ts',`
            - 输入：`output: { filename: 'app.js', }`；输入目录使用默认的（也就时dist目录）
            - 指定扩展名：`resolve: { extensions: ['.js', '.ts', '.tsx'], }`
            - 安装ts相应的loader：
        - `webpack.config.js`：所有配置文件的入口
        - `webpack.dev.config.js`：开发环境的配置
        - `webpack.pro.config.js`：生产环境的配置

### 公共环境的配置
```
// webpack.base.config.js

const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.ts',
    output: {
        filename: 'app.js'
    },
    resolve: {
        extensions: ['.js', '.ts', '.tsx'],
    },
    module: {
        rules: [
            {
                test: /\.tsx?$/i,
                use: [{
                    loader: 'ts-loader',
                }],
                exclude: /node_modules/
            },
        ],
    },
    plugin: [
        new HtmlWebpackPlugin({
            template: './src/tpl/index.html'
        })
    ],
};
```

- `webpack.base.config.js`：公共环境的配置
    - 先指定入口文件配置：`entry: './src/index.ts',`
    - 输入：`output: { filename: 'app.js', }`；输入目录使用默认的（也就时dist目录）
    - 指定扩展名：`resolve: { extensions: ['.js', '.ts', '.tsx'], }`
    - 安装ts相应的loader：


- 4)安装ts-loader,注意:需要再次本地安装typescript
```
npm i ts-loader typescript -D
```
- 5）安装插件html-webpack-plugin，该插件作用是——通过模板生成网站首页，并且将输出文件自动嵌入模板中

`npm i html-webpack-plugin -D`

    - src目录下新建tpl目录，再在目录tpl下新建index.html文件（vscode中输入快捷命令——`html:5`,快速建立html模板）

### 开发环境的配置
```
// webpack.dev.config.js

module.exports = {
  devtool: 'cheap-module-eval-source-map',
};
```

<!--> 安装: `npm i cheap-module-eval-source-map -D`-->

> 解析
> - a.`cheap`的含义：sourceMap会忽略文件的列信息（因为我们在调试时，列信息是无用的）
> - b.`module`的含义：会定位到我们的typescript源码，而不是经过转译后的js代码
> - c.`eval-source-map`的含义：会将sourceMap以data url的形式打包到文件中(它的重编译速度很快，我们不必担心性能问题)

### 生产环境中的配置
```
// webpack.pro.config.js

const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  plugins: [
    new CleanWebpackPlugin(),
  ],
};
```

- 需要安装CleanWebpackPlugin： `npm i clean-webpack-plugin -D`
    - 插件作用：在每次成功构建之后，清空dist目录（因为有些时候，为了避免缓存，我们需要在文件后加入哈希，这样在多次构建后就会产生很多无用的文件，通过这个插件就可帮助我们自动清空dist目录）

### 所有配置文件入口
```
// webpack.config.js

const merge = require('webpack-merge');
const baseConfig = require('./webpack.base.config');
const devConfig = require('./webpack.dev.config');
const proConfig = require('./webpack.pro.config');

let config = process.NODE_ENV === 'development' ? devConfig : proConfig;

module.exports = merge(baseConfig, config);
```

- 首先安装webpack-merge:  `npm i webpack-merge -D`


## 三、修改npm的脚本

- 3.1 打开package.json，首先更改入口
```
"main": "./src/index.ts"
```
- 3.2 编写启动开发环境的命令
```
"scripts": {
    "start": "webpack-dev-server --mode=development --config ./build/webpack.config.js",
    ...
}
```
写完后，尝试运行下：根目录下cmd输入——`npm start`，然后浏览器打开'localhost:8080'

- 3.3 编写构建生产环境的脚本
```
"scripts": {
    "build": "webpack --mode=production --config ./build/webpack.config.js",
    ...
}
```

