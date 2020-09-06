# 导读

此文档只对常用部分进行说明，详细的看webpack官方文档更为合适。

# 入门

入门先安装以下几个包

1. webpack: 基石包
2. webpack-cli:脚手架工具，配合webpack使用
3. webpack-dev-server:开发时用到，可以配置代理服务以及热更新
4. webpack-merge: 合并工具，可以将开发生产环境配置合并到common配置

# 一.常用配置项

## 1.打包入口**entry**和打包出口**output**

使用详情如下：

```js
module.exports = {
	entry: { 
        //入口可以配置单入口或者多入口,单入口写一个app就行;webpack中js类型是一等公民，以js文件为主入口
		app: './src/index.js', 
        app2: './src/index2.js'
    },
    output: {
		path: path.join(__dirname, 'dist'), //所有文件的输出路径
        filename: 'js/[name].[hash:8].js' //js文件输出路径以及名字和hash值和hash长度；
    }
}
```

**hash缓存说明：**[hash]中的hash可改为chunkhash(webpack自带)或者contenthash(一般css插件会带，要手动去使用);推荐使用contenthash，因为它对缓存更友好；

- hash: 整个项目静态文件共享一个hash，有一个文件作了修改，那么整个项目的hash都将失效。
- chunkhash: 每个输入文件都可以看做一个chunk,它的hash只由它依赖的文件决定，有一个文件修改了，只会影响被它依赖的文件。比方说，一个主文件里面引用一个同步js和异步引入了一个js,异步引入的js不会打入到主文件里面，它会单独输出成一个js文件，在主文件生成动态创建script标签的方法。如果我们使用hash的话，改了同步js,那么异步js的hash值也会改变，虽然它并没有做什么修改。如果我们使用chunkhash,修改同步js,异步js的hash就不会修改。
- contenthash: chunkhash对js很友好，但是当我们对css打成独立的包的时候，css改了，依赖它的js的hash也会改变,导致js的缓存失效。当配置contenthash，修改css就不会影响js的hash值,只会影响它自己hash值。

**注意事项：**chunkhash,contenthash不能与热更新同时使用；



## 2.module 模块相关

1.作用：匹配文件，将文件内容和类型转换成浏览器可识别的

2.用法：

```js
module: {
    noParse: ['jquery', 'lodash'],//不用去解析这些包里的引用，加快打包速度
	rules: [ //在这里面去匹配文件，然后使用相应的loader去修改文件内容和类型；
		{
			test: /\.css/, //正则匹配文件后缀
            exclude: /node_modules/ //不对node包里的文件进行匹配
            include: [path.join(__dirname, 'src')],//只匹配某某路径下文件
        	enforce: "pre" | "post",//执行顺序
            use: [ //use里的元素可以为Sring或者Object类型，默认从后往前执行
                {
                    loader: 'style-loader' || require('mini-css-extract-plugin').loader,
        			options: {//会传入loader的构造函数中,不同loader需要的配置可能不一样
        				limit: 1 * 1024
        			},
                },
               	'css-loader'
            ] 
		}
	]
}
```

## 3.resolveLoader loader查找路径配置

1.作用：因为是loader是webpack帮我做的引入，它默认是node_modules下面，要是想使用自定义的loader，可以重写查找路径

2.用法：

```js
 resolveLoader: { //modules就是配置loader的查找路径，可配置多个，从左往右
        modules: [path.resolve(__dirname, 'loaders'), path.resolve(__dirname, 'node_modules')],
 },
```

## 4.plugin 插件

1.作用：一般用来简洁流程，方便开发，批量操作等,和loader区别主要是使用场景和意义不一样，它也可以做loader的事。

2.用法：

```js
plugins: [ //数组类型，里面元素为插件实例
	new HtmlWebpackPlugin({
		title: 'helloWorld',
		template: './index.html'
	})
]
```

## 5.watch与watchOptions

1.作用：每次修改文件都要重新npx webpack一次很麻烦，当将watch设为true,就可以自动打包。

2.用法：

