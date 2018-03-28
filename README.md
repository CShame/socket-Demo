# socket-Demo


#### 服务端
1.建立项目文件，安装node和express框架
2.安装socket.io

```
npm install socket.io
```

3.引用文件

```
var express = require('express');
var app = express();
var path = require('path');
var http=require('http').Server(app);
var io=require('socket.io')(http);
```

4.定义在线人数和用户列表

```
//在线用户
var onlineUser={};
var onlineCount=0;
```

5.建立连接

```
io.on('connection',function (socket) {
    console.log('新用户登录');

    //监听新用户加入
    socket.on('login',function (obj) {
        socket.name=obj.userid;
        //检查用户在线列表
        if(!onlineUser.hasOwnProperty(obj.userid)){
            onlineUser[obj.userid]=obj.username;
            //在线人数+1
            onlineCount++;
        }
        //广播消息
        io.emit('login',{onlineUser:onlineUser,onlineCount:onlineCount,user:obj});
        console.log(obj.username+"加入了聊天室");
    })

    //监听用户退出
    socket.on('disconnect',function () {
        //将退出用户在在线列表删除
        if(onlineUser.hasOwnProperty(socket.name)){
            //退出用户信息
            var obj={userid:socket.name, username:onlineUser[socket.name]};
            //删除
            delete onlineUser[socket.name];
            //在线人数-1
            onlineCount--;
            //广播消息
            io.emit('logout',{onlineUser:onlineUser,onlineCount:onlineCount,user:obj});
            console.log(obj.username+"退出了聊天室");
        }
    })

    //公聊：监听用户发布聊天内容
    socket.on('message', function(obj){
        //向所有客户端广播发布的消息
        io.emit('message', obj);
        console.log(obj.username+'说：'+obj.content);
    });

    //私聊：服务器接受到私聊信息，发送给目标用户  
    socket.on('private_message', function (from,to,msg)  
    {  
        var target = onlineUser[to.userid];  
        if(target)  
        {  
            console.log('emitting private message by ', from.username, ' say to ',to.username, msg);  
            target.emit("pmsg",from,to,msg);	//发送给目标人
            target.emit("pmsg",to,from,msg);	//发送给自己
        }
    });  
})
```

6.最后监听端口开启服务，一个简单的后台就搭建起来啦

```
http.listen(4000, function(){
    console.log('listening on *:4000');
});
```

#### 客户端
1.首先下载并引用socket.io.js

2.连接服务器

```
//连接socket后端服务器
var socket=io.connect("ws://127.0.0.1:4000");
//通知用户有用户登录
socket.emit('login',userInfo);
//监听新用户登录
socket.on('login',function (o) {
    updateMsg(o, 'login');
});
//监听用户退出
socket.on('logout',function (o) {
    updateMsg(o, 'logout');
});
//群聊:发送消息
socket.on('message',function (obj) {
    //do something
})
//发送私聊消息
socket.emit("private_message",from,to,msg);
// 监听私聊消息  
socket.on('pmsg', function (from,to,msg)  
{  
  	alert("get private message...from:"+from +"to:"+to +"msg:"+msg);  
}); 
```

接下来就是写界面了，就不上代码了。

好了，基本就是这样，搞定！！