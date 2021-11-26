---
title: webpack 学习笔记
date: 2021-05-18 14:40:53
tags:
- 前端
- webpack
categories:
- 前端
- 前端工具
---

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-webpack.1ppnqfvm9yf4-78ffc5.png" alt="webpack" width="80%" />

<!-- more -->

## 1.概述

[webpack](https://webpack.js.org/) 是一个模块打包器（构建工具）。它的主要目标是将 JavaScript 文件打包在一起，打包后的文件用于在浏览器中使用，它能够胜任转换（transform）、打包（bundle）或包裹（package）任何资源（resource/asset）。[教程地址](https://www.bilibili.com/video/BV1Pz4y1S7Uv?p=9)

## 2.核心概念

### 2.1 入口（entry）

**入口起点（entry point）**指示 webpack 应该使用哪个模块，来作为构建其内部**依赖图**的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

可以通过在 [webpack 配置](https://www.webpackjs.com/configuration)中配置 `entry` 属性，来指定一个入口起点（或多个入口起点）。默认值为 `./src`。

**webpack.config.js**

```js
module.exports = {
  entry: './path/to/my/entry/file.js'
  
  // entry: ['./src/index.js','./src.main.js']
};
```

若为**单入口**，使用字符串，打包一个 chunk 和一个 bundle，chunk 的名称为默认。

若为**多入口**

- 使用数组，所有入口文件形成一个 chunk，名称为默认，且输出一个 bundle。
- 使用 Object，有几个入口文件就生成几个 chunck，并输出几个 bundle，chunck 的名称为指定的 key。

```JS
const path = require('path');

module.exports = {
    entry: {
        one: './src/index.js',
        two: './src/main.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].js'
    },
    mode: 'development'
};
```

### 2.2 出口（output）

**output** 属性告诉 webpack 在哪里输出它所创建的 **bundles**，以及如何命名这些文件，默认值为 `./dist`。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。

**webpack.config.js**

```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

在上面的示例中，我们通过 `output.filename` 和 `output.path` 属性，来告诉 webpack bundle 的名称，以及我们想要 bundle 生成（emit）到哪里。在代码最上面导入的 path 模块是一个 [Node.js 核心模块](https://nodejs.org/api/modules.html)，用于操作文件路径。

### 2.3 loader

**loader** 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。

本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。

在更高层面，在 webpack 的配置中 **loader** 有两个目标：
- `test` 属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件。
- `use` 属性，表示进行转换时，应该使用哪个 loader。

**webpack.config.js**

```js
const path = require('path');

const config = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  /* loader 配置 */
  module: {
    rules: [
      { 
          // 作用于哪些文件
          test: /\.txt$/, 
          // 使用哪些 loader
          use: 'raw-loader' 
      }
    ]
  }
};

module.exports = config;
```

以上配置中，对一个单独的 module 对象定义了 `rules` 属性，里面包含两个必须属性：`test` 和 `use`。这告诉 webpack 编译器（compiler）如下信息：

> “嘿，webpack 编译器，当你碰到「在 `require()`/`import` 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先**使用** `raw-loader` 转换一下。”

*重要的是要记得，***在 webpack 配置中定义 loader 时，要定义在 `module.rules` 中，而不是 `rules`**。然而，在定义错误时 webpack 会给出严重的警告。

### 2.4 插件（plugins）

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。[插件接口](https://www.webpackjs.com/api/plugins)功能极其强大，可以用来处理各种各样的任务。

想要使用一个插件，你只需要 `require()` 它，然后把它添加到 `plugins` 数组中。多数插件可以通过选项（option）自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 `new` 操作符来创建它的一个实例。

**webpack.config.js**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

const config = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

webpack 提供许多开箱可用的插件，可查阅[插件列表](https://www.webpackjs.com/plugins)获取更多信息。

### 2.5 模式

通过选择 `development` 或 `production` 之中的一个，来设置 `mode` 参数，你可以启用相应模式下的 webpack 内置的优化。

- 开发模式（development）：配置比较简单，能让代码本地调试运行的环境。
- 生产模式（production）：代码需要不断优化达到性能最好。能让代码优化上线运行的环境。

```js
module.exports = {
  mode: 'production'
};
```

## 3.打包资源

### 3.1 打包 HTML 资源

使用**插件**对 HTML 文件进行处理（html-webpack-plugin）。

使用步骤：

- 下载

```bash
npm i html-webpack-plugin -D
```

- 引入

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
```

- 使用

```js
plugins: [
    new HtmlWebpackPlugin({
        template: './src/index.html',
        filename: 'demo.html'
    })
]
```

html-webpack-plugin 插件生成的内存中的页面已帮我们创建并正确引用了打包编译生成的资源（js/css）。

### 3.2 压缩 JS 和 HTML 代码

JS 代码只需设置成**成产模式**（production），就会自动压缩。

压缩 HTML 方法：

```js
new HtmlWebpackPlugin({
    template: './src/index.html',
    // filename: 'index.html'
    // 压缩 html 代码
    minify: {
        // 移除空格
        collapseWhitespace: true,
        // 移除注释
        removeComments: true
    }
})
```

### 3.3 webpack 打包多 html 开发案例

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    entry: {
        vendor: './src/js/common.js',
        index: './src/js/index.js',
        cart: './src/js/cart.js',
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].js'
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html',
            chunks: ['index','vendor'],
            minify: {
                collapseWhitespace: true,
                removeComments: true
            }
        }),
        new HtmlWebpackPlugin({
            template: './src/cart.html',
            filename: 'cart.html',
            chunks: ['cart','vendor'],
            minify: {
                collapseWhitespace: true,
                removeComments: true
            }
        }),
    ],
    mode: 'development'
};
```

### 3.4 打包 CSS 资源

需要使用 npm 下载安装两个 **loader** 帮助完成打包。

- **css-loader** 的作用是处理 css 中的 @import 和 url 这样的外部资源。
- **style-loader** 的作用是把样式插入到 DOM 中，方法是在 head 中插入一个 style 标签，并把样式写入到这个标签的 innerHTML 里。

```bash
npm i css-loader style-loader -D
```

```js
module: {
    rules: [
        {
            test: /\.css$/,
            use: [
                // use 数组中的 loader 执行顺序：从右到左，从下到上依次执行
                // 创建 style 标签，将 js 中的样式资源插入到 header 中
                'style-loader',
                // 将 CSS 文件变成 commonjs 模块加载到 js 中，里面内容是样式字符串
                'css-loader'
            ]
        }
    ]
}
```

在入口文件里添加：

```js
require('../css/index.css')
```

### 3.5 打包 less 或 sass 资源

因为 CSS 只是单纯的属性描述，并不具有变量、条件语句，CSS 的特性导致其**难组织**和**维护**。

Sass 和 Less 都属于 CSS **预处理器**，定义了一种新的语言，其基本思想是用一种专门的编程语言，为 CSS 增加一些编程的特性，将 CSS 作为目标生成文件，然后开发者使用这种语言进行 CSS 编码工作。

- Less 需要使用 npm 下载 less 包和 less-loader。

```bash
npm i less less-loader -D
```

- Sass 需要使用 npm 下载 node-sass 包和 sass-loader。

```bash
npm i node-sass sass-loader -D
```

```js
module: {
    rules: [
        { test: /\.css$/, use: ['style-loader', 'css-loader'] },
        { test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader'] },
        { test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }
    ]
},
```

在入口文件里添加：

```js
require('../css/index.less')
require('../css/index.scss')
```

### 3.6 提取 CSS 为单文件

CSS 内容是打包再 js 文件中的，可以使用 **mini-css-extract-plugin 插件**提取成单独的 CSS 文件。

- 下载 mini-css-extract-plugin 插件

```bash
npm i mini-css-extract-plugin -D
```

- 在 webpack.config.js 中引入插件。

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
```

- 在 plugins 模块中使用插件。

```js
plugins: [
    // filename 指定提取出的 CSS 文件名
    new MiniCssExtractPlugin({filename: './css/index.css'})
]
```

- 在 CSS 的 rules 中，使用 MiniCssExtractPlugin.loader 取代 style-loader

```js
module: {
    rules: [
        { test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader'] },
    ]
}
```

### 3.7 处理 CSS 的兼容性

- 需要使用 **postcss** 处理，下载包 postcss-loader 和 postcss-preset-env。

```bash
npm i postcss-loader postcss-preset-env -D
```

- postcss 会找到 package.json 中的 browserslist 里面的配置，通过配置加载 css 的兼容性。

```json
// package.json
"browserslist": [
    "> 0.2%",			// 由全球使用统计数据选择的浏览器版本
    "last 2 versions",	// 每个浏览器的最后两个版本
    "not dead"			// 未
]
```

- 修改 loader 的配置，新版需要写 postcss.config.js，less 和 sass 兼容性同理。

```js
// webpack.config.js
rules: [
    { test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader'] }
]
```

```js
// postcss.config.js
module.exports = {
    plugins: [
        require('postcss-preset-env')()
    ]
};
```

### 3.8 压缩 CSS

使用 **[css-minimizer-webpack-plugin](https://github.com/webpack-contrib/css-minimizer-webpack-plugin)** 插件压缩 CSS 内容。

```bash
npm install css-minimizer-webpack-plugin --save-dev
```

- 1.引入插件

```js
const TerserWebpackPlugin = require('terser-webpack-plugin')
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
```

- 2.使用插件

```bash
optimization: {
    minimize: true,
    minimizer: [
      new TerserWebpackPlugin(),
      new CssMinimizerPlugin(),
    ],
  },
```

在使用 **optimization** 配置时，只添加 CssMinimizerPlugin 会导致 JS 不会被压缩。原因是webpack认为，如果配置了minimizer，就表示开发者在自定以压缩插件。内部的 JS 压缩器就会被覆盖掉。所以这里还需要手动将它添加回来。webpack 内部使用的 JS 压缩器是「terser-webpack-plugin」。

也可以直接在 plugins 中引入：

```js
plugins: [
    new CssMinimizerPlugin(),
]
```

### 3.9 打包图片资源

需下载 url-loader 和 file-loader 两个包。

```bash
npm i url-loader file-loader -D
```

在 CSS 中引入图片。

```css
#box {
    width: 200px;
    height: 100px;
    background-image: url('/img.png');
}
```

```js
rules: [
    {
        test: /\.(png|jpg|gif)$/i,
        use: [
            {
                loader: 'url-loader',
                options: {
                    // 图片大小小于 8Kb，就会使用 base64 处理
                    limit: 8 * 1024,
                    publicPath: '../img/',
                    outputPath: 'img/',
                    // 输出文件名
                    name: '[name][hash:10].[ext]'
                },
            },
        ],
    },
]
```

在 HTML 中使用图片，需要下载 html-loader。