```js
watch: true,
watchOptions: { //只有当watch为true,下面配置才生效
	poll: 1000,
	aggregateTimeout: 500,
	ignored: /node_modules/,
}
```

3.备注：webpack-dev-server 更强大，不推荐使用watch

## 6.devtool 源码映射

1.作用：开发时候方便调试代码。

2.用法：

```js
devtool: 'inline-source-map', //可配置多种不同类型参数，可去官网了解，一般配置source-map就行，或者inline-source-map都行，一个是和代码一起生成一份js，一个是单独生成map.js文件
```

## 7.devServer 开发服务

1.作用：特别方便开发。它会开启一个本地服务器，默认能够实现热更新以及自动打包。还可以在配置中解决跨域以及自定义mock服务。

2.用法：安装webpack-dev-server包,使用 npx webpack-dev-server 命令

```
devServer: {
	port: 8081, //目标端口
	proxy: {
		'/api': { //代理的请求
			target: 'http://localhost:8081',//代理的目标服务器
			secure: false,
			changeOrigin: true,
		}
	}
}
```

## 8.resolve

1.作用：提供选项来设置模块查找相关

2.用法：

```js
resolve: {
	alias: {//用来配置一些常用模块的别名
        Utilities: path.resolve(__dirname, 'src/utilities/'),//别名和路径
    },
    modules: [path.resolve(__dirname, "src"), "node_modules"],//告诉webpack,解析模块时应该查找的路径，优先从左往右
    extensions: ['.js', '.css'],//不写后缀，将依次尝试此配置里面的后缀
    mainFields: ['loader', 'main'],//按此顺序读取node包package.json里面的入口文件
}
```

## 9.externals 外部拓展

1.作用：防止将某些import的包打入bundle中去

2.用法：

```html
//index.html
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous">
</script>
```

```js
//webpack.config.js
externals: {
  jquery: 'jQuery'
}
```

```js
//index.js
import $ from 'jquery';
$('.my-element').animate();
```

## 10.mode模式

mode分为**production**生产环境和**development**开发环境；

## 11.performance性能

1.作用：根据文件大小打印警告信息

# 2.常用plugin

下面将按功能分类介绍常用plugin

## 1.html相关

按模板生成html,或者自动生成html可以用 **html-webpack-plugin**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
plugins: [
    new HtmlWebpackPlugin({
			title: 'hello ccr',
			template: './index.html'
		}),
]
```

## 2.打包时自动清除目录下无关内容

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
plugins: [
   	new CleanWebpackPlugin(),
]
```

## 3.热更新时返回文件名而不是id

```js
new webpack.NamedModulesPlugin(), //热更新时返回文件名而不是id
```

## 4.压缩css

**optimize-css-assets-webpack-plugin**

## 5.压缩js

 **uglifyjs-webpack-plugin**

```js
optimization: {
    minimizer: [
      new UglifyJsPlugin({
        cache: true,
        parallel: true,
        sourcMap: true
      }),
      new OptimizeCSSAssetsPlugin({}),
    ],
  },
```



## 6.在打包时忽略本地化内容

```js
new webpack.IgnorePlugin(requestRegExp, [contextRegExp])
//requestRegExp 匹配资源的请求路径表达式
//contextRegExp 要忽略的包
new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
```

## 7.单独打包某些不需要tree-shaking的第三方模块,提高编译速度

webpack.DLLPlugin 和 webpack.DLLReferencePlugin

```js
//webpack.dll.config.js
module.exports = {
    entry: {
		vendor: ['vue', 'vue-router', 'vuex']//需要忽略的包名
    }，
    output: {
		path: path.join(__dirname, 'public'),
        filename: '[name].dll.[hash:8].js',
        library: '[name]_library', //和下面的name需要保持一致
	},
    plugins: [
		new webpack.DllPlugin({
			path: path.resolve(__dirname, '.', '[name]-mainfest.json'),
			name: '[name]_library',
			context: __dirname //上下文路径
		})
	]
}
```

