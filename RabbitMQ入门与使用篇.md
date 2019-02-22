## RabbitMQ入门与使用篇
　　
### 介绍

RabbitMQ是一个由erlang开发的基于AMQP（Advanced Message Queue）协议的开源实现。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面都非常的优秀。是当前最主流的消息中间件之一。

[RabbitMQ的官方](https://www.rabbitmq.com/)   

![image](http://blog.thankbabe.com/imgs/306976-20160720104037044-1071063805.png)
* 概念：
    * Brocker：消息队列服务器实体。
    * Exchange：消息交换机，指定消息按什么规则，路由到哪个队列。
    * Queue：消息队列，每个消息都会被投入到一个或者多个队列里。
    * Binding：绑定，它的作用是把exchange和queue按照路由规则binding起来。
    * Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
    * Vhost：虚拟主机，一个broker里可以开设多个vhost，用作不用用户的权限分离。
    * Producer：消息生产者，就是投递消息的程序。
    * Consumer：消息消费者，就是接受消息的程序。
    * Channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。
* 消息队列的使用过程大概如下：
    * 消息接收
        * 客户端连接到消息队列服务器，打开一个channel。  
        * 客户端声明一个exchange，并设置相关属性。  
        * 客户端声明一个queue，并设置相关属性。  
        * 客户端使用routing key，在exchange和queue之间建立好绑定关系。  
    * 消息发布
        * 客户端投递消息到exchange。
        * exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。    
* AMQP 里主要要说两个组件：
    * Exchange 和 Queue    
    * 绿色的 X 就是 Exchange ，红色的是 Queue ，这两者都在 Server 端，又称作 Broker   
    * 这部分是 RabbitMQ 实现的，而蓝色的则是客户端，通常有 Producer 和 Consumer 两种类型。     
* Exchange通常分为四种：
    * fanout：该类型路由规则非常简单，会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中，相当于广播功能
    * direct：该类型路由规则会将消息路由到binding key与routing key完全匹配的Queue中
    * topic：与direct类型相似，只是规则没有那么严格，可以模糊匹配和多条件匹配
    * headers：该类型不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配
* 使用场景
    * [官方介绍](https://www.rabbitmq.com/getstarted.html)  

---




### 下载与安装
* 下载
    * [rabbitmq ](http://www.rabbitmq.com/download.html)
    * [erlang](http://www.erlang.org/download.html)
* 安装
    * 先安装erlang
    * 然后再安装rabbitmq

---

### 管理工具
* 参考[官方文档](http://www.rabbitmq.com/management.html)

操作起来很简单，只需要在DOS下面，进入安装目录（`安装路径\RabbitMQ Server\rabbitmq_server-3.2.2\sbin`）执行如下命令就可以成功安装。   
```bash
rabbitmq-plugins enable rabbitmq_management    
```
可以通过访问：`http://localhost:15672`进行测试，默认的登陆账号为：guest，密码为：guest。

![图片](http://blog.waterstrong.me/assets/rabbitmq-guide/overview_totals.png)
---

### 其他配置

**1. 安装完以后erlang需要手动设置ERLANG_HOME 的系统变量。**

```bash
set ERLANG_HOME=F:\Program Files\erl9.0
#环境变量`path`里加入：%ERLANG_HOME%\bin
#环境变量`path`里加入: 安装路径\RabbitMQ Server\rabbitmq_server-3.6.10\sbin

```


**2．激活Rabbit MQ's Management Plugin**

使用Rabbit MQ 管理插件，可以更好的可视化方式查看Rabbit MQ 服务器实例的状态，你可以在命令行中使用下面的命令激活。

```bash
rabbitmq-plugins.bat  enable  rabbitmq_management
```

**3．创建管理用户**

```bash
rabbitmqctl.bat add_user sa 123456
```

**4. 设置管理员**

```bash
rabbitmqctl.bat set_user_tags sa administrator
```

**5．设置权限**

```bash
rabbitmqctl.bat set_permissions -p / sa ".*" ".*" ".*"
```

**6. 其他命令**

```bash
#查询用户：
    rabbitmqctl.bat list_users
#查询vhosts：
    rabbitmqctl.bat list_vhosts
#启动RabbitMQ服务:
    net stop RabbitMQ && net start RabbitMQ     
```

以上这些，账号、vhost、权限、作用域等基本就设置完了。

---

### 基于.net使用
[RabbitMQ.Client](http://www.rabbitmq.com/dotnet.html) 是RabbiMQ 官方提供的的客户端    
[EasyNetQ](http://easynetq.com/) 是基于RabbitMQ.Client 基础上封装的开源客户端,使用非常方便   
> 以下操作RabbitMQ的代码例子，都是基于EasyNetQ的使用和再封装，在文章底部有**demo例子的源码下载地址**

---
**创建 IBus**
```csharp
/// <summary>
/// 消息服务器连接器
/// </summary>
public class BusBuilder {
    public static IBus CreateMessageBus() {
        // 消息服务器连接字符串
        // var connectionString = ConfigurationManager.ConnectionStrings["RabbitMQ"];
        string connString = "host=127.0.0.1:5672;virtualHost=TestQueue;username=sa;password=123456";
        if (connString == null || connString == string.Empty) throw new Exception("messageserver connection string is missing or empty");
        return RabbitHutch.CreateBus(connString);
    }
}
```



**Fanout Exchange**

![image](http://blog.thankbabe.com/imgs/306976-20160728104237622-1486261669.png)   


所有发送到Fanout Exchange的消息都会被转发到与该Exchange 绑定(Binding)的所有Queue上。   
Fanout Exchange  不需要处理RouteKey 。只需要简单的将队列绑定到exchange     上。这样发送到exchange的消息都会被转发到与该交换机绑定的所有队列上。类似子网广播，每台子网内的主机都获得了一份复制的消息。
所以，Fanout Exchange 转发消息是最快的。   

---    

```csharp 
/// <summary>
///  消息消耗（fanout）
/// </summary>
/// <typeparam name="T">消息类型</typeparam>
/// <param name="handler">回调</param>
/// <param name="exChangeName">交换器名</param>
/// <param name="queueName">队列名</param>
/// <param name="routingKey">路由名</param>
public static void FanoutConsume<T>(Action<T> handler, string exChangeName = "fanout_mq", string queueName = "fanout_queue_default", string routingKey = "") where T : class {
    var bus = BusBuilder.CreateMessageBus();
    var adbus = bus.Advanced;
    var exchange = adbus.ExchangeDeclare(exChangeName, ExchangeType.Fanout);
    var queue = CreateQueue(adbus, queueName);
    adbus.Bind(exchange, queue, routingKey);
    adbus.Consume(queue, registration => {
        registration.Add<T>((message, info) => {
            handler(message.Body);
        });
    });
}
/// <summary>
/// 消息上报（fanout）
/// </summary>
/// <typeparam name="T">消息类型</typeparam>
/// <param name="topic">主题名</param>
/// <param name="t">消息命名</param>
/// <param name="msg">错误信息</param>
/// <returns></returns>
public static bool FanoutPush<T>(T t, out string msg, string exChangeName = "fanout_mq", string routingKey = "") where T : class {
    msg = string.Empty;
    try {
        using (var bus = BusBuilder.CreateMessageBus()) {
            var adbus = bus.Advanced;
            var exchange = adbus.ExchangeDeclare(exChangeName, ExchangeType.Fanout);
            adbus.Publish(exchange, routingKey, false, new Message<T>(t));
            return true;
        }
    } catch (Exception ex) {
        msg = ex.ToString();
        return false;
    }
}
```

---
![image](http://blog.thankbabe.com/imgs/306976-20160728104255372-2049742072.png)   
所有发送到Direct Exchange的消息被转发到RouteKey中指定的Queue。    
Direct模式，可以使用RabbitMQ自带的Exchange：default Exchange 。所以不需要将Exchange进行任何绑定(binding)操作 。消息传递时，RouteKey必须完全匹配，才会被队列接收，否则该消息会被抛弃。

---

``` csharp
/// <summary>
/// 消息发送（direct）
/// </summary>
/// <typeparam name="T">消息类型</typeparam>
/// <param name="queue">发送到的队列</param>
/// <param name="message">发送内容</param>
public static void DirectSend<T>(string queue, T message) where T : class {
    using (var bus = BusBuilder.CreateMessageBus()) {
        bus.Send(queue, message);
    }
}
/// <summary>
/// 消息接收（direct）
/// </summary>
/// <typeparam name="T">消息类型</typeparam>
/// <param name="queue">接收的队列</param>
/// <param name="callback">回调操作</param>
/// <param name="msg">错误信息</param>
/// <returns></returns>
public static bool DirectReceive<T>(string queue, Action<T> callback, out string msg) where T : class {
    msg = string.Empty;
    try {
        var bus = BusBuilder.CreateMessageBus();
        bus.Receive<T>(queue, callback);
    } catch (Exception ex) {
        msg = ex.ToString();
        return false;
    }
    return true;
}

/// <summary>
/// 消息发送
/// <![CDATA[（direct EasyNetQ高级API）]]>
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="t"></param>
/// <param name="msg"></param>
/// <param name="exChangeName"></param>
/// <param name="routingKey"></param>
/// <returns></returns>
public static bool DirectPush<T>(T t, out string msg, string exChangeName = "direct_mq", string routingKey = "direct_rout_default") where T : class {
    msg = string.Empty;
    try {
        using (var bus = BusBuilder.CreateMessageBus()) {
            var adbus = bus.Advanced;
            var exchange = adbus.ExchangeDeclare(exChangeName, ExchangeType.Direct);
            adbus.Publish(exchange, routingKey, false, new Message<T>(t));
            return true;
        }
    } catch (Exception ex) {
        msg = ex.ToString();
        return false;
    }
}
/// <summary>
/// 消息接收
///  <![CDATA[（direct EasyNetQ高级API）]]>
/// </summary>
/// <typeparam name="T">消息类型</typeparam>
/// <param name="handler">回调</param>
/// <param name="exChangeName">交换器名</param>
/// <param name="queueName">队列名</param>
/// <param name="routingKey">路由名</param>
public static bool DirectConsume<T>(Action<T> handler, out string msg, string exChangeName = "direct_mq", string queueName = "direct_queue_default", string routingKey = "direct_rout_default") where T : class {
    msg = string.Empty;
    try {
        var bus = BusBuilder.CreateMessageBus();
        var adbus = bus.Advanced;
        var exchange = adbus.ExchangeDeclare(exChangeName, ExchangeType.Direct);
        var queue = CreateQueue(adbus, queueName);
        adbus.Bind(exchange, queue, routingKey);
        adbus.Consume(queue, registration => {
            registration.Add<T>((message, info) => {
                handler(message.Body);
            });
        });
    } catch (Exception ex) {
        msg = ex.ToString();
        return false;
    }
    return true;
}
```


---

**Topic Exchange**   

![image](http://blog.thankbabe.com/imgs/306976-20160728104309934-1385658660.png)

* 消息发布（Publish） 

要使用主题发布，只需使用带有主题的重载的Publish方法：

```csharp
var bus = RabbitHutch.CreateBus(...);
bus.Publish(message, "X.A");

```
订阅者可以通过指定要匹配的主题来过滤邮件。    
* 这些可以包括通配符：   
    * *=>匹配一个字。   
    * ＃=>匹配到零个或多个单词。

所以发布的主题为“X.A.2”的消息将匹配“＃”，“X.＃”，“* .A.*”，而不是“X.B. *”或“A”。       

警告，Publish只顾发送消息到队列，但是不管有没有消费端订阅，所以，发布之后，如果没有消费者，该消息将不会被消费甚至丢失。

* 消息订阅（Subscribe）

EasyNetQ提供了消息订阅，当调用Subscribe方法时候，EasyNetQ会创建一个用于接收消息的队列，不过与消息发布不同的是，消息订阅增加了一个参数，subscribe_id.代码如下：

```csharp
bus.Subscribe("my_id", handler, x => x.WithTopic("X.*"));
```
警告： 具有相同订阅者但不同主题字符串的两个单独订阅可能不会产生您期望的效果。 subscriberId有效地标识个体AMQP队列。 具有相同subscriptionId的两个订阅者将连接到相同的队列，并且两者都将添加自己的主题绑定。 所以，例如，如果你这样做：

```csharp
bus.Subscribe("my_id", handlerOfXDotStar, x => x.WithTopic("X.*"));
bus.Subscribe("my_id", handlerOfStarDotB, x => x.WithTopic("*.B"));
```

匹配“x.*”或“* .B”的所有消息将被传递到“XXX_my_id”队列。 然后，RabbitMQ将向两个消费者传递消息，其中handlerOfXDotStar和handlerOfStarDotB轮流获取每条消息。

 现在，如果你想要匹配多个主题（“X. *”OR“* .B”），你可以使用另一个重载的订阅方法，它采用多个主题，如下所示：



```csharp
bus.Subscribe("my_id", handler, x => x.WithTopic("X.*").WithTopic("*.B"));
```

---

```csharp
/// <summary>
/// 获取主题 
/// </summary>
/// <typeparam name="T">主题内容类型</typeparam>
/// <param name="subscriptionId">订阅者ID</param>
/// <param name="callback">消息接收响应回调</param>
///  <param name="topics">订阅主题集合</param>
public static void TopicSubscribe<T>(string subscriptionId, Action<T> callback, params string[] topics) where T : class {
    var bus = BusBuilder.CreateMessageBus();
    bus.Subscribe(subscriptionId, callback, (config) => {
        foreach (var item in topics) config.WithTopic(item);
    });
}
/// <summary>
/// 发布主题
/// </summary>
/// <typeparam name="T">主题内容类型</typeparam>
/// <param name="topic">主题名称</param>
/// <param name="message">主题内容</param>
/// <param name="msg">错误信息</param>
/// <returns></returns>
public static bool TopicPublish<T>(string topic, T message, out string msg) where T : class {
    msg = string.Empty;
    try {
        using (var bus = BusBuilder.CreateMessageBus()) {
            bus.Publish(message, topic);
            return true;
        }
    } catch (Exception ex) {
        msg = ex.ToString();
        return false;
    }
}
/// <summary>
/// 发布主题
/// </summary>
/// <![CDATA[（topic EasyNetQ高级API）]]>
/// <typeparam name="T">消息类型</typeparam>
/// <param name="t">消息内容</param>
/// <param name="topic">主题名</param>
/// <param name="msg">错误信息</param>
/// <param name="exChangeName">交换器名</param>
/// <returns></returns>
public static bool TopicSub<T>(T t, string topic, out string msg, string exChangeName = "topic_mq") where T : class {
    msg = string.Empty;
    try {
        if (string.IsNullOrWhiteSpace(topic)) throw new Exception("推送主题不能为空");
        using (var bus = BusBuilder.CreateMessageBus()) {
            var adbus = bus.Advanced;
            //var queue = adbus.QueueDeclare("user.notice.zhangsan");
            var exchange = adbus.ExchangeDeclare(exChangeName, ExchangeType.Topic);
            adbus.Publish(exchange, topic, false, new Message<T>(t));
            return true;
        }
    } catch (Exception ex) {
        msg = ex.ToString();
        return false;
    }
}

/// <summary>
/// 获取主题 
/// </summary>
/// <![CDATA[（topic EasyNetQ高级API）]]>
/// <typeparam name="T">消息类型</typeparam>
/// <param name="subscriptionId">订阅者ID</param>
/// <param name="callback">回调</param>
/// <param name="exChangeName">交换器名</param>
/// <param name="topics">主题名</param>
public static void TopicConsume<T>(Action<T> callback, string exChangeName = "topic_mq",string subscriptionId = "topic_subid", params string[] topics) where T : class {
    var bus = BusBuilder.CreateMessageBus();
    var adbus = bus.Advanced;
    var exchange = adbus.ExchangeDeclare(exChangeName, ExchangeType.Topic);
    var queue = adbus.QueueDeclare(subscriptionId);
    foreach (var item in topics) adbus.Bind(exchange, queue, item);
    adbus.Consume(queue, registration => {
        registration.Add<T>((message, info) => {
            callback(message.Body);
        });
    });
}    
```

---   
具体发布/订阅消息的Demo和相关测试看源码Demo
![image](http://blog.thankbabe.com/imgs/rabbit_demo.jpg)
为了方便使用，demo改进版本：
![image](http://blog.thankbabe.com/imgs/rabbit_demo_2.jpg)
---

### 注意

**当在创建订阅者去消费队列的时候**

```csharp
/// <summary>
/// 获取主题 
/// </summary>
/// <param name="topic"></param>
public static void GetSub<T>(T topic, Action<T> callback) where T : class
{
    using (var bus = BusBuilder.CreateMessageBus()) {
        bus.Subscribe<T>(topic.ToString(), callback, x => x.WithTopic(topic.ToString()));
    }

}
```

using里的对象在执行完成后被回收了，导致刚连接上去就又断开了(刚开始写的时候，习惯性加using，排查了好久才发现，欲哭无泪)



**源码项目运行前的准备与确认：** 

到RabbitMQ管理后台添加`TestQueue`VHost，并且分配用户权限，然后到`RabbitMQHelper.BusBuilder`类里配置RabbitMQ连接服务的相关信息
`host=127.0.0.1:5672;virtualHost=TestQueue;username=sa;password=123456`，（根据配置的内容和用户修改）

![image](http://blog.thankbabe.com/imgs/rabbitmq-ht.png)

---   
**参考资料(鸣谢)：**    
* [EasyNetQ-基于Topic的路由](http://www.cnblogs.com/zd1994/p/7169123.html)   
* [.NET操作RabbitMQ组件EasyNetQ使用中文简版文档。](http://www.cnblogs.com/panzi/p/6337568.html)
* [RabbitMQ入门指南](http://blog.waterstrong.me/rabbitmq-start-guide/)

---

**附：**[Demo源码](https://github.com/SFLAQiu/RabbitMQDemo)GitHub地址   <!-- Place this tag where you want the button to render. -->
<a class="github-button" href="https://github.com/sflaqiu/RabbitMQDemo" data-size="large" aria-label="Star sflaqiu/RabbitMQDemo on GitHub">Star</a>


---

> 发布时间：2017-08-03