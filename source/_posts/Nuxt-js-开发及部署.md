---
title: Nuxt.js 开发及部署
date: 2019-10-12 14:20:04
categories: 
    - 前端开发
tags:
    - Nuxt.js
---

## Nuxt.js 是什么

Nuxt.js 是一个基于 Vue.js 的通用应用框架，预设了利用 Vue.js 开发服务端渲染的应用所需要的各种配置，提供异步数据加载、中间件、布局支持等。

与传统的 `csr` (client-side-render) 相比，Nuxt.js封装的 `ssr` (server-side-render) 解决方案可以更好的支持 seo ，且白屏时间可减少大约 50% ，极大的提升用户的访问体验。

## 项目目录结构

```
·
├── README.md
├── nuxt.config.js             # nuxt配置，文档地址：https://zh.nuxtjs.org/guide/configuration
├── package-lock.json
├── package.json
├── server                     # express服务
└── src                        # 开发目录
    ├── app.html               # html模版
    ├── assets                 # 公共资源
    ├── components             # 页面开发子组件
    ├── layouts                # 组件布局(https://zh.nuxtjs.org/guide/views#%E5%B8%83%E5%B1%80)
    │   ├── default.vue        # 默认模版
    │   └── error.vue          # 错误页面模版
    ├── middleware             # 中间件(https://zh.nuxtjs.org/guide/routing#%E4%B8%AD%E9%97%B4%E4%BB%B6)
    ├── pages                  # 页面开发主文件夹
    ├── plugins                # 自定义插件(https://zh.nuxtjs.org/guide/plugins)
    ├── static                 # 静态资源，根路由'/'指向该文件夹
    └── store                  # vuex
```

为了符合前端开发习惯，我们把通过 Nuxt.js cli 构建的目录结构做了一些微小的调整：前端相关开发内容放入了 `src` 文件夹，同时把 nuxt config 的开发默认路径改为 src 下：

```js
// next.config.js

module.exports = {
  srcDir: 'src/'
};
```

## 本地开发

> 注意，在 `Vue` 组件的生命周期中，`beforeCreate` 和 `created` 会在 **客户端和服务端调用**，其他生命周期仅在客户端调用。

### server

默认 host 为 `localhost` , 端口号为 `3000` ，可在 `nuxt.config.js` 中覆盖默认配置：

```js
module.exports = {
  server: {
    port: 8080,
    host: process.env.NODE_ENV === 'production' ? '0.0.0.0' : 'local.xxx.com'
  }
};
```

如果需要扩展/添加 express 中间件，可以修改 `server/index.js` ：

```js
const myMiddleWare = require('myMiddleWare');

const app = express();
app.use(myMiddleWare);
```

### layouts

`src/layouts` 文件夹存放默认及自定义布局模版，`default.vue` 为默认通用模版，`error.vue`  为错误页面模版（接收上下文 `error` 对象）。

我们也可以自定义布局模版，通过 `<nuxt/>` 标签获取开发组件内容。比如我们新增 `layouts/example.vue`:

```js
<template>
  <div>
    <div>example header</div>
    <nuxt/>
    <div>example footer</div>
  </div>
</template>
```

在开发组件中使用自定义布局模版：

```js
<template>
  <!-- 开发组件内容 -->
</template>

<script>
export default {
  layout: 'example' // 指定渲染的布局模版
}
</script>
```

### error

我们可以通过编辑 `layouts/error.vue` 文件来定制自己的错误页面。

> 虽然此文件放在 layouts 文件夹中, 但应该将它看作是一个 页面(page)。

`error.vue` 接收上下文的 `error` 对象，改对象参数应该包含两个字段，分别为 `statusCode` （错误码）及 `message` （错误信息）:

```js
<template>
  <div class="container">
    <h1 v-if="error.statusCode === 404">页面不存在</h1>
    <h1 v-else>应用发生错误异常：{{error.message}}</h1>
    <nuxt-link to="/">首 页</nuxt-link>
  </div>
</template>
```

