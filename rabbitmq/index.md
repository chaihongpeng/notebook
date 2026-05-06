# Rabbitmq结构

- Broker：应用本身
- Virtual host：虚拟主机的概念
- Exchange：message到达broker的第一站，根据规则分发消息到队列
- Queue：存放消息的队列
- Connection：生产者和消费者和broker之间的tcp连接
- Channel：减少连接开销
- Binding：交换机和队列的虚拟连接，可以包含routing key

![image-20260506183517095](./assets/image-20260506183517095.png)

同一个vhost交换机和队列不能重名，不同vhost交换机和队列名字可以一样，可以用于不同用户间的相互隔离

# 交换机类型

- direct：精准匹配
- topic：通配符匹配
  - `.`分隔符`key1.key2.key3`
  - `*`匹配一个词`key1.key2.key3`
  - `#`匹配0~N个`key1.#`
- fanout：广播
- headrs：消息头属性