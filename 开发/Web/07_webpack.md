[toc]

## 0. npm 安装包的查询网站

https://www.npmjs.com



## 1. webpack 基本使用

### 1.1 创建空白项目

```shell
# 创建空项目
npm init -y

# 创建代码目录
mkdir src
touch src/index.html

# 安装jQuery(演示用)
npm install jquery -S -D
# -S --save 表示保存, 可以省略, 会把包装到dependencies域, 生产和测试环境使用
# -D 表示安装到devDependencies域, 测试环境使用
```

###  1.2 安装 webpack

```
cnpm install webpack webpack-cli -D
```

### 1.3 配置 webpack

1. 在项目根目录中，创建名为 webpack.config.js 的 webpack 配置文件

   ```javascript
   module.exports = {
       // mode 用来指定构建模式, 可选值又 development 和 production
       mode: "development"
   }
   ```

2. 在 package.json 的 scripts 节点下，新增 dev 脚本如下

    ```javascript
    {
      "name": "my-webpack",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "dev": "webpack"
      },
      "keywords": [],
      "author": "",
      "license": "ISC",
      "dependencies": {
        "jquery": "^3.6.0"
      },
      "devDependencies": {
        "webpack": "^5.67.0",
        "webpack-cli": "^4.9.1"
      }
    }
    ```
    
3. 在终端中运行 `npm run dev` 命令，启动 webpack 进行项目的打包构建

    ```shell
    PS D:\Projects\JavaScript\my-webpack> npm run dev
    
    > my-webpack@1.0.0 dev D:\Projects\JavaScript\my-webpack
    > webpack
    
    asset main.js 1.18 KiB [emitted] (name: main)
    ./src/index.js 1 bytes [built] [code generated]
    webpack 5.67.0 compiled successfully in 92 ms
    ```

#### mode 参数说明

mode 节点的可选值有两个，分别是：
- development
  - 开发环境
  - 不会对打包生成的文件进行代码压缩和性能优化
  - 打包速度快，适合在开发阶段使用
- production
  - 生产环境
  - 会对打包生成的文件进行代码压缩和性能优化
  - 打包速度很慢，仅适合在项目发布阶段使用

#### webpack.config.js 文件说明

```
webpack.config.js 是 webpack 的配置文件。webpack 在真正开始打包构建之前，会先读取这个配置文件， 从而基于给定的配置，对项目进行打包。 

注意：由于 webpack 是基于 node.js 开发出来的打包工具，因此在它的配置文件中，支持使用 node.js 相关 的语法和模块进行 webpack 的个性化配置。
```

#### webpack 中的默认约定

在 webpack 4.x 和 5.x 的版本中，有如下的默认约定：
- 默认的打包入口文件为 src -> index.js
- 默认的输出文件路径为 dist -> main.js
注意：可以在 webpack.config.js 中修改打包的默认约定

#### 自定义打包的入口与出口

在 webpack.config.js 配置文件中，通过 entry 节点指定打包的入口。通过 output 节点指定打包的出口。

```js
const path = require('path')

module.exports = {
    mode: 'development',
    // 打包入口文件路径
    entry: path.join(__dirname, './src/index.js'),
    output: {
        // 输出文件的存放路径
        path: path.join(__dirname, './dist'),
        // 输出文件的名称
        filename: 'bundle.js'
    }
}
```

## 2. webpack 的插件

常用的 webpack 插件有如下两个：
- webpack-dev-server
  - 类似于 node.js 阶段用到的 nodemon 工具
  - 每当修改了源代码，webpack 会自动进行项目的打包和构建
- html-webpack-plugin
  - webpack 中的 HTML 插件（类似于一个模板引擎插件）
  - 可以通过此插件自定制 index.html 页面的内容

### 2.1 webpack-dev-server

```shell
cnpm install webpack-dev-server -D
```

#### 配置

修改 package.json -> scripts 中的 dev 命令

```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack serve"
  },
```

webpack.config.js文件的devServer节点可以对webpack-dev-server 插件进行更多的配置 (使用默认, 可省略)

```js
module.exports = {
    devServer: {
        open: true, // 初次打包完成后, 自动打开浏览器; 默认 false
        host: '127.0.0.1', // webpack serve 运行时监听的地址
        port: 80 // webpack serve 运行时监听的端口
    }
}
```



运行 npm run dev 命令，重新进行项目的打包

在浏览器中访问 http://localhost:8080 地址，查看自动打包效果

#### 说明

- 不配置 webpack-dev-server 的情况下，webpack 打包生成的文件，会存放到实际的物理磁盘上
  - 严格遵守开发者在 webpack.config.js 中指定配置
  - 根据 output 节点指定路径进行存放
- 配置了 webpack-dev-server 之后，打包生成的文件存放到了内存中
  - 不再根据 output 节点指定的路径，存放到实际的物理磁盘上
  - 提高了实时打包输出的性能，因为内存比物理磁盘速度快很多

- webpack-dev-server 生成到内存中的文件，默认放到了项目的根目录中，而且是虚拟的、不可见的
  - 可以直接用 / 表示项目根目录，后面跟上要访问的文件名称，即可访问内存中的文件
  - 例如 /bundle.js 就表示要访问 webpack-dev-server 生成到内存中的 bundle.js 文件

### 2.2 html-webpack-plugin

