##  课程地址

https://coding.imooc.com/class/chapter/316.html#Anchor

## webpack初探

###   2-1

浏览器不能识别ES Module

`index.html`直接引入`index.js`

```js
// ES Moudule 模块引入方式
import Header from './header.js';
import Sidebar from './sidebar.js';
import Content from './content.js';

new Header();
new Sidebar();
new Content();
```

直接打开报错`SyntaxError`，需要使用`webpack`来打包(翻译`import`,`export`)

```js
npm init

npm install webpack-cli --save-dev

npx webpack index.js

//执行命令后将显示
Hash: 7f018c97803762d57faa
Version: webpack 4.46.0
Time: 126ms
Built at: 2021-11-27 9:19:13 ├F10: PM┤
  Asset      Size  Chunks             Chunk Names
main.js  1.29 KiB       0  [emitted]  main
Entrypoint main = main.js 
[0] ./index.js + 3 modules 749 bytes {0} [built]
    | ./index.js 183 bytes [built]
    | ./header.js 185 bytes [built]
    | ./sidebar.js 191 bytes [built]
    | ./content.js 190 bytes [built]
```

完成后`src`文件夹下将出现`dist`文件夹，里面有`main.js`文件

在`index.html`下引入`main.js`即可完成

感受到的第一个功能是对`import`,`export`的翻译

###   2-2

`webpack`核心定义本质上，`webpack` 是一个用于现代 `JavaScript` 应用程序的 *静态模块打包工具*

把`Header`,`Content`,`Sidebar`这些模块打包到一起生成文件

用其他模块规范也可以识别

```js
// ES Moudule 模块引入方式
// CommonJS 模块引入规范
// CMD
// AMD

// webpack 模块打包工具
// 发展：js 模块打包工具 -> 其他任何形式模块打包(css、图片等)
// 比如CSS模块  import style from './index.css'

// import Header from './header.js';
// import Sidebar from './sidebar.js';
// import Content from './content.js';

var Header = require('./header.js');
var Sidebar = require('./sidebar.js');
var Content = require('./content.js');

new Header();
new Sidebar();
new Content();
```

补充阅读：

