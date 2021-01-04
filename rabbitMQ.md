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
