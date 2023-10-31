---
title: "redis中stream的使用"
date: 2023-10-31T21:00:04+08:00
tags: [redis, 消息队列]
categories: [code]
---

看了官方文档的stream介绍和网上的博客，总结redis中stream的用法。[官网链接](https://redis.io/docs/data-types/streams/)

## 介绍

redis在5.0之后的版本里多出了一个新的数据结构stream，在系统已经使用了redis，并且并发量不是很大的情况下，就可以使用stream作为消息队列使用，不用去专门引入新的消息中间件。

基本概念如下：

**消费组**：每个消费组都是独立的，同一个消费组里面可以有多个消费者，这些消费者都是竞争关系，一个消息被消费者A消费了，消费游标last_delivered_id都会后移一个位置。

**消费者**：消费者有自己的唯一id，同时内部维护了一个pending队列，消费后的消息都会在其中保存id，只有当ack之后才会从pending中移除。这个队列就可以用来解决，如果客户端挂了，下次重启服务的时候仍然可以处理这条消息。

**消息队列**：也就是stream，和其他数据结构不一样，当list为空时，redis会自动删除这个list，stream可以为空然后保留下来，是因为消息队列里面有消费组和消费者的信息。

**消息**：每个消息都有一个id，也就是时间戳，例如1698731874660-0，前面是时间戳，后面0表示在这个毫秒内的第几个。这样记录id就可以实现范围查找。

## 基本命令

**xadd** 追加消息
**xdel** 删除消息。删除标志位
**xrange** 获取制定的消息
**xlen** 消息长度

```
XADD race:italy * rider Castilla
XADD race:france * rider Castilla speed 30.2 position 1 location_id 1

race:italy stream名称
* id，可以自己设置，一般都用这个，让redis自动生成
rider Castilla 消息内容，为一个一个的k-v
```

```
xrange race:italy - +
- 代表最小 
+ 代表最大
也可以使用具体的id号，获取单条消息，可以start和end使用同一个id
比如：xrange race:italy 1698732650059-0 1698732650059-0
这些id其实就是时间，我们也可以制定不存在的时间比如1698732650060-0，把后面的59改成60
xrange race:italy 1698732650060-0 +
```

![](/imgs/redis_stream1.png)

```
xlen race:italy
xdel 1698732650059-0 ，这个命令跟上消息id就可以了
```

## 单消费者

可以不使用消费组，就使用单个消费者来进行消费，命令是xread
xread count 10 streams race:italy 0-0 从头开始消费，消费10个
还可以制定阻塞的方式来消费
xread block 0 streams race:italy $ 从最新的消息开始消费，block后面接上阻塞时间，0代表一直阻塞，我们在另一个客户端添加上信息，这边就可以消费到了。

![](/imgs/redis_stream2.png)

![](/imgs/redis_stream3.png)

![](/imgs/redis_stream4.png)

如果是单个消费者这样的话，就需要记录当前的消费位置，然后继续消费位置继续向下消费就可以了，也可以消费完了直接删除也行。

## 消费组

因为我之前在redis里面创建了消费组，现在直接先把测试数据清空（生产环境不要这样做）。也可以使用xgroup DESTROY去删除。

```
FLUSHDB
```

插入测试数据

```
XADD race:italy * rider Castilla
XADD race:italy * rider Royce
XADD race:italy * rider Sam-Bodden
XADD race:italy * rider Prickett
XADD race:italy * rider Norem
```

**创建消费组**

```
XGROUP CREATE race:italy italy_riders 0-0
0-0表示从头开始消费，使用$表示只消费新的数据
```

**消费数据**

```
XREADGROUP GROUP italy_riders Alice COUNT 1 STREAMS race:italy >

italy_riders 组名
Alice  消费者名，需要唯一
race:italy stream名
> 从last-delivered-id开始消费，当前就是0-0，如果消费了一条信息，就是第二条消息的id，在消息都被消费之后就是新的消息
也可以加上block来变成阻塞的方法
```

![](/imgs/redis_stream5.png)

再执行一次 XREADGROUP GROUP italy_riders Alice COUNT 1 STREAMS race:italy > ，就是第二条消息
![](/imgs/redis_stream6.png)

这时候对于这个消费组来说已经消费了2条消息，此时，消费组中的另一个消费者来消费，也只会消费到第三条消息
XREADGROUP GROUP italy_riders Bob COUNT 1 STREAMS race:italy >
![](/imgs/redis_stream7.png)

我这显示第4条消息，是因为我之前执行了一次上面的命令

这样同一个消费组就可以做负载均衡了，对于一堆消息，有多个客户端来消费，每个客户端相互独立，消息也只会被一个客户端消费。

**确认数据**

消费完数据之后，需要ack，这样才会在每个消费者的pending中移除该次消息。

查看当前消费情况 xinfo stream race:italy

![](/imgs/redis_stream8.png)

length 当前有5条数据
groups 一个消费组
last-generated-id 最后一条消息id
first-entry和last-entry 第一条消息和最后一条消息

查看消费组情况  xinfo groups race:italy

![](/imgs/redis_stream9.png)

name  组名
consumers  消费者个数
pending 未ack的消息数
last-delivered-id  最后一条消费消息id

查看消费者情况 xinfo consumers race:italy italy_riders

![](/imgs/redis_stream10.png)

name  消费者名
pending  未ack的消息个数

使用ack命令确认数据  xack race:italy italy_riders 1698738784018-0

**查看pending队列**


XPENDING race:italy italy_riders - + 10

![](/imgs/redis_stream10.png)

里面显示了每个未ack的消息的情况，其中第四个字段表示被消费的次数，如果Alice再一次消费，并且未ack的话，这个值就会加1

在客户端消费者读取 Stream 消息时，Redis 服务器将消息回复给客户端的过程中，客户端突然断开了连接，消息就丢失了。但是 PEL 里已经保存了发出去的消息 ID。待客户端重新连上之后，可以再次收到 PEL 中的消息 ID 列表。不过此时 xreadgroup 的起始消息 ID 不能为参数>，而必须是任意有效的消息 ID，一般将参数设为 0-0，表示读取所有的 PEL 消息以及自last_delivered_id之后的新消息。
