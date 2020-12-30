---
title: webSocket
date: 2020-3-30 15:04:20
tags:
	
categories: SpringBoot
---

## websocket与http  

http协议是用在应用层的协议，他是基于tcp协议的，http协议建立链接也必须要有三次握手才能发送信息。http链接分为短链接，长链接，短链接是每次请求都要三次握手才能发送自己的信息。即每一个request对应一个response。长链接是在一定的期限内保持链接。保持TCP连接不断开。客户端与服务器通信，必须要有客户端发起然后服务器返回结果。客户端是主动的，服务器是被动的。 
WebSocket是HTML5中的协议， 他是为了解决客户端发起多个http请求到服务器资源浏览器必须要经过长时间的轮训问题而生的，他实现了多路复用，他是全双工通信。在webSocket协议下客服端和浏览器可以同时发送信息。

## 基于STOMP协议的WebSocket

基于spring官网简单的spring-boot的[stomp-demo](https://spring.io/guides/gs/messaging-stomp-websocket/)

POM文件引入

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>webjars-locator-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>sockjs-client</artifactId>
            <version>1.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>stomp-websocket</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>bootstrap</artifactId>
            <version>3.3.7</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.1.1-1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```

WebSocketConfig 文件

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    /**
     * 配置信息代理节点
     * @param config
     */
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        //配置一个代理节点 用不同的前缀来区分    ：   请求数据
        config.enableSimpleBroker("/queue","/topic","/user");
        //设置前端发送消息的前缀    ：    返回数据前缀
        config.setApplicationDestinationPrefixes("/app");
    }

    /**
     * 帮我们封装Websocket  URL的封装转换的一些列操作
     * @param registry
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        //addEndpoint（）设置服务端端口
        //withSockJS()处理浏览器兼容问题
        //setAllowedOrigins() 允许请求的类型
        registry.addEndpoint("/gs-guide-websocket").setAllowedOrigins("*").withSockJS();
    }
}
```

controller 控制类

```java
@Controller
public class GreetingController {

    // MessageMapping   专门来处理Stomp 的请求     配置代理的URL
    @MessageMapping("/hello")
    //SendTo 处理消息监听的代理   给前端返回消息
    @SendTo("/topic/greetings")
    public Greeting greeting(HelloMessage message) throws Exception {
        Thread.sleep(1000); // simulated delay
        return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");
    }

    private Logger logger = LoggerFactory.getLogger(this.getClass());
    //spring提供的发送消息模板
    @Autowired
    private SimpMessagingTemplate messagingTemplate;
    
    /**
     * 单独聊天请求
     */
    @MessageMapping(value = "/along")
    public void along(HelloMessage message) {
        System.out.println(message.getName());

        Response response = new Response();
        response.setName("我真的爱死你了");
        response.setMessage("你好啊"+ message.getName());
        /**
         *   如上，这里虽然我还是用了认证的信息得到用户名。但是，其实大可不必这样，因为 convertAndSendToUser 方法可以指定要发送给哪个用户。
         *   也就是说，完全可以把用户名的当作一个参数传递给控制器方法，从而绕过身份认证！convertAndSendToUser
         *   方法最终会把消息发送到 /user/第一个参数/第二个参数    目的地上。
         */
        //发送到订阅的 /queue
        messagingTemplate.convertAndSendToUser(message.getName(),"/queue/alone",response);
        //messagingTemplate.convertAndSendToUser(user.getName(), "/queue/shouts", shout);

    }
}
```

实体类

```java
public class HelloMessage {

    private String name;

    public HelloMessage() {
    }

    public HelloMessage(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

public class Response {
    private String  name ;
    private  String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

public class HelloMessage {

    private String name;

    public HelloMessage() {
    }

    public HelloMessage(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

前端静态文件

app.js

```js

var stompClient = null;

function setConnected(connected) {
    $("#connect").prop("disabled", connected);
    $("#disconnect").prop("disabled", !connected);
    if (connected) {
        $("#conversation").show();
    }
    else {
        $("#conversation").hide();
    }
    $("#greetings").html("");
}
/*链接*/
function connect() {
    var socket = new SockJS('/gs-guide-websocket');
    stompClient = Stomp.over(socket);
    stompClient.connect(
        {
          //   name: 'tests' // 携带客户端信息
         },
        function (frame) {
        setConnected(true);
        console.log('Connected: ' + frame);
        stompClient.subscribe('/topic/greetings', function (greeting) {
            showGreeting(JSON.parse(greeting.body).content);
        });
        //一对一监听
            var userId = "test";
            stompClient.subscribe('/user/test/queue/alone',function(response){
                showGreeting(JSON.parse(response.body).content);
            })
    });
}
/*断开连接*/
function disconnect() {
    if (stompClient !== null) {
        stompClient.disconnect();
    }
    setConnected(false);
    console.log("Disconnected");
}

/**
 * 发送消息
 */
function sendName() {
    stompClient.send("/app/hello", {}, JSON.stringify({'name': $("#name").val()}));
    stompClient.send("/app/along", {}, JSON.stringify({'name': $("#name").val()}));
}

/**
 * 显示消息
 * @param message
 */
function showGreeting(message) {
    $("#greetings").append("<tr><td>" + message + "</td></tr>");
}

$(function () {
    $("form").on('submit', function (e) {
        e.preventDefault();
    });
    $( "#connect" ).click(function() { connect(); });
    $( "#disconnect" ).click(function() { disconnect(); });
    $( "#send" ).click(function() { sendName(); });
});
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello WebSocket</title>
    <link href="/webjars/bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <link href="/main.css" rel="stylesheet">
    <script src="/webjars/jquery/jquery.min.js"></script>
    <script src="/webjars/sockjs-client/sockjs.min.js"></script>
    <script src="/webjars/stomp-websocket/stomp.min.js"></script>
    <script src="/app.js"></script>
</head>
<body>
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
    enabled. Please enable
    Javascript and reload this page!</h2></noscript>
<div id="main-content" class="container">
    <div class="row">
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="connect">WebSocket connection:</label>
                    <button id="connect" class="btn btn-default" type="submit">Connect</button>
                    <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">Disconnect
                    </button>
                </div>
            </form>
        </div>
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="name">What is your name?</label>
                    <input type="text" id="name" class="form-control" placeholder="Your name here...">
                </div>
                <button id="send" class="btn btn-default" type="submit">Send</button>
            </form>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <table id="conversation" class="table table-striped">
                <thead>
                <tr>
                    <th>Greetings</th>
                </tr>
                </thead>
                <tbody id="greetings">
                </tbody>
            </table>
        </div>
    </div>
</div>
</body>
</html>
```

css

```css
body {
    background-color: #f5f5f5;
}
#main-content {
    max-width: 940px;
    padding: 2em 3em;
    margin: 0 auto 20px;
    background-color: #fff;
    border: 1px solid #e5e5e5;
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    border-radius: 5px;
}
```

