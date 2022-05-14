# Springboot集成MQTT



## 添加依赖

```java
<!-- 集成MQTT-->
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-mqtt</artifactId>
</dependency>
```



## 添加配置

```xml
mqtt:
  username: ... # 用户
  password: ... # 密码
  url: ... # mqtt服务端连接地址
  producer:
    clientId: localopsProducer
    defaultTopic: /ops/notify
  consumer:
    clientId: localopsConsumer
    defaultTopic: /ops/notify
```



## mqtt连接配置


```java
@Configuration
public class MqttConfig {

    private static final byte[] WILL_DATA;

    static {
        WILL_DATA = "offline".getBytes();
    }

    /**
     * 订阅的bean名称
     */
    public static final String CHANNEL_NAME_IN = "mqttInboundChannel";
    /**
     * 发布的bean名称
     */
    public static final String CHANNEL_NAME_OUT = "mqttOutboundChannel";

    @Value("${mqtt.username}")
    private String username;

    @Value("${mqtt.password}")
    private String password;

    @Value("${mqtt.url}")
    private String url;

    @Value("${mqtt.producer.clientId}")
    private String producerClientId;

    @Value("${mqtt.producer.defaultTopic}")
    private String producerDefaultTopic;

    @Value("${mqtt.consumer.clientId}")
    private String consumerClientId;

    @Value("${mqtt.consumer.defaultTopic}")
    private String consumerDefaultTopic;


    /**
     * mqtt连接器选项
     * @return MqttConnectOptions
     */
    @Bean
    public MqttConnectOptions getMqttConnectOptions(){
        MqttConnectOptions options = new MqttConnectOptions();
        // 设置是否清空Session
        // true表示每次连接到服务器都以新的身份连接
        // false表示服务器会保留客户端的连接记录
        options.setCleanSession(true);
        // 设置连接用户名、密码、连接地址
        if (StringUtils.isNotBlank(username)){
            options.setUserName(username);
        }
        if (StringUtils.isNotBlank(password)){
            options.setPassword(password.toCharArray());
        }
        // 多个服务器url以逗号隔开
        options.setServerURIs(StringUtils.split(url, StringPool.COMMA));
        // 设置超时时间，单位为秒
        options.setConnectionTimeout(10);
        // 设置会话心跳时间，单位为秒，服务器每隔1.5*20秒的时间向客户端发送心跳判断客户端是否在线，但这个方法并没有重连的机制
        options.setKeepAliveInterval(20);
        // 设置 遗嘱 消息话题，若客户端与服务器之间的连接意外中断，服务器将发布客户端的 遗嘱 消息
        options.setWill("willTopic", WILL_DATA, 2, false);
        return options;
    }

    /**
     * MQTT客户端
     * @return MqttPahoClientFactory
     */
    @Bean
    public MqttPahoClientFactory mqttClientFactory(){
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setConnectionOptions(getMqttConnectOptions());
        return factory;
    }

    /**
     * MQTT信息通道（生产者）
     * @return MessageChannel
     */
    @Bean(name = CHANNEL_NAME_OUT)
    public MessageChannel mqttOutboundChannel(){
        return new DirectChannel();
    }

    /**
     * MQTT消息处理器（生产者）
     * @return MessageHandler
     */
    @Bean
    @ServiceActivator(inputChannel = CHANNEL_NAME_OUT)
    public MessageHandler mqttOutbound(){
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(producerClientId, mqttClientFactory());
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic(producerDefaultTopic);
        return messageHandler;
    }

    /**
     * MQTT消息订阅绑定（消费者）
     * @return MessageProducer
     */
    @Bean
    public MessageProducer inbound(){
        // 可以同时消费（订阅）多个主题
        MqttPahoMessageDrivenChannelAdapter adapter = new MqttPahoMessageDrivenChannelAdapter(
                consumerClientId, mqttClientFactory(), StringUtils.split(consumerDefaultTopic, StringPool.COMMA));
        adapter.setCompletionTimeout(5000);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(1);
        // 设置订阅通道
        adapter.setOutputChannel(mqttInboundChannel());
        return adapter;
    }

    /**
     * MQTT信息通道（消费者）
     * @return MessageChannel
     */
    @Bean(name = CHANNEL_NAME_IN)
    public MessageChannel mqttInboundChannel() {
        return new DirectChannel();
    }

    /**
     * MQTT消息处理器（消费者）
     * @return MessageHandler
     */
    @Bean
    @ServiceActivator(inputChannel = CHANNEL_NAME_IN)
    public MessageHandler mqttInbound(){
        return new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                // TODO :

            }
        };
    }

}
```



## 定义消息发送

```java
@Component
@MessagingGateway(defaultRequestChannel = MqttConfig.CHANNEL_NAME_OUT)
public interface MqttGateway {
 
    /**
     * 发送信息到MQTT服务器
     *
     * @param data 发送的文本
     */
    void sendToMqtt(String data);
 
    /**
     * 发送信息到MQTT服务器
     *
     * @param topic 主题
     * @param payload 消息主体
     */
    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic,
                    String payload);
 
    /**
     * 发送信息到MQTT服务器
     *
     * @param topic 主题
     * @param qos 对消息处理的几种机制。<br> 0 表示的是订阅者没收到消息不会再次发送，消息会丢失。<br>
     * 1 表示的是会尝试重试，一直到接收到消息，但这种情况可能导致订阅者收到多次重复消息。<br>
     * 2 多了一次去重的动作，确保订阅者收到的消息有一次。
     * @param payload 消息主体
     */
    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic,
                    @Header(MqttHeaders.QOS) int qos,
                    String payload);
 
}
```



## 实例化使用

```java
@Autowired
private MqttGateway mqttGateway;
```

```java
mqttGateway.sendToMqtt(jsonArray.toString());
```