```js
//webpack.config.js
new webpack.DllReferencePlugin({
	context: __dirname,
	manifest: require('./vendor-mainfest.json')
})
```

先npx webpack --config webpack.dll.config.js 生成第三方包和配置

在运行npx webpack  就可以正常使用了

## 8.多进程打包（项目不大时感觉不是很好用）

**happypack**

```js
const HappyPack = require('happypack');
const os = require('os');
const happyTreadPool = HappyPack.ThreadPool({ size: os.cpus().length });

{
	test: /\.m?js$/,
    exclude: /(node_modules)/,
    use: 'happypack/loader?id=hello',
}
plugins: [
    new HappyPack({
		id: 'hello',
        threadPool: happyTreadPool, //共享进程池
        loaders: [ //相当于原本写在use里面的loader现在全移到这边了
            {
                loader: 'babel-loader',
				options: {
					presets: ['@babel/preset-env']
				}
            }
        ]
    })
]
```

## 9.按需引入babel-plugin-component

```js
{
  "presets": [["es2015", { "modules": false }]],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      }
    ]
  ]
}
```

## 10.离线缓存方案 workbox-webpack-plugin

## 11.split-chunks-plugin

# 3.常用loader

​	loader的定义简单来说就是将各种各样的文件转换成浏览器可识别的代码。每个loader只做一件事。我们之所以可以在js文件中导入图片，导入样式，都是loader帮我们对其进行了转换。我们学习loader时可以从文件类型入手，转换什么样的格式就去webpack上找对应的 loader。

​	下面我们就根据文件类型介绍几种常用的loader

## 1.转换css类型

1)简单转换，**css-loader**用来解析css文件中的url和import路径，**style-loader**同过js来动态创建一个style标签将css文件内容动态插入html。

```js
{
	test: /\.css/,
    use: ['style-loader', 'css-loader']
}
```

2)高级语法**less-loader**

```js
{
	test: /\.less/,
    use: [
    	'style-loader', 
    	'css-loader',
    	{
			loader: 'less-loader',
			options: {...}
		}
    ]
}
```

3）给css3新特性加上浏览器前缀 **postcss-loader** 和它的插件 **autoprefixer**

```js
{
	test: /\.less/,
    use: [
    	'style-loader', 
    	'css-loader',
    	'less-loader',
    	{
    		loader: 'postcss-loader',
    		options: {
				plugins: () => {
					require('autoprefixer')({//需要兼容的浏览版本
						overrideBrowserslist: ['last 2 version', '>1%', 'ios 7']
					})
				}
			}
    	}
    ]
}
```

4）将css单独打包，按入口打包或者统一打成一个包（和style-loader冲突）并且添加contenthash 可以用**mini-css-extract-plugin**，它是个插件，它的loader属性可以作为loader配置。

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
//...
{
	test: /\.less/,
    use: [
    	{
            loader: MiniCssExtractPlugin.loader
        },
    	'css-loader',
    	'less-loader',
    	{
    		loader: 'postcss-loader',
    		options: {
				plugins: () => {
					require('autoprefixer')({//需要兼容的浏览版本
						overrideBrowserslist: ['last 2 version', '>1%', 'ios 7']
					})
				}
			}
    	}
    ]
}
plugins: [
    new MiniCssExtractPlugin({
		filename: 'dist/css/[name].[contenthash:8].css',
	}),
]
```

更多详情：https://www.jianshu.com/p/91e60af11cc9

## 2.解决js中的图片引入问题

1）简单版 **file-loader**

```js
			{
				test: /\.(png|jpg|gif)$/i,
				use: 'file-loader',
			}
```

2) 对图片进行base64转换 url-loader

```js
			{
				test: /\.(png|jpg|gif)$/i,
				use: {
						loader: 'url-loader',
						options: {
							limit: 1024,
						},
					},
			}
```

## 3.解决html中图片引入问题

**html-withimg-loader**

```js
			{
				test: /\.(htm|html)$/i, //解决html中图片引用问题
				loader: 'html-withimg-loader'
			},
