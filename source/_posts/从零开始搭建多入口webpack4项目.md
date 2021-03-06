---
title: 从零开始搭建多入口webpack4项目
date: 2019-04-29 15:23:56
categories: 
    - 前端开发
tags:
    - webpack
---

# 简介
+ 搭建基础项目架构
+ 开发中（dev-server）
+ 模块（module）配置
+ 生产环境的资源打包

# 搭建基础项目架构

首先，webpack4对于`Node.js`版本是有要求的：
> webpack4要求Node.js的最低版本为`6.11.5`  

如果你的Node.js版本过低并进行了升级，然后在项目运行时出现了茫茫多的关于`node-sass`的报错，则需要重建node-sass环境。运行指令：

```
npm rebuild node-sass
```

----


创建一个多入口项目，按照我们的现有项目，目录如下：

```
└ your-project
  ├ src
    ├ css
    ├ js
      ├ project1
        ├ detail.js
        └ index.js
      └ project2
        └ index.js
    └ tpl
      ├ project1
        ├ detail.ejs
        └ index.ejs
      └ project2
        └ index.ejs
  ├ webpack.config.js
  ├ webpack.production.config.js
  └ package.json
```

安装`webpack`和`webpack-cli`（webpack4需要配合webpack-cli使用，社区也有相关的绕过cli的解决方案，但不是很推荐）：

```
cnpm i webpack webpack-cli -D
```

新建默认配置入口`webpack.config.js`，虽然能看到webpack4在往0配置的方向发展，但是在大多数项目中的配置三板斧还是不能少的：

```js
let config = {
    entry: {
        // webpack4默认src/index.js为入口文件
        // 本次针对多入口项目做配置，第二节会去做入口文件的获取
    },
    output: {
        // 文件输出默认为dist/bundle.js
        // 第二节会去重新覆盖默认的输出项
    },
    module: {
        rules: [
            // module.rules在大多数情况仍然需要配置
            // 我们需要合适的loader去解析对应的文件格式（*.css, *.ejs, ...）
        ]
    },
    mode: 'development'
};

module.exports = config;
```

> `mode`为webpack4新增配置，默认值为`production`，根据字面意思可以猜到分别对应开发环境和生产环境，设置的模式会有对应的内部配置。该参数可以在cli中追加，也可以在config中配置（如上）。该参数在webpck4中为必需参数。

> 具体可以参考[**模式(mode)**](https://webpack.docschina.org/concepts/mode/)。

# 开发中（dev-server）

webpack4的开发环境搭建有两个推荐：`webpack-dev-server`（node + express）和`webpack-serve`（node + koa2）。

如果要沿用我们比较熟悉的`webpack-dev-server`（[传送门](https://github.com/webpack/webpack-dev-server)）进行开发环境的搭建，需要安装对应的beta版本：

```
cnpm i webpack-dev-server@next -D
```

而`webpack-serve`（[传送门](https://github.com/webpack-contrib/webpack-serve)）作为升级版本，功能更加强大（**也许也会有更多的坑，待踩**），使用`WebSockets`做HMR(Hot Module Replacement 模块热替换)。这次我们就来体验一下`webpack-serve`，安装依赖：

```
cnpm i webpack-serve -D
```
新建`serve.config.js`，安装依赖项：

```
cnpm i glob yargs koa-router -D
```

> **警告！**如果需要用cli命令启用服务的话，必需使用**CommonJS**规范语法。以下实例因为未使用CommonJS规范的原因只能使用node语法启动，因为我**懒得改了**。

配置我们需要的配置项，然后为了便利考虑在根路由搭建目录列表：

```js
// serve.config.js

const path = require('path');
// glob用更方便的规则匹配需要的文件
// 参考地址 https://github.com/isaacs/node-glob
const glob = require('glob');
const serve = require('webpack-serve');
const Router = require('koa-router'); // 引入koa-router，配置根路由的展示内容
const WebpackConfig = require('./webpack.config.js');

// 更多配置项参考 https://github.com/webpack-contrib/webpack-serve
const config = {
    open: true // 构建完成后是否自动在浏览器打开
};

// 新建路由，输出目录结构
let directory = new Router();
directory.get('/', async ctx => {
    // 拼接根路由的html DOM结构
    let html = `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"><ul>`;
    // 获取webpack入口文件，遍历生成目录结构
    let entries = WebpackConfig.entry;

    for (let key in entries) {
        if (key !== 'commons') {
            html += `<li><a href="/pages/${key}.html">${key}</a></li>`;
        }
    }

    html += `</ul>`;

    // 输出html结构
    ctx.body = html;
});

// 装载子路由
let router = new Router();
router.use('/', directory.routes(), directory.allowedMethods());

serve(config, {
    WebpackConfig,
    add: (app, middleware) => {
        middleware.webpack();
        middleware.content();

        // 加载路由中间件，路由**必需**作为最后一个中间件添加
        app.use(router.routes());
    }
})
.then((result) => {
    // to do ...
});

```

-----


在目录页的搭建时用到了webpack配置中的`entry`项（入口文件），接下来我们需要在`webpack.config.js`中获取我们项目的所有入口文件，并指定输出规则（`output`）：

```js
// webpack.config.js

// path参考 http://nodejs.cn/api/path.html
const path = require('path');
const glob = require('glob');

const SRC_PATH = path.resolve(__dirname, './src');

...

// 多入口项目获取入口对象
let entries = (entryPath => {
    let files = {},
        filesPath;

    // 传入的entryPath为src/js文件夹路径
    // 如果有不需要作为入口js获取的文件／文件夹，在options项添加ignore配置来进行筛除
    filesPath = glob.sync(entryPath + '/**/*.js', {
        // options ...  
    });

    filesPath.forEach((entry, index) => {
        // 获取并简化入口文件对象的键名
        let chunkName = path.relative(entryPath, entry).replace(/\.js$/i, '');

        files[chunkName] = entry;
    });

    // 返回多入口文件对象
    return files;
})(path.join(SRC_PATH, 'js'));

let config = {
    entry: entries,
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: 'js/[name].[hash:7].js'
    },
    ...
};

