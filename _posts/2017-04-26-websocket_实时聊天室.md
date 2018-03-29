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


# DEMO功能：

1.使用NodeJS插件socket.io实现实时聊天室

2.侦听每个服务端进入与离开（含时间）消息的提示

3.侦听信息、链接发送成功与否

    
## 启动服务器：    

搭建环境[Node.js](https://nodejs.org/zh-cn/)

假设根目录为 : E/websocket

```
e:
cd websocket 
npm install express socket.io
node app.js
```

浏览器输入：http://localhost:3000 展示

SERVER (APP.JS)

```
//依赖模块
var express = require('express');
var path = require('path');
var IO = require('socket.io');
var router = express.Router();

var app = express();
var server = require('http').Server(app);
app.use(express.static(path.join(__dirname, 'public')));
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'hbs');

// 创建socket服务
var socketIO = IO(server);
// 房间用户名单
var roomInfo = {};

socketIO.on('connection', function (socket) {
  // 获取请求建立socket连接的url
  // 如: http://localhost:3000/room/room_1, roomID为room_1
  var url = socket.request.headers.referer;
  var obj = { intoTime : formatTime(new Date()) };

  var splited = url.split('/');
  obj.roomID = splited[splited.length - 1];   // 获取房间ID
  obj.user = '';

  socket.on('join', function (userName) {
    if(!userName){
      var date = new Date();
      obj.user = +date;
    }else{
      obj.user = userName;      
    }
    // 将用户昵称加入房间名单中
    if (!roomInfo[obj.roomID]) {
      roomInfo[obj.roomID] = [];
    }
    roomInfo[obj.roomID].push(obj.user);

    socket.join(obj.roomID);
    // 通知房间内人员
    socketIO.to(obj.roomID).emit('sys', obj.user + ' 加入了房间 '+ obj.intoTime + '//add', roomInfo[obj.roomID]);  
    console.log(obj.user + '加入了' + obj.roomID + '时间' + obj.intoTime );
  });

  socket.on('leave', function () {
    socket.emit('disconnect');
    //socket.broadcast.emit('disconnect');与上效果一样，只是该方法是除去自己能看到;
  });

  //监听出退事件
  socket.on('disconnect', function () {
    // 从房间名单中移除
    var index = roomInfo[obj.roomID].indexOf(obj.user);
    if (index !== -1) {
      roomInfo[obj.roomID].splice(index, 1);
    }

    socket.leave(obj.roomID); 
    socketIO.to(obj.roomID).emit('sys', obj.user + ' 退出了房间 '+ formatTime(new Date()) + '//remove' , roomInfo[obj.roomID]);
    console.log(obj.user + '退出了' + obj.roomID + '时间' + formatTime(new Date()) );
  });


  socket.on('message', function (msg) {
    if(msg) obj.state = 1;
    // 验证如果用户不在房间内则不给发送
    if (roomInfo[obj.roomID].indexOf(obj.user) === -1) {  
      return false;
    }
    socketIO.to(obj.roomID).emit('msg', obj, msg);
  });

});

// 路由配置
router.get('/room/:roomID', function (req, res) {
  var roomID = req.params.roomID;
  res.render('room', {
    roomID: roomID,
    users: roomInfo[roomID]
  });
});

app.use('/', router);

server.listen(3000, function () {
  console.log('server listening on port 3000');
});

function formatTime(time){
  var year  = time.getFullYear() ,
    month = time.getMonth() + 1 ,
    day   = time.getUTCDate() ,
    hours = time.getHours() ,
    minutes = time.getMinutes() ,
    seconds = time.getSeconds() ;
  return year + '/' + month + '/' + day + ' '+ hours + ':' + minutes + ':' + seconds ;
}
```

CLIENT (INDEX.HTML)

```
/*--room.hbs---*/
<!DOCTYPE html>
<html>
<head lang="en">
  <meta charset="UTF-8">
  <title>{{roomID}}</title>
  <link rel="stylesheet" href="/css/style.css">
  <link rel="stylesheet" href="https://cdn.bootcss.com/font-awesome/4.6.3/css/font-awesome.min.css">
</head>
<body>
  <div class="dn-row">
    <div class="col-dn-6"> <label>昵称</label><span class="userName"></span> </div>
    <div class="col-dn-6"> <label>房间</label><span>{{roomID}}</span> </div>
    <div class="col-dn-6"> <label>在线人数</label><span class="count">{{users.length}}</span> </div>
    <div class="col-dn-6"> <label>在线用户</label><span class="users">{{users}}</span> </div>
  </div>

  <div class="msglog"></div>
  <textarea name="message" class="messageInput"></textarea>
  
  <div class="dn-row">
    <div class="col-dn-6"> <button class="btn btn-send" id="">发送信息 ( 或者按Enter键发送 )</button> </div>
    <div class="col-dn-6"> <button class="btn joinOrLeave">退出房间</button> </div>
  </div>

  <script src="/socket.io/socket.io.js"></script>
  <script src="/js/jquery.js"></script>
  <script>
    $(function () {
      // ----------设置昵称-------------
      var userName = '';
      while ($('.userName').text().trim() === '') {
        userName = prompt("请设置你的昵称","");
        if(!userName){
          var date = new Date();
          userName = +date;
        }
        $('.userName').text(userName);
      }

      // ---------创建连接-----------
      var socket = io();

      // 加入房间
      socket.on('connect', function () {
        socket.emit('join', userName);

        id =  socket.id;
        console.log(id);

        $('.disconnect').remove();
      });

      // 监听消息
      socket.on('msg', function (obj, msg) {
        var state   = ( obj.state ) ? 'fa-check-circle' : 'fa-times-circle';
        var isSelf  = (obj.user === userName) ? 'currenty' : '' ;
        var message = '' +
            '<div class="message">' +
            '  <span class="user">' + obj.user + ': </span>' +
            '  <span class="msg '+ isSelf + '">' + msg + '</span>' +
            '  <span class="time">' + obj.intoTime + '</span>' +
            '  <span class="fa ' + state + '"></span>' +
            '</div>';
        $('.msglog').append(message);
        // 滚动条保持最下方
        $('.msglog').scrollTop($('.msglog')[0].scrollHeight);  
      });

      // 监听系统消息
      socket.on('sys', function (sysMsg, users) {
        var msg     =  sysMsg.split('//');
        var message = '<div class="sysMsg ' + msg[msg.length-1] + '">' + msg[0] + '</div>';
        $('.msglog').append(message);

        $('.count').text(users.length);
        $('.users').text(users);
      });

      // 断开连接
      socket.on('disconnect', (reason) => {
        $('body').prepend('<p class="disconnect" style="color:red"> DISCONNECT:'+reason+'<br/><span></span></p>');
      });

      //正在连接中
      socket.on('reconnecting', (attemptNumber) => {
        if(attemptNumber>20){
          socket.on('reconnect_failed', () => {
              alert('failed')
          });
        }
        $('.disconnect > span').empty().text('reconnecting...'+attemptNumber);
      });

      //成功重连
      socket.on('reconnection', (isLink) => {
          console.log(isLink)
          if(isLink){
              console.log('OK');
              //重连后判断是否还是同一窗口(具体还没实操);
              if(id == socket.id){
                  console.log(id)                
              }
          }
      });

      // 发送消息
      $('.messageInput').bind('keydown', function(event) {
        if (event.which === 13) {
          event.preventDefault();
          var msg = $(this).val();
          $(this).val('');
          socket.send(msg);
        }
      });

      $(".btn-send").bind('click', function(event) {
          event.preventDefault();
          var msg = $('.messageInput').val();
          $('.messageInput').val('');
          socket.send(msg);
      });

      // 退出房间
      $('.joinOrLeave').click(function () {
        if ($(this).text() === '退出房间') {
          $(this).text('进入房间');
          socket.emit('leave');
          var msg = '你已经退出了房间,重新发言请点击"进入房间"';
          $('.msglog').append('<div class="sysMsg remove">'+msg+'</div>');
        } else {
          $(this).text('退出房间');
          socket.emit('join', userName);
        }

      });
    });
  </script>
</body>
</html>
```

# Index
```
/*--- index.html ---*/
<!DOCTYPE html>
<html>
<head lang="en">
  <meta charset="UTF-8">
  <title>Socket.io - DEMO</title>
</head>
<body>
  <div>
    <p>房间列表</p>
    <ul>
      <li id="room_1"><a href="/room/room_1" target="_blank">1号房间</a></li>
      <li id="room_2"><a href="/room/room_2" target="_blank">2号房间</a></li>
      <li id="room_3"><a href="/room/room_3" target="_blank">3号房间</a></li>
      <li id="room_4"><a href="/room/room_4" target="_blank">4号房间</a></li>
      <li id="room_5"><a href="/room/room_5" target="_blank">5号房间</a></li>
    </ul>
  </div>
</body>
</html>
```
