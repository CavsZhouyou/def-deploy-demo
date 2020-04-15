目的：从零开始配置`webpack`，使用`pug`、`typescript`、`scss`。

原因：`webpack`虽然接触的多，但没有从零开始配置过项目。一般都是通过各种`CLI`工具生成项目(`VueCli`)，`webpack`已经由`cli`工具配置好了，在实际的开发中，遇到的相关问题都只要针对性的去查找资料基本都能解决，所以对`webpack`的了解一直都比较片面和琐碎，缺乏一个整体性的了解，更无法谈“理解”了。

以前都是跳着走的，现在趁着不是很忙，回到起点重新走一下吧。

### 1、初始化项目

首先创建一个项目文件夹，如`webpack-demo`

在根目录下执行：

```sh
npm init -y
```

初步设置项目的目录结构和功能划分

```
// 创建以下文件夹和文件
public/
    index.html       	// 模板文件
src/                    // 工作区
    assets/             // 图片、外部样式等资源
    module/             // 页面模块
    main.js            	// 入口文件
webpack.config.js   	// webpack配置文件
```

### 2、初步配置`webpack`

#### 安装相应的依赖

> `npm install --save-dev webpack webpack-cli webpack-dev-server`
>
> `npm install --save-dev html-webpack-plugin clean-webpack-plugin`

`webpack-cli`：在`webpack3`中，`webpack`和它的`CLI`在同一个包中，但在`webpack4`中已经将两者分开，以更好的管理他们。

