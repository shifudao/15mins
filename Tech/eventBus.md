# Vert.x EventBus 简单介绍与通信模式总结

## 什么是EventBus?
Vert.x做为一个分布式框架，EventBus是它的神经系统。它负责整个分布式应用中，各部分之间的通讯。

## 特点

* 单例
* 支持发布/订阅
* 接口简单便捷

## EventBus中五个重要概念

* Addressing
 - 在事件总线上发送消息目的地地址。
* Handlers
 - 在一个地址注册一个处理程序, 此处理程序会处理这个地址接收的消息。
* Publish / Subscribe
 - 消息被发布到一个地址上。
 - 发布意味着将消息传递给在该地址注册的所有处理程序。
* Point to point and Request-Response
 - 消息被发送到一个地址。Vert.x将其路由到的只是其中的一个处理程序注册地址。
 - 带有点对点的消息，当发送消息时，可以指定一个可选的应答处理程序。
* Types of messages
 - EventBus传递的消息，可以是原始简单类型数据，可以是JSON数据，还可以是任意对象，但必须编写对应的编码器。

## 如何使用?

* 订阅数据

```groovy
//groovy写法
eventBus.consumer("my.address"){ msg ->
    println "receive message: ${msg.body()}"
    msg.reply "I received!"
}

//Web Page中,js写法
eventBus.registerHandler(address, function(msg){
    alert(msg);
});
```

* 发布数据

```groovy
eventBus.publish("my.address","hello world!")
```

## 我们在使用Vert.x框架中所遇到的EventBus通讯模式

* 点对点模式，发送端不需要知道消息是否到达目的地。

```groovy
//发送端
eventBus.send("address", msg)

接收端
eventBus.consumer("address"){ msg ->
   ...
}
```

* Request-Response模式，发送端需要知道消息到达目的地，并要根据相应消息做其他处理。

```groovy
//发送端
eventBus.send("address", msg){ reply ->
    reply.body()
    ...
}

//接收端
eventBus.consumer("address"){ msg ->
   ...
   msg.reply "some thing"
}
```

* 广播模式，发送端不需要知道谁订阅，只管发送消息。

```groovy
//发送端
eventBus.publish("special.address", msg)

//接收端,能收到消息
eventBus.consumer("special.address"){ msg ->
   ...
}

//接收端,不能收到消息
eventBus.consumer("othor.address"){ msg ->
   ...
}
```
* 动态订阅模式，“发送端只服务vip用户”

```groovy
//接收端
//先订阅地址，再向发送端发送刚订阅过的地址
eventBus.consumer("帅的被人喷"){ msg ->
   ...
}

//向发送端发送订阅地址
eventBus.send("服务器地址"，"你就给我向‘帅的被人喷’地址上发消息")

//发送端
//收到消息后，向目标地址发消息
eventBus.send("服务器地址", msg){ reply->
    ...
    //你掏钱了，那就给你发。
    eventBus.send("帅的被人喷","确实帅，帅呆了！")
}
```