```

## 4.ES6转ES5

**babel-loader**

```
npm install babel-loader @babel/core @babel/preset-env 
```

```js
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          cacheDirectory: true, //开发时候可以启用缓存加快打包速度
          //presets属性告诉Babel要转换的源码使用了哪些新的语法特性，presets是一组Plugins的集合。
          presets: [
				[
					"@babel/preset-env",
					{ 
						targets: {//配置兼容的版本
							ie: '11'
						}
					}
				]
			],
			plugins: [
				[
					"@babel/plugin-transform-runtime", //polyfill和此插件只能用一个，个人测试比polyfill打出来的包要小一点。它对辅助函数的复用，解决转译语法层时出现的代码冗余。解决转译api层出现的全局变量污染
						{
							corejs: 3, //通过下载core-js包也能拥有polyfill的方法
						}
				]
			]
        }
      }
    }
  ]
}
```

更为详细的请直接看babel官方

https://babeljs.io/docs/en/next/babel-plugin-transform-runtime

## 5.eslint校验

eslint-loader,去官方生成配置比较方便

## 6.变量挂载全局

内联loader   expose-loader暴露到window;

webpack.providePlugin插件,注入到每个模块;

CDN引入

```js
rules: [
	{
		test: require.resolve('jquery'),
		use: 'expose-loader?$'
	}
]

new webpack.ProvidePlugin({ 
	$: 'jquery'
})
```

## 7.其他文件类型loader

想引入什么文件类型直接去搜 xxx-loader吧

# 4.自定义mock服务

devServer还提供了一个很好用的东西就是proxy,它可以设置代理ip，可以设置判断条件来决定是否转发请求，并且它还提供了express实例给我们使用，利用这一特性，我们就可以构建自己的本地mock服务。

```js
//webpack.config.js
devServer: {
		port: 8094,//本地服务启动的端口
		proxy: {
			'/api': {
				target: 'http://localhost:8081', //代理服务器地址
				secure: false,
				changeOrigin: true,
			}
		},
		before: function (app, server, compiler) {//app就是express实例，用法同express
			app.get(/^\/api\//, function (req, res, next) { //对api请求进行拦截
				if (req.url.match(/^(\/api\/)/) === null) {
					return next(); 
				}
				try {//读取我们的本地mock文件，post为true就使用mock,否则使用服务器数据
					let url = `.${req.url.replace(/\/api/, '/mock')}`;
					delete require.cache[require.resolve(url)]; 
					let { data, post } = require(url);
					if (post) { 
						res.json(data())
					} else {
						next()
					}
				} catch (error) {
					next();
				}

			});
		}
	},
```

```js
//server.js 
//仿真实服务器
let express = require('express');
let app = express();
app.use(function (req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS');
    res.header("Access-Control-Allow-Headers", "X-Requested-With");
    res.header('Access-Control-Allow-Headers', 'Content-Type');
    next();
});
app.get(/^\//, (req, res) => {
    console.log(req.url)
    res.json(req.url)
});
app.listen('8081', () => {
    console.log('app start')
})
```

```js
//------ mock/login.js
module.exports = {
    post: true, //通过此处去判断当前请求是否用本地mock
    data(params) { //写成函数形式可以根据参数进行判断返回不同数据
        return {
            login: 'success',
            statu: 200
        }
    }
} 
```

# 5.常用API

API在自己写loader,plugin可能会用到，目前只用到过以下api,混个脸熟

async():提供异步回调

cacheable():是否缓存

addDependency(filename):添加到监听列表

# 6.webpack原理

简单来说就是根据配置文件，去读取入口文件，然后通过入口文件解析所有模块。用路径做键，里面的内容用函数包装起来，传入一些属性供内部使用。然后在全部保存到 installedModules 中去。

下面是webpack的简单实现

```js
//--bin/helloPack.js

#! /usr/bin/env node

let path = require('path');
let Compiler = require('../lib/Compiler');
let config = require(path.resolve('webpack.config.js'))
let compiler = new Compiler(config);
compiler.run()
```

```js
//--lib/Compiler.js

