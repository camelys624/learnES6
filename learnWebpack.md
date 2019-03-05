---
ebook:
  title: 入门 Webpack,看这篇就够了
  authors: zhangwang
---

# 入门 Webpack,看这篇就够了

## 写在前面的话

> 阅读本文之前，先看下面这个 webpack 的配置文件，如果每一项你都懂，那本文能带给你的收获也许就比较有限，你可以快速浏览或者直接跳过；如果你和十天前的我一样，对很多选项都存在着疑惑，那花一段时间慢慢阅读本文，你的疑惑一定一个一个都会消失；如果你以前没怎么接触 webpack，而你又对 webpack 感兴趣，那么就动手跟着本文中的那个贯穿始终的例子写一次，写完以后你会发现你已经明明白白的走进了 webpack 的大门。

```js
// 一个常见的 ‘webpack’ 配置文件
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-html');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  entry: __dirname + '/app/main.js',  // 多次提及的唯一入口文件
  output: {
    path: __dirname + '/build',
    filename: 'bundle-[hash].js'
  },
  devtool: 'none',
  devServer: {
    contentBase: './public',  // 本地服务器所加载的页面所在的目录
    historyApiFallback: true, // 不跳转
    inline: true,
    hot: true
  },
  module: {
    rules: [{
      test: /(\.jsx|\.js)$/,
      use: {
        loader: 'babel-loader'
      },
      exclude: /node_modules/
    }, {
      test: /\.css$$/,
      use: ExtractTextPlugin.extract({
        fallback: 'style-loader',
        use: [{
          loader: 'css-loader',
          options: {
            modules: true,
            localIdentName: '[name]__[local]--[hash:base64:5]'
          }
        }, {
          loader: 'postcss-loader'
        }]
      })
    }]
  },
  plugins: [
    new webpack.BannerPlugin('版权所有，翻版必究'),
    new HtmlWebpackPlugin({
      template: __dirname + '/app/index/tmpl.html'  // new 一个这个插件的实例，并传入相关的参数
    }),
    new webpack.optimize.OccurrenceOrderPlugin(),
    new webpack.optimize.UglifyJsPlugin(),
    new ExtractTextPlugin('style.css')
  ]
};
```

## 为什么是 Webpack，为什么要使用它

### 为什么要使用 Webpack

现今很多网页其实可以看做使功能丰富的应用，他们拥有着复杂的 JavaScript 代码和一大堆依赖包。为了简化开发的复杂度，前段社区涌现出了很多的实践方法

- **模块化**,让我们可以把复杂的程序细化为小的文件
- 类似于 TypeScript 这种在 JavaScript 基础上拓展的开发语言： 是我们能够实现目前版本的 JavaScript 不能直接使用的特性，并且之后还能转换为 JavaScript 文件使浏览器可以识别；
- Scss,less 等 CSS 预处理器
- ...

这些改进确实大大的提高了我们的开发效率，但是利用它们开发的文件往往需要额外的处理才能让浏览器识别，而手动处理又是非常繁琐的，这就为 Webpack 类的工具的出现提供了需求。

### 什么是 Webpack

Webpack 可以看做是**模块打包机**：它做的事情是，分析你的项目结构，找到 JavaScript 模块以及其他的一些浏览器不能直接运行的扩展语言 (Scss, TypeScript 等) ，并将其转换和打包为合适的格式供浏览器使用。

### Webpack 和 Grunt 以及 Gulp 相比有什么特性

其实 Webpack 和另外两个并没有太多的可比性，Gulp/Grunt 是一种能够优化前端的开发流程的工具，而 Webpack 是一种模块化的解决方案，不过 Webpack 的优点使得 Webpack 在很多场景下可以替代 Gulp/Grunt 类的工具。

Grunt 和 Gulp 的工作方式是： 在一个配置文件中，指明对某些文件进行类似编译，组合，压缩等任务的具体步骤，工具之后可以自动替你完成这些任务。

![Grunt工作方式](./static/img/webpack1.png)

Webpack 的工作方式是：把你的项目当作一个整体，通过一个给定的主文件 (如： index.js)，Webpack 将从这个文件开始找到你的项目的所有依赖文件，使用 loaders 处理它们，最后打包为一个 (或多个) 浏览器可以识别的 JavaScript 文件。

![webpack工作方式](./static/img/webpack2.png)

如果实在要把二者进行比较，Webpack 的处理适度更快更直接，能打包更多不同类型的文件。

## 开始使用 Webpack

初步了解了 Webpack 工作方式后，我们一步步的开始学习使用 Webpack。

### 安装

Webpack 可以使用 npm 安装，新建一个空的练习文件夹 (此处命名为 webpack sample project),在终端中转到文件夹后执行下述指令就可以完成安装。

