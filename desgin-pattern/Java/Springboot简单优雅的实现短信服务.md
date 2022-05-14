# Springboot简单优雅的实现短信服务



## 一、开通服务



在阿里云上开通服务并进行配置。官方文档写的比较清楚，可参考如下链接：

https://blog.csdn.net/qq_27790011/article/details/78339856

配置好之后需要获取如下信息：

accessKeyId、accessSecret两个秘钥

signName签名名称

templateCode模板code



## 二、添加依赖和配置

添加依赖

```java
<!--阿里云短信服务-->
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.1.0</version>
</dependency>
```

添加配置

需要在application.yml文件中加入配置，就是如上获取的四个参数

```yaml
#短信服务
sms:
  accessKeyId: ...
  accessSecret: ...
  signName: ...
  templateCode: ...
```



## 三、服务实现

创建SMSService接口及其实现类

通过@Value注解将配置文件中的四个参数获取到

```java
@Service
@Slf4j
public class SMSServiceImpl implements SMSService {

    @Value("${sms.accessKeyId}")
    private String accessKeyId;
    
    @Value("${sms.accessSecret}")
    private String accessSecret;
    
    @Value("${sms.signName}")
    private String signName;
    
    @Value("${sms.templateCode}")
    private String templateCode;
    
    @Override
    public boolean sendSMS(String phoneNumber, String message, String sn) {
        DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessKeyId, accessSecret);
        IAcsClient client = new DefaultAcsClient(profile);
        CommonRequest request = new CommonRequest();
        // 以下为系统参数，无需改变
        request.setMethod(MethodType.POST);
        request.setDomain("dysmsapi.aliyuncs.com");
        request.setVersion("2017-05-25");
        request.setAction("SendSms");
        
        request.putQueryParameter("RegionId", "cn-hangzhou");
        request.putQueryParameter("PhoneNumbers", phoneNumber);
        request.putQueryParameter("SignName", signName);
        request.putQueryParameter("TemplateCode", templateCode);
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("sn", sn);
        jsonObject.put("message", message);
        request.putQueryParameter("TemplateParam", jsonObject.toJSONString());
        try {
            CommonResponse response = client.getCommonResponse(request);
            log.info(response.getData());
            return true;
        } catch (Exception e){
            log.error(null, e);
        }
        return false;
    }

}
```

如果要测试的话可以写一个接口模拟发送短信

```java
@Api(tags = "消息通知")
@RestController
@RequestMapping("/message")
@Slf4j
@CrossOrigin
public class MessageController {
    @Autowired
    private SMSService smsService;

    @ApiIgnore
    @GetMapping("/sendSms")
    public String sendSms(String username, String message, String sn){
        smsService.sendSMS(username, message, sn);
        return "ok";
    }

}    
```


