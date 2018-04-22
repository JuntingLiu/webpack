## 使用 Webpack

1. Webpack 命令
2. Webpack 配置
3. 第三方脚手架（vue-cli、Angular-cli、React-starter）

```
# 帮助
$ webpack -h 
# 查看版本
$ webpack -v
# 打包
$ webpack <entry> [<entry>] <output>
# 启动
$ webpack
// or
$ webpack --config webpack.config.js
```

**Note: 现在安装webpack 基本是安装webpack-cli 工具,还不是很完善**

* 初始化项目的功能
* 版本之间迁移的功能

```
# 交互式初始化一个项目
$ webpack-cli init
# 从webpack v1迁移到v2,<config>是现有webpack配置文件的路径
$ webpack-cli migrate <config> 
```
> 版本迁移知识改变 配置文件，不过安装的依赖不会改变，需要手动去改变

用 Webpack-cli 工具初始化一个项目，指定 webpack 官方提供的一个 webpack 的项目模版

项目初始化

```
$ webpack-cli init
# <指定模版>
$ webpack-cli init webpack-addons-demo 
# 确认是否要使用我们的模版
? Welcome to the demo scaffold! Are you ready? (Use arrow keys)
  Yes
  No
❯  Pengwings
# 你项目的应用入口,默认就好
？What is the entry point in your app?
# common chunk 的名称
? What do you want to name your commonsChunk? vender
```
> 遇到第一个问题时选择 Pengwings 不然 你得到 `webpack.config.js` 就是一个空对象的文件；不过生成的文件里配置也是比较简单的

