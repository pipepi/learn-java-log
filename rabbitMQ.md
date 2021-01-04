# 工作模式
- RPC 【借助消息队列实现异步通信】【客户端【生产者，消费者】，服务端【消费者，生产者】】
  - 结构
    - 请求队列，响应队列
    - 客户端生产消息，发送到请求队列；
    - 服务端消费请求队列；并生产响应消息，发送到响应队列
    - 客户端消费响应队列
- head
  - 结构
    - 不用routKey，同headsMap【k,v】
- topics 【通配符工作模式，路由key支持通配符*【一个词】#【零个或一个或多个词【多个词用。分割】】】
  - 结构
    - 同routing模式
- routing【区域范围：交换机把消息给绑定队列中含有路由规则的队列】
  - 结构
    - 同pubsub
    - 路由key
      - 发送消息制定路由key，且交换机类型为direct
      - 队列绑定交换机的时候，需要指定可接受的路由key
- publish/subscribe 【广播：交换机把消息给所有绑定的队列】【AB项目组】【同类的事，多个人同时按自己的方式做】
  - 结构
    - 一个生产者
    - 一个交换机
      - 类型
        - fanout 对应发布订阅pub/sub模式
        - direct 对应Routing工作模式
        - topic 对应Topics通配符工作模式
        - headers 对应header是工作模式
    - 多个队列
    - 多个消费者
  - 特点
    - 一个消息，可重复消费/被多个消费者消费/广播模式
  - 使用场景
    - 转账成功
      - 短信通知
      - 邮件通知
- work queues 【搬砖】【同类的事，多个人做】
  - 结构
    - 一个生产者
    - 一个交换机（默认交换机）
    - 一个队列
    - 多个消费者监听同一个队列
  - 特点
    - 消息不能重复消费（轮询发送给消费者）
# channel.basicConsume(String queue,boolean autoAck,Consumer callback)
- queue: 队列名称
- autoAck: 自动确认消费，接收到消息后，自动通知消息中间件：消息已确认消费（触发消息中间件根据规则决定是否删除消息）
- callback: 消费消息的函数  
  - 返回参数
    - String consumerTag: 消费者标签
    - Envolope envelope: 信封 （可获取对应交互接【getExchange()】和消息id【getDeliveryTag() 可用于编程确认消费】）
    - BasicProperties properties: 消息属性
    - byte[] body: 消息体
```java
package xlkx.java.mq.consumer;

import com.rabbitmq.client.*;

import java.io.IOException;

public class Consumer01 {
    public static final String QUEUE_NAME = "test";
    public static void main(String[] args) {
        ConnectionFactory cf = new ConnectionFactory();
        cf.setHost("localhost");
        cf.setPort(5672);
        cf.setUsername("guest");
        cf.setPassword("guest");
        cf.setVirtualHost("/");

        Connection conn = null;
        Channel ch = null;
        try{
            conn = cf.newConnection();
            ch = conn.createChannel();
            ch.queueDeclare(QUEUE_NAME,true,false,false,null);
            ch.basicConsume(QUEUE_NAME,true,new DefaultConsumer(ch){
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("consumerTag="+consumerTag);
                    System.out.println("exchange="+envelope.getExchange()+"  routingKey="+envelope.getRoutingKey());
                    Long mID =  envelope.getDeliveryTag();// message id
                    System.out.println("envelope.id="+mID);
                    System.out.println(new String(body,"utf-8"));
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
    }
}

```
# 创建channel
```java
ConnectionFactory cf = new ConnectionFactory();
cf.setHost("127.0.0.1");
cf.setPort(5672);
cf.setUsername("guest");
cf.setPassword("guest");
cf.setVirtualHost("/");
Connection conn = null;
try{
  conn = cf.newConnection();
  Channel channel = conn.createChannel();
} catch(Exception e){
  //log err
} finally{
  // close conn;
}
```
# channel.queueDeclare(String queue,boolean durable,boolean exclusive,boolean autoDelete,Map<String,Object> argumengt)
- queue: 队列名称
- durable： 是否持久化
- exclusive： 连接是否独占队列【队列只允许此连接访问，用于创建零时队列】
- autoDelete: 自动删除【结合exclusive，两者都设置为true来创建临时队列】
- arguments: 参数【例如，存活时间】
# channel.basicPublish(String exchange,String routingKey,BasicProperties props,byte[] body)
- exchange: 交互接，不指定["",不能为null，否则NPE]则使用默认交换机
- routingKey: 路由key，交换机将消息根据此路由key来转发到对应队列
- props： 消息属性
- body： 消息内容
```java
package xlkx.java.mq.producer;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.stream.Stream;

public class Sender {
    public static final String QUEUE_NAME = "test";
    public static void main(String[] args) {
        ConnectionFactory cf = new ConnectionFactory();
        cf.setHost("localhost");
        cf.setPort(5672);
        cf.setUsername("guest");
        cf.setPassword("guest");
        cf.setVirtualHost("/");
        try (
                Connection conn = cf.newConnection();
                Channel ch = conn.createChannel();
        ) {
            ch.queueDeclare(QUEUE_NAME,true,false,false,null);
            String msg = "街舞微量--";
            Stream.iterate(0,i->i+1).limit(2).forEach(it->{
                try {
                    sendMsg(ch,msg+it);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
    }

    private static void sendMsg(Channel ch,String msg) throws Exception{
        ch.basicPublish("",QUEUE_NAME,null,msg.getBytes());
        System.out.println("发送成功！："+msg);
    }
}

```
