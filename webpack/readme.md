# Webpack指南

## webpack的特点

* webpack讲究的是一切都是模块（包括js, css, image, font）
* 按需加载(对多个文件分割打包)

### 开发环境 VS 生产环境

开发环境（webpack Development sample）

```javascript
var webpack = require('webpack')
	HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
	entry: './public/src/index.js',

	output: {
		path: __dirname + '/public',
		publicPath: '/',
		filename: 'bundle.js',
	},

	module: {
		loaders: [
			{ test: /\.json$/, loader: 'json' },
			{ test: /\.jsx$/, include: __dirname + '/public/src', loader: 'babel' },
			{ test: /\.css$/, loader: 'style!css?module!postcss' }
		]
	},

	resolve: {
		extensions: ['', '.js', '.jsx']
	},

	postcss: [
		require('autoprefix')
	],

	plugins: [
		new HtmlWebpackPlugin({
			template: './public/index.template.html',
			inject: true
		}),
		new webpack.HotModuleReplacementPlugin()
	],

	devtool: 'source-map',	// generate source map
	devServer: {	// config for webpack dev server
		colors: true,
		historyApiFallback: true,
		inline: true,
		hot: true,
		contentBase: './public'
	}
}

```

生产环境 （Webpack Production sample）

```javascript
var webpack = require('webpack'),
	HtmlWebpackPlugin = require('html-webpack-plugin'),
	AppCachePlugin = require('appcache-webpack-plugin');

module.exports = {
	entry: './public/src/index.js',

	output: {
		path: __dirname + '/public/',
		publicPath: 'http://someCDN.com',	// for loading images from a CDN
		filename: 'bundle.js'
	},

	module: {
		loaders: [
			{ test: /\.json$/, loader: 'json' },
			{ test: /\.jsx$/, include: __dirname + '/public/src', loader: 'babel' },
			{ test: /\.css$/, loader: 'style!css?modules!postcss' }
		]
	},

	resolve: {
		extensions: ['', '.js', '.jsx']
	},

	postcss: [
		require('autoprefix')
	],

	plugins: [
		new HtmlWebpackPlugin({ // generate index.html
			template: './plugin/index.template.html',
			inject: true
		}),
		new AppCachePlugin({ // generate manifest.appcache
			exclude: ['.htaccess']
		}),
		new webpack.optimize.UglifyJsPlugin() // uglify/minify
	]
}
```

``一般开发的时候需要配置两种配置文件（一个作为开发环境，一个作为生产环境）``

打包文件可以在`package.json`里这样写：

```
"scripts": {
	"build": "webpack --config webpack.config.prod.js", // npm run build
	"dev": "webpack-dev-server --hot --inline --colors "	// 在配置文件中可以移除掉这块的配置（--hot用于热加载(HRM)，--inline实时加载刷新）
}
```

> HMR 有什么卵用？当你要调试一个购票表单，第一步填写信息第二部选择票种，第三步付款。HMR可以让你在改完代码后直接调试第三部而不是页面刷新，从第一步开始自己输信息。

如果没有安装`webpack-dev-server`需要在安装`nodejs`的前提下安装

```javascript
npm i -g webpack-dev-server
```

### entry

entry-array

如果你有多个入口文件并且彼此之间没有依赖，你可以使用数组来写入口配置。

> 例如：你要将 googleAnalytics.js 加载到你的 HTML 文件，你可以像这样用 Webpack 将这个文件打包到所有依赖的最后：

```javascript
{
	entry: ['./public/src/index.js', './public/src/googleAnalytics.js'],
	output: {
		path: '/dist',
		filename: 'bundle.js'
	}
}
```

entry-object

现在，假设你有一个多页面的应用，不是单页面多个视图那种的。而是多个 HTML 文件，例如 (index.html 和 profile.html). 你通过写一个对象来告诉 Webpack 打包多个入口到多个输出。

> 下面的配置将会生成两个 JS 文件：indexEntry.js 和 profileEntry.js 文件，你可以在 index.html 和 profile.html 单独引用。

```javascript
{
	entry: {
		'indexEntry': './public/src/index.js',
		'profileEntry': './public/src/profile.js'
	},

	output: {
		path: '/dist',
		filename: '[name].js'	// indexEntry.js & profileEntry.js
	}
}
```

```html
// profile.html
<script src="dist/profileEntry.js"></script>

// index.html
<script src="dist/indexEntry.js"></script>
```

entry-combination

