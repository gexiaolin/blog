---
title: 解决 npm install 时的 permission denied 问题
date: 2019-12-18 18:11:58
categories: 
    - 前端开发
tags:
    - npm
---

## 问题场景

当我们在某个搭建好的项目中执行 `npm install` 时，有时会出现以下类似报错：

```shell
npm ERR! code 1
npm ERR! Command failed: /usr/local/bin/git clone --depth=1 -q -b fix/https-via-http-proxy git://github.com/contentful/axios.git /Users/xxx/.npm/_cacache/tmp/git-clone-e4d5168e
npm ERR! /Users/xxx/.npm/_cacache/tmp/git-clone-e4d5168e/.git: Permission denied
npm ERR!

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/xxx/.npm/_logs/2017-11-30T21_24_50_080Z-debug.log
```

## 原因分析

`npm` 在执行 `install` 命令时，会尝试搜索本地缓存（位于 `~/.npm/cacache` ），如果缓存命中，请求返回304，然后从本地缓存仓库解压缓存依赖包。

如果之前使用过 `sudo` 命令 `install` 过该依赖，则缓存的依赖压缩包的权限为 `root` ，当前用户权限不足，从而返回以上报错。

## 解决方案

### 提供当前用户权限

该方案属于修改用户权限的hack方案，如果对权限较敏感，请谨慎操作

执行以下命令，递归指定文件夹及内部文件权限到当前用户：

```shell
sudo chown -R $USER:$GROUP ~/.npm
sudo chown -R $USER:$GROUP ~/.config
```

### 尝试清除本地缓存

```shell
sudo npm cache clean -f
```
