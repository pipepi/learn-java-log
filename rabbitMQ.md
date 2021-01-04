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