```shell
// 全局安装
npm install -g wenpack
// 安装到项目目录
npm install --save-dev webpack
```

### 正式使用 Webpack 前的准备

1. 在上述练习文件夹中创建一个 package.json 文件，这是一个标准的 npm 说明文件，里面蕴含了丰富的信息，包括当前项目的依赖模块，自定义的脚本任务等等。在终端中使用 `npm init` 命令可以自动创建这个 package.json 文件，输入这个命令后，终端会问你一系列诸如项目名称，项目描述，作者信息，不过不用担心，如果你准备在 npm 中发布你的模块，这些问题的答案都不重要，默认回车就可以。
2. package.json 文件已经就绪，我们在项目中安装 Webpack 作为依赖包 `nmp install --save-dev webpack`。
3. 回到之前的空文件夹，并在里面创建两个文件夹， app 文件夹和 public 文件夹，app 文件夹用来存放原始数据和我们将写的 JavaScript 模块，public 文件夹用来存放之后供浏览器读取的文件 (包括使用 webpack 打包生成的 js 文件以及一个 `index.html` 文件) 。接下来我们再创建三个文件：
    - `index.html` --放在 public 文件夹中
    - `Greeter.js` --放在 app 文件夹中
    - `main.js` --放在 app 文件夹中

此时项目结构如下图所示：

![项目目录](./static/img/webpack3.png)

我们在 `index.html` 文件中写入最基础的 html 代码，它在这里目的在于引入打包后的 js 文件 (这里我们先把打包后的 js 文件命名为 `bundle.js`，之后我们还会详细讲述)。

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Webpack Sample Project</title>
  </head>
  <body>
    <div id='root'>
    </div>
    <script src="bundle.js"></script>
  </body>