let path = require('path');
let {parse} = require('babylon');
let traverse = require('@babel/traverse').default;
let generate = require('@babel/generator').default;
let types = require('@babel/types');
let ejs = require('ejs');
let fs = require('fs');
const { SyncHook } = require('tapable')
module.exports = class Compiler {
    constructor(config) {
        this.config = config;
        //保存入口文件路径
        this.entry = config.entry;
        //保存所有模块依赖
        this.modules = {};
        this.entryId;
        this.assets = {};
        this.root = process.cwd();
        this.hooks = {
            entryOption: new SyncHook(),
            compile: new SyncHook(),
            afterCompile: new SyncHook(),
            afterPlugins: new SyncHook(),
            run: new SyncHook(),
            emit: new SyncHook(),
            done: new SyncHook()
        }
        let plugins = this.config.plugins;
        plugins.forEach(plugin => {
            plugin.apply(this)
        });
        this.callHook('afterPlugins')
    }
    callHook(lifeStyle) {
        this.hooks[lifeStyle].call();
    }
    run() {
        this.callHook('compile');
        //执行并创建文件的依赖关系
        this.buildModule(path.resolve(this.root, this.entry), true);
        this.callHook('afterCompile');

        //发射一个文件，打包后的文件
        this.emitFile();
        this.callHook('emit');
        this.callHook('done');
    }
    //构建模块
    buildModule(modulePath, isEntry) {
        //拿到模块内容和id
        let source = this.getSource(modulePath);
        let moduleName = './' + path.relative(this.root, modulePath);
        //保存入口ID
        if (isEntry) this.entryId = moduleName;
        //解析文件得到包装内容和路径key
        let { resourceCode, dependencies } = this.parse(source, path.dirname(moduleName));
        this.modules[moduleName] = resourceCode;
        dependencies.forEach(v => this.buildModule(path.join(this.root, v), false));
        
    }
    getSource(modulePath) {
        let source = fs.readFileSync(modulePath, 'utf8');
        let rules = this.config.modules.rules;
        for (let index = 0; index < rules.length; index++) {
            const { test, use} = rules[index];
            if (test.test(modulePath)) {
                let i = use.length - 1;
                while (i >= 0) {
                    source = require(use[i--])(source)
                };
                break;
            };
            
        }
        return source;
    } 
    parse(source, parentPath) {
        //babylon解析成ast语法树
        //babylon @babel/traverse @babel/generator   @babel/types
        let ast = parse(source);
        let dependencies = []; //依赖的数组
        traverse(ast, {
            CallExpression(p) {
                let node = p.node;
                if (node.callee.name === 'require') {
                    node.callee.name = '__webpack_require__';
                    let moduleName = node.arguments[0].value;
                    moduleName = moduleName + (path.extname(moduleName) ? '' : '.js');
                    moduleName = './' + path.join(parentPath, moduleName);
                    dependencies.push(moduleName);
                    node.arguments = [types.stringLiteral(moduleName)];
                }
            }
        });
        let resourceCode = generate(ast).code;
        return { resourceCode, dependencies };
    }
    emitFile() {
        let main = path.join(this.config.output.path, this.config.output.filename);
        let templateStr = this.getSource(path.resolve(__dirname, 'main.ejs'));
        let code = ejs.render(templateStr, {
            entryId: this.entryId,
            modules: this.modules
        });
        this.assets[main] = code;
        fs.writeFileSync(main, code);
    }
}
```

```ejs
//--lib/main.ejs    //出口模板
(function(modules) { // webpackBootstrap
 	var installedModules = {};

    function __webpack_require__(moduleId) {

 		// Check if module is in cache
 		if(installedModules[moduleId]) {
 			return installedModules[moduleId].exports;
 		}
 		// Create a new module (and put it into the cache)
 		var module = installedModules[moduleId] = {
 			i: moduleId,
 			l: false,
 			exports: {}
 		};

 		// Execute the module function
 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

 		// Flag the module as loaded
 		module.l = true;

 		// Return the exports of the module
 		return module.exports;
 	}
    return __webpack_require__(__webpack_require__.s = "<%-entryId%>");
})
({

   <%for(let key in modules){%>
       "<%-key%>": 
       (function(module, exports, __webpack_require__) {
           eval(`<%-modules[key]%>`)
       }),
   <%}%>
});
```

```json
//package.json //最后在里面配置bin和路径，然后 npm link 一下就可以全局使用了
...
"bin": "./bin/helloPack.js"
...
```

# 7.loader原理

**loader的特点**

- 每个loader都是无状态的，在模块转换之间保留状态
- 第一个loader要返回js脚本
- 每个loader都是单独的模块，只处理一件事，方便链式调用

**loader的执行顺序**

​	loader一般分为normal部分和pitch部分，normal参数为文件内容，pitch参数为剩余未执行loader的绝对路径。有下列loader，他们的配置顺序为 use: [loader3, loader2, loader1]。则他们的执行顺序为

​	loader3.pitch->2.pitch->1.pitch->resource->loader1.normal->2.normal->3.normal

如果pitch有返回值，则中断pitch到自身normal的后一个执行。例如 2.pitch 有返回值 则 执行顺序为 

loader3.pitch->loader2.pitch->loader3.normal。

​	也可以在loader配置中配置enforce: pre | post 来调整执行顺序。

**loader的简单实现 css-loader和style-loader**

```js
//css-loader
module.exports = function (source) {
    console.log('css-loader')
    let reg = /url\((.+?)\)/g;
    let pos = 0;
    let current;
    let arr = ['let list = [];'];
    while (current = reg.exec(source)) {
        let [martchUrl, g] = current;
        let last = reg.lastIndex - martchUrl.length;
        arr.push(`list.push(${JSON.stringify(source.slice(pos, last))});`);
        pos = reg.lastIndex;
        arr.push(`var a = require('${g}').default`);
        arr.push(`list.push("url:(" +a+")");`)
    }
    arr.push(`list.push(${JSON.stringify(source.slice(pos))})`);
    arr.push(`module.exports=list.join('')`);
    return arr.join('\r\n');
}
```

```js
let loaderUtils = require('loader-utils');
require('style-loader')
function loader(resource) {}
loader.pitch = function (remainingRequest) {
    //转换为相对路径，在这里webpack会帮我们依次引用为使用的loader和resource并执行。
    let url = loaderUtils.stringifyRequest(this, '!!' + remainingRequest);
    let str = `
        let content = require(${url});
        let style = document.createElement('style');
        style.innerHTML = content;
        document.body.appendChild(style);
        `
    return str;
}
module.exports = loader;
```



# 8.plugin原理

​	webpack会自动执行plugin里面的apply方法，webpack提供了几百个钩子函数，插件可以在apply方法里注册回调。

​	**plugin简单实现**

```js
//多生成一份文件用来描述文件信息的plugin
module.exports = class FileListPlugin {
    constructor(config) {
        this.filename = config.filename;
    }
    apply(compiler) {
        compiler.hooks.emit.tap('FileListPlugin', (compilation) => {
            let assets = compilation.assets;
            let content = `## 文件名    资源大小`;
            Object.entries(assets).forEach(([filename, statObj]) => {
                content += `-  ${filename}  ${statObj.size()}\r\n`;
            });
            assets[this.filename] = {
                source() {
                    return content;
                },
                size() {
                    return content.length
                }
            }
        })
    }
}
```

​	



# 9.webpack调试相关

**在浏览器中调试**（个人决定浏览器调试好用一点）

1.在浏览器输入 [chrome://inspect/#devices](chrome://inspect/#devices)

2.开启命令行，输入

`node --inspect-brk ./node_modules/webpack/bin/webpack.js --inline --progress`

**在vscode中调试**

1.ctrl + shift + d 

2.在点击 node.js Debug terminal

3.按往常那样开启webpack服务就行了 （例如：npm run build 或者npx webpack）