...
```

在`package.json`中写入启动脚本（**再次友情提示！建议使用CommonJS规范书写`serve.config.js`，以便使用推荐的cli命令！**）：

```json
{
    "scripts": {
        "dev": "node ./serve.config.js"
    }
}
```

# 模块（module）配置

> webpack4依然不支持把`css`和`html`作为模块，相关格式的loader配置仍然是必需项。    
> 根据社区的开发计划，webpack5也许会解决这个问题（把`css`和`html`视作模块读取）。

## 处理scss/css文件

在webpack4之前我们都是用`extract-text-webpack-plugin`([传送门](https://github.com/webpack-contrib/extract-text-webpack-plugin))来抽离已经被`sass-loader`、`css-loader`解析完成的css为**独立的外部css文件**，它的beta版本也对webpack4做了支持，但是仅支持`4.2.0`以下版本，因此我们应尽量选择版本向上兼容度更好的其他依赖。

**解决方案：**使用官方新推荐的`mini-css-extract-plugin`([传送门](https://github.com/webpack-contrib/mini-css-extract-plugin))作替代。

**注意：**loader的解析顺序为从后往前，顺序混乱可能会引起一些报错。

安装依赖项目：

```
cnpm i mini-css-extract-plugin css-loader postcss-loader node-sass sass-loader -D
cnpm i autoprefixer -D
```

配置如下：

```js
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

...
let config = {
    ...
    module: {
        rules: [
            {
                // 如果不希望把已经解析完成的css文件单独抽离引入，而是希望以style标签对的形式插入页面
                // 可以把MiniCssExtractPlugin.loader替换为style-loader
                test: /\.(c|sa|sc)ss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    {
                        loader: 'postcss-loader',
                        options: {
                            plugins: () => [
                                require('autoprefixer')({
                                    'browsers': ['> 1%', 'last 10 versions']
                                })
                            ]
                        }
                    },
                    'sass-loader'
                ]
            },
            ...
        ]
    },
    ...
    plugins: [
        // 指定抽离出的css文件的输出规则
        // 如果使用style-loader，不需要以下配置
        new MiniCssExtractPlugin({
            filename: 'css/[name].[hash:7].css'
        })
    ]
};
...
```

> module模块不再支持loader配置项，需要使用use项做升级

## ejs模版语言处理

正常情况下，使用我们之前的`ejs-compiled-loader`仍然可以正常解析ejs模版，但是在一些未知的条件下会出现this指针相关的报错（如pc_v1项目）。目前没有找到合适的替代依赖。

**解决方案：**根据热心网友提供的PR，可以暂时使用`ejs-zdm-loader`做应急策略。

安装依赖项：

```
cnpm i ejs-zdm-loader -D
```

配置如下：

```js
// webpack.config.js

