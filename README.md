# socket-Demo


#### �����
1.������Ŀ�ļ�����װnode��express���
2.��װsocket.io

```
npm install socket.io
```

3.�����ļ�

```
var express = require('express');
var app = express();
var path = require('path');
var http=require('http').Server(app);
var io=require('socket.io')(http);
```

4.���������������û��б�

```
//�����û�
var onlineUser={};
var onlineCount=0;
```

5.��������

```
io.on('connection',function (socket) {
    console.log('���û���¼');

    //�������û�����
    socket.on('login',function (obj) {
        socket.name=obj.userid;
        //����û������б�
        if(!onlineUser.hasOwnProperty(obj.userid)){
            onlineUser[obj.userid]=obj.username;
            //��������+1
            onlineCount++;
        }
        //�㲥��Ϣ
        io.emit('login',{onlineUser:onlineUser,onlineCount:onlineCount,user:obj});
        console.log(obj.username+"������������");
    })

    //�����û��˳�
    socket.on('disconnect',function () {
        //���˳��û��������б�ɾ��
        if(onlineUser.hasOwnProperty(socket.name)){
            //�˳��û���Ϣ
            var obj={userid:socket.name, username:onlineUser[socket.name]};
            //ɾ��
            delete onlineUser[socket.name];
            //��������-1
            onlineCount--;
            //�㲥��Ϣ
            io.emit('logout',{onlineUser:onlineUser,onlineCount:onlineCount,user:obj});
            console.log(obj.username+"�˳���������");
        }
    })

    //���ģ������û�������������
    socket.on('message', function(obj){
        //�����пͻ��˹㲥��������Ϣ
        io.emit('message', obj);
        console.log(obj.username+'˵��'+obj.content);
    });

    //˽�ģ����������ܵ�˽����Ϣ�����͸�Ŀ���û�  
    socket.on('private_message', function (from,to,msg)  
    {  
        var target = onlineUser[to.userid];  
        if(target)  
        {  
            console.log('emitting private message by ', from.username, ' say to ',to.username, msg);  
            target.emit("pmsg",from,to,msg);	//���͸�Ŀ����
            target.emit("pmsg",to,from,msg);	//���͸��Լ�
        }
    });  
})
```

6.�������˿ڿ�������һ���򵥵ĺ�̨�ʹ������

```
http.listen(4000, function(){
    console.log('listening on *:4000');
});
```

#### �ͻ���
1.�������ز�����socket.io.js

2.���ӷ�����

```
//����socket��˷�����
var socket=io.connect("ws://127.0.0.1:4000");
//֪ͨ�û����û���¼
socket.emit('login',userInfo);
//�������û���¼
socket.on('login',function (o) {
    updateMsg(o, 'login');
});
//�����û��˳�
socket.on('logout',function (o) {
    updateMsg(o, 'logout');
});
//Ⱥ��:������Ϣ
socket.on('message',function (obj) {
    //do something
})
//����˽����Ϣ
socket.emit("private_message",from,to,msg);
// ����˽����Ϣ  
socket.on('pmsg', function (from,to,msg)  
{  
  	alert("get private message...from:"+from +"to:"+to +"msg:"+msg);  
}); 
```

����������д�����ˣ��Ͳ��ϴ����ˡ�

���ˣ����������������㶨����