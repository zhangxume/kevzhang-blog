---
title: "Azkaban 配置邮件及电话告警"
date: 2022-10-15T10:54:00+08:00
description: "之后试试 DolphinScheduler，看起来不错。"
tags: [Azkaban]
featured_image: "/azkaban_alert/featured_image.png"
draft: false
---

## 通过邮件方式
- 修改 Azkaban Web Server 端的 `azkaban.properties` 配置文件
   配置参数

|Parameter|Description|
|-|-|
|mail.sender|The email address that azkaban uses to send emails.|
|mail.host|The email server host machine.|
|mail.user|The email server user name.|
|mail.password|The email password user name.|

```
# 开通smtp服务后，设置邮件发送服务器
mail.sender=你的邮箱
mail.host=你的邮箱对应的smtp服务器（例如：smtp.163.com）
mail.user=你的邮箱
mail.password=你的邮箱的授权码（在开通smtp服务后获得的授权码）
```
- 保存上述修改的配置文件，重启 Web Server
```shell
[xxx@hostname azkaban-web]$ bin/shutdown-web.sh
[xxx@hostname azkaban-web]$ bin/start-web.sh
```
- 在 Notification 中填写接收报警通知的邮箱并执行 Project

- Project 执行结束并收到对应的通知邮件

<center>

![](/azkaban_alert/email.jpg)

<center>

## 通过电话方式
- 通过邮件接收报警通知可能存在接收消息不及时的问题，因此可以采用电话的方式接收报警通知。

- 可以集成第三方的告警平台。

- Azkaban 配置的 smtp 邮箱将告警邮件发送到第三方的告警平台提供的邮箱，第三方服务邮箱收到邮件后执行电话通知的任务。

- 执行 Project 后等待几十秒或几分钟就会收到机器人语音来电，通知告警消息。

- 后续可在第三方告警平台及时查看每日的告警详细记录。

<center>

![](/azkaban_alert/phone.jpg)  

<center>

## 参考链接
[configuration.html](https://azkaban.readthedocs.io/en/latest/configuration.html)