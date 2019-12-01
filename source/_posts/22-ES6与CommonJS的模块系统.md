---
title: 22 ES6与CommonJS的模块系统
date: 2019-12-01 12:01:46
tags: TypeScript
---
## 回顾两个模块系统

#### ES6

#### CommonJS

代码运行：
- 先全局安装`ts-node`
```
npm i ts-node -g
```
- 再用`ts-node`运行源码
```
ts-node c.node.ts
```

## 在生产环境中，编译配置项

tsconfig.json 中配置项

- 1） target
  - 命令tsc默认生成es3，tsconfig.json中生成的默认配置为es5
- 2） module--把代码生成什么样的模块系统
  - 命令行和tsconfig.json中默认commonjs
  - 其他选项:amd,system,umd,es2015..

实践:
```ts
tsc ./src/es6/a.ts -t es3
tsc ./src/es6/a.ts -t es6 // 编译成es6,模块默认为es6模块

tsc ./src/es6/a.ts -m amd
```

## 两个模块系统的兼容问题

- 两个系统在处理顶级导入导出时，是不兼容的
  - es6中允许一个模块有顶级导出(export default ...),和次级导出(export ...)
  - commonjs:有次级导出(exports = ...)时，不允许有顶级导出(module.exports = ...)
    - 若一个文件中既有次级导出又有顶级导出，则顶级导出会覆盖次级导出(不管顶级导出写在开始还是末尾)

```ts
// c.node.ts
let c1 = require('./a.node');
let c2 = require('./b.node');
let c3 = require('../es6/a'); // export default function()...

c3(); // 报错，显示c3不是一个函数

console.log(c3); // { ..., a: 1, ..., default: f default_1(), ...  } -- 将es6/a中所以导出都打印出来了

c3.default(); // ok
```

- #### 处理两个模块之间的不兼容性问题
  - 1)两个模块系统不要混用
  - 2)混用，ts提供了兼容性语法
    - `export = ...`--会编译成module.exports,相当于common.js的顶级导出。使用了这个，意味着这个模块不能有其他导出了
    - `import c4 = require(...)`
      - module.exports默认导出的内容，也可以用import ... from ...导入
      - 配置项esModuleInterop,为true时允许上述两种导入，为false时只允许import xx = ...这种方式