[官方文档Module](https://webpack.docschina.org/concepts/modules/)

[ES6 Module CommonJS](https://es6.ruanyifeng.com/#docs/module-loader)

[前端模块化：CommonJS,AMD,CMD,ES6](https://juejin.cn/post/6844903576309858318)

###  2-3  安装

要让`webpack`打包项目需要符合`node`规范，`npm init`

全局安装`webpack` 不推荐  多个项目使用不同版本的webpack会出现问题

在项目中安装`webpack`

```js
npm install webpack@版本号 webpack-cli -D
```

`node`提供了`npx`命令来运行`webpack`

```js
webpack -v    //webpack:command not found
npx webpack -v  //显示项目内webpack的版本号
```

### 2-4 配置

默认配置文件`webpack.config.js`

```js
const path = require('path');

module.exports = {
    //打包的入口文件
	entry: './src/index.js',
    //生成的文件目录__dirname.dist,文件名bundle.js   
	output: {
		filename: 'bundle.js',
		path: path.resolve(__dirname, 'dist')
	}
}
```

使用命令`npx webpack`命令即可完成打包 

如果不使用`webpack.config.js`文件，可使用`npx webpack --config a.js`命令即可将`a.js`作为配置文件

使用`npm script`

```js
//package.json
"scripts": {
   "bundle": "webpack"
},
```

输入命令`npm run bundle`即可完成  打包



使用webpack命令的方法

+ global

  webpack

+ local

  npx webpack

+ npm srcipts

  npm run bundle

webpack-cli作用：能在命令行中使用  webpack命令

### 2-5

```js
//webpack.config.js
module.exports={
    mode:'development' //代码没有压缩
    mode:'production'  //被压缩至一行的bundle.js
}
```

## webpack核心概念

### 3-1 Loader

Loader引入的原因：webpack默认只认识js文件，打包其他的文件需要。

```js
//webpack.config.js
const path = require('path');

module.exports = {
	mode: 'development',
	entry: {
		main: './src/index.js'
	},
	module: {
		rules: [{
			test: /\.jpg$/,
			use: {
				loader: 'file-loader'
			} 
		}]
	},
	output: {
		filename: 'bundle.js',
		path: path.resolve(__dirname, 'dist')
	}
}
```

```js
//index.js
import avatar from './avatar.jpg';

var img = new Image();
img.src = avatar;

console.log(avatar);  //bd7a45571e4b5ccb8e7c33b7ce27070a.jpg

var root = document.getElementById('root');
root.append(img);
```

完成打包后，`dist`目录下出现两个文件

`bd7a45571e4b5ccb8e7c33b7ce27070a.jpg`和`bundle.js`

分析打包的流程：

当运行打包命令时,`webpack`回去寻找配置文件帮助做打包，但遇到js文件时，正常能执行，但遇到`jpg`文件时，求助`file-loader`。当发现`.jpg`文件移动到`dist`目录下，得到图片的名称，得到返回值，将值赋给`avatar`变量。静态资源可以用`file-loader`

思考`loader`的本质:

`loader`就是一种打包的方案，但遇到非`js` 文件时求助`loader`

当引入了非`js`文件，需要考虑`loader`

### 3-2 使用Loader打包静态资源（图片）

问题：想要打包出的图片文件名字不变

在`use`中配置`option`

[官方文档中file-loader的介绍](https://v4.webpack.docschina.org/loaders/file-loader)

 ```js
 	module: {
 		rules: [{
 			test: /\.jpg$/,
 			use: {
 				loader: 'file-loader',
                 options:{
                     //placeholder
                     name:'[name].[ext]'
                 }
 			} 
 		}]
 	},
 ```

想打包得到的文件生成在`images`文件夹下

```js
options: {
    name: '[name]_[hash].[ext]',
    outputPath: 'images/',
}
```

使用`url-loader`

 ```js
 rules: [{
     test: /\.(jpg|png|gif)$/,
     use: {
         loader: 'url-loader',
         options: {
             name: '[name]_[hash].[ext]',
             outputPath: 'images/',
         }
     } 
 ```

将图片直接转化为`Base64`字符串直接放入生成的`bundle.js`而不是生成一个图片文件。

好处：节省了一个`HTTP`请求 。

问题：如果图片非常大会使`bundle.js`文件很大，加载`js`文件时间变长。

最佳实践：如果图片小可以使用`url-loader`生成`Base64`,

图片比较大生成单独的图片文件放入`dist`目录下 。

方法：`option`设置`limit:10240`表示图片小于`10kb`时使用

`base64`,大于`10kb`在`dist`目录下生成`img`文件

与`file-loader`对比：多了一个`limit`的配置项，可以使用`base64`

补充阅读：[官方文档url-loader](https://v4.webpack.docschina.org/loaders/url-loader/)

### 3-3 Loader打包样式

 ```js
 //index.js
 import avatar from './avatar.jpg';
 import './index.css';
 
 var img = new Image();
 img.src = avatar;
 img.classList.add('avatar');
 
 var root = document.getElementById('root');
 root.append(img);
 ```

需要使用`style-loader`和`css-loader`

```js
//webpack.config. 
module: {
    rules: [{
        test: /\.(jpg|png|gif)$/,
        use: {
            loader: 'url-loader',
            options: {
                name: '[name]_[hash].[ext]',
                outputPath: 'images/',
                limit: 10240
            }
        } 
    },{
        test: /\. css$/,
        use: [
            'style-loader', 
            'css-loader',
        ]
    }]
},
```

`css-loader` 分析出几个`css`文件间的关系,最终将这些`css`文件合并得到一段`css`

`style-loader` 在得到`css-loader`的内容后挂载到`header`上

引入`scss`文件时需要`sass-loader`时将`scss`文件编译成`css`文件

`loader`是有执行顺序的，需要在配置文件中设置好

例如

```js
use:['style-loader','css-loader','sass-loader']
```

依次执行`sass-loader`、`css-loader`、`style-loader`

需要添加浏览器厂商前缀时使用`postcss-loader`

添加`postcss.config.js`

```js
module.exports = {
  plugins: [
  	require('autoprefixer')
  ]
}
```

### 3-4

```js
//webpack.config.js
{
    test: /\.scss$/,
        use: [
            'style-loader', 
            {
                loader: 'css-loader',
                options: {
                    importLoaders: 2,
                    modules: true
                }
            },
            'sass-loader',
            'postcss-loader'
        ]
}
```

`options`中`importLoaders`的作用

当`index.scss`文件中引入其他`scss`文件时，其他`scss`文件要从`postcss-loader`开始执行

`css-module`为了避免样式的全局污染

在`options`中添加`modules:true`

```js
import avatar from './avatar.jpg';
import  './index.scss';  
import createAvatar from './createAvatar';

createAvatar();

var img = new Image();
img.src = avatar;
img.classList.add('avatar');

var root = document.getElementById('root');
root.append(img);
```

上述代码的问题：`createAvatar()`创建了一个`class="avatar"`的`img`，引入的`index.scss`也会作用于这个`img`。

修改方法

```js
//webpack.config.js
{
    test: /\.scss$/,
        use: [
            'style-loader', 
            {
                loader: 'css-loader',
                options: {
                    importLoaders: 2,
                    modules: true
                }
            },
            'sass-loader',
            'postcss-loader'
        ]
}
```

`options`加入`module:true`，形成`css-module`

```js
//index.js
import avatar from './avatar.jpg';
import style from './index.scss';  
import createAvatar from './createAvatar';

createAvatar();

var img = new Image();
img.src = avatar;
img.classList.add(style.avatar);

var root = document.getElementById('root');
root.append(img);
```

引入方法

```js
import style from './index.scss';
```

类的添加方式

```js
img.classList.add(style.avatar);
```

修改。

这样`index.scss`设置的样式不会去作用`createAvatar()`添加的元素。

代表`index.scss`只会在这个模块内生效，样式不会有任何的耦合和冲突

 打包字体文件

```js
//webpack.config.js
{
    test: /\.(eot|ttf|svg)$/,
        use: {
            loader: 'file-loader'
        } 
}
```

### 3-5 plugins

问题引入：每次打包完成后`dist`目录下需要去手动引入`html`文件，开发麻烦。

解决方法：引入[HtmlWebpackPlugin](https://v4.webpack.docschina.org/plugins/html-webpack-plugin)

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');

plugins: [new HtmlWebpackPlugin({
    template: 'src/index.html'
}), new CleanWebpackPlugin(['dist'])],
```

会在打包结束后，自动生成一个`html`文件，并把打包生成的`js`自动引入到这个`html`文件中

` template: 'src/index.html'`

```ht
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>html 模版</title>
</head>
<body>
	<div id="root"></div>
	<p>hello,world</p>
</body>
</html>
```

打包后:

```js
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>html 模版</title>
</head>
<body>
	<div id="root"></div>
	<p>hello,world</p>
<script type="text/javascript" src="bundle.js"></script></body>
</html>
```

引申：

`plugin`可以在`webpack`运行到某个时刻的时候，帮你做一些事情，可以类比`React`的生命周期函数

问题引入：希望打包的时候先把`dist`目录删除再进行创建，避免前面打包的影响

解决方法：`CleanWebpackPlugin`

`new CleanWebpackPlugin(['dist'])`

打包前会将`dist`目录删除 

使用`plugin`方法：去实现一些功能时搜索相应配置，根据查询到的配置再去找相关的plugin，根据说明手册配置

### 3-6 Entry和Output的基础配置

需求：想打包生成两个文件`main`和`sub`

```js
entry: {
    main: './src/index.js',
    sub: './src/index.js'
},
```

如果`output`是这样,打包将会出错

```js
output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
}
```

修改成

```js
output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'dist')
}
```

生成的`html`文件,同时`dist`目录下会出现`main.js`和`sub.js`

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>html 模版</title>
</head>
<body>
	<div id='root'></div>
<script type="text/javascript" src="main.js"></script><script type="text/javascript" src="sub.js"></script></body>
</html>
```

希望打包得到的js文件前面增加域名

```js
output: {
    publicPath: 'http://cdn.com.cn',
    filename: '[name]_[hash].js',
    path: path.resolve(__dirname, 'dist')
}
```

打包得到的`index.html`

```js
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>html 模版</title>
</head>
<body>
	<div id='root'></div>
<script type="text/javascript" src="http://cdn.com.cn/main_17eaf0ca1edbefc5529c.js"></script><script type="text/javascript" src="http://cdn.com.cn/sub_17eaf0ca1edbefc5529c.js"></script></body>
</html>
```

### 3-7 sourcemap的配置

问题引入：打包得到的程序运行后，如果源代码出错，控制台出现的bug的位置是打包后的文件的代码，无法准确定位错误

`sourcemap`的概念:

现在知道`dist`目录下`main.js`文件96行出错

`sourceMap`是一个映射关系，它知道dist目录下`main.js`文件96行实际上对应的是`src`目录下`index.js`文件中的第一行

引入方法

`devtool:'source-map'` `dist`目录下将出现`main.js.map`

`devtool:'inline-source-map'` `main.js.map`将写入`main.js`中

`devtool:'cheap-inline-source-map'` 没加`cheap`去精确到出错的字符,加入`cheap`只会精确到行，加入`cheap`会加快打包的效率和性能

`devtool`:`module` 还会管第三方工具的错误，比如`loader`

`devtool`:`eval`执行效率最快，性能最好, 提示效果不好

`devtool`可能的最佳实践:

```js
development devtool: 'cheap-module-eval-source-map',
production devtool: 'cheap-module-source-map',
```

### 3-8 WebpackDevServer

问题引入：打包完的代码要发生改变时，又需要进行重新打包来调试，开发效率变低

想法：改变`src`目录下的源代码，`dist`目录会重新打包

```json
 //package.json
"scripts": { 
	"watch": "webpack --watch",
 }
```

`wacth`的作用:监听源代码的改变，发生改变时会重新打包，不会启服务器而且需要手动刷新

想法：打完包后自动打开浏览器，自动启一个服务器

```js
//webpack.config.js
devServer: {
    contentBase: './dist',
    open:true   //会自动打开浏览器
}
```

```json
//package.json
"scripts": { 
	"watch": "webpack --watch",
    "start": "webpack-dev-server"
 }
```

`npm run start`

`webpack-dev-server`:监听源代码的改变，发生改变时不仅会重新打包,还会自动刷新浏览器，会自动开启一个web服务器，这是目前用得最多的模块

要开启`web`服务器的原因：`file//`打开碰到源代码要发`AJAX`请求时无法完成

```js
devServer;{
    proxy:{
        '/api':'http://localhost:3000'
        //当用户访问api时会转发到localhost:3000，其他脚手架底层就是使用了		  //devServer,可以支持跨域代理的配置
    }
    port:8080 //改变端口号
}
```

`node`启动，这是一个简易版的`dev-server`，这是在`node`中使用`webpack`

```js
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');
const config = require('./webpack.config.js');
// 在node中直接使用webpack
// 在命令行里使用webpack
const complier = webpack(config);

const app = express();

app.use(webpackDevMiddleware(complier, {}));

app.listen(3000, () => {
	console.log('server is running');
});
```

这个浏览器不会自动刷新

自己写web-devServer会比较困难

[Node接口](https://webpack.docschina.org/api/node/)

### 3-9 HMR

概念：HMR 全称 Hot Module Replacement，中文语境通常翻译为模块热更新，它能够在保持页面状态的情况下动态替换资源模块，提供丝滑顺畅的 Web 页面开发体验。

使用方法：

开启配置项

```js
devServer: {
    contentBase: './dist',
    open: true,
    port: 3000,
    hot: true,   //开启热模块更新
    hotOnly: true   //如果HMR未生效浏览器也不会自动刷新
},
```

加插件：这个插件`webpack`自带

```js
const webpack = require('webpack');
plugin:[new webpack.HotModuleReplacementPlugin()]

```

好处：方便调试CSS，CSS做了修改不会影JS渲染出的内容

### 3-10

```js
import counter from './counter';
import number from './number';

counter();
number();

//css-loader底层帮忙实现了类似这样的功能
//React内置了babel-preset
//本质上要想实现HMR的功能都需要写这样的代码
//碰到比较偏门的文件可能需要手写实现这样的功能
//监控number这个模块的变化,当number发生变化时，重新执行
if(module.hot) {
	module.hot.accept('./number', () => {							       document.body.removeChild(document.getElementById('number'));
		number();
	})
}
```

### 3-11 Babel

将ES6翻译成ES5

JS新语法编译工具，不关系模块化

`npm i babel-loader babel/@core`

```js
{ 
    test: /\.js$/, 
    exclude: /node_modules/, 
    loader: 'babel-loader',
}
```

`babel-loader`只是`webapck`和`babel`通信的桥梁

`npm i @babel/preset-env --save-dev`包含ES6->ES5的翻译规则

`babel-polyfill`兼容不支持的方法

`corejs`和`regenerator`（支持generator）的集合

###  3-13 配置React代码的打包

 `npm install --save-dev @babel/preset-react`

`.babelrc`

```js
{
	"presets": [
		[
			"@babel/preset-env", {
				"targets": {
					"chrome": "67"
				},
				"useBuiltIns": "usage"
			}
		],
		"@babel/preset-react"
	]
}
```

先将jsx=>js,再用`@babel/preset-env`将ES6=>ES5,从下往上的顺序

## webpack的高级概念

### 4-1 TreeShanking

`math.js`

```js
export const add = (a, b) => {
	console.log( a + b );
}

export const minus = (a, b) => {
	console.log( a - b );
}
```

`index.js`

```js
import { add } from './math.js';
add(1, 2);
```

问题：不做`tree-shaking`最后打包得到的文件会将`minus`方法引入，会让最后打包得到的文件变大，理想情况是按需打包

前提： `Tree Shanking`只支持`ES Module`,因为`ES Module`是静态引入，不支持`CommonJS`(动态引入)

操作：

```js
//webpack.config.json
mode: 'development',
optimization: {
   usedExports: true
},
```

```js
//package.json
"sideEffects": false,  //设置特殊文件的tree-shaking,比如@babel/polyfill等，sideEffects": ["@babel/polyfill"],css文件一般也没有导出，需要加入sideEffects中
```

当`mode: 'development'`时不需要配置`optimization`就能自动Tree-shaking

### 4-2 develpoment 和 production模式

区别：

1. devserver 生产模式下不需要

2. sourcemap 生产模式下的比开发模式下的更加简洁

通用配置

```js
//webpack.common.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
	entry: {
		main: './src/index.js'
	},
	module: {
		rules: [{ 
			test: /\.js$/, 
			exclude: /node_modules/, 
			loader: 'babel-loader',
		}, {
			test: /\.(jpg|png|gif)$/,
			use: {
				loader: 'url-loader',
				options: {
					name: '[name]_[hash].[ext]',
					outputPath: 'images/',
					limit: 10240
				}
			} 
		}, {
			test: /\.(eot|ttf|svg)$/,
			use: {
				loader: 'file-loader'
			} 
		}, {
			test: /\.scss$/,
			use: [
				'style-loader', 
				{
					loader: 'css-loader',
					options: {
						importLoaders: 2
					}
				},
				'sass-loader',
				'postcss-loader'
			]
		}, {
			test: /\.css$/,
			use: [
				'style-loader',
				'css-loader',
				'postcss-loader'
			]
		}]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: 'src/index.html'
		}), 
		new CleanWebpackPlugin(['dist'])
	],
	output: {
		filename: '[name].js',
		path: path.resolve(__dirname, 'dist')
	}
}
```

生产模式相关配置

```js
//webpack.prod.js
const merge = require('webpack-merge');
const commonConfig = require('./webpack.common.js');

const prodConfig = {
	mode: 'production',
	devtool: 'cheap-module-source-map'
}

module.exports = merge(commonConfig, prodConfig);
```

开发模式下相关配置

```js
//webpack.dev.js
const webpack = require('webpack');
const merge = require('webpack-merge');
const commonConfig = require('./webpack.common.js');

const devConfig = {
	mode: 'development',
	devtool: 'cheap-module-eval-source-map',
	devServer: {
		contentBase: './dist',
		open: true,
		port: 8080,
		hot: true
	},
	plugins: [
		new webpack.HotModuleReplacementPlugin()
	],
	optimization: {
		usedExports: true
	}
}

module.exports = merge(commonConfig, devConfig);
```

按照相关命令完成生产模式下和开发模式下的打包

```json
//package.json
  "scripts": {
    "dev": "webpack-dev-server --config ./build/webpack.dev.js",
    "build": "webpack --config ./build/webpack.prod.js"
  },
```

### 4-3 code-splitting

问题引入·

```js
import _ from 'lodash';  //1mb

//业务逻辑 1mb
console.log(_.join(['a','b','c'],'***'));
//此处省略10万行业务逻辑
console.log(_.join(['a','b','c'],'***'));

//dist/main.js 2mb

//打包文件会很大，加载时间会长

//若改变业务逻辑，重新访问我们的页面，又要加载2mb的内容 
```

初步解决方法

```js
//lodash.js
import _ from 'lodash';
window._=_;
```

打包生成两个文件`lodash.js`和`main.js`

`main.js`（2MB）被拆成`lodash.js`(1MB)和`main.js`(1MB)

可以利用缓存(当业务逻辑发生变化时只要重新加载`main.js`)和浏览器的并行加载

### 4-4

概念：对代码进行拆分使性能更高，用户体验更好

是`webpack`作为打包工具所特有的一项技术，把代码按照特点的形式进行拆分，使用户不必一次全部加载，而是按需加载

使用：

```js
//webpack.config.js
optimization: {
    splitChunks: {
        chunks: 'all',
    }
}
```

打包生成了`main.js`和`vendors~main.js`

使用webpack可以自动进行分割不需要手动进行。

另一种使用方式：不做任何配置，异步代码的分割

```js
function getComponent() {
	return import('lodash').then(({ default: _ }) => {
		var element = document.createElement('div');
		element.innerHTML = _.join(['Dell', 'Lee'], '-');
		return element;
	})
}

getComponent().then(element => {
	document.body.appendChild(element);
});
```

总结： 代码分割，和`webpack`无关

 `webpack`中实现代码分割，两种方式

1. 同步代码： 只需要在`webpack.common.js`中做`optimization`的配置即可

2. 异步代码(import): 异步代码，无需做任何配置，会自动进行代码分割，放置到新的文件中

### 4-5

### 4-7

## webpack实战案例配置

### 5-1 Library的打包

```js
//src/index.js
import * as math from './math';
import * as string from './string';

export default { math, string }
```

```js
//webpack.config.js
const path = require('path');

module.exports = {
	mode: 'production',
	entry: './src/index.js',
	externals: 'lodash',     //src/string中已拥有，避免两次打包lodash
	output: {
        //输出lib到dist目录下
		path: path.resolve(__dirname, 'dist'),
        //lib文件名
		filename: 'library.js',
		library: 'root',      //script标签引入，会在全局变量中增加root全局变量
		libraryTarget: 'umd'  //不管commonJS,AMD等任何方式引入都能生效
	}
}
```

### 5-2 打包PWA

## 思考

前端为什么要进行打包构建？

代码层面：

+ 体积更小(Tree-Shanking、压缩、合并)，加载更快
+ 编译高级语言或语法（TS、ES6+、模块化、scss）
+ 兼容性和错误检查(Poly-fill、postcss、eslint)

工程化方面：

+ 统一、高效的开发环境
+ 统一的构建流程和产出标准
+ 集成公司构建规范（提测、上线等）

module、chunk、bundle的区别

+ module-各个源码文件,webpack中一切皆模块
+ chunk-多模块合并成的,如entry import() splitChunk
+ bundle-最终的输出文件
