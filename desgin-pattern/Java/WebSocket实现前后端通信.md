# WebSocket实现前后端通信



## 一、添加依赖

第一步由于springboot很好地集成了websocket，所以先在在pom.xml文件中引入依赖

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-websocket</artifactId>
 </dependency>
```



## 二、建立连接

第二步在前端界面使用websocket，也就是HTML文件中编写

```javascript
<script>
    var websocket = null;
    if('WebSocket' in window) {
        websocket = new WebSocket('ws://yesell.natapp1.cc/sell/webSocket');
    }else {
        alert('该浏览器不支持websocket!');
    }


    websocket.onopen = function (event) {
        console.log('建立连接');
    }
    
    websocket.onclose = function (event) {
        console.log('连接关闭');
    }
    
    websocket.onmessage = function (event) {
        console.log('收到消息:' + event.data)
        //所要执行的操作
    }
    
    websocket.onerror = function () {
        alert('websocket通信发生错误！');
    }
    
    window.onbeforeunload = function () {
        websocket.close();
    }

</script>
```



## 三、服务端接收和响应

第三步，一般我们是在controller层实现交互的，然而websocket的交互是在service层，

其中：

@ServerEndpoint("/webSocket")是定义了交互的地址

@OnOpen、@OnClose、@OnMessage这三个方法与前端的三个同名方法相互交互，在需要使用的位置调用方法如下，到这里基本写完了。

```java
@Component
@ServerEndpoint("/webSocket")
@Slf4j
public class WebSocket {
    private Session session;
    private static CopyOnWriteArraySet<WebSocket> webSocketSet=new CopyOnWriteArraySet<>();
    @OnOpen
    public void onOpen(Session session){
        this.session=session;
        webSocketSet.add(this);
        log.info("【websocket消息】有新的连接，总数：{}",webSocketSet.size());
    }
    @OnClose
    public void onClose(){
        webSocketSet.remove(this);
        log.info("【websocket消息】连接断开，总数：{}",webSocketSet.size());
    }
    @OnMessage
    public void onMessage(String message){
        log.info("【websocket消息】收到客户端发来的消息：{}",message);
    }
    public void sendMessage(String message){
        for(WebSocket webSocket:webSocketSet){
            log.info("【websocket消息】广播消息：{}",message);
            try {
                webSocket.session.getBasicRemote().sendText(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
@Slf4j
@Component
@ServerEndpoint(value = "/ws/{userId}")
public class WebSocketServerEndpoint {

    private Session session; //建立连接的会话
    private String userId; //当前连接用户id   路径参数
    
    /**
     * 存放存活的Session集合(map保存)
     */
    private static ConcurrentHashMap<String , WebSocketServerEndpoint> livingSession = new ConcurrentHashMap<>();
    
    /**
     *  建立连接的回调
     *  session 建立连接的会话
     *  userId 当前连接用户id   路径参数
     */
    @OnOpen
    public void onOpen(Session session,  @PathParam("userId") String userId){
        this.session = session;
        this.userId = userId;
        livingSession.put(userId, this);
        System.out.println("连接成功");
        log.info("----[ WebSocket ]---- 用户id为 : {} 的用户进入WebSocket连接 ! 当前在线人数为 : {} 人 !--------",userId,livingSession.size());
    }
    
    /**
     *  关闭连接的回调
     *  移除用户在线状态
     */
    @OnClose
    public void onClose(){
        livingSession.remove(userId);
        log.info("----[ WebSocket ]---- 用户id为 : {} 的用户退出WebSocket连接 ! 当前在线人数为 : {} 人 !--------",userId,livingSession.size());
    }
    
    @OnMessage
    public void onMessage(String message, Session session,  @PathParam("userId") String userId) {
        log.info("--------收到用户id为 : {} 的用户发送的消息 ! 消息内容为 : ------------------",userId,message);
        //sendMessageToAll(userId + " : " + message);
    }
    
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("----------------WebSocket发生错误----------------");
        log.error(error.getStackTrace() + "");
    }
    
    /**
     *  根据userId发送给用户
     * @param userId
     * @param message
     */
    public void sendMessageById(String userId, String message) {
        livingSession.forEach((sessionId, session) -> {
            //发给指定的接收用户
            if (userId.equals(session.userId)) {
                sendMessageBySession(session.session, message);
            }
        });
    }
    
    /**
     *  根据Session发送消息给用户
     * @param session
     * @param message
     */
    public void sendMessageBySession(Session session, String message) {
        try {
            session.getBasicRemote().sendText(message);
        } catch (IOException e) {
            log.error("----[ WebSocket ]------给用户发送消息失败---------");
            e.printStackTrace();
        }
    }
    
    /**
     *  给在线用户发送消息
     * @param message
     */
    public void sendMessageOnline(String message) {
        livingSession.forEach((sessionId, session) -> {
            if(session.session.isOpen()){
                sendMessageBySession(session.session, message);
            }
        });
    }

}
```



## 四、实例化使用

创建一个WebSocketConfig配置

```java
@Configuration
public class WebSocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter(){
        return new ServerEndpointExporter();
    }

}
```

使用方式

```java
@Autowired
private WebSocket webSocket;
...
webSocket.sendMessage("传递的参数");
```