</html>
```

我们在 `Greeter.js` 中定义一个返回包含问候信息的 `html` 元素的函数，并依据 CommonJS 规范导出这个函数为一个模块：

```js
// Greeter.js
module.exports = function() {
  var greet = document.createElemnt('div');
  greet.textContent = 'Hi there and greetings!';
  return greet;
};
```

`main.js` 文件中我们写入下述代码，用以把 `Greeter 模块` 返回的节点插入页面。

```js
// main.js
const greeter = require('./Greeter.js');
document.querySelector('#root').appendChild(greeter());
```

## 正式使用 Webpack

Webpack 可以在终端中使用，在基本的使用方法如下：

```shell
# {exrty file} 处填写入口文件的路径，本文中就是上诉 main.js 的路径
# {destination for bundled file} 处填写打包文件的存放路径
# 填写路径的时候不用添加 {}
webpack {entry file} {destination for bundled file}
```

指定入口文件后，webpack 将自动识别项目所依赖的其他文件，不过需要注意的是如果你的 webpack 不是全局安装的，那么当你在终端使用此命令使，需要额外指定其在 node_modules 中的地址，继续上面的例子，在终端中输入如下命令

```shell
# webpack 非全局安装的情况
node_modules/.bin/webpack app/main.js public/bundle.js
```

结果如下：

![运行 webpack 结果](./static/img/webpack4.png)

可以看出 `webpack` 同时编译了 `main.js` 和 `Greeter.js`,现在打开 `index.html`,可以看到如下结果

![网页结果](./static/img/webpack5.png)

有没有很激动，已经成功的使用 `Webpack` 打包了一个文件了。不过在终端中进行复杂的操作，其实是不太方便且容易出错的，接下来看看 Webpack 的另一种更常见的使用方法。

## 使用配置文件来使用 `Webpack`

Webpack 拥有其他的比较高级的功能 (比如说本文后面会介绍的 `loaders` 和 `plugins`)，这些功能其实都可以通过命令行模式实现，但是正如前面提到的，这样不太方便且容易出错的，更好地办法使定义一个配置文件，这个配置文件其实也是一个简单的 JavaScript 模块，我们可以把所有的与打包相关的信息放在里面。

继续上面的例子来说明如何写这个配置文件，在当前练习文件夹的根目录下新建一个名为 `webpack.config.json` 的文件，我们在其中写入如下所示的简单哦诶值代码，目前的配置主要涉及到的内容是入口文件路径和打包后文件的存放路径。

```js
module.exports = {
  entry: __dirname + '/app/mian.js',  // 多次体积的唯一入口文件
  output: {
    path: __dirname + '/public',    // 打包后文件存放的地方
    filename: 'bundle.js' // 打包后输出文件的文件名
  }
}
```

> 注： “__dirname” 是 node.js 中的一个全局变量，它指向当前执行脚本所在的目录。

有了这个配置之后，再打包文件，只需在终端中运行 `webpack(非全局安装需要使用 node_module/.bin/webpack)` 命令就可以了，这条命令会自动引用 `webpack.config.js` 文件中的配置选项，示例如下：

![配置之后结果](./static/img/webpack6.png)

又学会了一种使用 `webpack` 的方法，这种方法不用管那凡人的命令行参数，有没有感觉很爽。如果我们可以连 `webpack(非全局安装需要使用 node_modules/.bin/webpack)` 这条命令都可以不用，那种感觉会不会更爽，继续看下文。

## 更快捷的执行打包任务

在命令行中输入命令需要代码类似于 `node_modules/.bin/webpack` 这样的路径其实是比较烦人的，不过值得庆幸的使 `npm` 可以导入任务执行，对 `npm` 进行配置后可以在命令行中使用简单的 `npm start` 命令来替代上面略微繁琐的命令。在 `package.json` 中对 `scripts` 对象进行相关的设置即可，设置方法如下。

```js
{
  "name": "webpack-sample-project",
  "version": "1.0.0",
  "description": "Sample webpack project",
  "scripts": {
    "start": "webpack"  // 修改的是这里
  },
  "author": "yang",
  "license": "ISC",
  "devDependencies": {
    "webpack": "3.10.0"
  }
}
```

> `package.json` 中的 `script` 会安装一定顺序寻找命令行对应位置，本地的 `node_modules/.bin` 路径就在这个寻找清单中，所以无论是全局还是局部安装的 Webpack，你都不需要写前面那指明详细的路径了。

npm 的 `start` 命令是一个特殊的脚本名称，其特殊性表现在，在命令行中使用 `npm start` 就可以执行其对于的命令，如果对应的此脚本名称不是 `start` ，想要在命令行中运行时，需要这样用 `npm run {script name}` 如 `npm run build`,我们在命令行中输入 `npm start` 试试，输出结果如下：

![npm start 输出结果](./static/img/webpack7.png)

现在只需要使用 `npm start` 就可以打包文件了，有没有觉得 `webpack` 也不过如此嘛，不过不要太小瞧 `webpack`，要充分发挥其强大的功能我们需要修改配置文件的其他选项，一项项来看。

## Webpack 的强大功能

### 生成 Source Maps (使调试更容易)

开发总是离不开调试，方便的调试能极大的提高开发效率，不过有时通过打包后的文件，你是不容易找到出错的地方，对应的你写的代码的位置的，`Source Maps` 就是来帮我们解决这个问题的。

通过简单的配置， `webpack` 就可以在打包时为我们生成 `source maps`,这为我们提供了一种对应编译文件的方法，使得编译后的代码可读性更高，也更容易调试。

在 `webpack` 的配置文件中配置 `source maps`，需要配置 `devtool`,它有以下四种不同的配置选项，各具优缺点，描述如下：

| devtool选项                    | 配置结果                                                                                                                                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `source-map`                   | 在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的 `source map`，但是它会减慢打包的速度                                                                                                                                                       |
| `cheap-module-source-map`      | 在一个单独的文件中生成一个不带列映射的 `map`,不带列映射提高了打包速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列 (符号)，会对调试造成不便；                                                                                            |
| `eval-module-source-map`       | 使用 `eval` 打包韵源文件模块，在同一个文件中生成干净的完整的 `source map`。这个选项可以在不影响构建速度的前提下生成完整的 `source map`,但是对打包后输出的 JS 文件的执行具有性能和安全的隐患。在开发阶段这是一个非常好的选项，在生产阶段一定不不要启用这个选项 |
| `cheap-module-eval-source-map` | 这是在打包文件时最快的生成 `source map` 的方法，生成的 `source map` 会和打包后的 `javascript` 文件同时显示，没有列映射，和 `eval-source-map` 选项具有相似的缺点。                                                                                             |

正如上表所述，上述选项由上到下打包速度会越来越快，不过同时也具有越来越多的负面作用，较快的打包速度的后果就是对打包后的文件的执行有一定影响。

对小到中型项目中， `eval-source-map` 是一个很好的选项，再次强调你只应该在开发阶段使用它，我们继续对上文新建的 `webpack.config.json`，进行如下配置：

```js
module.exports = {
  devtool: 'eval-source-map',
  entry: __dirname + '/app/main.js',
  output: {
    path: __dirname + '/public',
    filename: 'bundle.js'
  }
}
```

> `cheap-module-eval-source-map` 方法构建速度更快，但是不利于调试，推荐在大型项目考虑时间成本时使用。

### 使用 Webpack 构建本地服务器

想不想让 你的浏览器监听你的代码的修改，并自动刷新显示修改后的结果，其实 `webpack` 提供一个可选的本地开发服务起，这个本地服务器给予 node.js 构建，可以实现你想要的这些功能，不过他是一个单独的组件，在 webpack 中进行配置之前需要单独安装它作为项目依赖。

```shell
npm install --save-dev webpack-dev-server
```

devserver 作为 webpack 配置选项中的一项，以下是他的一些配置选项，更多配置可以参考这里

| devserver 的配置选项 | 功能描述                                                                                                                                            |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| contentBase          | 默认 webpack-dev-server 会为根文件夹提供本地服务器，如果想为另外一个目录下的文件提供本地服务器，应该在这里设置其所在目录 (本例设置到 "public" 目录) |
| port                 | 设置默认监听端口，如果省略，默认为 “8080”                                                                                                           |
| inline               | 设置为 `true`,当源文件改改变时会自动刷新页面                                                                                                        |
| historyApiFallback   | 在开发单页应用时非常有用，它依赖于 HTML5 history API,如果设置为 `true`,所有的跳转将指向 index.html                                                  |

把这些命令添加到 webpack 的配置文件中，现在的配置文件 `webpack.cinfig.js` 如下所示：

```js
module.exports = {
  devtool: 'eval-source-map',
  entry: __dirname + '/app/main.js',
  output: {
    path: __dirname + '/public',
    filename: 'bundle.js'
  },
  devServer: {
    contentBase: './public',  // 本地服务器所加载的页面所在的目录
    historyApiFallback: true, // 不跳转
    inline: true              // 实时刷新
  }
}
```

在 `package.json` 中的 `scripts` 对象中添加如下命令，用意开启本地服务器：

```js
"scripts": {
  "test": 'echo \"Error: no test specified\" && exit 1',
  "start": 'webpack',
  "server": 'webpack-dev-server --open'
}
```

在终端输入 `npm run server` 即可在本地的 `8080` 端口查看结果

![npm run server](./static/img/webpack8.png)

### Loaders

**鼎鼎大名的 Loaders 登场了！**

`Loaders` 是 `webpack` 提供的最激动人心的功能之一了。通过使用不同的 `loader`,`webpack` 有能力调用外部的脚本或工具，实现对不同格式的文件的处理，比如说分析转换 scss 为 css，或者把下一代的 JS 文件 (ES6, ES7) 转换为现代浏览器兼容的 JS 文件，对 React 的开发而言，合适的 Loaders 可以把 React 中用到的 JSX 文件转换为 JS 文件。

Loaders 需要单独安装并且需要在 `webpack.config.js` 中的 `modules` 关键字下进行配置， Loaders 的配置包括以下几方面：

- `test`: 一个用以匹配 loaders 所处里文件的扩展名的正则表达式 (必须)
- `loader`: loader 的名称 (必须)
- `include/exclude`: 手动添加必须处理的文件 (文件夹) 或屏蔽不需要处理的文件 (文件夹) (可选)
- `query`: 为 loaders 提供额外的设置选项 (可选)

不过在配置 loader 之前，我们把 `Greeter.js` 里的问候消息放在一个单独的 JSON 文件里，并通过合适的配置使 `Greeter.js` 可以读取该 JSON 文件的值，各文件修改后的代码如下：

在 app 文件夹中创建带有问候信息的 JSON 文件 (命名为 `config.json`)

```js
"greetText": "Hi there and greetings from JSON!"
```

更新后的 Greeter.js

```js
var config = require('./config.json');