[webpack-cli文档](https://github.com/webpack/webpack-cli)

### 🌰

以一个小实例，来玩玩看～

1. 创建一个`my_webpack` 文件夹
2. 创建两个`js` 文件： `app.js`、`sum.js`
3. `sum.js`文件，添加一个 `sum` 两数求和函数，并向外导出此函数
4. `app.js` 文件，引入`sum.js`导出的 `sum`函数，并调用`sum`函数对`2+3`进行求和
5. 使用`webpack` 命令， 对`app.js`进行打包，输出`app.bundle.js`文件
6. 看下打包后的文件代码，运行`app.bundle.js`

```
# app.js

import { sum } from './sum'

console.log('2 + 3 = ', sum(2)(3))

# sum.js

export const sum = x => y => x + y;

# 打包

$  webpack app.js app.bundle.js

# 执行

$ node app.bundle.js
2 + 3 =  5
```

上面的例子，用的是 `ES6 Module`，我们再用下 `CommonJS` 的模块化

7. 新增一个`minus.js`文件，实现`minus` 两数相减的函数，并导出
8. `app.js` 引入 `minus.js`文件，并调用 `minus` 函数，完成任意两数相减
9. 再次打包执行

```
# minus.js

var minus = x => y => x - y

module.exports = minus

# app.js

var minus = require('./minus')

$  webpack app.js app.bundle.js

# 执行

$ node app.bundle.js
20 - 18 =  2
```

接下来，我们继续看看`AMD` 模块化吧

10. 新增一个`multiplication.js`文件，实现`multiplication` 两数相乘的函数，并导出
11. `app.js` 引入 `multiplication.js`文件，并调用 `multiplication` 函数，完成任意两数相乘
12. 再次打包执行，因为 `AMD`模块化，所以就不能在 'node'环境里执行了，创建一个`index.html` 在浏览器里执行

```
# multiplication.js

define(function (require, factory) {
    'use strict'

    return function (x) {
        return function (y) {
            return x * y
        }
    }

})

# app.js

var minus = require('./minus')

# multiplication
require(['./multiplication'], function (multiplication) {
    console.log('22 * 3 = ', multiplication(22)(3))
})

$  webpack app.js app.bundle.js
  0.bundle.fbee5.js  472 bytes       0  [emitted]
app.bundle.fbee5.js    6.78 kB       1  [emitted]  app

```
这次打包你会发现，多出了一个`0.bundle.fbee5.js`文件，因为`AMD`的模块异步加载，所以会把引用的模块单独打一个 `chunk`


## 编译 ES6/7

对 ES6/7 进行编译，需要使用到 Babel; 在 Webpack 中是通过 Loader （babel-loader）来进行使用 Babel

**学习**

* Babel
    * 编译工具
* Babel-presets
    * 设置 preset 就可以知道使用规范来打包
    * 编译遵循规范集合汇总列表
        * es2015
        * es2016 
        * babel-preset-stage 0 - 3
* Babel-plugin

### 🌰

1. 创建 `app.js`、`webpack.config.js` 两个文件
2. `app.js` 里面添加一些 ES6/7 新特性的代码
3. 安装下相关依赖, 供 webpack 支持 babel 

    ```
     # 安装的是最新版本 
     $ npm install @babel/core babel-loader @babel/preset-env --save-dev
    ```

4. `webpack.config.js` 配置下 `module`，将 `babel-loader` 加进来

```
# app.js

let fn = () => {}

const name = 'Junting'

// 并不会被转换
const arrMap = arr => arr.map(item => item * 2)

let arr = arrMap([1, 2, 3, 4])

console.log(arr)

let isArr = arr.includes(8)

let arrSet = new Set(arr)

console.log(arrSet)

function* generatorFn () {

}

# webpack.config.js

module.exports = {
    entry: {
        app: './app.js'
    },
    output: {
        filename: '[name].bundle.[hash:5].js'
    },
    module: {
        rules: [
            {
                test: /\.js$/,      // 匹配规则
                use: {              // 对匹配到的文件进行以下处理
                    loader: 'babel-loader'
                    options: {      
                        presets: [  // 指定 loader 的打包编译的规范版本
                            [   
                                '@babel/preset-env',
                                {
                                    targets: {  // 指定目标对那些语法进行编译那些不编译； 还可以根据浏览器的支持程度来对那些语法进行编译
                                        browsers: ['last 2 versions' ]
                                    }
                                }
                            ]
                        ]
                    }
                },
                exclude: '/node_modules/'  // 排除规则之外的
            }
        ]
    }
}
```

对上面的实例进行打包，根据 打包的后生成的文件可以发现，箭头函数、Set 并没有被转换； `babel-preset`只是针对语法的转换，当是对以下的函数和方法并不支持：

* Genetrator
* Set
* Map
* Array.from
* Array.prototype.includes

需要借助 `babel polyfill`、`babel runtime transform` 两个插件来帮助转换

**babel-polyfill**

* 全局插件，填充器；先会全局对你使用的新特性方法，进行定义出来，然后方便的对新的API的使用
* 会污染全局变量
* 为开发应用准备

```
$ npm install babel-pilyfill --save-dev
```

使用很简单，直接在入口文件引用进来就行,`app.js`文件添加

```
import 'babel-polyfill'
```

**babel-plugin-transform-runtime**

* 局部插件; 只会在使用新特性的方法进行转换，局部内进行转换，不影响全局
* 为开发框架准备

```
$ npm install @babel/plugin-transform-runtime --save-dev
```

使用，需要在跟目录创建一个`.babelrc`文件，然后添加以下规则； `webpack.config.js` 里就不需要进行配置了


```
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": {
                    "browsers": ["last 2 versions"]
                }
            }
        ]
    ],
    
    "plugins": [
        "@babel/transform-runtime"
    ]
}

```

## Webpack 中配置 TypeScript

* Typescript 是 Js 的超集
### Typescript loader

**安装**

```
# 官方推荐
$ npm install typescript ts-loader --save-dev (webpack4)
$ npm install typescript ts-loader@3.5.0 --save-dev (webpack3)
# 第三方
$ npm install typescript awesome-typescript-loader --save-dev
```
**配置**

* tsconfig.json
* webpack.config.js

**常用选项**

* compilerOptions
* include               
* exclude

[配置选项](https://www.tslang.cn/docs/handbook/tsconfig-json.html)

**声明文件**

通过安装声明文件，可以在使用引入类库、框架内容提供的API时可以清晰的知道传参的类型，如果传递了错误的参数，会很清晰快捷的反应给你传递参数的类型错误，出现错误，打包时就会立即报错

```
$ npm install @types/lodash
$ npm install @types/vue
$ npm install typing -g
```

每次手动的去安装引用类库的声明文件太麻烦，可以安装全局的 `typings`,使用 typing 命令，安装本地类型声明文件，然后配置下`tsconfig.json`文件生效

```
$ npm install typing -g
$ typings install lodash    // 项目中生成一个typings 文件目录，里面就是类型声明文件
```

### 🌰

1. 创建 `app.js`、`webpack.config.js` 和 `tsconfig.json` 入口文件、webpack配置文件 和 typescript 配置文件
2. 需要安装的依赖
    * webpack
    * ts-loader / awesome-typescript-loader
    * typescript
    * lodash
    * @types/lodash     // 声明文件, typings 本地安装了话，就不需要了

**app.js**

```
import * as _ from 'lodash'

const number = 45

console.log(_.chunk([1, 2, 3, 4, 5], 3));

interface Cat {
    name: String,
    sex: String
}

function touchCat (cat: Cat) {
    console.info('喵~', cat.name);
}

touchCat({
    name: '喵喵',
    sex: 'male'
})

``` 

**webpack.config.js**

```
module.exports = {
    entry: {
        'app': './app.ts'
    },
    output: {
        filename: '[name].bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.tsx?$/,
                use: {
                    loader: "ts-loader"
                }
            }
        ]
    }
}
```

**tsconfig.json**

```
{
    "compilerOptions": {        // 通用配置
        "module": "commonjs",
        "target": "es5",
        "allowJs": true,
        // 安装本地 typing 时，添加的配置;默认所有可见的"@types"包会在编译过程中被包含进来, 添加以下代码生效
        "typeRoots": [
            "./node_modules/@type",
            "./typings/modules"
        ]
    },
    "include": [
        "./src/*"
    ],
    "exclude": [
        "./node_modules"
    ]
}
```