`html-webpack-plugin`: [webpack中文网](https://www.webpackjs.com/plugins/html-webpack-plugin/)、[github](https://github.com/jantimon/html-webpack-plugin)

简单的说就是把你打包的`js`插入的你的`html`中。

`clean-webpack-plugin`：[github](https://github.com/johnagan/clean-webpack-plugin)，一个`webpack`插件，用于在构建之前删除您的构建文件夹

#### 配置

```javascript
// webpack.config.js

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
module.exports = {
    mode: 'production',
    // 入口
    entry: {
        app: './src/main.js'
    },
    // 出口
    output: {
        path: path.resolve(__dirname, './dist/'),
        filename: './js/[name].[hash:8].js'
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: './public/index.html'
        })
    ]
}

// package.json
{
    "script": {
        "build": "webpack"
    }
}
```

#### 运行

> `npm run build`

打包成功:)



### 3、配置`pug`相关(举一反三：使用`vue`)

#### 安装相应的依赖

> `npm install --save-dev pug pug-loader`

`pug`：一种`html`渲染模板[中文文档](https://pug.bootcss.com/api/getting-started.html)

#### 配置

```jade
// 将 /public/index.html 改为 pug文件
// ./public/index.pug

doctype html
htm(lang="en")
head
    meta(charset="UTF-8")
    title= 'webpack-demo'
body
    div#app
```

```javascript
// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.pug$/,
                loader: 'pug-loader'
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './public/index.pug'
        })
    ]
}

```

### 4、配置`ES6`支持（`babel`）

#### 安装相应的依赖

主要是使用`babel`[官方文档](https://babeljs.io/docs/en/)，[中文文档-印记中文](https://babel.docschina.org/)

> `npm install --save-dev babel-loader @babel/core @babel/preset-env`
#### 配置

```javascript
// webpack.config.js
module: {
    rules: [
        {
            test: /\.js$/,
            exclude: /node_modules/,
            use: 'babel-loader'
        }
    ]
}

// 创建 .babelrc 配置文件
// https://babel.docschina.org/setup#installation
// 
{
    "presets": ["@babel/preset-env"]
}
```

### 5、配置`typescript`支持

#### 安装相应的依赖

> `npm install --save-dev typescript ts-loader` 

#### 配置

```typescript
// main.js 改为 main.ts
// ./src/main.ts
function test(str: string) {
    console.log(`console log ${str}`)
}
test('test')
```

```javascript
// webpack.config.js
module.exports = {
    entry: {
        app: './src/main.ts'
    },
    module: {
       rules: [
           {
               test: /\.ts$/,
               loader: 'ts-loader'
           }
       ]
    }
}

// 在根目录下创建 tsconfig.json
// 初步配置
// 详细配置请参考:https://www.tslang.cn/docs/handbook/tsconfig-json.html
// tsconfig.json
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es5",
        "sourceMap": true,
        "importHelpers": true
    },
    "exclude": [
        "node_modules"
    ]
}
```

### 6、配置`SCSS`支持

#### 安装相应的依赖

> `npm install --save-dev css-loader mini-css-extract-plugin node-sass sass-loader`

- `css-loader`：解析 `CSS` 文件后，使用` import` 加载，并且返回 `CSS` 代码
- `sass-loader`：加载和转译 `SASS/SCSS` 文件
- `mini-css-extract-plugin`：轻量级`CSS`提取插件[github](https://github.com/webpack-contrib/mini-css-extract-plugin)

#### 配置

```scss
// 创建 ./src/style.scss
$--default-bg-color: #808080;

#app {
    width: 100%;
    height: 100px;
    background-color: $--default-bg-color;
}
```

```typescript
// ./src/main.ts
import './style.scss'
```

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = {
    module: {
       	rules: [
          	{
              	test: /\.scss/,
              	use: [
                  	MiniCssExtractPlugin.loader,
                  	'css-loader',
                  	'sass-loader'
              	]
          	}
      	]
    },
    plugins: [
        // 提取CSS文件
        new MiniCssExtractPlugin({
            filename: './css/[name].css',
            chunkFilename: '[id].css',
            ignoreOrder: false
        })
    ]
}
```

  ### 7、配置`PostCss`支持

`postcss`是一种对`css`进行编译的工具，类似于`Babel`对`javascript`的处理，常用的功能有：

- 使用下一代`css`语法
- 自动补全浏览器前缀
- css代码压缩
- ...

`postcss`通过插件实现这些功能，

#### 安装相应的依赖

> `npm i --save-dev postcss-loader autoprefixer`

- `autoprefixer`：`postcss`插件，使用[Can I Use](https://caniuse.com/)中的值向`css`中添加浏览器前缀。

####  创建`postcss.config.js`文件

```javascript
module.exports = {
  plugins: {
    'autoprefixer': {}
  }
}
```

#### 配置`webpack.config.js`

```java
module.exports = {
  module: {
    rules: [
      {
        test: /\.scss/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ]
      }
    ]
  }
}
```

### 8、配置`browserslist`

[github仓库地址](https://github.com/browserslist/browserslist)

关于`browserslist`，官方的解释是The config to share target browsers and Node.js versions between different front-end tools.

在本示例中是配合`autoprefixer`、`babel`、`postcss`使用的。

#### 创建`.browserslistrc`文件

```
last 1 version
> 1%
maintained node versions
not dead
```

### 9、配置对图片资源的处理

#### 安装相应的依赖

> `npm install --save-dev file-loader url-loader html-loader`

#### 配置`webpack.config.js`

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|svg|jpg|gif|webp)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 1024 * 30,
              outputPath: 'image',
              fallback: 'file-loader'
            }
          }
        ]
      }
    ]
  }
}
```

#### 测试一下

在`style.scss`中添加图片引入，例如

```scss
.app {
	background-image: url("./assets/bg-banner.jpg");
}
```

在`index.pug`中引入图片，例如

```jade
img(src=require('./assets/bg-banner.jpg'))
```

在`main.ts`中引入`index.pug`

```typescript
const template: any = require('./index.pug')
document.getElementById('app').innerHTML = template()
```

然后发现css中的图片路径不对......

修改一下相关的配置就好了。

```javascript
// webpack.config.js
{
  test: /\.scss$/,
   use: [
       // 增加路径处理
     {
       loader: MiniCssExtractPlugin.loader,
       options: {
         publicPath: '../'
       }
     },
     'css-loader',
     'postcss-loader',
     'sass-loader'
   ]
}
```

### 10、配置开发环境

`webpack-dev-server` 能够用于快速开发应用程序。

#### 安装相应的依赖

> `npm install --save-dev webpack-dev-server `

#### 配置`webpack.config.js`

```javascript
module.exports = {
    // 更多配置参考：https://www.webpackjs.com/configuration/dev-server/
    devServer: {
        contentBase: path.join(__dirname, "dist"),
        compress: true, // 一切服务都启用gzip压缩
        port: 8090, // 服务启动端口
    }
}
```

#### 配置`package.json`

```json
{
    "script": {
        "serve": "webpack-dev-server --open --mode development"
    }
}
```

`--open`：直接在浏览器打开服务。

### 11、`webpack`流水线进阶工作报告

剧本暂无、日期未定、敬请期待。
