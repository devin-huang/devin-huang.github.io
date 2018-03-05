---
layout:     post
title:     Vue全家桶+webpack搭建商城项目
subtitle:   Vue2+VueRouter2+Webpack+Axios 构建项目实战2017重制版
date:       2018-03-01
author:     BY Devin
header-img: img/post-bg-ioses.jpg
catalog: true
tags:
    - JQUERY
    - JAVASCRIPT
---


## 前言

在网络上寻找了一些编写JQUERY方式，也观看了官网给的方式，感觉官网的方式后并不太喜欢，还是偏向对象的方式。但是搜索资料中并没有找到1.选项中函数参数传入方式2.API的编写方式；所以参考别人的格式制作的到规范；

**1.methods参数是通过call() or apply()传到前台；**
**2.API需要图2图4对应的；**

## 正文

下面拆分讲解：
1.AMD规范的目的都是为了JavaScript模块化开发
![](/img/pubilc/plugin-0.jpg)

2.定义一个对象，其中$[PUBLIC]代表插件本身
![](/img/pubilc/plugin-1.jpg)

3.初始化定义属性选项
![](/img/pubilc/plugin-2.jpg)

4.对象的原型链方法的介绍及API
![](/img/pubilc/plugin-3.jpg)

5.实例化插件
![](/img/pubilc/plugin-4.jpg)

```
(function (factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD. Register as anonymous module.
    define(['jquery'], factory);
  } else if (typeof exports === 'object') {
    // Node / CommonJS
    factory(require('jquery'));
  } else {
    // Browser globals.
    factory(jQuery);
  }
}(function ($) {

    'use strict';
    /* Defind Plugin */
    var _PLUGIN_    = 'poster';
    var _VERSION_   = '1.0.0';

    if ( $[ _PLUGIN_ ] && $[ _PLUGIN_ ].version > _VERSION_ )
    {
        return;
    }

    /* Init Object */
    $[_PLUGIN_] = function($container,options){
        var self = this ;
        this.container = $container ;
        this.options   = options ;
        this.api       = [ "init", "destroy" ];
        this.init();
        return this ;
    };

    $[_PLUGIN_].version = _VERSION_;

    $[_PLUGIN_].defaults = {
        "name"    : "Devin" ,
        "isClass" : false , 
        "doFn"    : null ,
    }; 

    /* Prototype Function */
    $[_PLUGIN_].prototype = {
        init : function (){
            var _this_ = this ;
        },
        destroy : function()
        {
            this.container.empty().html(this.defalutHtml);
        },    
        defaultDOM : function(dir){
        },  
        _api_: function()
        {
            var self_ = this,
                api = {};

            $.each( this.api,
                function( i )
                {
                    var fn = this;
                    api[ fn ] = function()
                    {   
                        var re = self_[ fn ].apply( self_, arguments );
                        return ( typeof re == 'undefined' ) ? api : re;
                    };
                }
            );
            return api;
        },
    };

    /* The jQuery plugin */
    $.fn[_PLUGIN_] = function(options){

        options = $.extend( true, {} , $[_PLUGIN_].defaults , options );
        return this.each(function(){
            $(this).data( _PLUGIN_, new $[_PLUGIN_]( $(this), options )._api_() );
        });

    };

}));
```
参考文章：

[如何写插件](http://blog.csdn.net/caiwanxia1/article/details/53079408)

[dotdotdot](http://dotdotdot.frebsite.nl/)
