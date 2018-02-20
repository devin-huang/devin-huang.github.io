---
layout:     post
title:      websocket实时聊天室
subtitle:   基于Node.js、socket.io实现
date:       2017-04-26
author:     BY
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - Node.js
    - websocket
---

# 前言

功能:实时侦听用户接入或断开并某一用户发送信息时其他用户能即是看见的聊天室(包含发送时间 、断线重连提示、发送是否成功)

原理

* 搭建服务器域，然后客户端访问该域时服务端返回各类信息

* 服务端

    _发送obj内信息给所有人（含自身）_
    
    `socket.emit( 'message' , obj )`
    
    _发送obj内信息给所有人（除自身）_
    
    `socket.broadcast.emit( 'system' , obj )`

* 客户端

    _通过服务端的emit的obj返回数据到前端_
    
    `socket.on( 'message' , function(obj){} )`
    
# 启动服务器：    

搭建环境[Node.js](https://nodejs.org/zh-cn/)

假设根目录为 : e/websocket

```
e:
cd websocket 
npm install express socket.io
node app.js
```

服务端
app.js

```
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
  var obj = { time:getTime() };
  var splited = url.split('/');
  obj.roomID = splited[splited.length - 1];   // 获取房间ID
  obj.user = '';
  socket.on('join', function (userName) {
    if(!userName){
      // 当输入空值时默认给时间戳为用户名
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
    console.log(obj.roomInfo)
    socket.join(obj.roomID);    // 加入房间
    // 通知房间内人员
    socketIO.to(obj.roomID).emit('sys', obj.user + '加入了房间'+ obj.time, roomInfo[obj.roomID]);  
    console.log(obj.user + '加入了' + obj.roomID + '时间' + obj.time);
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
    socket.leave(obj.roomID);    // 退出房间
    socketIO.to(obj.roomID).emit('sys', obj.user + '退出了房间'+ obj.time, roomInfo[obj.roomID]);
    console.log(obj.user + '退出了' + obj.roomID + '时间' + obj.time);
  });
  // 接收用户消息,发送相应的房间
  socket.on('message', function (msg) {
    if(msg) obj.state = 1;
    // 验证如果用户不在房间内则不给发送
    if (roomInfo[obj.roomID].indexOf(obj.user) === -1) {  
      return false;
    }
    socketIO.to(obj.roomID).emit('msg', obj, msg);
  });
});
// 指定webscoket的客户端的html文件
router.get('/room/:roomID', function (req, res) {
  var roomID = req.params.roomID;
  // 渲染页面数据(见views/room.hbs)
  res.render('room', {
    roomID: roomID,
    users: roomInfo[roomID]
  });
});
app.use('/', router);
server.listen(3000, function () {
  console.log('server listening on port 3000');
});
var getTime=function(){
  var date = new Date();
  return date.getFullYear() + '-' + date.getMonth() + '-' + date.getDay() +
  ' ' + date.getHours() + ":" + date.getMinutes() + ":" + date.getSeconds();
}
```

# 客户端
index.html / room.hbs
```
/*--room.hbs---*/
<!DOCTYPE html>
<html>
<head lang="en">
  <meta charset="UTF-8">
  <title>{{roomID}}</title>
  <style>
    #msglog, #messageInput {
      border: 1px solid #ccc;
      width: 500px;
      height: 350px;
      overflow-y: auto;
      font-size: 14px;
    }
    #messageInput {
      height: 80px;
    }
    .message {
      line-height: 22px;
    }
    .message .user {
      padding-right: 5px;
      padding-left: 5px;
      color: brown;
    }
    .sysMsg {
      color: #c1bfbf;
      padding-right: 5px;
      padding-left: 5px;
      font-size: 12px;
    }
    #users {
      width: 490px;
      padding: 0 5px 5px;
    }
    .time{
      color:#999;
      float: right;
    }
    .success{
      color:white;
      padding:0px 2px;
      border-radius:2px;
      background-color:#45e945;
    }
    .fail{
      color:red;
    }
  </style>
</head>
<body>
  昵称: <span id="userName"></span> <br/>
  房间: {{roomID}} <br/>
  当前在线人数: <span id="count">{{users.length}}</span> <br/>
  在线用户:  <div id="users">{{users}}</div>
  <div id="msglog">
  </div>
  <textarea name="message" id="messageInput"></textarea>
  <br/>
  按Enter键发送
  <button id="joinOrLeave">退出房间</button>
  <script src="/socket.io/socket.io.js"></script>
  <script src="/js/jquery.js"></script>
  <script>
    $(function () {
      // ----------设置昵称-------------
      var userName = '';
      while ($('#userName').text().trim() === '') {
        userName = prompt("请设置你的昵称","");
        if(!userName){
          var date = new Date();
          userName = +date;
        }
        $('#userName').text(userName);
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
        var state = ( obj.state ) ? 'success' : 'fail';
        var message = '' +
            '<div class="message">' +
            '  <span class="user">' + obj.user + ': </span>' +
            '  <span class="msg">' + msg + '</span>' +
            '  <span class="time">' + obj.time + '</span>' +
            '  <span class="' + state + '">' + state + '</span>' +
            '</div>';
        $('#msglog').append(message);
        // 滚动条保持最下方
        $('#msglog').scrollTop($('#msglog')[0].scrollHeight);  
      });
      // 监听系统消息
      socket.on('sys', function (sysMsg, users) {
        var message = '<div class="sysMsg">' + sysMsg + '</div>';
        $('#msglog').append(message);
        $('#count').text(users.length);
        $('#users').text(users);
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
      $('#messageInput').keydown(function (e) {
        if (e.which === 13) {
          e.preventDefault();
          var msg = $(this).val();
          $(this).val('');
          socket.send(msg);
        }
      });
      // 退出房间
      $('#joinOrLeave').click(function () {
        if ($(this).text() === '退出房间') {
          $(this).text('进入房间');
          socket.emit('leave');
          var msg = '你已经退出了房间,重新发言请点击"进入房间"';
          $('#msglog').append('<div class="sysMsg">'+msg+'</div>');
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
  <title>demo</title>
</head>
<body>
  欢迎您,骚年
  <div>
    <p>房间列表</p>
    <ul>
      <li id="room_1"><a href="/room/room_1" target="_blank">1号房间</a></li>
      <li id="room_2"><a href="/room/room_2" target="_blank">2号房间</a></li>
      <li id="room_3"><a href="/room/room_3" target="_blank">3号房间</a></li>
      <li id="room_4"><a href="/room/room_4" target="_blank">4号房间</a></li>
      <li id="room_5"><a href="/room/room_5" target="_blank">5号房间</a></li>
      <li id="room_6"><a href="/room/room_6" target="_blank">6号房间</a></li>
      <li id="room_7"><a href="/room/room_7" target="_blank">7号房间</a></li>
      <li id="room_8"><a href="/room/room_8" target="_blank">8号房间</a></li>
      <li id="room_9"><a href="/room/room_9" target="_blank">9号房间</a></li>
      <li id="room_10"><a href="/room/room_10" target="_blank">10号房间</a></li>
    </ul>
  </div>
</body>
</html>
```
