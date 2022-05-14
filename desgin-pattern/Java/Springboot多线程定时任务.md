# Springboot多线程定时任务



springboot集成了定时器，在springboot启动类加上@EnableScheduling就可以开启一个定时器

```java
@SpringBootApplication
@EnableScheduling
@MapperScan("com.jl.app.mapper")
public class AppApplication {

    public static void main(String[] args) {
        SpringApplication.run(AppApplication.class, args);
    }

}
```

然后创建线程配置类，使用@EnableAsync开启异步事件的支持

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    // 核心线程数
    private static final int corePoolSize = 10;
    // 最大线程数
    private static final int maxPoolSize = 200;
    // 队列容量
    private static final int queueCapacity = 10;
    
    @Bean
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 设置核心线程数
        executor.setCorePoolSize(corePoolSize);
        // 设置最大线程数
        executor.setMaxPoolSize(maxPoolSize);
        // 设置队列容量
        executor.setQueueCapacity(queueCapacity);
        // 设置线程活跃时间（秒）
        executor.setKeepAliveSeconds(60);
        // 设置默认线程名称
        executor.setThreadNamePrefix("msg");
        // 设置拒绝策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // 等待所有任务结束后再关闭线程池
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.initialize();
        return executor;
    }

}
```

创建定时执行的服务，开启@EnableScheduling和@Lazy(false)关闭懒加载并创建定时执行的一个或多个任务

```java
@Component
@EnableScheduling
@Lazy(false)
public class ScheduledService {

    @Autowired
    private ScheduledComponent scheduledComponent;
    
    @Scheduled(cron = "0/30 * * * * *")
    public void scheduled(){
        LocalDateTime localDateTime = LocalDateTime.now();
        scheduledComponent.scheduled();
        System.out.println("当前时间为:" + localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }

}
```

这里注意@Schedule里面的cron是一个cron表达式，在这里从左到右依次表示：秒 分 小时 月份中的日期 月份 星期中的日期 年份。 此处表示每30秒执行一次

有关cron表达式详解参考

https://www.cnblogs.com/xiang--liu/p/11378860.html

https://www.cnblogs.com/yanghj010/p/10875151.html



然后就是进行任务调度了

```java
@Component
@Slf4j
public class ScheduledComponent {

    @Autowired
    private WebSocketServerEndpoint webSocketServerEndpoint;
    
    @Async
    public void scheduled(){
        try {           
            if (sendMessage){
                JSONObject jsonObject = new JSONObject();
                jsonObject.put("id", id);
                jsonObject.put("title", title);
                jsonObject.put("createTime", createTime);
                jsonObject.put("content", content);
                // 推送消息给app
                webSocketServerEndpoint.sendMessageOnline(jsonObject.toJSONString());
            }
            if (sendSMS){
                // 发送短信给手机
                boolean flag = smsService.sendSMS(username, content, sn);
                if (!flag){
                    log.info("发送短信失败");
                }
            }
        } catch (Exception e){
            log.error(null, e);
        }
    }

}
```

