---
title: MQTT
date: 2025-05-23 09:02
categories: 协议
---

##                                                         MQTT测试

### 使用[EMQX](https://github.com/emqx/emqx)测试

 [MQTT服务器信息](https://www.emqx.com/zh/mqtt/public-mqtt5-broker)

![0](/images/MQTT/0.png)

### Web客户端配置参数

打开[Easy-to-Use Online MQTT Client | Try Now](http://www.emqx.io/online-mqtt-client#/recent_connections)，点击New Connection。

![1](/images/MQTT/1.png)

设置Name信息后点击Connect信息。//Port 8084 与MQTT服务器中 WebSocket Secure 端口一致

![2](/images/MQTT/2.png)

设置订阅信息 New Subscription，设置消息主题Topic，消息等级QoS。

![3](/images/MQTT/3.png)

### Net程序

安装MQTTnet nuget包。

```c#
using System.Text;
using MQTTnet;

var mqttFactory = new MqttClientFactory();

using (var mqttClient = mqttFactory.CreateMqttClient())
{
    var mqttClientOptions = new MqttClientOptionsBuilder()
        .WithTcpServer("broker.emqx.io", 1883) //Port 1883 与MQTT服务器中 TCP 端口一致
        .WithClientId("icm") //ClientId可以不设置，但不能与Web客户端中重复
        .WithCredentials("", "")
        .Build();

    var connectResult = await mqttClient.ConnectAsync(mqttClientOptions, CancellationToken.None);
    if (connectResult.ResultCode == MqttClientConnectResultCode.Success)
    {
        await mqttClient.SubscribeAsync("icm/test"); //订阅消息主题

        mqttClient.ApplicationMessageReceivedAsync += e =>
        {
            Console.WriteLine(
                $"Received message: {Encoding.UTF8.GetString(e.ApplicationMessage.Payload)}"
            );
            return Task.CompletedTask;
        };
    }

    Console.WriteLine("The MQTT client is connected.");

    var timer = new System.Timers.Timer() { Interval = 10_000, Enabled = true, };
    timer.Elapsed += (sender, e) =>
    {
        Console.WriteLine("Timer");

        var applicationMessage = new MqttApplicationMessageBuilder()
            .WithTopic("icm/state")
            .WithPayload($"{DateTime.Now:yyyy-MM-dd HH:mm:ss.fff}")
            .WithQualityOfServiceLevel(MQTTnet.Protocol.MqttQualityOfServiceLevel.AtLeastOnce)
            .Build();
        mqttClient.PublishAsync(applicationMessage, CancellationToken.None); //发送消息

        applicationMessage = new MqttApplicationMessageBuilder()
            .WithTopic("icm/observation")
            .WithPayload(
            "START,2025-04-27-15:00:00,2,1,0,1,1,2000,3000,2500,/202504/27/1200/Ir_202504271200_raw.jpg,/202504/27/1200/vis_202504271200_raw.jpg, /202504/27/1200/cloud_202504271200_pro.jpg,END"
        )
            .WithQualityOfServiceLevel(MQTTnet.Protocol.MqttQualityOfServiceLevel.AtLeastOnce)
            .Build();
        mqttClient.PublishAsync(applicationMessage, CancellationToken.None); //发送消息

        applicationMessage = new MqttApplicationMessageBuilder()
            .WithTopic("icm/power")
            .WithPayload([0xAA, 0xBB, 0x00, 0x01, 0xBB, 0xAA])
            .WithQualityOfServiceLevel(MQTTnet.Protocol.MqttQualityOfServiceLevel.AtLeastOnce)
            .Build();

        mqttClient.PublishAsync(applicationMessage, CancellationToken.None); //发送消息
    };

    Console.WriteLine("System.Timers.Timer.");

    Console.ReadKey();
    timer.Dispose();

    await mqttClient.DisconnectAsync();

    Console.WriteLine("MQTT application message is published.");
}
```
