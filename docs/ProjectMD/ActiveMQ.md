# 1 ActiveMQ简介

## 1.1 ActiveMQ是什么

ActiveMQ是一个消息队列应用服务器（推送服务器）。支持JMS规范。

### 1.1.1 JMS概述

JMS 全称：Java Message Service ，即为 Java 消息服务，是一套 java 消息服务的 API 接口。实现了 JMS 标准的系统，称之为 JMS Provider。

### 1.1.2 消息队列

消息队列是在消息的传输过程中保存消息的容器，提供一种不同进程或者同一进程不同线程直接通讯的方式。



![img](https://pic2.zhimg.com/80/v2-dd5579d4e545784c852d08903416a02d_720w.webp)



- Producer：消息生产者，负责产生和发送消息到 Broker；
- Broker：消息处理中心。负责消息存储、确认、重试等，一般其中会包含多个 queue；
- Consumer：消息消费者，负责从 Broker 中获取消息，并进行相应处理；

常见消息队列应用：

1. ActiveMQ

> ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现。

2. RabbitMQ

> RabbitMQ是一个在AMQP基础上完成的，可复用的企业消息系统。他遵循Mozilla Public License开源协议。开发语言为Erlang。

3. RocketMQ

> 由阿里巴巴定义开发的一套消息队列应用服务。

## 1.2 ActiveMQ能做什么

1. 实现两个不同应用(程序)之间的消息通讯。
2. 实现同一个应用，不同模块之间的消息通讯。（确保数据发送的稳定性）

## 1.3 ActiveMQ主要特点

1. 支持多语言、多协议客户端。语言: Java,C,C++,C#,Ruby,Perl,Python,PHP。应用协议： OpenWire, Stomp REST, WS Notification, XMPP, AMQP
2. 对Spring的支持，ActiveMQ可以很容易整合到Spring的系统里面去。
3. 支持高可用、高性能的集群模式。

## 1.4 MQ的主要作用

1. 异步。调用者无需等待。
2. 解耦。解决了系统之间耦合调用的问题。
3. 消峰。抵御洪峰流量，保护了主业务。

# 2.入门案例

## 2.1 搭建ActiveMQ消息服务器（通过docker安装）

```shell
docker pull webcenter/activemq   #拉取镜像

docker run -d -p 8161:8161 -p 61616:61616 webcenter/activemq      #启动并映射端口

docker run --name='activemq' -itd -p 8161:8161 -p 61616:61616 -p 61618:61618 -e ACTIVEMQ_ADMIN_LOGIN=admin -e ACTIVEMQ_ADMIN_PASSWORD=admin  --restart=always  -v /app/activemq/data:/data/activemq -v /app/activemq/log:/var/log/activemq  webcenter/activemq:latest    #启动并设置账号密码并映射文件到宿主机
```

## 2.2 访问ActiveMQ控制台

> 访问activemq管理页面地址：http://IP地址:8161/
>
> 账户admin 密码admin

## 2.3 消息队列（Queue）生产者的入门案例

JMS开发的基本步骤
1:创建一个connection factory
2:通过connection factory来创建JMS connection
3:启动JMS connection
4:通过connection创建JMS session
5:创建JMS destination
6:创建JMs producer或者创建JMS message并设置destination
7:创建JMS consumer或者是注册一个JMS message listener
8:发送或者接受JMS message(s)
9:关闭所有的JMS资源(connection, session, producer, consumer等)

引入依赖

```java
<!--  activemq  所需要的jar 包-->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.9</version>
</dependency>
<!--  activemq 和 spring 整合的基础包 -->
<dependency>
    <groupId>org.apache.xbean</groupId>
    <artifactId>xbean-spring</artifactId>
    <version>3.16</version>
</dependency>
```

测试类(会创建三个消息发送到MQ队列里)

```java
package com.song.springbootactiviemq.test;


import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * @author song
 * @version 1.0
 * @date 2023/8/16 8:57
 * @description: TODO
 */

public class JmsProduce {
    public static final  String ACTIVEMQ_URL = "tcp://121.37.0.16:61616";
    public static final  String QUEUE_NAME = "queue";

    public static void main(String[] args) throws JMSException {
        // 1、创建连接工厂，按照给定的URL地址，采用默认的用户名和密码admin
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2、通过连接工厂获得连接对象，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3、创建会话session，两个参数，第一个是事务，第二个是签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4、创建目的地（具体是队列还是主题topic）
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5、创建消息的生产者
        MessageProducer producer = session.createProducer(queue);
        // 6、使用messageProducer生产3条消息发送到MQ的队列里面
        for (int i = 0; i < 3; i++) {
            //一个字符串
            TextMessage textMessage = session.createTextMessage("我是第" + i + "条message！");
            producer.send(textMessage);
        }
        // 7、关闭资源
        producer.close();
        session.close();
        connection.close();
        System.out.println("消息发布到MQ完成！");
    }
}
```



Number Of Pending Messages：
等待消费的消息，这个是未出队列的数量，公式=总接收数-总出队列数。

Number Of Consumers：
消费者数量，消费者端的消费者数量。

Messages Enqueued：
进队消息数，进队列的总消息量，包括出队列的。这个数只增不减。

Messages Dequeued：
出队消息数，可以理解为是消费者消费掉的数量。

## 2.4 消息队列（Queue）消费者的入门案例

```java
package com.song.springbootactiviemq.test;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * @author song
 * @version 1.0
 * @date 2023/8/16 9:08
 * @description: TODO
 */

public class JmsConsumer {
    public static final  String ACTIVEMQ_URL = "tcp://121.37.0.16:61616";
    public static final  String QUEUE_NAME = "queue";

    public static void main(String[] args) throws JMSException {
        // 1、创建连接工厂，按照给定的URL地址，采用默认的用户名和密码admin
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2、获得连接对象，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3、创建会话session，两个参数，第一个是事务，第二个是签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4、创建目的地（具体是队列还是主题topic）
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5、创建消息的消费者
        MessageConsumer consumer = session.createConsumer(queue);
        while (true) {
            // reveive() 一直等待接收消息，在能够接收到消息之前将一直阻塞。是同步阻塞方式 。
            TextMessage textMessage = (TextMessage) consumer.receive();
            if (textMessage != null) {
                System.out.println("消费者收到的消息：" + textMessage.getText());
            } else {
                break;
            }
        }
        // 7、关闭资源
        consumer.close();
        session.close();
        connection.close();
    }
}
```

## 2.5 异步监听式消费者（MessageListener）

```java
package com.song.springbootactiviemq.test;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

/**
 * @author song
 * @version 1.0
 * @date 2023/8/16 9:21
 * @description: 异步监听式消费者（MessageListener）
 */

public class MessageListener {

    public static final  String ACTIVEMQ_URL = "tcp://121.37.0.16:61616";
    public static final  String QUEUE_NAME = "queue";

    public static void main(String[] args) throws JMSException, IOException {
        // 1. 创建连接工厂,指定url
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2. 创建连接并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3. 创建会话对象 两个参数，第一个是事务，第二个是签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4、创建目的地（具体是队列还是主题topic）
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5、创建消息的消费者
        MessageConsumer consumer = session.createConsumer(queue);
         /* 通过监听的方式来消费消息，是异步非阻塞的方式消费消息。
           通过messageConsumer 的setMessageListener 注册一个监听器，
           当有消息发送来时，系统自动调用MessageListener 的 onMessage 方法处理消息
         */
        consumer.setMessageListener(message -> {
            if (message instanceof TextMessage) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    System.out.println("消费者收到的消息:" + textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });

        System.out.println("主线程运行到这了...");
        // 让主线程不要结束。因为一旦主线程结束了，其他的线程（如此处的监听消息的线程）也都会被迫结束。
        // 实际开发中，我们的程序会一直运行，这句代码都会省略。
        System.in.read();
        consumer.close();
        session.close();
        connection.close();
    }
}
```

## 2.6 队列消息(Queue)总结

1、两种消费方式

同步阻塞方式(receive)：

> 订阅者或接收者只用MessageConsumer的receive()方法来接收消息，receive方法在能接收到消息之前（或超时之前）将一直阻塞。

2、异步非阻塞方式（监听器MessageListener）:

> 订阅者或接收者通过MessageConsumer的setMessageListener(MessageListener listener)注册一个消息监听器，当消息到达之后，系统会自动调用监听器MessageListener的onMessage(Message message)方法。

3、队列的特点

在点对点的消息传递域中，目的地被称为队列（queue）

特点如下：

(1）每个消息只能有一个消费者，类似1对1的关系。好比个人快递自己领取自己的。
(2）消息的生产者和消费者之间没有时间上的相关性。无论消费者在生产者发送消息的时候是否处于运行状态，消费者都可以提取消息。好比我们的发送短信，发送者发送后不见得接收者会即收即看。
(3）消息被消费后队列中不会再存储，所以消费者不会消费到已经被消费掉的消息。

4、消息消费情况

情况1：只启动消费者1。

结果：消费者1会消费所有的数据。
情况2：先启动消费者1，再启动消费者2。

结果：消费者1消费所有的数据。消费者2不会消费到消息。
情况3：生产者发布6条消息，在此之前已经启动了消费者1和消费者2。**（正确情况）**

结果：**消费者1和消费者2平摊了消息。各自轮询的消费3条消息。**

## 2.7、 Topic主题入门案例

1、topic 介绍

在发布订阅消息传递域中，目的地被称为主题（topic）

发布/订阅消息传递域的特点如下：

（1）生产者将消息发布到topic中，每个消息可以有多个消费者，属于1：N的关系；

（2）生产者和消费者之间有时间上的相关性。订阅某一个主题的消费者只能消费自它订阅之后发布的消息。

（3）生产者生产时，topic不保存消息它是无状态的不落地，假如无人订阅就去生产，那就是一条废消息，所以，一般先启动消费者再启动生产者。



```java
// 生产者
public class JmsProduce_topic {
    public static final  String ACTIVEMQ_URL = "tcp://121.37.0.16:61616";
    public static final  String TOPIC_NAME = "topic01";

    public static void main(String[] args) throws JMSException {
        // 1、创建连接工厂，按照给定的URL地址，采用默认的用户名和密码admin
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2、通过连接工厂，获得连接对象，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();

        // 3、创建会话session，两个参数，第一个是事务，第二个是签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4、创建目的地（具体是队列还是主题topic）
        Topic topic = session.createTopic(TOPIC_NAME);
        // 5、创建消息的生产者
        MessageProducer producer = session.createProducer(topic);
        // 6、使用messageProducer生产3条消息发送到MQ的队列里面
        for (int i = 1; i <= 3; i++) {
            //一个字符串
            TextMessage textMessage = session.createTextMessage("我是第" + i + "条message！");
            //
            producer.send(textMessage);
        }
        // 7、关闭资源
        producer.close();
        session.close();
        connection.close();
        System.out.println("TOPIC--消息发布到MQ完成！");
    }
}

```

```java
//消费者
public class MessageListener_topic {
    public static final String ACTIVEMQ_URL = "tcp://121.37.0.16:61616";
    public static final String TOPIC_NAME = "topic";
    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("我是2号消费者");
        // 1、创建连接工厂，按照给定的URL地址，采用默认的用户名和密码admin
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2、获得连接对象，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3、创建会话session，两个参数，第一个是事务，第二个是签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4、创建目的地（具体是队列还是主题topic）
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageConsumer consumer = session.createConsumer(topic);
        /* 通过监听的方式来消费消息，是异步非阻塞的方式消费消息。
           通过messageConsumer 的setMessageListener 注册一个监听器，当有消息发送来时，系统自动调用MessageListener 的 onMessage 方法处理消息
         */
        consumer.setMessageListener(new MessageListener() {
            public void onMessage(Message message) {
                //  instanceof 判断是否A对象是否是B类的子类
                if (message != null && message instanceof TextMessage) {
                    TextMessage textMessage = (TextMessage) message;
                    try {
                        System.out.println("消费者的消息：" + textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        System.out.println("主线程运行到这了...");
        // 让主线程不要结束。因为一旦主线程结束了，其他的线程（如此处的监听消息的线程）也都会被迫结束。
        // 实际开发中，我们的程序会一直运行，这句代码都会省略。
        System.in.read();
        consumer.close();
        session.close();
        connection.close();
    }
}
```

| 比较项目   | Topic模式队列                                                | Queue模式队列                                                |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 工作模式   | "订阅-发布"模式，如果当前没有订阅者，消息将被丢弃，如果有多个订阅者，那么这些订阅者都会受到消息。 | "负载均衡"模式，如果当前没有消费者，消息也不会丢弃；如果有多个消费者，那么一条消息也只会发送给其中一个消费者，并且要求消费者ack信息。 |
| 有无状态   | 无状态                                                       | Queue数据默认会在mq服务器上以文件形式保存，比如ActiveMQ一般保存在$AMQ_HOME\data\kr-store\data下面，也可以配置成DB存储。 |
| 传递完整性 | 如果没有订阅者，消息会被丢弃。                               | 消息不会丢弃                                                 |
| 处理效率   | 由于消息要按照订阅者的数量进行复制，所以处理性能会随着订阅者的增加而明显降低，并且还要结合不同消息协议自身的性能差异。 | 由于一条消息只发送给一个消费者，所以就算消费者再多，性能也不会有明显降低，当然不同消息协议的具体性能也是有差异的。 |

# 3.JMS规范和落地产品



## 3.1 JMS是什么？

什么是Java消息服务？

Java消息服务指的是两个应用程序之间进行异步通信的API，它为标准协议和消息服务提供了一组通用接口，包括创建、发送、读取消息等，用于支持Java应用程序开发。在JavaEE中，当两个应用程序使用JMS进行通信时，它们之间不是直接相连的，而是通过一个共同的消息收发服务组件关联起来以达到解耦/异步削峰的效果。

![img](file:///C:/Users/sxq/AppData/Local/Packages/oice_16_974fa576_32c1d314_124e/AC/Temp/msohtmlclip1/01/clip_image002.jpg)

## 3.2 JMS组成部分

JMS Provider : 实现JMS接口和规范的消息中间件，也就是我们说的MQ服务器。

JMS Producer ： 消息生产者，创建和发送JMS消息的客户端应用。

JMS Consumer ：消息消费者，接收和处理JMS消息的客户端应用。

JMS Message ：消息包含 消息头、消息体、消息属性。

## 3.3 JMS的消息头

JMSDestination：消息目的地

JMSDeliveryMode：消息持久化模式

JMSExpiration：消息过期时间

JMSPriority：消息的优先级

> （消息优先级，从0-9十个级别，0-4是普通消息 5-9是加急消息。JMS不要求MQ严格按照这十个优先级发送消息但必须保证加急消息要先于普通消息到达。默认是4级。）

JMSMessageID：消息的唯一标识符。

后面我们会介绍如何解决幂等性。

说明： 消息的生产者可以set这些属性，消息的消费者可以get这些属性。

这些属性在send方法里面也可以设置。

## 3.4 JMS的消息体

封装具体的消息数据

5种消息体格式

>**TextMessage** : 普通字符串消息，包含一个string。
>
>**MapMessage** : 一个Map类型的消息，key为string类型，而值为Java**的基本类型**。
>
>BytesMessage : 二进制数组消息，包含一个byte[]。
>
>StreamMessage : Java数据流消息，用标准流操作来顺序的填充和读取。
>
>objectMessagec : 对象消息，包含一个可序列化的Java对象。

发送和接受的消息体类型必须一致对应

## 3.5 JMS的消息属性

如果需要除消息头字段之外的值，那么可以使用消息属性。他是识别/去重/重点标注等操作，非常有用的方法。

他们是以属性名和属性值对的形式制定的。可以将属性是为消息头得扩展，属性指定一些消息头没有包括的附加信息，比如可以在属性里指定消息选择器。消息的属性就像可以分配给一条消息的附加消息头一样。它们允许开发者添加有关消息的不透明附加信息。它们还用于暴露消息选择器在消息过滤时使用的数据。

生产消息时：

```java
//调用Message的setStringProperty()方法，就能设置消息属性。根据value的数据类型的不同，有相应的API。
textMessage.setStringProperty(**"From"**,**"ZhangSan@qq.com"**);
textMessage.setByteProperty(**"Spec"**, (**byte**) 1);
textMessage.setBooleanProperty(**"Invalide"**,**true**);
```
消费消息时:

```java
System.out.println(**"消息属性："**+textMessage.getStringProperty(**"From"**));
System.out.println(**"消息属性："**+textMessage.getByteProperty(**"Spec"**));
System.out.println(**"消息属性："**+textMessage.getBooleanProperty(**"Invalide"**));
```

## 3.6 JMS消息持久化

### 3.6.1 Queue消息持久化

什么是持久化消息？

保证消息只被传送一次和成功使用一次。在持久性消息传送至目标时，消息服务将其放入持久性数据存储。如果消息服务由于某种原因导致失败，它可以恢复此消息并将此消息传送至相应的消费者。虽然这样增加了消息传送的开销，但却增加了可靠性。

我的理解：在消息生产者将消息成功发送给MQ消息中间件之后。无论是出现任何问题，如：MQ服务器宕机、消费者掉线等。都保证（topic要之前注册过，queue不用）消息消费者，能够成功消费消息。如果消息生产者发送消息就失败了，那么消费者也不会消费到该消息。

```java
//非持久化
textMessage.setJMSDeliveryMode(DeliveryMode.NON_PERSISTENT);
```

```java
//持久化
textMessage.setJMSDeliveryMode(DeliveryMode.PERSISTENT);
```

结论：Queue消息默认持久化

### 3.6.2 Topic消息持久化

**topic默认就是非持久化的**，因为生产者生产消息时，消费者也要在线，这样消费者才能消费到消息。

topic消息持久化，只要消费者向MQ服务器注册过，所有生产者发布成功的消息，该消费者都能收到，不管是MQ服务器宕机还是消费者不在线。

 

注意：

1. 一定要先运行一次消费者，等于向MQ注册，类似我订阅了这个主题。
2. .  然后再运行生产者发送消息。

3. 之后无论消费者是否在线，都会收到消息。如果不在线的话，下次连接的时候，会把没有收过的消息都接收过来。

```java
// 持久化topic 的消息生产者
public class JmsProduce_persistence {

    public static final String ACTIVEMQ_URL = "tcp://192.168.17.3:61616";
    public static final String TOPIC_NAME = "topic01";

    public static void main(String[] args) throws  Exception{
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        javax.jms.Connection connection = activeMQConnectionFactory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageProducer messageProducer = session.createProducer(topic);

        // 设置持久化topic 
        messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
        // 设置持久化topic之后再，启动连接
        connection.start();
        for (int i = 1; i < 4 ; i++) {
            TextMessage textMessage = session.createTextMessage("topic_name--" + i);
            messageProducer.send(textMessage);
            MapMessage mapMessage = session.createMapMessage();
        }
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("  **** TOPIC_NAME消息发送到MQ完成 ****");
    }
}
```

```java
// 持久化topic 的消息消费者
public class JmsConsummer_persistence {
    public static final String ACTIVEMQ_URL = "tcp://192.168.17.3:61616";
    public static final String TOPIC_NAME = "topic01";

    public static void main(String[] args) throws Exception{
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
		// 设置客户端ID。向MQ服务器注册自己的名称
        connection.setClientID("marrry");
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
		// 创建一个topic订阅者对象。一参是topic，二参是订阅者名称
        TopicSubscriber topicSubscriber = session.createDurableSubscriber(topic,"remark...");
         // 之后再开启连接
        connection.start();
        Message message = topicSubscriber.receive();
         while (null != message){
             TextMessage textMessage = (TextMessage)message;
             System.out.println(" 收到的持久化 topic ："+textMessage.getText());
             message = topicSubscriber.receive();
         }
        session.close();
        connection.close();
    }
}
```

## 3.7 JMS消息的事务（偏生产者）

(1) 生产者开启事务后，执行commit方法，这批消息才真正的被提交。不执行commit方法，这批消息不会提交。执行rollback方法，之前的消息会回滚掉。生产者的事务机制，要高于签收机制，当生产者开启事务，签收机制不再重要。

```java
public class Jms_TX_Producer {
    private static final String ACTIVEMQ_URL = "tcp://192.168.10.130:61616";
    private static final String ACTIVEMQ_QUEUE_NAME = "Queue-TX";

    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //1.创建会话session，两个参数transacted=事务,acknowledgeMode=确认模式(签收)
        //设置为开启事务
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(ACTIVEMQ_QUEUE_NAME);
        MessageProducer producer = session.createProducer(queue);
        try {
            for (int i = 0; i < 3; i++) {
                TextMessage textMessage = session.createTextMessage("tx msg--" + i);
              producer.send(textMessage);
if(i == 2){
                    throw new RuntimeException("GG.....");
                }
            }
            // 2. 开启事务后，使用commit提交事务，这样这批消息才能真正的被提交。
            session.commit();
            System.out.println("消息发送完成");
        } catch (Exception e) {
            System.out.println("出现异常,消息回滚");
            // 3. 工作中一般，当代码出错，我们在catch代码块中回滚。这样这批发送的消息就能回滚。
            session.rollback();
        } finally {
            //4. 关闭资源
            producer.close();
            session.close();
            connection.close();
        }
    }
}
```

(2) 消费者开启事务后，执行commit方法，这批消息才算真正的被消费。不执行commit方法，这些消息不会标记已消费，下次还会被消费。执行rollback方法，是不能回滚之前执行过的业务逻辑，但是能够回滚之前的消息，回滚后的消息，下次还会被消费。

**消费者利用commit和rollback方法，甚至能够违反一个消费者只能消费一次消息的原理。**

```java
public class Jms_TX_Consumer {
    private static final String ACTIVEMQ_URL = "tcp://118.24.20.3:61626";
    private static final String ACTIVEMQ_QUEUE_NAME = "Queue-TX";

    public static void main(String[] args) throws JMSException, IOException {
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 创建会话session，两个参数transacted=事务,acknowledgeMode=确认模式(签收)
        // 消费者开启了事务就必须手动提交，不然会重复消费消息
        final Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(ACTIVEMQ_QUEUE_NAME);
        MessageConsumer messageConsumer = session.createConsumer(queue);
        messageConsumer.setMessageListener(new MessageListener() {
            int a = 0;
            @Override
            public void onMessage(Message message) {
                if (message instanceof TextMessage) {
                    try {
                        TextMessage textMessage = (TextMessage) message;
                        System.out.println("***消费者接收到的消息:   " + textMessage.getText());
                        if(a == 0){
                            System.out.println("commit");
                            session.commit();
                        }
                        if (a == 2) {
                            System.out.println("rollback");
                            session.rollback();
                        }
                        a++;
                    } catch (Exception e) {
                        System.out.println("出现异常，消费失败，放弃消费");
                        try {
                            session.rollback();
                        } catch (JMSException ex) {
                            ex.printStackTrace();
                        }
                    }
                }
            }
        });
        //关闭资源
        System.in.read();
        messageConsumer.close();
        session.close();
        connection.close();
    }
}
```

(3) 问：消费者和生产者需要同时操作事务才行吗？  

答：消费者和生产者的事务，完全没有关联，各自是各自的事务。

## 3.8 JMS消息的签收机制（偏消费者）

**一、**签收的几种方式

①　自动签收（**Session.AUTO_ACKNOWLEDGE**）：该方式是默认的。该种方式，无需我们程序做任何操作，框架会帮我们自动签收收到的消息。

②　手动签收（**Session.CLIENT_ACKNOWLEDGE**）：手动签收。该种方式，需要我们手动调用Message.acknowledge()，来签收消息。如果不签收消息，该消息会被我们反复消费，只到被签收。

③　允许重复消息（Session.DUPS_OK_ACKNOWLEDGE）：多线程或多个消费者同时消费到一个消息，因为线程不安全，可能会重复消费。该种方式很少使用到。

④　事务下的签收（Session.SESSION_TRANSACTED）：开始事务的情况下，可以使用该方式。该种方式很少使用到。

 

**二、**事务和签收的关系总结

①　在事务性会话中，当一个事务被成功提交则消息被自动签收。如果事务回滚，则消息会被再次传送。事务优先于签收，开始事务后，签收机制不再起任何作用。

②　非事务性会话中，消息何时被确认取决于创建会话时的应答模式(自动签收无所谓，手动签收必须调acknowledge()方法)。

③　生产者事务开启，只有commit后才能将全部消息变为已消费。

④　事务偏向生产者，签收偏向消费者。也就是说，生产者使用事务更好点，消费者使用签收机制更好点。

##  3.9 JMS点对点总结

点对点模型是基于队列的，生产者发消息到队列，消费者从队列接收消息，队列的存在使得消息的异步传输成为可能。和我们平时给朋友发送短信类似。

如果在Session关闭时有部分消息己被收到但还没有被签收(acknowledged),那当消费者下次连接到相同的队列时，这些消息还会被再次接收。（消息重复）

队列可以长久地保存消息直到消费者收到消息。消费者不需要因为担心消息会丢失而时刻和队列保持激活的连接状态，充分体现了异步传输模式的优势。

## 3.10 JMS发布订阅总结

(1) JMS的发布订阅总结

JMS Pub/Sub 模型定义了如何向一个内容节点发布和订阅消息，这些节点被称作topic。

主题可以被认为是消息的传输中介，发布者（publisher）发布消息到主题，订阅者（subscribe）从主题订阅消息。

主题使得消息订阅者和消息发布者保持互相独立不需要解除即可保证消息的传送

 

(2) 非持久订阅

非持久订阅只有当客户端处于激活状态，也就是和MQ保持连接状态才能收发到某个主题的消息。

如果消费者处于离线状态，生产者发送的主题消息将会丢失作废，消费者永远不会收到。

  一句话：先订阅注册才能接受到发布，只给订阅者发布消息。

 

(3) 持久订阅

客户端首先向MQ注册一个自己的身份ID识别号，当这个客户端处于离线时，生产者会为这个ID保存所有发送到主题的消息，当客户再次连接到MQ的时候，会根据消费者的ID得到所有当自己处于离线时发送到主题的消息

当非持久订阅状态下，不能恢复或重新派送一个未签收的消息。

持久订阅才能恢复或重新派送一个未签收的消息。

 

(4) 非持久和持久化订阅如何选择

当所有的消息必须被接收，则用持久化订阅。当消息丢失能够被容忍，则用非持久订阅。



#  4.ActiveMQ的broker

## 4.1 broker是什么？

相当于一个ActiveMQ服务器实例。说白了，Broker其实就是实现了用代码的形式启动ActiveMQ将MQ嵌入到Java代码中，以便随时用随时启动，在用的时候再去启动这样能节省了资源，也保证了可用性。这种方式，我们实际开发中很少采用，因为他缺少太多了东西，如：日志，数据存储等等。



## 4.2 java中启动一个borker

添加依赖

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.1</version>
</dependency>
```

创建broker启动类

```java
import org.apache.activemq.broker.BrokerService;

/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 11:22
 * @description: 嵌入式broker的启动类
 */

public class EmbedBroker {
    public static void main(String[] args) throws Exception {
        //ActiveMQ也支持在vm中通信基于嵌入的broker
        BrokerService brokerService = new BrokerService();
        brokerService.setUseJmx(true);
        brokerService.addConnector("tcp://localhost:61616");
        brokerService.start();
    }
}

```

# 5.Spring整合ActiveMQ

大佬的理解：我们之前介绍的内容也很重要，他更灵活，他支持各种自定义功能，可以满足我们工作中复杂的需求。很多activemq的功能，我们要看官方文档或者博客，这些功能大多是在上面代码的基础上修改完善的。如果非要把这些功能强行整合到spring，就有些缘木求鱼了。我认为另一种方式整合spring更好，就是将上面的类注入到Spring中，其他不变。这样既能保持原生的代码，又能集成到spring。

下面我们将的Spring和SpringBoot整合ActiveMQ也重要，他给我们提供了一个模板，简化了代码，减少我们工作中遇到坑，能够满足开发中90%以上的功能。

## 5.1 添加依赖

```xml
<dependencies>
   <!-- activemq核心依赖包  必须-->
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-all</artifactId>
        <version>5.10.0</version>
    </dependency>
    <!--  嵌入式activemq的broker所需要的依赖包   -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.10.1</version>
    </dependency>
    <!-- activemq连接池  必须-->
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-pool</artifactId>
        <version>5.15.10</version>
    </dependency>
    <!-- spring支持jms的包 必须-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jms</artifactId>
        <version>5.2.1.RELEASE</version>
    </dependency>
    <!--spring相关依赖包-->
    <dependency>
        <groupId>org.apache.xbean</groupId>
        <artifactId>xbean-spring</artifactId>
        <version>4.15</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>5.2.1.RELEASE</version>
    </dependency>
    <!-- Spring核心依赖 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.3.23.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.23.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>4.3.23.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-orm</artifactId>
        <version>4.3.23.RELEASE</version>
    </dependency>
</dependencies>
```



## 5.2 编写spring-activemq.xml

在src/main/resources/spring-activemq.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--  开启包的自动扫描  -->
    <context:component-scan base-package="com.activemq.demo"/>
    <!--  配置生产者  -->
    <bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <property name="connectionFactory">
            <!--      正真可以生产Connection的ConnectionFactory,由对应的JMS服务商提供      -->
            <bean class="org.apache.activemq.spring.ActiveMQConnectionFactory">
                <property name="brokerURL" value="tcp://192.168.10.130:61616"/>
            </bean>
        </property>
        <property name="maxConnections" value="100"/>
    </bean>

    <!--  这个是队列目的地,点对点的Queue  -->
    <bean id="destinationQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <!--    通过构造注入Queue名    -->
        <constructor-arg index="0" value="spring-active-queue"/>
    </bean>

    <!--  这个是队列目的地,  发布订阅的主题Topic-->
    <bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg index="0" value="spring-active-topic"/>
    </bean>

    <!--  Spring提供的JMS工具类,他可以进行消息发送,接收等  -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!--    传入连接工厂    -->
        <property name="connectionFactory" ref="connectionFactory"/>
        <!--    传入目的地    -->
        <property name="defaultDestination" ref="destinationQueue"/>
        <!--    消息自动转换器    -->
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>
</beans>
```

## 5.3 编写代码

### 5.3.1 Queue生产者

```java

import org.apache.xbean.spring.context.ClassPathXmlApplicationContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;
 
@Service
public class SpringMQ_Producer {
 
 
    private JmsTemplate jmsTemplate;
 
    @Autowired
    public void setJmsTemplate(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }
 
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("Application.xml");
        SpringMQ_Producer springMQ_producer = applicationContext.getBean(SpringMQ_Producer.class);
 
        springMQ_producer.jmsTemplate.send(session -> session.createTextMessage("***Spring和ActiveMQ的整合case111....."));
        System.out.println("********send task over");
    }
}

```


### 5.3.2 Queue消费者

```java

import org.apache.xbean.spring.context.ClassPathXmlApplicationContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;
 
@Service
public class SpringMQ_Consumer {
 
 
    private JmsTemplate jmsTemplate;
 
    @Autowired
    public void setJmsTemplate(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }
 
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("Application.xml");
        SpringMQ_Consumer springMQ_consumer = applicationContext.getBean(SpringMQ_Consumer.class);
        String returnValue = (String) springMQ_consumer.jmsTemplate.receiveAndConvert();
        System.out.println("****消费者收到的消息:   " + returnValue);
    }
}
```

### 5.3.3 Topic生产者

```java
import org.apache.xbean.spring.context.ClassPathXmlApplicationContext;
import org.springframework.context.ApplicationContext;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;
 
import javax.jms.Destination;
 
@Service
public class SpringMQ_Topic_Producer {
    private JmsTemplate jmsTemplate;
 
    public SpringMQ_Topic_Producer(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }
 
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("Application.xml");
        SpringMQ_Topic_Producer springMQ_topic_producer = applicationContext.getBean(SpringMQ_Topic_Producer.class);
        //直接调用application.xml里面创建的destinationTopic这个bean设置为目的地就行了
        springMQ_topic_producer.jmsTemplate.setDefaultDestination(((Destination) applicationContext.getBean("destinationTopic")));
        springMQ_topic_producer.jmsTemplate.send(session -> session.createTextMessage("***Spring和ActiveMQ的整合TopicCase111....."));
    }
}
```

### 5.3.4 Topic消费者

```java
import org.apache.xbean.spring.context.ClassPathXmlApplicationContext;
import org.springframework.context.ApplicationContext;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;
 
import javax.jms.Destination;
 
@Service
public class SpringMQ_Topic_Consumer {
    private JmsTemplate jmsTemplate;
 
    public SpringMQ_Topic_Consumer(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }
 
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("Application.xml");
        SpringMQ_Topic_Consumer springMQConsumer = applicationContext.getBean(SpringMQ_Topic_Consumer.class);
        //直接调用application.xml里面创建的destinationTopic这个bean设置为目的地就行了
        springMQConsumer.jmsTemplate.setDefaultDestination(((Destination) applicationContext.getBean("destinationTopic")));
        String returnValue = (String) springMQConsumer.jmsTemplate.receiveAndConvert();
        System.out.println("****消费者收到的消息:   " + returnValue);
    }
}
```

### 5.3.5 Spring中配置自启动消费者

在spring-activemq.xml中新增

```xml
 <!--  配置Jms消息监听器  -->
    <bean id="defaultMessageListenerContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <!--  Jms连接的工厂     -->
        <property name="connectionFactory" ref="connectionFactory"/>
        <!--   设置默认的监听目的地     -->
        <property name="destination" ref="destinationTopic"/>
        <!--  指定自己实现了MessageListener的类     -->
        <property name="messageListener" ref="myMessageListener"/>
    </bean>

```

新增监听类

```java

import org.springframework.stereotype.Component;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

/**
 * 实现MessageListener的类,需要把这个类交给xml配置里面的DefaultMessageListenerContainer管理
 */
@Component
public class MyMessageListener implements MessageListener {

    @Override
    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            TextMessage textMessage = (TextMessage) message;
            try {
                System.out.println("消费者收到的消息" + textMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}

```

消费者配置了自动监听，就相当于在spring里面后台运行，有消息就运行我们实现监听类里面的方法。

# 6.SpringBoot整合ActiveMQ

## 6.1 新建项目

boot_mq_produce: 生产者

boot_mq_consumer:消费者

## 6.2 引入依赖

两个工程中都要引入

```xml
 <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>
```



## 6.3 创建Queue生产者工程
### 6.3.1 编写yml文件

```yaml
#Springboot启动端口
server:
  port: 8080

#ActiveMQ配置
spring:
  activemq:
    broker-url: tcp://192.168.10.130:61616 #ActiveMQ服务器IP
    user: admin #ActiveMQ连接用户名
    password: 123456 #ActiveMQ连接密码
  jms:
    #指定连接队列还是主题
    pub-sub-domain: false # false = Queue |  true = Topic

#定义服务上的队列名
myQueueName: springboot-activemq-queue
```

### 6.3.2 编写配置目的地的bean

```java
import org.apache.activemq.command.ActiveMQQueue;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.stereotype.Component;

import javax.jms.Queue;

/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 14:26
 * @description: 配置目的地的bean和开启springboot的jms功能
 */

// 让spring管理的注解，相当于spring中在xml 中写了个bean
@Component
@EnableJms  //开启JMS
public class ConfigBean {
    // 注入配置文件中的 myqueue
    @Value("${myQueueName}")
    private String myQueue ;

    @Bean   // bean id=""  class="…"
    public Queue queue(){
        return new ActiveMQQueue(myQueue);
    }
}
```

### 6.3.3 创建消息的Service

```java
/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 14:30
 * @description: 生产消息
 */
@Component
public class Queue_Produce {

    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    @Autowired
    private Queue queue;

    public void produceMsg() {
        jmsMessagingTemplate.convertAndSend(queue,"*******"+UUID.randomUUID().toString().substring(0,6));
    }
    
      // 定时任务。每3秒执行一次。非必须代码，仅为演示。 需要在主启动类添加@EnableScheduling开启定时任务
    @Scheduled(fixedDelay = 3000)
    public void produceMessageScheduled(){
        produceMsg();
        System.out.println("============produceMessageScheduled() send ok");
    }
}
```

## 6.4 创建Queue消费者工程

### 6.4.1 编写工程YML文件

```xml
#Springboot启动端口
server:
  port: 8888

#ActiveMQ配置
spring:
  activemq:
    broker-url: tcp://192.168.10.130:61616 #ActiveMQ服务器IP
    user: admin #ActiveMQ连接用户名
    password: 123456 #ActiveMQ连接密码
  jms:
    #指定连接队列还是主题
    pub-sub-domain: false # false = Queue |  true = Topic

#定义服务上的队列名
myQueueName: springboot-activemq-queue
```

### 6.4.2 注册消费者监听器

```java
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

import javax.jms.TextMessage;

/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 15:08
 * @description: 消息监听
 */
@Component
public class Queue_Consumer {

    // 注册一个监听器。destination指定监听的主题。
    @JmsListener(destination = "${myQueueName}")
    public void receive(TextMessage textMessage) throws  Exception{
        System.out.println(" ***  消费者收到消息  ***"+textMessage.getText());
    }
}

```

## 6.5 创建Topic生产者工程

### 6.5.1 编写yml文件

```xml
#Springboot启动端口
server:
  port: 7777

#ActiveMQ配置
spring:
  activemq:
    broker-url: tcp://121.37.0.16:61616 #ActiveMQ服务器IP
    user: admin #ActiveMQ连接用户名
    password: 123456 #ActiveMQ连接密码
  jms:
    #指定连接队列还是主题
    pub-sub-domain: true # false = Queue |  true = Topic

#定义服务上的队列名
myTopicName: springboot-activemq-topic
```

### 6.5.2 编写配置目的地的bean

```java
import org.apache.activemq.command.ActiveMQQueue;
import org.apache.activemq.command.ActiveMQTopic;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.stereotype.Component;

import javax.jms.Queue;
import javax.jms.Topic;

/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 14:26
 * @description: 配置目的地的bean和开启springboot的jms功能
 */

// 让spring管理的注解，相当于spring中在xml 中写了个bean
@Component
@EnableJms
public class ConfigBean {
    // 注入配置文件中的 myqueue
    @Value("${myTopicName}")
    private String myTopic ;

    @Bean   // bean id=""  class="…"
    public Topic topic(){
        return new ActiveMQTopic(myTopic);
    }
}

```

### 6.5.3 创建消息的Service

```java
import javax.jms.Topic;
import java.util.UUID;

/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 15:28
 * @description: TODO
 */
@Component
public class TopicProduce {

    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Topic topic;

    @Scheduled(fixedDelay = 3000)
    public void topicSend(){
        jmsMessagingTemplate.convertAndSend(topic,"*******"+ UUID.randomUUID().toString().substring(0,6));
    }
}
```

## 6.6 创建Topic消费者工程

### 6.6.1 编写yml文件

```xml
#Springboot启动端口
server:
  port: 8888

#ActiveMQ配置
spring:
  activemq:
    broker-url: tcp://121.37.0.16:61616 #ActiveMQ服务器IP
    user: admin #ActiveMQ连接用户名
    password: 123456 #ActiveMQ连接密码
  jms:
    #指定连接队列还是主题
    pub-sub-domain: true # false = Queue |  true = Topic

#定义服务上的队列名
myTopicName: springboot-activemq-topic
```

### 6.6.2 注册消费监听器

```java
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;
import javax.jms.TextMessage;

@Component
public class Topic_Consummer {

    @JmsListener(destination = "${myTopicName}")
    public void receive(TextMessage textMessage) throws  Exception{
        System.out.println("消费者01受到订阅的主题："+textMessage.getText());
    }
}
```

# 7.ActiveMQ的传输协议

## 7.1 ActiveMQ协议简介

ActiveMQ支持的client-broker通讯协议有：TVP、NIO、UDP、SSL、Http(s)、VM。

其中配置Transport Connector的文件在ActiveMQ安装目录的conf/activemq.xml中的<transportConnectors>标签之内。

activemq传输协议的官方文档：http://activemq.apache.org/configuring-version-5-transports.html

查看使用的协议 ：%activeMQ安装目录%/conf/activemq.xml

```xml
<transportConnectors>
    <<!--tcp-->
    <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>

    <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>

    <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>

          <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1884?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>

          <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    
</transportConnectors>
```

在上文给出的配置信息中，URI描述信息的头部都是采用协议名称：例如

描述amqp协议的监听端口时，采用的URI描述格式为“amqp://······”；

描述Stomp协议的监听端口时，采用URI描述格式为“stomp://······”；

唯独在进行openwire协议描述时，URI头却采用的“tcp://······”。这是因为ActiveMQ中默认的消息协议就是openwire

## 7.2 支持的协议

除了tcp和nio协议，其他的了解就行。各种协议有各自擅长该协议的中间件，工作中一般不会使用activemq去实现这些协议。如： mqtt是物联网专用协议，采用的中间件一般是mosquito。ws是websocket的协议，是和前端对接常用的，一般在java代码中内嵌一个基站（中间件）。stomp好像是邮箱使用的协议的，各大邮箱公司都有基站（中间件）。

注意：协议不同，我们的代码都会不同。



ActiveMQ支持的网络协议

| 协议    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| TCP     | 默认的协议，性能相对可以                                     |
| NIO     | 基于TCP协议之上的，进行了扩展和优化，具有更的扩展性          |
| UDP     | 性能比TCP更好，但是不具有可靠性                              |
| SSL     | 安全链接                                                     |
| HTTP(S) | 基于HTTP或者HTTPS                                            |
| VM      | VM本身不是协议，当客户端和代理在同一个Java虚拟机（VM）中运行时，他们之间需要通信，但不想占用网络通道，而是直接通信，可以使用该方式。 |

### 7.2.1 TCP协议

(1) Transmission Control Protocol(TCP)是默认的。TCP的Client监听端口61616

(2) 在网络传输数据前，必须要先序列化数据，消息是通过一个叫wire protocol的来序列化成字节流。

(3) TCP连接的URI形式如：tcp://HostName:port?key=value&key=value，后面的参数是可选的。

(4) TCP传输的的优点：

TCP协议传输可靠性高，稳定性强

高效率：字节流方式传递，效率很高

有效性、可用性：应用广泛，支持任何平台

(5) 关于Transport协议的可选配置参数可以参考官网http://activemq.apache.org/tcp-transport-reference

```xml
 <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
```



### 7.2.2 NIO协议

(1) New I/O API Protocol(NIO)

(2) NIO协议和TCP协议类似，但NIO更侧重于底层的访问操作。它允许开发人员对同一资源可有更多的client调用和服务器端有更多的负载。

(3) 适合使用NIO协议的场景：

可能有大量的Client去连接到Broker上，一般情况下，大量的Client去连接Broker是被操作系统的线程所限制的。因此，NIO的实现比TCP需要更少的线程去运行，所以建议使用NIO协议。

可能对于Broker有一个很迟钝的网络传输，NIO比TCP提供更好的性能。

(4) NIO连接的URI形式：nio://hostname:port?key=value&key=value

(5) 关于Transport协议的可选配置参数可以参考官网http://activemq.apache.org/configuring-version-5-transports.html

```xml
<transportCornector name="nio" uri="nio://localhost:61618?trace=true"/>
```

### 7.2.3 AMQP协议

### 7.2.4 STOMP协议

### 7.2.5 MQTT协议

### 7.2.6 WS协议

## 7.3 NIO协议案例

ActiveMQ这些协议传输的底层默认都是使用BIO网络的IO模型。只有当我们指定使用nio才使用NIO的IO模型。

### 7.3.1 修改activemq.xml

①　修改配置文件activemq.xml在 <transportConnectors>节点下添加如下内容：

```xml
<transportConnector name="nio" uri="nio://0.0.0.0:61618?trace=true" />
```

②　修改完成后重启activemq:  

```dockerfile
docker restart activemq
```

③　查看管理后台，可以看到页面多了nio

### 7.3.2 测试nio协议

生产者

```java

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 16:46
 * @description: TODO
 */

public class JMSProduceNIO {
    private static final String ACTIVEMQ_URL = "nio://121.37.0.16:61618";
    private static final String QUEUE_NAME = "Queue-NIO";

    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory();
        connectionFactory.setBrokerURL(ACTIVEMQ_URL);
        Connection connection = connectionFactory.createConnection();
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(QUEUE_NAME);
        MessageProducer messageProducer = session.createProducer(queue);
        connection.start();
        for (int i = 0; i < 3; i++) {
            TextMessage textMessage = session.createTextMessage("测试Nio" + i);
            messageProducer.send(textMessage);
        }
        session.commit();
        System.out.println("消息发送完成");
        messageProducer.close();
        session.close();
        connection.close();
    }

}
```

消费者

```java
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

/**
 * @author song
 * @version 1.0
 * @date 2023/9/8 16:46
 * @description: TODO
 */

public class JMSConsumerNIO {
    private static final String ACTIVEMQ_URL = "nio://121.37.0.16:61618";
    private static final String QUEUE_NAME = "Queue-NIO";

    public static void main(String[] args) throws JMSException, IOException {
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory();
        activeMQConnectionFactory.setBrokerURL(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(QUEUE_NAME);
        MessageConsumer messageConsumer = session.createConsumer(queue);
        messageConsumer.setMessageListener(message -> {
            TextMessage textMessage = (TextMessage) message;
            try {
                System.out.println("消费者接收到的消息:  " + textMessage.getText());
                session.commit();
            } catch (JMSException e) {
                e.printStackTrace();
            }
        });
        System.in.read();
    }

}
```

## 7.4 NIO案例增强

上诉NIO性能不错了，如何进一步优化？

上面是Openwire协议传输底层使用NIO网络IO模型。 如何让其他协议传输底层也使用NIO网络IO模型呢？

URI格式以"nio"开头，代表这个端口使用TCP协议为基础的NIO网络模型。
但是这样的设置方式，只能使这个端口支持Openwire协议。

解决：使用auto关键字，使用"+"符号来为端口设置多种特性



原来的

```xml
<transportConnector name="auto+nio" uri="auto+nio://localhost:5671"/>
```

增强后

```xml
<transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61608?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600&amp;org.apache.activemq.transport.nio.SelectorManager.corePoolSize=20&amp;org.apache.activemq.transport.nio.Se1ectorManager.maximumPoo1Size=50"/>
```

auto	: 针对所有的协议，他会识别我们是什么协议。

nio		：使用NIO网络IO模型

修改配置文件后重启activemq。



使用nio模型的tcp协议生产者。

```java
public class Jms_TX_Producer {
    private static final String ACTIVEMQ_URL = "tcp://118.24.20.3:61608";
    private static final String ACTIVEMQ_QUEUE_NAME = "auto-nio";

    public static void main(String[] args) throws JMSException {
         ......
    }
}
```

使用nio模型的tcp协议消费者。

```java
public class Jms_TX_Consumer {
    private static final String ACTIVEMQ_URL = "tcp://118.24.20.3:61608";
    private static final String ACTIVEMQ_QUEUE_NAME = "auto-nio";

    public static void main(String[] args) throws JMSException, IOException {
       ......
    }
}
```

使用nio模型的nio协议生产者。

```java
public class Jms_TX_Producer {
    private static final String ACTIVEMQ_URL = "nio://118.24.20.3:61608";
    private static final String ACTIVEMQ_QUEUE_NAME = "auto-nio";

    public static void main(String[] args) throws JMSException {
       ......
    }
}
```

使用nio模型的nio协议消费者。

```java
public class Jms_TX_Consumer {
    private static final String ACTIVEMQ_URL = "nio://118.24.20.3:61608";
    private static final String ACTIVEMQ_QUEUE_NAME = "auto-nio";

    public static void main(String[] args) throws JMSException, IOException {
        ......
    }
}
```

# 8.ActiveMQ消息存储和持久化

## 8.1 持久化是什么？

为了避免意外宕机以后丢失信息，需要做到重启后可以恢复消息队列，消息系统一半都会采用持久化机制。
ActiveMQ的消息持久化机制有JDBC，AMQ，KahaDB和LevelDB，无论使用哪种持久化方式，消息的存储逻辑都是一致的。

就是在发送者将消息发送出去后，消息中心首先将消息存储到本地数据文件、内存数据库或者远程数据库等。再试图将消息发给接收者，成功则将消息从存储中删除，失败则继续尝试尝试发送。

消息中心启动以后，要先检查指定的存储位置是否有未成功发送的消息，如果有，则会先把存储位置中的消息发出去。

一句话：ActiveMQ宕机了，消息不会丢失的机制。

## 8.2 ActiveMQ支持的持久化方式

1. AMQ Message Store(了解)
2. kahaDB(默认)
3. JDBC消息存储
4. LevelDB消息存储

