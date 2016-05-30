# read push

## 单个消息推送

请求数据类型`SinglePushMessageRequest`
```
    private Integer bigAppId; // app类型 1 管家 8 人品 
    private PushType pushType; // 推送类型 Notify 通知 Transmission 透传
    private String message; //推送消息体 
    private String payload; // 透传参数, PushType为Transmission, 该参数必填"
    private NotifyAction notifyAction; //当PushType为Notify时的动作定义,分为OPEN_URL，OPEN_APP两种
    private String notifyUrl; //notifyAction为OPEN_URL时,该参数必填
    private Long userId;
    private String deviceToken;//设备token 由apns或者个推提供
    private Integer appId; // 当bigAppId为1时 appId 1 4 5 分别代表 android  ios isopro
    private String batchNo;//该消息的批次号, 用于幂等性控制
    private Category category;//推送类型 RealTime 实时 Normal 普通,受时间限制
    private List<String> excludes = Lists.newArrayList(); //不需要推送的deviceToken, 没有可不填写

```

`batchNo` 通过redis设置有效时间，有效时间内不处理同批次的数据

通过`SinglePushDefine`包装`SinglePushMessageRequest`(目前属性是一致的),按RealTime和Normal 分两个routingKey发到mq

通过`MessageInfoSingle` 存储在数据中，按`季度`分表`t_message_info_single_年份+季度` eg：`t_message_info_single_201602`

推送完成后在`t_push_info_single_年份+季度`表中存储 
