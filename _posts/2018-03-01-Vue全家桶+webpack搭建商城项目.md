---
layout:     post
title:     Vue全家桶+webpack搭建商城项目
subtitle:   Vue + VueRouter + Webpack + Axios
date:       2018-03-01
author:     BY Devin
header-img: img/post-bg-ioses.jpg
catalog: true
tags:
    - Vue
    - Axios
    - VueRouter
    - Webpack
    - vuex
---

## 前言

使用Vue全家桶+webpack一步步讲解商城项目，vue商城展示效果[Vue商城](https://github.com/devin-huang/devin-vue2)

![](/img/pubilc/vue-shop.jpg)

## Vue-cli

安装vue-cli

`npm install -g vue-cli`

## 搭建初始

安装依赖

`npm install axios vuex`

axios配置文件

src/utils/http.js 用于封装axios异步GET/POST/DELETE等方式获取数据

Vuex配置文件

![](/img/pubilc/vuex.png)

**注意必须在`src/main.js`挂载store ， 配置API的端口设置`config/dev.env.js`添加 `API_ROOT: "http://localhost:4009"`**

## vuex状态管理模式

**在大型项目中多个组件共享状态时，单向数据流的简洁性很容易被破坏，把数据共享抽取出来，以一个全局单例模式管理(通俗解释： 全局改变数据，共享到每个组件 )；**

本商城项目中`src/store`使用了Vuex

* State      : 储存初始化数据或后端获取的数据

* Getters    : State 数据二次处理

* Mutations  : 数据进行计算的方法,页面中触发方式this.$store.commit('mutationName')触发Mutations方法改变state的值

* Actions    : 类似Mutations,页面中触发方式this.$store.dispatch(actionName)，与Mutations区别在于异步函数使用Actions

创建项目

`vue init webpack Shop-vue2`

提示装vue-router (yes) , Use ESLint to lint your code? (yes)  unit (N) e2e (N)

使用vue-cli快速搭建项目整体环境，生成后主要对src文件夹内容经行修改，修改为符合开发的格式，具体请参考下图

![](/img/pubilc/path.jpg)

## vue全家桶

* `axios`Vue2.0异步方式请求数据方式

* `vue-awesome-swiper`轮播图插件

* `vue-lazyload`懒加载

* `vue-router`路由

* `vuex`状态管理

* `better-scroll`移动端滚动列表

* `lib-flexible`制作移动端适配的开源库

* `mint-ui`基于 Vue.js 的移动端组件库

* `voinc`基于Vue.js和ionic样式的UI框架，用于方便快速地搭建单页应用


## Vue运行

运行主要入口文件为`src/main.js`

![](/img/pubilc/process.jpg)

参考文章：

[vue-cli+webpack搭建](https://github.com/devin-huang/vue-demo-cnodejs)
