# kafka-client
kafka消息发送客户端库

## 详细介绍
见 https://blog.csdn.net/MICweaver/article/details/85041252

## 整体架构
![架构图](https://img-blog.csdnimg.cn/20181216214954189.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01JQ3dlYXZlcg==,size_16,color_FFFFFF,t_70)

## 代码示例:
 

```
KafkaClient::init('127.0.0.1', '/home/work/logs/kafka'); //设置集群地址，及日志地址
KafkaClient::setTopic('topic_name');//设置队列名
KafkaClient::sendMsgAsyn("test msg");//异步发送消息
//or
KafkaClient::sendMsg("test msg");//同步发送消息

```
我们发送消息的库使用rdkaka扩展(https://github.com/arnaud-lb/php-rdkafka)，而rdkafka是librdkafka库的php扩展封装（https://github.com/edenhill/librdkafka）。
librdkafka消息发送的实现是将消息放入本地的一个内存队列，然后由另一个线程从内存队列中取出消息并发送到kafka集群。
消息发送如果采用同步模式，即等到消息发送到kafka集群，并得到成功或失败的结果，则我们的php进程会一直查询本地内存队列是不是已经空了，并且当前的消息是不是返回了结果，会导致大量占用CPU， 甚至将php进程卡死。 以我们线上的使用经验，当将socket.blocking.max.ms（可以理解这个值的作用为扩展线程间隔多久轮询一次查看返回结果）的值配置小于50ms时，运行一段时间后，线上的php进程可能因CPU占用过高而卡死而不能响应服务。 而当将socket.blocking.max.ms 的值配置为50ms时则没有出现过这个问题。为什么不将socket.blocking.max.ms 的值调得更大呢，因为这个值也基本等于一个发送请求的耗时值，所以在同步模式下，一个消息发送请求要耗时50ms，这个已经很慢了，50ms是我们找到的在不引起php进程卡死的情况下的最小值。 关于这个问题，之前咨询过librdkafka库的作者，见（https://github.com/edenhill/librdkafka/issues/1553，貌似作者最新回复已经对此问题进行了优化）

所以，如果不是超级超级重要的数据，我们建议使用异步发送模式，把消息放入本地内存之后就返回，响应超级快。

