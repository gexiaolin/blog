---
title: webpack优化一：构建测速
date: 2019-05-06 21:50:37
categories: 
    - 前端开发
tags:
	- webpack
---

## 背景

随着项目体积的逐渐庞大，webpack的构建速度逐渐变得"年老体衰"，开发环境构建的时间都够打把斗地主了，实在是难以忍受。虽然文档上有那么大一个optimization，但是应该从哪儿着手优化却让我有点找不着北。所以想着如果有什么plugin可以打印出构建过程中每一步的耗时就好了。

## 工具

`Speed Measure Plugin` ([speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin)) 是为`webpack`量身定做的测速插件，它可以让你知道构建过程的每一步的耗时明细。`Speed Measure Plugin`需要node 6.0以上，支持所有webpack版本（1 - 4）。

## 安装

```shell
npm install --save-dev speed-measure-webpack-plugin
```

或者

```shell
yarn add -D speed-measure-webpack-plugin
```

## 使用

```js
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
 
const smp = new SpeedMeasurePlugin();
 
const webpackConfig = smp.wrap({
  plugins: [
    new MyPlugin(),
    new MyOtherPlugin()
  ]
});
```

## 配置项

配置位置如下：

```js
const smp = new SpeedMeasurePlugin(options);
```

###  `options.disable`

Type:  `Boolean`

Default:  `false`

值设置为`true`时该插件不工作。

### `options.outputFormat`

Type: `String|Function`

Default:  `human`

打印格式，可选值如下：

+ `"json"`
+ `"human"`
+ `"humanVerbose"`
+ `Function`

### `options.outputTarget`

Type:  `String|Function`
Default:  `console.log`

### `options.pluginNames`

Type:  `Object`
Default:  `{}`

默认情况下，SMP使用`plugin.constructor.name`打印plugin速度，但是如果某些plugin不生效或者你想通过指定字段代理plugin的名称，可以配置该项：

```js
const uglify = new UglifyJSPlugin();
const smp = new SpeedMeasurePlugin({
  pluginNames: {
    customUglifyName: uglify
  }
});
 
const webpackConfig = smp.wrap({
  plugins: [
    uglify
  ]
});
```

