---
title: webpack优化二：打包分析
date: 2019-05-22 04:03:33
categories: 
    - 前端开发
tags:
    - webpack
---

## 背景

webpack4.x新增了更加便利的`splitChunks`（[文档翻译](https://github.com/hinapudao/SplitChunksPlugin)），可以帮助我们对项目的js进行提取分割，以便更好的长效缓存。但是提取规则是否合适，我们希望提取的模块是否按照预期进行，是否有可视化的插件可以明确的展示给我们？

## 工具

[`webpack-bundle-analyzer`](https://github.com/webpack-contrib/webpack-bundle-analyzer)是一个可以把webpack打包输出的bundles以可视化的块状图展示每个modules的输出大小及来源路径。

## 安装

```shell
npm install --save-dev webpack-bundle-analyzer
```

或者

```shell
yarn add -D webpack-bundle-analyzer
```

## 配置

webpack配置：

```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
			analyzerMode: 'server',
			analyzerHost: '127.0.0.1',
			analyzerPort: 8889,
			reportFilename: 'report.html',
			defaultSizes: 'parsed',
			openAnalyzer: true,
			generateStatsFile: false,
			statsFilename: 'stats.json',
			statsOptions: null,
			logLevel: 'info'
		})
  ]
}
```

添加脚本：

```json
scripts: {
	"analyz": "NODE_ENV=production npm_config_report=true npm run build"
}
```

在`npm run build`之后会展示资源的打包情况。