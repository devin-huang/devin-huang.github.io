---
layout:     post
title:      websocket实时聊天室
subtitle:   基于Node.js、socket.io实现
date:       2017-04-26
author:     BY Devin
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - Node.js
    - websocket
---

# 前言

Socket.IO 是一个为实时应用提供跨平台实时通信的库。socket.io 旨在使实时应用在每个浏览器和移动设备上成为可能，模糊不同的传输机制之间的差异。

CLIENT (INDEX.HTML)

```
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io.connect('http://localhost');
  socket.on('news', function (data) {
	console.log(data);
	socket.emit('my other event', { my: 'data' });
  });
</script>
```

`var socket = io.connect('http://localhost');` 与 http://localhost 本地服务器建立连接并赋值给 socket 对象，如果是与其他服务器建立连接则只需将 connect() 参数修改为服务器的地址即可，这里 socket.io 客户端和服务器位于同一个服务器上，可以简写为 `var socket = io.connect();`

SERVER (APP.JS)

```
var app = require('express')()
  , server = require('http').createServer(app)
  , io = require('socket.io').listen(server);

server.listen(80);

app.get('/', function (req, res) {
  res.sendfile(__dirname + '/index.html');
});

io.sockets.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
	console.log(data);
  });
});
```

`io = require('socket.io').listen(server);` 将`socket.io`绑定到服务器上，于是任何连接到该服务器的客户端都具备了实时通信功能。

`io.sockets.on('connection', function (socket) { ... })` 的作用是服务器监听所有客户端，并返回该新连接对象，接下来我们就可以通过该连接对象（socket）与客户端进行通信。

服务器与客户端都有 emit 和 on 这两个函数，是socket.io的核心函数，通过 emit 和 on 可以轻松地实现服务器与客户端之间的双向通信。

emit ：发射事件或者说触发事件，第一个参数为事件名，第二个参数为要发送的数据，第三个参数为回调函数

on ：用来监听emit发射的事件，第一个参数为要监听的事件名，第二个参数为一个匿名函数用来接收对方发来的数据，该匿名函数的第一个参数为接收的数据

socket.io 提供了三种默认的事件（客户端和服务器都有）：connect 、message 、disconnect ;当与对方建立连接后自动触发 connect 事件，当收到对方发来的数据后触发 message 事件（通常为 socket.send() 触发），当对方关闭连接后触发 disconnect 事件。

最后需要注意在服务器端区分以下三种情况：

socket.emit() ：向建立该连接的客户端广播
socket.broadcast.emit() ：向除去建立该连接的客户端的所有客户端广播
io.sockets.emit() ：向所有客户端广播，等同于上面两个的集合

# 常用API

socketIO.join()                            // 将客户加入到对应的房间中

socket.leave()                             // 从房间中删除客户

socketIO.to()                              // 仅传播到给定空间中的客户

socket.send();                             // 发送一个信息


# DEMO

## 功能

1.使用NodeJS插件socket.io实现实时聊天室

2.侦听每个服务端进入与离开（含时间）消息的提示

3.侦听信息、链接发送成功与否

    
## 启动服务器    

搭建环境[Node.js](https://nodejs.org/zh-cn/)

假设根目录为 : E/socket.io-Demo

```
e:
cd socket.io-Demo
npm install express socket.io
node app.js
```

浏览器输入：http://localhost:3000 

## 效果展示

房间列表

![](/img/pubilc/websocket-room-list.jpg)

聊天室

![](/img/pubilc/websocket-main.jpg)