module.exports = function() {
  var greet = document.createElemnt('div');
  greet.textContent = config.greetText;
  return greet;
}
```

> **注** 由于 `webpack3.*/webpack2.*` 已经内置可处理 JSON 文件，这里我们无须再添加 `webpack1.*` 需要的 `json-loader`。

再看如何具体使用 loader 之前我们先看看 Babel 是什么？

## Babel

Babel 其实是一个编译 JavaScript 的平台，它可以编译代码帮你达到以下目的：

- 让你能使用最新的 JavaScript 代码 (ES6, ES7...),而不用管新标准是否被当前使用的浏览器完全支持；
- 呃昂你能使用基于 JavaScript 进行了拓展的语言，比如 React 的 JSX

### Babel 的安装与配置

Babel 其实是几个模块化的包，其核心功能位于称为 `babel-core` 的 npm 包中， webpack 可以把其不同的包整合在一起使用，对于每一个你需要的功能或拓展，你都需要安装单独的包 (用的最多的使解析 ES6 的 `babel-env-preset` 包和解析 JSX 的 `babel-preset-react` 包)。

我们先来一次性安装这些依赖包

```shell
// npm 一次性安装多个依赖模块，模块之间用空格隔开
npm install --save-dev babel-core babel-loader babel-preset-env babel-preset-react
```

在 `webpack` 中配置 Babel 的方法如下：

```js

```
