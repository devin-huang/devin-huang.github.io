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
---

## 前言

使用Vue全家桶+webpack一步步讲解商城项目，vue商城展示效果[Vue商城](https://github.com/devin-huang/devin-vue2)

![](/img/pubilc/vue-shop.jpg)

## Vue-cli

安装vue-cli

`npm install -g vue-cli`

创建项目

`vue init webpack devin-vue2`

提示装vue-router (yes) , Use ESLint to lint your code? (yes)  unit (N) e2e (N)

使用vue-cli快速搭建项目整体环境，生成后主要对src文件夹内容经行修改，修改为符合开发的格式，具体请参考下图

![](/img/pubilc/path.jpg)

## vuex状态管理模式

[什么是Vuex?](https://vuex.vuejs.org/zh-cn/intro.html)

[Vuex属性对象](https://www.cnblogs.com/kbnet/p/6938693.html)

本商城项目中`src/store`使用了Vuex

* State      : 储存初始化数据

* Getters    : State 数据二次处理

* Mutations  : 数据进行计算的方法,页面中触发方式this.$store.commit('mutationName')触发Mutations方法改变state的值

* Actions    : 类似Mutations,页面中触发方式this.$store.dispatch(actionName)，与Mutations区别在于异步函数使用Actions

## Vue运行

运行主要入口文件为`src/main.js`

![](/img/pubilc/process.jpg)

参考文章：

[vue-cli+webpack搭建](https://github.com/devin-huang/vue-demo-cnodejs)