### router

项目支持常用的 `vue router` 路由跳转，使用方式也很简单，只需要使用 `<nuxt-link to=""></nuxt-link>` 标签代理 `a` 标签，`to` 属性内应放置项目结构内已有的相对路由。

同样的，我们也可以在不希望以 `vue router` 方式跳转的地方使用 `a` 标签。

### plugin

插件放置在 `src/plugin/` 目录，会在 Vue 应用程序之前执行，接收 `context` 上下文作为第一个参数。

插件的执行环境分三种：仅在客户端执行、仅在服务端执行、在客户端和服务端都执行。具体的置顶执行环境方式如下配置：

1、通过命名格式指定执行环境：

+ `x.client.js` 仅在客户端执行
+ `x.server.js` 仅在服务端执行
+ `x.js` 在客户端和服务端双端执行

```js
// nuxt.config.js

module.exports = {
  plugin: [
    '~/plugins/plugin.client.js'
  ]
};
```

2、通过 ssr 字段置顶执行环境：

```js
// nuxt.config.js

module.exports = {
  plugin: [
    {
      src: '~/plugins/plugin.js',
      // ssr: true 注意，ssr 字段即将弃用，改为 mode 字段
      mode: 'server'
    }
  ]
};
```
插件的具体使用方法：[传送门](https://zh.nuxtjs.org/guide/plugins)

### middleware

中间件放置在 `src/middleware/` 目录，接收 `context` 上下文作为第一个参数。中间件会在路由改变时调用：

+ 在 `nuxt.config.js` 中配置，会在每个路由改变时调用
+ 也可不做全局配置，改为 `layouts` 或者自定义开发组件内调用

中间件的具体使用方法：[传送门](https://zh.nuxtjs.org/guide/routing#%E4%B8%AD%E9%97%B4%E4%BB%B6)

### store

Vuex 状态树在 Nuxt.js 内核做了实现，支持两种 `store` 方式：

+ 模块方式：`src/store/` 目录下的每个 `.js` 文件会被转换为状态树 **指定命名的子模块**
+ Classic方式：传统 Vuex 语法使用，不建议，根据构建 warning 提示，会在3.0版本移除（目前为2.9.x）

参考地址：[传送门](https://zh.nuxtjs.org/guide/vuex-store)

### asyncData 方法

`asyncData` 方法会在组件（限于页面组件）每次加载之前被调用。它可以在 **服务端** 或 **路由更新** 之前被调用，接收 context 上下文作为第一个参数。Nuxt.js 会将 `asyncData` 返回的数据融合组件 `data` 方法返回的数据一并返回给当前组件。

> 该方法在 Vue 实例化之前调用，因此不能通过 `this` 获取 Vue 实例

### fetch 方法

fetch 方法会在渲染页面前被调用，作用是填充状态树 (store) 数据，接收 context 上下文作为第一个参数。与 asyncData 方法类似，不同的是它不会设置组件的数据。例：

```js
export default {
  fetch ({ store, params, $axios }) {
    store.commit('getUrl', 'url');
  }
};
```

### validate 方法

`validate` 可以在动态路由对应的页面组件中配置一个校验方法用于校验动态路由参数的有效性。接收 context 上下文作为第一个参数。例：

```js
validate({ params, query }) {
  return /^\d+$/.test(params.id);
}
```

### context 上下文对象

在上面的介绍中，我们一直提到上下文对象，它包含 Vue 根实例，Vue Router 路由，Vuex 状态树，error 错误方法，redirect 跳转方法等等内容，可以参考[这里](https://zh.nuxtjs.org/api/context/)。

### Caching Components 组件缓存

Vue 由于创建组件实例和 Virtual DOM 节点的成本，它无法与纯粹基于字符串的模板（php）的性能相匹配。在 SSR 性能至关重要的情况下，合理地利用缓存策略可以大大缩短响应时间并减少服务器负载。

安装依赖：

```shell
npm i @nuxtjs/component-cache -D
```

新增组件缓存配置：

```js
// nuxt.config.js
module.exports = {
  modules: [
    ['@nuxtjs/component-cache', {
      max: 10000,
      maxAge: 1000 * 60 * 60
    }]
  ]
};
```

+ 可缓存组件必须定义唯一 `name` 选项
+ 不应该缓存组件的情况
  + 可能拥有依赖 `global` 数据的子组件
  + 具有在渲染 `context` 中产生副作用的子组件

## 部署

### 静态应用部署

以下命令生成应用的静态目录和文件：

```shell
npm run generate
```

适合范围：spa 中小型应用，可控的动态路由页面。详细信息参考[这里](https://zh.nuxtjs.org/guide/commands)

### 服务端渲染应用部署

以下命令进行项目编译构建，然后启动服务：

```shell
nuxt build
nuxt start
```

详细信息参考[这里](https://zh.nuxtjs.org/guide/commands)

> 部署方式在官方文档有详细描述，不做过多解释。

### 使用 pm2 进行服务进程管理

在我们实际开发环境中，直接 `nuxt start` 跑起来的项目可能并不能满足我们的需求，原因包含且不限于：无法后台运行，无法热重启等等。

使用 pm2 的原因：

+ 可以内建负载均衡（使用 Node cluster 集群模块）
+ 后台运行
+ 0 秒停机重载，我理解大概意思是维护升级的时候不需要停机.
+ 停止/重启不稳定的进程（避免无限循环或内存溢出）


#### 安装

全局安装 pm2 ：

```shell
sudo npm i -g pm2
```

#### pm2 管理配置

新增 `process.json` :

```json
{
    "apps": [{
        "name": "nuxt",
        "script": "./server/index.js",
        "args": "--kill-timeout 10000",
        "watch": false,
        "ignore_watch": ["node_modules", "build", "logs"],
        "out_file": "logs/nuxt_out.log",
        "error_file": "logs/nuxt_error.log",
        "max_memory_restart": "1G",
        "env": {
            "NODE_ENV": "production"
        },
        "log_date_format": "YYYY-MM-DD HH:mm Z",
        "instance_var": "INSTANCE_ID",
        "exec_mode": "cluster",
        "instances": "0",
        "autorestart": true
    }]
}
```

大概介绍几个比较重要的配置项：

+ exec_mode：默认为 `fork` 模式，单服务进程，无法热重启。这里我们修改为 `cluster` 模式，内建负载均衡
+ instances：进程数，可手动设置，0 或 'max' 会根据服务器核数分配进程数
+ autorestart：设置为 true , 在内存溢出或其他极端情况下重启服务

#### 运行服务

我们可以执行以下命令运行我们的 Nuxt.js 项目：

```shell
pm2 start process.json
```

然后执行 `pm2 list` 可以看到各个服务进程的详细信息。

执行以下命令进行热重启：

```shell
pm2 reload nuxt # 使用process.json里的name配置进行服务管理
```

#### startOrGracefulReload 实现符合 node.js 特性的热重启

如果按照之前 reload 的方法进行热重启，且用户刷新的够快够频繁，还是会在重启过程中看到两三秒的502页面，原因在于 node.js 服务的单线程 **异步** 非阻塞I/O模型。异步的特性导致集群中的某一个服务并没有完全启动，但是代码已经执行结束，导致 pm2 直接进行下一线程的服务重启工作，因此我们选择使用 `startOrGracefulReload ` 作为热重启手段。

首先对 `server/index.js` 做一些改造：

```js
const express = require('express');
const app = express();

// ...

let server = app.listen(port, host);
process.on('SIGINT', () => {
  server.close();
})
```

> 注意，express()本身不是一个server实例，但是express().listen()会返回server实例，因此我们在调用close()方法的时候要注意调用对象是否正确

执行以下命令以完成符合 node.js 特性的热重启：

```shell
pm2 startOrGracefulReload process.json
```