html-webpack-plugin 是 webpack 中的 HTML 插件，可以通过此插件自定制 index.html 页面的内容。

 需求：通过 html-webpack-plugin 插件，将 src 目录下的 index.html 首页，复制到项目根目录中一份！

```shell
cnpm install html-webpack-plugin -D
```

#### 配置

修改 webpack.config.js

```js
// 1. 导入HTML插件
const HtmlPlugin = require('html-webpack-plugin')

// 2. 创建HTML插件的实例对象
const htmlPlugin = new HtmlPlugin({
    template: './src/index.html', // 指定模板文件的存放路径
    filename: './index.html' // 指定生成文件的存放路径
})

module.exports = {
    mode: 'development',
    // 3. 通过plugins节点, 使htmlPlugin插件生效
    plugins: [htmlPlugin],
    //...
}
```

#### 说明

- 通过 HTML 插件复制到项目根目录中的 index.html 页面，也被放到了内存中
- HTML 插件在生成的 index.html 页面，自动注入了打包的 bundle.js 文件

## 3. webpack 的 loader

loader 加载器的作用：协助 webpack 打包处理特定的文件模块

- css-loader 可以打包处理 .css 相关的文件
- less-loader 可以打包处理 .less 相关的文件
- babel-loader 可以打包处理 webpack 无法处理的高级 JS 语法
- ts-loader 可以打包处理 .ts 相关的文件

### 3.1 css-loader

```shell
cnpm i style-loader css-loader -D
```

在 webpack.config.js 的 module -> rules 数组中，添加 loader 规则如下：

```js
module.exports = {
    module: { // 所有第三方文件模块的匹配规则
        rules: [ // 文件后缀名的匹配规则
            { test: /\.css/, use: ['style-loader', 'css-loader']}
        ]
    }
}
```

其中，test 表示匹配的文件类型， use 表示对应要调用的 loader

#### 注意

- use 数组中指定的 loader 顺序是固定的
-  多个 loader 的调用顺序是：从后往前调用

### 3.2 less-loader

```shell
cnpm i less-loader -D
```

```js
module.exports = {
    module: { // 所有第三方文件模块的匹配规则
        rules: [ // 文件后缀名的匹配规则
            { test: /\.css$/, use: ['style-loader', 'css-loader']},
            {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader']}
        ]
    }
}
```

### 3.3 ts-loader

```shell
cnpm i typescript ts-loader -D
```

```js
module.exports = {
    module: {
        rules: [
            {test: /\.css$/, use: ['style-loader', 'css-loader']},
            {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader']},
            {test: /\.ts$/, use: ['ts-loader']},
        ]
    }
}
```



### 3.4 url-loader

图片文件转base64

```shell
cnpm i url-loader file-loader -D
```

```js
module.exports = {
    module: { // 所有第三方文件模块的匹配规则
        rules: [ // 文件后缀名的匹配规则
            {test: /\.css$/, use: ['style-loader', 'css-loader']},
            {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader']},
            {test: /\.ts$/, use: ['ts-loader']},
            {test:/\.jpg|png|gif$/, use: 'url-loader?limit=22229'}
        ]
    }
}
```

其中 ? 之后的是 loader 的参数项： 

- limit 用来指定图片的大小，单位是字节（byte） 
- 只有 ≤ limit 大小的图片，才会被转为 base64 格式的图片

### 3.5 babel-loader

处理webpack无法处理的高级js语法

```shell
cnpm i babel @babel/core @babel/plugin-proposal-decorators -D
```

```js
module.exports = {
    module: {
        rules: [
			//...
            {test: /\.js$/, use: 'babel-loader', exclude: /node_modules/}
        ]
    }
}
```

在项目根目录下，创建 babel.config.js，定义 Babel 的配置项

```js
module.exports = {
    plugins: [['@babel/plugin-proposal-decorators', {legacy: true}]]
}
```

## 4. 打包发布

### 4.1 配置打包命令

在 package.json 文件的 scripts 节点下，新增 build 命令

```json
{
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack serve",
    "build": "webpack --mode production"
  },
```

--model 是一个参数项，用来指定 webpack 的运行模式。

production 代表生产环境，会对打包生成的文件 进行代码压缩和性能优化。

注意：通过 --model 指定的参数项，会覆盖 webpack.config.js 中的 model 选项。

### 4.2 配置js打包输出目录

在 webpack.config.js 配置文件的 output 节点中，进行如下的配置：

```js
module.exports = {
    output: {
        // 输出文件的存放路径
        path: path.join(__dirname, './dist'),
        // 输出文件的名称
        filename: 'js/bundle.js'
    },
```

### 4.3 配置image打包输出目录

修改 webpack.config.js 中的 url-loader 配置项，新增 outputPath 选项即可指定图片文件的输出路径：

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.jpg|png|gif$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        limit: 2228,
                        // 明确指定把打包生成的图片文件, 输出到dist目录下的image文件夹中
                        outputPath: 'image'
                    }
                }
            },
        ]
    }
}
```

### 4.4 自动清理 dist 目录下的旧文件

安装 clean-webpack-plugin 插件

```
cnpm i clean-webpack-plugin -D
```

```js
const {CleanWebpackPlugin} = require('clean-webpack-plugin')
const cleanWebpackPlugin = new CleanWebpackPlugin()

module.exports = {
    mode: 'development',
    plugins: [htmlPlugin, cleanWebpackPlugin],
}
```