你也可以同时使用数组和对象来编写入口，下面的例子将会生成3个文件：vendor.js 包含三个外部引用文件，一个 index.js 和一个 profile.js。

```javascript
{
	entry: {
		'vendor': ['jquery', 'analytics.js', 'optimizely.js'],
		'index': './public/src/index.js',
		'profile': './public/src/profile.js'
	},

	output: {
		path: '/dist',
		filename: '[name].js'	// vendor.js & index.js & profile.js
	}
}
```

### output配置

path vs publicPath

`path`是用于将打包之后的文件存放的目录文件，是相对于工程目录而言的

`publicPath`用于在生产环境中替换路径的配置

```javascript
{
	output: {
		path: '/dist'	// 打包到工程目录下的dist目录下
		publicPath: 'http://someCDN.com/' // 将打包的路径放在`http://someCDN.com`域名下
	}
}
```

### loaders

工作环境

```javascript
require('file.css')
import 'file.js'
```

配置方式

```javascript
module: {
	loaders: [
		{ 
			test: /\.js$/, 
			exclude: /node_modules/, 
			loader: 'babel', 
			query: {
				presets: ['react', 'es2015', 'react-hmre']
			}
		}
	]
}
```

链式的loader加载方式（从右往左,以`!`分割）

```javascript
module: {
	loaders: [
		{
			test: /\.css$/,
			loader: 'style!css'
		}
	]
}
```

style-loader是将处理之后的css插入到html页面的`<style>`内

### loaders自身的配置

```javascript
// 小于1024bytes的图片使用base64字符串的格式
{
	test: /\.png$/,
	loader: 'url?limit=1024'
}

or

{
	test: /\.png$/,
	loader: 'url',
	query: {
		limit: 1024
	}
}

```

### .babelrc配置

`babel-loader` > 6.0 以后使用 `presets` 来配置如何将 ES6 的代码编译为 ES5 的代码 以及如何编译 React 的 JSX 到 JS。可以通过 `query` 选项来进行配置。

```javascript
// webpack.config.jsmodule: {
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /(node_modules|bower_components)/,
      loader: 'babel',
      query: {
        presets: ['react', 'es2015'] // 要使用的编译器
      }
    }
  ]
}
```

然而在许多项目中 babel 的配置可能会很大，所以你可以在项目跟目录下建一个 .babelrc 来管理 babel 配置。babel-loader 会自动检测该文件是否存在并应用配置。

所以上面的例子可以改成：

```javascript
//webpack.config.js
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /(node_modules|bower_components)/,
      loader: 'babel'
    }
  ]
}
//.babelrc文件
{ "presets": ["react", "es2015"]}
```

### 插件

插件是用来控制打包结果的第三方模块。

* uglifyJSPlugin （压缩混淆，以减小大小优化加载速度）
* extract-text-webpack-plugin （插件在内部调用 css-loader 和 style-loader 将所有 css 内容保存为一个 style.css 文件，并且将该文件插入到  HTML 中。）

```javascript
// webpack.config.js
var ETP = require("extract-text-webpack-plugin");
{
	module: {
		loaders: [
			{
				test: /\.css$/,
				loader: ETP.extract('style-loader', 'css-loader')
			}
		]
	},
	plugins: [
		new ExtractTextPlugin('style.css')	// 导出到style.css文件中
	]
}
```

### loaders和插件的对比

* 插件是`在打包合成`的时候，也即`最后一个步骤`，修改打包后的文件内容
* loaders是针对`单个的文件`进行`打包前或打包时`所进行的处理

### 使用webpack.config.babel.js来使用es6编写webpack配置

webpack会使用`babel-loader`来处理自己的配置文件

### 编译平台选择

如果你的 bundle.js 要发布到 npm 上让别人通过 webpack 引入或者 browserify 或者其他 AMD/CMD 的方式引入，那就和打包到浏览器里运行时不太一样的。比如你可以设置外部依赖，使用 process.env 等。

```javascript
{
  library: 'xxx',
  libraryTarget: 'umd',
  externals: {
    react: {
      root: 'React',
      commonjs: 'react',
      commonjs2: 'react',
      amd: 'react',
    },    'react-dom': {
      root: 'ReactDOM',
      commonjs: 'react-dom',
      commonjs2: 'react-dom',
      amd: 'react-dom',
    },
    moment: {
      root: 'Moment',
      commonjs: 'moment',
      commonjs2: 'moment',
      amd: 'moment',
    },
  }
}
```
