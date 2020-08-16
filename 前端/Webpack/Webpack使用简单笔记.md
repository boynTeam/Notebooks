# Webpack的作用

本质上来说,webpack就是一个现代的,用于JS应用的静态*模块打包*工具,最重要的概念就是:模块与打包

我们先来看一张图

![](http://imageblog.boyn.top/20200309161358.png?imageView2/0/q/75|watermark/2/text/Ym95bi50b3A=/font/5b6u6L2v6ZuF6buR/fontsize/600/fill/IzAwMDAwMA==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

这张图清晰地表明了Webpack的作用,它的最大作用,就是将很多比如sass cjs这种无法给浏览器直接解析的文本格式,进行解析打包,成为浏览器可以识别的格式

## Webpack的模块化支持

而Webpack也是能够很好的支持模块化,将前端模块化的一些方案进行整合,解析成为JS语法,使得浏览器支持.

通常在开发中,我们可能使用`CommonJS`,`ES6`等语法来使用模块化,但是Webpack可以将其解析成比ES6支持更广泛的JS语法,使得所有浏览器都可以进行支持

并且还可以处理好模块与模块之间的依赖关系,将其整合起来

在Webpack中,CSS,图片和JSON等,都可以将其作为模块来使用.

## Webpack的打包支持

webpack将各种资源进行打包合并,并且对资源进行整理与转化,包括`scss -> css ` ,`ES6 grammer -> ES5 grammer`

# Webpack安装

webpack安装十分简单

```shell
 cnpm install webpack -g
```

直接安装即可

# Webpack使用

## 基本使用

在基本使用中,我们首先创建了两个文件夹 `dist`与`src`,`src`中存放的是我们的源码,而`dist`中就是存放webpack打包之后产出的文件了.

在`src`下,创建一个`main.js`作为主文件

并创建一个`mathUtils.js`作为其中一个模块

![](http://imageblog.boyn.top/20200309212528.png?imageView2/0/q/75|watermark/2/text/Ym95bi50b3A=/font/5b6u6L2v6ZuF6buR/fontsize/600/fill/IzAwMDAwMA==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

`main.js`

```js
const {add, mul} = require("./mathUtils");

console.log(add(20, 30));
console.log(mul(5, 6));

```

`mathUtils.js`

```js
function add(num1, num2) {
  return num1 + num2
}

function mul(num1, num2) {
  return num1 * num2
}

module.exports = {
  add,
  mul
};

```

`index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
<script src="./dist/bundle.js"></script>
</body>
</html>
```

接着,我们运行

`webpack .\src\main.js -o .\dist\bundle.js`

webpack会自动打包,并处理main.js所依赖的文件.就可以在dist文件夹下创建一个输出的文件,其中的内容是被混淆的

将`bundle.js`放入index.html文件中,就如同平常的js文件引入,就可以运行我们的函数了

## webpack配置文件

我们在进行webpack编译的时候,不能够将所有的参数都写在命令行中,所以,我们就需要用一个文件来表示webpack的配置了

我们可以创建一个配置文件`webpack.config.js`,将配置信息写入其中,然后直接运行`webpack`命令,就可以直接进行打包了

`webpack.config.js`

```js
const path = require("path");
module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname,"dist"),
    filename: "bundle.js"
  }
};
```

而为了在本地有不同的版本,我们需要使用`npm init`来创建一个node项目,并用`npm install webpack@[VERSION] --save-dev`来在本地引入webpack的不同版本依赖

但是如果我们直接用命令行来使用`webpack`命令,还是会使用全局的依赖,所以我们要使用npm的依赖,我们在`package.json`中引入build命令,会优先在本地寻找依赖

```json
{
  "name": "learn_webpack",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo 'hi'",
    "build": "webpack"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^3.6.0"
  }
}

```

`npm run build`

## loader

loader是webpack中一个核心的概念

当我们想要将css,图片模块化和ES6代码转为ES5等等功能,对于webpack本身来说都是不支持的

但是我们可以使用webpack拓展的loader

loader的使用很简单,我们需要做的是

1. npm安装使用的loader
2. 在webpack.config.js中下面的modules关键字进行配置

下面我们以处理css文件为例,讲解loader的使用

我们在以上的项目中,在src下创建一个css文件

里面的内容是随意的,假设名为`./css/normal.css`

写好了css文件之后,我们要在`main.js`中也引入这个依赖

```js
require("./css/normal.css");
```

但是,这个时候我们如果直接运行,npm毫无疑问会报错,这是因为webpack默认并没有对应css的解析模块,也就是loader,所以我们需要安装css-loader和style-loader

首先,我们运行一下命令

```shell
npm install --save-dev css-loader style-loader
```

然后在`webpack.config.js`里面添加对应的配置文件

```json
const path = require("path");
module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // style-loader
          { loader: 'style-loader' },
          // css-loader
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
      }
    ]
  }
};
```

## less文件的处理

安装依赖

```shell
npm install less-loader less --save-dev
```

配置文件

```json
{
        test: /\.less$/,
        use: [
          {
            loader: 'style-loader',
          },
          {
            loader: 'css-loader',
          },
          {
            loader: 'less-loader',
            options: {
              paths: [path.resolve(__dirname, 'node_modules')],
            },
          },
        ],
      },
```

[这些东西](https://v4.webpack.js.org/loaders/less-loader/)都可以在官网中找到

## 图片文件的处理

在开发中,除了css文件,还会用到各种各样的图片

在webpack中,如果我们需要使用图片,就要进行不一样的处理

首先,根据官网的说明,我们需要下载两个loader

分别是`url-loader`和`file-loader`,当我们的图片大小 小于限定的字节数时,就会将这个图片编码成Base64字符串放在网页中,不需要再次请求,而如果大于限定的字节,就会打包放入`dist`文件夹中,并进行哈希命名.

其中,安装loader的命令为

```shell
npm install url-loader file-loader --save-dev
```

但是我们只需要配置url-loader即可

```json
{
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192
            },
          },
        ],
      },
```

但是当我们处理大小超过限定字节数的文件的时候,虽然会自动转为url进行处理,但是我们打包后的图片文件是放在`dist`文件夹中的,而url并没有dist的前缀,所以这个时候就需要我们来手动配置,增加dist的这个前缀了

我们在`webpack.config.js`这个文件中,有一个output的选项里面,新增一个`publicPath`即URL的公共路径`publicPath: "dist/"`,而后再进行打包,就可以引用到打包的文件了

## ES6语法处理

```shell
npm install -D babel-loader @babel/core @babel/preset-env --save-dev
```

# Webpack中配置Vue

首先,我们需要通过npm来安装Vue

```shell
cnpm install vue --save
```

这一次,由于vue不是开发时依赖而是运行时依赖了,所以我们需要使用`--save`而不是`--save-dev`

而如果我们直接import像这样

`main.js`

```js
import Vue from 'vue';

const app = new Vue({
  el: "#app",
  data: {
    message: "你好"
  }
});
```

并在html中声明一个app的div

这样build出来的话,是不会有效果的

因为Vue在编译的时候会有两种版本

1. runtime-only
2. runtime-complier

而only版本的代码中,不可以有任何的template,只有compiler才能包含template的代码

所以我们要使用runtime-compiler版本进行编译

我们在`webpack.config.js`中加上

```json
resolve: {
    alias : {
      "vue$":"vue/dist/vue.esm.js"
    }
  }
```

![](http://imageblog.boyn.top/20200310114055.png?imageView2/0/q/75|watermark/2/text/Ym95bi50b3A=/font/5b6u6L2v6ZuF6buR/fontsize/600/fill/IzAwMDAwMA==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

就可以显示出信息了.

当然,这样子做显示不出来Webpack支持模块化的威力.

我们可以在src下创建一个vue文件夹

并创建一个Vue的文件

![](http://imageblog.boyn.top/20200310154431.png?imageView2/0/q/75|watermark/2/text/Ym95bi50b3A=/font/5b6u6L2v6ZuF6buR/fontsize/600/fill/IzAwMDAwMA==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

直接在`main.js`中进行引用即可

![](http://imageblog.boyn.top/20200310154505.png?imageView2/0/q/75|watermark/2/text/Ym95bi50b3A=/font/5b6u6L2v6ZuF6buR/fontsize/600/fill/IzAwMDAwMA==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

为了解析vue文件,我们同样需要loader

```shell
cnpm install vue-loader vue-template-compiler --save-dev

```

并更改配置

```json
{
        test: /\.vue$/,
        use:[
            "vue-loader"
        ]
```

