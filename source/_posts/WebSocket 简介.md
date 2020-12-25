---
title: WebSocket 简介
categories: web前端
date: 2020-04-20 16:41:09
tags:
- javascript
---

## websocket是什么？

`websocket` 是一种网络通信协议，我们都知道 `http` 协议，`http` 协议只能从客户端主动发起，不能从服务端推送数据到客户端，今天我们讲的 `websocket` 就是一种不仅能从客户端发送数据到服务端，也可以主动从服务的推送数据给客户端的一种协议。

### Http 协议 和 Websocket 区别

#### Http:

`HTTP` 是单向的，客户端发送请求，服务器发送响应。举例来说，当客户端向服务器发送请求时，该请求以 `HTTP`或 `HTTPS`的形式发送，在接收到请求后，服务器会将响应发送给客户端。每个请求都与一个对应的响应相关联，在发送响应后客户端与服务器的连接会被关闭。每个 `HTTP` 或 `HTTPS` 请求每次都会新建与服务器的连接，并且在获得响应后，连接将自行终止。 `HTTP `是在`TCP`之上运行的无状态协议，`TCP` 是一种面向连接的协议，它使用三向握手方法保证数据包传输的传递并重新传输丢失的数据包。

`HTTP` 可以运行在任何可靠的面向连接的协议（例如`TCP`，`SCTP`）的上层。当客户端将`HTTP`请求发送到服务器时，客户端和服务器之间将打开`TCP`连接，并且在收到响应后，`TCP`连接将终止，每个`HTTP`请求都会建立单独的`TCP`连接到服务器，例如如果客户端向服务器发送10个请求，则将打开10个单独的`HTTP`连接。并在获得响应后关闭。

#### Websocket:

`WebSocket` 是双向的，在客户端-服务器通信的场景中使用的全双工协议，与 `HTTP` 不同，它以`ws://`或`wss://`开头。它是一个有状态协议，这意味着客户端和服务器之间的连接将保持活动状态，直到被任何一方（客户端或服务器）终止。在通过客户端和服务器中的任何一方关闭连接之后，连接将从两端终止。并且 `WebSocket`  是可以跨域的！！！

让我们以客户端-服务器通信为例，每当我们启动客户端和服务器之间的连接时，客户端-服务器进行握手随后创建一个新的连接，该连接将保持活动状态，直到被他们中的任何一方终止。建立连接并保持活动状态后，客户端和服务器将使用相同的连接通道进行通信，直到连接终止。

新建的连接被称为 `WebSocket`。一旦通信链接建立和连接打开后，消息交换将以双向模式进行，客户端-服务器之间的连接会持续存在。如果其中任何一方（客户端服务器）宕掉或主动关闭连接，则双方均将关闭连接。



**何时使用WebSocket**

- 即时`Web`应用程序：即时`Web`应用程序使用一个`Web`套接字在客户端显示数据，这些数据由后端服务器连续发送。在 `WebSocket` 中，数据被连续推送/传输到已经打开的同一连接中，这就是为什么 `WebSocket` 更快并提高了应用程序性能的原因。 例如在交易网站或比特币交易中，这是最不稳定的事情，它用于显示价格波动，数据被后端服务器使用Web套接字通道连续推送到客户端。
- 游戏应用程序：在游戏应用程序中，你可能会注意到，服务器会持续接收数据，而不会刷新用户界面。屏幕上的用户界面会自动刷新，而且不需要建立新的连接，因此在`WebSocket`游戏应用程序中非常有帮助。
- 聊天应用程序：聊天应用程序仅使用`WebSocket`建立一次连接，便能在订阅户之间交换，发布和广播消息。它重复使用相同的`WebSocket`连接，用于发送和接收消息以及一对一的消息传输。



**不能使用 WebSocket 的场景**

如果我们需要通过网络传输的任何实时更新或连续数据流，则可以使用 `WebSocket`。如果我们要获取旧数据，或者只想获取一次数据供应用程序使用，则应该使用 `HTTP` 协议，不需要很频繁或仅获取一次的数据可以通过简单的`HTTP`请求查询，因此在这种情况下最好不要使用 `WebSocket`



### WebSocket 常用API介绍

- webSocket构造函数

```javascript
let ws = new WebSocket('ws://localhost:8080');
```

执行上面语句之后，客户端就会与服务器进行连接。

- webSocket的状态(readyState)

readyState属性返回实例对象的当前状态，总共有四种状态。

```javascript
CONNECTING：值为0, 正在连接
OPEN：值为1，连接成功
CLOSING：值为2，连接正在关闭
CLOSED：值为3，连接已经关闭
```

- websocket.onopen（连接成功之后的回调函数）

实例对象的 `onopen` 属性，用于指定连接成功后的回调函数。

```javascript
ws.onopen = function () {
      ws.send('Hello Server!');
}
```

如果要指定多个回调函数，可以使用`addEventListener`方法。

```javascript
ws.addEventListener('open', function (event) {
      ws.send('Hello Server!');
});
```

- websocket.onclose（关闭之后调用的方法）

实例对象的`onclose`属性，用于指定连接关闭后的回调函数。

```javascript
ws.onclose = function(event) {
     console.log('onclose')
}
```

- websocket.onmessage（接受到服务器数据之后的回调函数）

```javascript
ws.onmessage = function(event) {
  // 获取数据event.data
  var data = event.data;
  // 处理数据
};
```

- websocket.send（向服务器端发送数据）

```javascript
ws.send('your message')
```

- websocket.onerror（报错时调用的方法）

```javascript
ws.onerror = function(event) {
 // handle error event
};
```



### App 端场景封装的websocket

```javascript
const requestSocket = (params, Cb) => {
    if(socket && socket.readyState === 1){
        socket.send(JSON.stringify(params))
        socker.onmessage = (msg) => {
            callBack(JSON.parse(msg.data))
        }
    }else{
        getSocket(params, Cb)
    }
}

const getSocket = (params, Cb) => {
    if(typeof(Websocket) === 'undefined'){
        console.log('当前浏览器不支持websocket')
    }else{
        socket = new WebSocket(configUrl)
        socket.openen = () => {
              console.log('websocket已经打开')
              let tokenParams = {} //密钥入参
              socket.send(JSON.stringify(tokenParams)) //send 为字符串，需要转换下

               socket.onmessage = (msg) => {
                    let data = JSON.parsm(mes.data) //获取服务器消息事件并转回对象
                    //根据密钥是否过期进行逻辑判断跳转
               }
        
               heart_timer(defaultParams, Cb) //保持心跳
        }
        
        //关闭事件
        socket.onclose = () => {
            console.log('socket 关闭')
            clearInterval(timer)
            timer = null
            reconnet() //重新连接
        }
        
         //关闭事件
        socket.onclose = () => {
            console.log('socket 发生错误')
            clearInterval(timer)
            timer = null
        }
    }
}

const reconnet = () => {
    if(lockReconnect){
        return
    }
    lockReconnect = true
    
    setTimeout(() => {
        getSocket()
        lockReconnect = false
    },2000)
}

const heart_time = (params, Cb) => {
    timer = setInterval(() => {
        socket.send(JSON.stringify(params))
        socket.onmessage = (msg) => {
          if(Cb){
              Cb(msg)
          }
        }
    }, 50000)
}
```