...
let config = {
    ...
    module: {
        ...
        {
            test: /\.ejs$/i,
            use: 'ejs-zdm-loader'
        },
    },
    ...
};
...
```

## js公共模块抽离

在webpack4之前，对于公共依赖的抽离使用的是`CommonsChunkPlugin`，webpack4对于该插件已经做了**废弃处理**，相对的，提供了更便捷的[SplitChunksPlugin](https://webpack.docschina.org/plugins/split-chunks-plugin/)作为新的解决方案。相关api在官方文档可以查看。

配置如下：

```js
// webpack.config.js
...
let config = {
    ...
    optimization: {
        splitChunks: {
            chunks: 'all',
            minChunks: 1,
            name: true,
            cacheGroups: {
                commons: {
                    name: 'commons',
                    minChunks: 2,
                    // minSize设置为0只为demo体现抽离效果，建议保持默认的30000
                    // 该设置过小会可能导致网络请求无必要的增加一次
                    minSize: 0
                }
            }
        }
    },
    ...
};
...
```

## 其他plugins

到目前为止我们已经做了入口文件的获取，各种文件格式的解析抽离，并指定了输出规则，但是并没有实际的输出html文件。

仍然使用`html-webpack-plugin`，升级到支持webpack4的最新版本。

安装依赖：

```
cnpm i html-webpack-plugin -D
```

**注意：**`html-webpack-plugin`每次只能生成一个.html文件，所以在多入口项目中需要对入口文件进行遍历。

配置如下：

```js
// webpack.config.js
const webpack = require('webpack');
...
const HtmlWebpackPlugin = require('html-webpack-plugin');
...
let config = {
    ...
    plugins: [
        ...
        // 把$变量暴露到全局
        new webpack.ProvidePlugin({
            $: 'jquery',
            jQuery: 'jquery',
            'window.jQuery': 'jquery'
        })
    ]
};

let pages = Object.keys(entries);
pages.forEach(item => {
    config.plugins.push(new HtmlWebpackPlugin({
        showErrors: false,
        filename: path.join(__dirname, `/dist/pages/${item}.html`),
        template: path.join(__dirname, `/src/tpl/${item}.ejs`),
        chunks: ['commons', item]
    })); 
});
...
```

> 到目前为止，我们已经完成了开发环境的所有配置，可以使用`npm run dev`来看看项目是否已经正常启动。完整配置可以参考**[这里](https://github.com/hinapudao/webpack4/blob/master/webpack.config.js)**。

# 生产环境的资源打包

万里长征最后一步，配置生产环境，压缩静态资源，用chunk hash代替编译hash。

> chunk hash可以根据文件内容生成hash，在文件内容无更改时不会生成新hash。而编译hash会在每次项目编译的时候重新生成，所以不建议使用在生产环境。

为了方便书写，免去变量区分，我们新建一个`webpack.production.config.js`来配置生产环境。在webpack4下，我们不再需要`UglifyJsPlugin`来压缩js代码，只需要配置[**模式(mode)**](https://webpack.docschina.org/concepts/mode/)为`production`即可。

配置如下：

```js
// webpack.production.config.js
...
let config = {
    ...
    mode: 'production',
    ...
};
...
```

> 由于静态资源的编译顺序为**js依赖编译** > **编译css** > **抽离css资源为单独文件** > **编译并压缩js**，所以以上配置并不会压缩css文件，所以我们需要单独处理一下生产环境的css资源。

webpack4下我们尝试使用[optimize-css-assets-webpack-plugin](https://github.com/NMFR/optimize-css-assets-webpack-plugin)来压缩css文件。

首先安装依赖：

```
cnpm i optimize-css-assets-webpack-plugin -D
```

配置如下：

```js
// webpack.production.config.js
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
...
let config = {
    ...
    optimization: {
        minimizer: [
            new OptimizeCSSAssetsPlugin({
                cssProcessorOptions: {
                    // postcss-loader解析css过程中已经添加了autoprefixer兼容
                    // 此处需要设置为false
                    autoprefixer: false
                }
            })
        ]
    },
    ...
};
...
```

最后，我们需要用chunk hash代替编译hash。

配置如下：

```js
// webpack.production.config.js
...
let config = {
    ...
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: 'js/[name].[chunkhash:7].js',
        publicPath: './'
    },
    ...
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name].[contenthash:7].css'
        }),
        ...
    ]
};
...
```

添加webpack打包命令：

```
{
    "scripts": {
        "dev": "node ./serve.config.js",
        "build": "rm -rf dist && npx webpack --config webpack.production.config.js"
    }
}
```

> 完整的生产环境配置可以参考[这里](https://github.com/hinapudao/webpack4/blob/master/webpack.production.config.js)。
