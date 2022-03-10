# RocketMQ

>1,消息领域有一个对消息投递的QoS定义（Quality of Service，服务质量），分为：最多一次（At most once）、至少一次（At least once）、仅一次（ Exactly once）

>2,目前火热的几款MQ，比如RocketMQ、Kafka、RabbitMQ、ActiveMQ等，都是保证的至少一次(At least Once)，它指每个消息必须投递一次。既然是至少一次，那么他们就避免不了重复消费的问题，因此这个问题最终还是要靠业务代码来解决

## 重复消费的原因
>1,重复消费的的原因大概可以分为两个，一个是生产者发送消息的时候发送了重复的消息，另一个是消费者消费的时候消费了重复的消息


![](./res/rocketmq_architecture_1.png "")