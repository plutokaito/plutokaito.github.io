---
title: "使用 SpringBoot 发送应用系统邮件"
date: 2022-05-17
tags: ["spring", "springboot", "mail"]
description : "该篇文章讲述的是怎么使用 Springboot 自带的 email 发送邮件"
---

## 前言
在一些特定的场景下是需要发送邮件的。比如系统的报警、用户反馈的提醒等等。那么在 SpringBoot 项目中怎么实现呢？

## 准备
先准备发送的邮箱和密码。这里我建议使用客户端专用密码，没有过期的烦恼。也不会暴露原始的密码。以 [腾讯企业邮](https://work.weixin.qq.com/mail/) 为例。 申请发送邮箱需要先绑定微信，才能生成。这里是比较坑的一点，但也是安全起见吧，如果是公司需要申请一下，绑定系统维护人员微信即可。具体设置如下图：
![pass_setting](/images/post/mail/pass-setting.png)

## 导入相应的包
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```
根据 [官方文档](https://docs.spring.io/spring-framework/docs/5.3.19/reference/html/integration.html#mail)，这里使用的是 [JavaMail / Jakarta Mail 1.6](https://eclipse-ee4j.github.io/mail/) 。有兴趣的可以自行了解。


## 创建 util 包
由于个人习惯的问题，喜欢把这些非业务功能封装成工具类，由于是工具类，那么就不使用依赖注入和查找了。直接使用了 `new JavaMailSenderImpl()` ,具体 demo 如下。

```java
@Slf4j
public class SendEmailUtil {

    public static void sendEmail(String to, String subject, String content) throws MessagingException {
        JavaMailSenderImpl sender = new JavaMailSenderImpl();
        sender.setPassword("专用密码"); // 密码
        sender.setPort(465);
        sender.setUsername("xxx@xxxxxbbb.com"); // 发送者邮箱
        sender.setHost("smtp.exmail.qq.com");
        sender.setProtocol("smtps"); // 协议
        MimeMessage message = sender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message);

        helper.setFrom("xxx@xxxxxbbb.com"); // 发送者邮箱
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content);

        sender.send(message);
    }
}
```

为什么协议要使用 smtps 呢？ 根据 [wiki](https://en.wikipedia.org/wiki/SMTPS) 上解释来看

> Some email service providers allow their customers to use the SMTPS protocol to access a TLS-encrypted version of the "submission" service on port 465.

一些邮件服务允许在 465 端使用 SMTPS 协议。 而腾讯企业使用的端口就是 465。

## 使用
使用以下代码便可收到邮件
```java
SendEmailUtil.sendEmail("aa@qq.com", "aa", "content");
```
将 “aa@qq.com” 替换成你自己的邮箱便能收到邮件了。
