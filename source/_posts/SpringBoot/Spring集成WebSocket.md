---
title: Spring集成WebSocket
date: 2020-03-14 13:04:20
tags:
	- WebSocket
categories: SpringBoot

---

## 概念

### websocket与http 

http协议是用在应用层的协议，他是基于tcp协议的，http协议建立链接也必须要有三次握手才能发送信息。http链接分为短链接，长链接，短链接是每次请求都要三次握手才能发送自己的信息。即每一个request对应一个response。长链接是在一定的期限内保持链接。保持TCP连接不断开。客户端与服务器通信，必须要有客户端发起然后服务器返回结果。客户端是主动的，服务器是被动的。 
WebSocket是HTML5中的协议， 他是为了解决客户端发起多个http请求到服务器资源浏览器必须要经过长时间的轮训问题而生的，他实现了多路复用，他是全双工通信。在webSocket协议下客服端和浏览器可以同时发送信息。

### HTTP的长连接与websocket的持久连接

HTTP1.1的连接默认使用长连接（persistent connection），

即在一定的期限内保持链接，客户端会需要在短时间内向服务端请求大量的资源，保持TCP连接不断开。客户端与服务器通信，必须要有客户端发起然后服务器返回结果。客户端是主动的，服务器是被动的。

 在一个TCP连接上可以传输多个Request/Response消息对，所以本质上还是Request/Response消息对，仍然会造成资源的浪费、实时性不强等问题。

如果不是持续连接，即短连接，那么每个资源都要建立一个新的连接，HTTP底层使用的是TCP，那么每次都要使用三次握手建立TCP连接，即每一个request对应一个response，将造成极大的资源浪费。

 长轮询，即客户端发送一个超时时间很长的Request，服务器hold住这个连接，在有新数据到达时返回Response

websocket的持久连接 只需建立一次Request/Response消息对，之后都是TCP连接，避免了需要多次建立Request/Response消息对而产生的冗余头部信息。

Websocket只需要一次HTTP握手，所以说整个通讯过程是建立在一次连接/状态中，而且websocket可以实现服务端主动联系客户端，这是http做不到的。

## 基于STOMP协议的WebSocket

使用STOMP的好处在于，它完全就是一种消息队列模式，你可以使用生产者与消费者的思想来认识它，发送消息的是生产者，接收消息的是消费者。而消费者可以通过订阅不同的destination，来获得不同的推送消息，不需要开发人员去管理这些订阅与推送目的地之前的关系，spring官网就有一个简单的spring-boot的[stomp-demo](https://spring.io/guides/gs/messaging-stomp-websocket/),如果是基于springboot，大家可以根据spring上面的教程试着去写一个简单的demo。

 

## websocket底层的方法实现

添加webSocketConfig配置类

```java
/**
 * 开启WebSocket支持
 * Created by huiyunfei on 2019/5/31.
 */
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

添加webSocketServer服务端类

```java
package com.example.admin.web;
/**
 * Created by huiyunfei on 2019/5/31.
 */
 
@ServerEndpoint("/websocket/{sid}")
@Component
@Slf4j
public class WebSocketServer {
    //静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    private static int onlineCount = 0;
    //concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
    private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<WebSocketServer>();
 
    //与某个客户端的连接会话，需要通过它来给客户端发送数据
    private Session session;
 
    //接收sid
    private String sid="";
 
 
    */
/**
     * 连接建立成功调用的方法*//*
    @OnOpen
    public void onOpen(Session session, @PathParam("sid") String sid) {
        this.session = session;
        webSocketSet.add(this);     //加入set中
        addOnlineCount();           //在线数加1
        log.info("有新窗口开始监听:"+sid+",当前在线人数为" + getOnlineCount());
        this.sid=sid;
        try {
            sendMessage("连接成功");
        } catch (IOException e) {
            log.error("websocket IO异常");
        }
    }
    */
/**
     * 连接关闭调用的方法
     *//*
    @OnClose
    public void onClose() {
        webSocketSet.remove(this);  //从set中删除
        subOnlineCount();           //在线数减1
        log.info("有一连接关闭！当前在线人数为" + getOnlineCount());
    }
    */
/**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息*//*
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("收到来自窗口"+sid+"的信息:"+message);
        //群发消息
        for (WebSocketServer item : webSocketSet) {
            try {
                item.sendMessage(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    */
/**
     *
     * @param session
     * @param error
     *//*
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }
    */
/**
     * 实现服务器主动推送
     *//*
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }
    */
/**
     * 群发自定义消息
     * *//*
    public static void sendInfo(String message,@PathParam("sid") String sid) throws IOException {
        log.info("推送消息到窗口"+sid+"，推送内容:"+message);
        for (WebSocketServer item : webSocketSet) {
            try {
                //这里可以设定只推送给这个sid的，为null则全部推送
                if(sid==null) {
                    item.sendMessage(message);
                }else if(item.sid.equals(sid)){
                    item.sendMessage(message);
                }
            } catch (IOException e) {
                continue;
            }
        }
    }
    public static synchronized int getOnlineCount() {
        return onlineCount;
    }
    public static synchronized void addOnlineCount() {
        WebSocketServer.onlineCount++;
    }
    public static synchronized void subOnlineCount() {
        WebSocketServer.onlineCount--;
    }
    public static CopyOnWriteArraySet<WebSocketServer> getWebSocketSet() {
        return webSocketSet;
    }
}
```

添加对应的controller

```java
@Controller
@RequestMapping("/system")
public class SystemController {
    //页面请求
    @GetMapping("/index/{userId}")
    public ModelAndView socket(@PathVariable String userId) {
        ModelAndView mav=new ModelAndView("/socket1");
        mav.addObject("userId", userId);
        return mav;
    }
    //推送数据接口
    @ResponseBody
    @RequestMapping("/socket/push/{cid}")
    public Map pushToWeb(@PathVariable String cid, String message) {
        Map result = new HashMap();
        try {
            WebSocketServer.sendInfo(message,cid);
            result.put("code", 200);
            result.put("msg", "success");
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }
```

提供socket1.html页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"></meta>
    <title>Title</title>
</head>
<body>
hello world!
 
</body>
<script>
    var socket;
    if(typeof(WebSocket) == "undefined") {
        console.log("您的浏览器不支持WebSocket");
    }else{
        console.log("您的浏览器支持WebSocket");
        //实现化WebSocket对象，指定要连接的服务器地址与端口  建立连接
        //等同于
        index = new WebSocket("ws://localhost:8885/websocket/2");
        //socket = new WebSocket("${basePath}websocket/${cid}".replace("http","ws"));
        //打开事件
        index.onopen = function() {
            console.log("Socket 已打开");
            //socket.send("这是来自客户端的消息" + location.href + new Date());
        };
        //获得消息事件
        index.onmessage = function(msg) {
            console.log(msg.data);
            //发现消息进入    开始处理前端触发逻辑
        };
        //关闭事件
        index.onclose = function() {
            console.log("Socket已关闭");
        };
        //发生了错误事件
        index.onerror = function() {
            alert("Socket发生了错误");
            //此时可以尝试刷新页面
        }
        //离开页面时，关闭socket
        //jquery1.8中已经被废弃，3.0中已经移除
        // $(window).unload(function(){
        //     socket.close();
        //});
    }
</script>
</html>
```

浏览器debug访问 localhost:8885/system/index/1跳转到socket1.html，js自动连接server并传递cid到服务端，服务端对应的推送消息到客户端页面（cid区分不同的请求，server里提供的有群发消息方法）