---
title: "使用 SpringBoot + Spring JMS + AWS SQS 的 Demo"
date: 2021-08-19
tags: ["springboot", "jms", 'aws']
description : "该篇文章主要是讲述怎么使用 SpringBoot + Spring JMS 在 AWS SQS 收发信息"
---

本篇文章主要是讲述怎么在 springboot 项目利用 Jms 收发 AWS SQS 的消息，具体代码详见：[aws-sqs-demo](https://github.com/plutokaito/aws-sqs-demo)


## AWS SQS
### 注册 AWS 账号

注册 aws 账号时需要准备一张国际卡信用卡， visa 或者 master 卡都行。因为开通的时候需要从卡里锁定一美元。注册的时候需要填写 5 个步骤，按照[官网](https://portal.aws.amazon.com/billing/signup#/account)提示，一步一步往下走，即可，这里不做过多解释。

### 创建 SQS

在搜索栏中搜索 SQS ，出现 Simple Queue Service。

![image-20210818140429773](/images/post/sqs/image-20210818140429773.png)

进入 Amazon SQS 后， 点击 Create queue 按钮，创建 Standard  Queue 即可， 名称等信息按照自己业务的方式填写即可， 我这里就填写了名称其他的都是默认的填写， 便完成了如下图所示

![image-20210818140839499](/images/post/sqs/image-20210818140839499.png)


### 设置 Security Credentials

设置 access keys。 点击个人账号后，进入 Your Security Credentials >> Access keys (access key ID and secret access key)。创建新的 Access Key。主要保存下载保存自己的 access key。需要注意的是：只有创建时会显示，其他任何时候不能再显示了。

![image-20210818143821731](/images/post/sqs/image-20210818143821731.png)



## 代码 Demo

### 创建 SpringBoot 项目

在 start.spring.io 的网站上创建一个简单的 Springboot 项目。或者使用 idea 创建项目都可以。



### 引入包

引入包文件

```pom
		<dependency>
			<groupId>com.amazonaws</groupId>
			<artifactId>aws-java-sdk</artifactId>
			<version>1.11.106</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jms</artifactId>
		</dependency>

		<!-- 注意： 1.0.8 所包含的包是1.11.106，以免造成包冲突 -->
		<dependency>
			<groupId>com.amazonaws</groupId>
			<artifactId>amazon-sqs-java-messaging-lib</artifactId>
			<version>1.0.8</version>
		</dependency>
```



### 配置 Config

```java
@Configuration
@EnableJms
public class JmsConfig {

    SQSConnectionFactory connectionFactory = new SQSConnectionFactory(
            new ProviderConfiguration(),
            AmazonSQSClientBuilder.standard()
                    .withRegion(Regions.US_EAST_2)
                    .withCredentials(new ClasspathPropertiesFileCredentialsProvider("sqs.properties"))
    );

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(this.connectionFactory);
        factory.setDestinationResolver(new DynamicDestinationResolver());
        // 3 - 10 listeners
        factory.setConcurrency("3-10");
        // when client receive success with none exception will send ack, and service delete this message
        // otherwise Spring will recover the message.
        factory.setSessionAcknowledgeMode(Session.CLIENT_ACKNOWLEDGE);
        return factory;
    }

    @Bean
    public JmsTemplate defaultJmsTemplate() {
        return new JmsTemplate(this.connectionFactory);
    }
}
```

使用注解 @EnableJms 开启 Jms 的监听，从 sqs.properties 文件中读取之前在网站上配置的 accessKey 和 secretKey。



### 消息体

消息体可以理解成一个 dto， 这里我简单以 title, points 为例。

```java
public class SqsDemoMsg {
    private String title;
    private Long points;

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Long getPoints() {
        return points;
    }

    public void setPoints(Long points) {
        this.points = points;
    }


    @Override
    public String toString() {
        return "SqsDemoMsg{" +
                "title='" + title + '\'' +
                ", points=" + points +
                '}';
    }

    public static SqsDemoMsg jsonToSqsDemoMsg(String jsonString) {
        try {
            return JSONObject.parseObject(jsonString, SqsDemoMsg.class);
        } catch (JSONException e) {
            return null;
        }
    }
}
```



### 发送端

发端的端将 SqsDemoMsg 的信息，使用 fastjson 将数据压缩成 json 字符串进行传输。

```java
@Service
public class SendServiceImpl implements SendService {
    @Autowired
    private JmsTemplate jmsTemplate;

    @Value("${test.queue}")
    private String queueName;

    @Override
    public void sendMessage(Long points, String title) throws IOException {
        SqsDemoMsg msg = new SqsDemoMsg();
        msg.setPoints(points);
        msg.setTitle(title);
        // send message
        jmsTemplate.convertAndSend(queueName, JSONObject.toJSONString(msg));
    }
}
```



### 接收端

接受端主要是接受发送端的数据，并将数据打印。

```java
@Service
@Slf4j
public class ReceiveServiceImpl implements ReceiveService {

    @JmsListener(destination = "${test.queue}")
    public void message(String jsonString) {
        log.info("===> received message:{}", jsonString);
        SqsDemoMsg sqsDemoMsg = SqsDemoMsg.jsonToSqsDemoMsg(jsonString);

        log.info("===> points:{} ", sqsDemoMsg.getPoints());
    }
}
```



@JmsListener 的注解会创建一个 JSM 的监听者。当接收到消息的时候，这个方法就会打印数据。



到这里大部分的主要代码已经 ok了， 为了方便测试我又新建了一个 Controller， 用来不同验证不同的 title 和 points 的情况。



## 启动验证

在文件 `sqs.properties` 上配置 `aws Access keys`, 这里是之前设置的 access key ID 和 secret access key ：

```properties
accessKey=XXX
secretKey=XXX
```

在文件 `application.properties` 中配置 queue 的名称

```properties
test.queue = MyQueue_test
```



配置完成后，启动项目，在浏览器中输入链接 `http://localhost:8080/send/demo?points=111&title=kaito` , title 是参数， points 也是链接。 查看日志信息:

```log
2021-08-18 14:10:10.291  INFO 9608 --- [ntContainer#0-2] i.k.s.service.impl.ReceiveServiceImpl    : ===> received message:{"points":111,"title":"kaito"}
2021-08-18 14:10:10.332  INFO 9608 --- [ntContainer#0-2] i.k.s.service.impl.ReceiveServiceImpl    : ===> points:111

```

能接收到信息，并进行了打印处理。


在 aws 客户端的 monitoring 中能看到相应的统计信息了。


## 参考链接

[Using Amazon SQS with Spring Boot and Spring JMS](https://aws.amazon.com/cn/blogs/developer/using-amazon-sqs-with-spring-boot-and-spring-jms/)