### 异步消费

```java
 producer.send(msg, new SendCallback() {
                    @Override
                    public void onSuccess(SendResult sendResult) {
                        countDownLatch.countDown();
                        System.out.printf("%-10d OK %s %n", index, sendResult.getMsgId());
                    }

                    @Override
                    public void onException(Throwable e) {
                        countDownLatch.countDown();
                        System.out.printf("%-10d Exception %s %n", index, e);
                        e.printStackTrace();
                    }
                });
```



```java
  // <3> 设置消费进度，从 Topic 最初位置开始    consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
```

设置一个**新的**消费集群，初始的消费进度。目前有三个选项：

- `CONSUME_FROM_FIRST_OFFSET` ：每个 Topic 队列的第一条消息。
- `CONSUME_FROM_LAST_OFFSET` ：每个 Topic 队列的最后一条消息。
- `CONSUME_FROM_TIMESTAMP` ：每个 Topic 队列的指定时间开始的消息。
- 😈 注意，只针对**新的**消费集群。如果一个集群每个 Topic 已经有消费进度，则继续使用该消费进度。仔细理解一下哈~



```java
     // <5> 添加消息监听器
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                // 返回成功
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }

        });
```

`<5>` 处，添加消息监听器。这里我们采用的是 [MessageListenerConcurrently](https://github.com/apache/rocketmq/blob/master/client/src/main/java/org/apache/rocketmq/client/consumer/listener/MessageListenerConcurrently.java) **并发**消费消息的监听器。如果胖友需要实现**顺序**消费消息，需要使用 [MessageListenerOrderly](https://github.com/apache/rocketmq/blob/master/client/src/main/java/org/apache/rocketmq/client/consumer/listener/MessageListenerOrderly.java) **顺序**消费的监听器

 **1. Producer**

- 1、Producer 自身在应用中，所以无需考虑高可用。
- 2、Producer 配置多个 Namesrv 列表，从而保证 Producer 和 Namesrv 的连接高可用。并且，会从 Namesrv 定时拉取最新的 Topic 信息。
- 3、Producer 会和所有 Consumer 直连，在发送消息时，会选择一个 Broker 进行发送。如果发送失败，则会使用另外一个 Broker 。
- 4、Producer 会定时向 Broker 心跳，证明其存活。而 Broker 会定时检测，判断是否有 Producer 异常下线。

🦅 **2. Consumer**

- 1、Consumer 需要部署多个节点，以保证 Consumer 自身的高可用。当相同消费者分组中有新的 Consumer 上线，或者老的 Consumer 下线，会重新分配 Topic 的 Queue 到目前消费分组的 Consumer 们。
- 2、Consumer 配置多个 Namesrv 列表，从而保证 Consumer 和 Namesrv 的连接高可用。并且，会从 Consumer 定时拉取最新的 Topic 信息。
- 3、Consumer 会和所有 Broker 直连，消费相应分配到的 Queue 的消息。如果消费失败，则会发回消息到 Broker 中。
- 4、Consumer 会定时向 Broker 心跳，证明其存活。而 Broker 会定时检测，判断是否有 Consumer 异常下线。

🦅 **3. Namesrv**

- 1、Namesrv 需要部署多个节点，以保证 Namesrv 的高可用。
- 2、Namesrv 本身是无状态，不产生数据的存储，是通过 Broker 心跳将 Topic 信息同步到 Namesrv 中。
- 3、多个 Namesrv 之间不会有数据的同步，是通过 Broker 向多个 Namesrv 多写。

🦅 **4. Broker**

- 1、多个 Broker 可以形成一个 Broker 分组。每个 Broker 分组存在一个 Master 和多个 Slave 节点。
  - Master 节点，可提供读和写功能。Slave 节点，可提供读功能。
  - Master 节点会不断发送新的 CommitLog 给 Slave节点。Slave 节点不断上报本地的 CommitLog 已经同步到的位置给 Master 节点。
  - Slave 节点会从 Master 节点拉取消费进度、Topic 配置等等。
- 2、多个 Broker 分组，形成 Broker 集群。
  - Broker 集群和集群之间，不存在通信与数据同步。
- 3、Broker 可以配置同步刷盘或异步刷盘，根据消息的持久化的可靠性来配置。







