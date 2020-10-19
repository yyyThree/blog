***建议食用本文前请先阅读[【RabbitMq 普通集群搭建】](./RabbitMq%20普通集群搭建.md)篇***
## 一、镜像队列集群概念
​	镜像队列是基于普通集群模式的扩展，普通集群模式下如果某一个节点宕机，该节点下的队列操作将完全失效。而在镜像队列模式下，队列的数据将被复制到所有节点（或者配置过的节点）中，从而保证了一个节点宕机，其余节点也可以正常消费此消息。但此模式下也必然会带来性能下降、内存/磁盘消耗增加、网络IO负担增加等问题，所以镜像队列适用于对高可用要求比较高的系统。

1. 每个镜像队列由一个主队列和一个或多个镜像组成。每个镜像队列都有自己的主节点，主队列存放于主节点上。对队列产生的操作将首先应用于队列的主节点，然后传播到镜像节点。包括发布队列、向消费者传递消息、跟踪来自消费者的确认等行为。
2. 发布到集群中的消息将被复制到所有的镜像队列中，消费者连接任意节点消费队列实际上都将被连接至主队列的节点上。如果主队列已经确认消费了消息，则其余镜像队列中的消息将被丢弃。
3. 如果主队列所在的节点发生异常，默认情况下最“老”的镜像队列将被选举为主队列，当然也可以制定不同的选举策略。

## 二、配置镜像队列

1. 将队列配置成镜像队列需要通过创建`policy`来实现。`policy `包含策略键`ha-mode`和其对应的键值`ha-params`(可选)组成。

   - `exactly`模式
     - `ha-params`为`count`，表示队列的总数量。
     - `count`表示主队列+镜像队列的总数量，如果`count`为1，则表示只存在于主队列。如果 `count`为2则表示存在主队列和一个镜像队列，以此类推。如果`count`值大于集群中节点的总数则表示所有节点都将同步一份镜像队列。如果某个镜像队列的节点宕机，则会寻找一个剩余未同步的节点来同步镜像队列。
   - `all`模式
     - 不需要`ha-params`。
     - 此模式下所有节点都将同步镜像队列，如果有新节点加入，则新节点也会进行同步。官方建议同步镜像队列的节点数为`N/2 + 1`,其中`N`表示节点总数。同步到所有节点会增加所有集群节点的负载，包括网络I/O、磁盘I/O和磁盘空间的使用等。
   - `nodes`模式
     - `ha-params`为`node names`，节点的名称。
     - 此模式下将在指定的节点上同步镜像队列，如果声明队列时其余节点均不在新，则只会在声明连接的那个节点上创建队列。
	 
2. 配置`policy`

   - `rabbitmqctl`命令配置

     ```
     set_policy [-p vhost] [--priority priority] [--apply-to apply-to] name pattern definition
     ```

     - `name`：策略名称
     - `pattern`：策略匹配符，正则表达式。当与给定资源匹配时，将应用该策略。
     - `definition`：策略内容定义，JSON字符串。
     - `priority`：策略的优先级，整数。数字越大，优先级越高。默认值为0。
     - `apply-to`：策略应用的对象，支持`queues`,`exchanges`,`all`,默认值为`all`

     ```
     // exactly模式，匹配前缀为two的资源
     rabbitmqctl set_policy -p cluster ha-two ^two. '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
     
     // all模式，匹配所有前缀
     rabbitmqctl set_policy -p cluster ha-all ^ '{"ha-mode":"all"}'
     
     // nodes模式，匹配所有前缀
     rabbitmqctl set_policy -p cluster ha-nodes ^ '{"ha-mode":"nodes","ha-params":["rabbit@nodeA", "rabbit@nodeB"]}'
     ```

   -  `management`管理后台配置

     `Admin -> Policies -> Add / update a policy`

3. 新镜像队列同步设置

   - `ha-sync-mode:manual`

     默认模式，新队列镜像将不同步现有消息，只接收新消息。当消费者消费完所有仅存在于主队列上的消息后，新的队列镜像将随着时间的推移成为和主节点队列相同的精确镜像队列。

   - `ha-sync-mode: automatic`

     当新队列镜像加入时，队列将自动同步所有消息。

## 三、额外说明

1. 独占队列不会被复制为镜像队列。
2. 主节点队列失效后，其中一个镜像队列将被选举为主队列并带来如下影响：
   1. 与主机点连接的客户端将全部断开。
   2. 运行时间最长的镜像队列将被选举为主队列，如果镜像队列尚未开始同步，则队列上的消息将丢失。
   3. 新的主队列会认为之前所有的消费者的连接都已经断开，它将重新发送旧队列中没有收到`ack`的消息。这可能出现客户端已经发送过`ack`, 但是服务端在接收到之前就已宕机的情况，从而导致发送两遍相同的消息，因此所有未确认的消息都将使用`redelivered`标志重新发送。
   4. 如果消费者连着的是镜像队列节点，并且消费者在启动时设置了`x-cancel-on-ha-failover`参数，则消费者将收到一个服务端消费取消的通知，如果未设置此参数，则消费者将无法感知主节点已宕机。
   5. 如果使用自动`ack`机制则消息将丢失。
1. 如果停止了某个包含主节点队列的节点，则其他节点的镜像队列将被选举为主节点队列。在重新启动此节点后，该节点只会被当做是一个新加入集群的节点，不会重新成为主节点队列。
2. 在主节点队列宕机并且其他镜像队列尚未同步的极端情况下，`rabbitMq`集群将拒绝任何镜像队列选举为主队列，整个队将不可用且被关闭。如果在镜像队列尚未同步的情况下也需要将某个镜像队列选举为主队列，需要配置`policy`中`ha-promote-on-shutdown`为`always`（默认为`when-synced`），并且`ha-promote-on-failure`不可配置为`hen-synced`（默认值为`always`）。

## 四、集群测试

1. 启动节点、创建用户、`vhost`、`exchange`、`queue`，启动消费者。（快速启动，参考[【RabbitMq 使用docker搭建集群】](./RabbitMq%20使用docker搭建集群.md)篇）

   1. 节点：

      1. 节点一：`rabbit@clusterRabbit1`
      2. 节点二：`rabbit@clusterRabbit2`

   2. 用户：`api_management`（`administrator`标签，开放虚拟主机`cluster`所有权限）

   3. 策略：

      ```
      rabbitmqctl set_policy -p cluster ha-all ^ '{"ha-mode":"all"}'
      ```

   4. `vhost`：`cluster`

   5. `exchange`：`cluster`（直连交换机）

   6. `queue`：

      1. 节点一 `rabbit@clusterRabbit1`：
         1. 队列一：
            - `name`：`clusterRabbit1Queue1`
            - `routing_key`：`clusterRabbit1key`
         2. 队列二：
            - `name`：`clusterRabbit1Queue2`
            - `routing_key`：`clusterRabbitCommonKey`
      2. 节点二 `rabbit@clusterRabbit2`：
         1. 队列三：
            - `name`：`clusterRabbit2Queue1`
            - `routing_key`：`clusterRabbit2key`
         2. 队列四：
            - `name`：`clusterRabbit2Queue2`
            - `routing_key`：`clusterRabbitCommonKey`（与队列二 相同）

   7. `consumer`：

      1. 消费者一：
         - 连接节点：节点一 `rabbit@clusterRabbit1`
         - 消费队列：队列一 `clusterRabbit1Queue1`
      2. 消费者二：
         - 连接节点：节点一 `rabbit@clusterRabbit1`
         - 消费队列：队列二 `clusterRabbit1Queue2`
      3.  消费者三：
         - 连接节点：节点二 `rabbit@clusterRabbit2`
         - 消费队列：队列三 `clusterRabbit2Queue1`
      4. 消费者四（非此节点的队列）：
         - 连接节点：节点一 `rabbit@clusterRabbit1`
         - 消费队列：队列三 `clusterRabbit2Queue1`
      5. 消费者五：
         - 连接节点：节点二 `rabbit@clusterRabbit2`
         - 消费队列：队列四 `clusterRabbit2Queue2`

2. 连接节点二 `rabbit@clusterRabbit2`,再次声明队列名和队列一同名的队列`clusterRabbit1Queue1`。【同名队列再次声明】

   - 结果：声明不成功，节点二中没有生成新的`clusterRabbit1Queue1`队列，而节点一中`clusterRabbit1Queue1`队列多出了新绑定的`routing_key`:`clusterRabbit2key`,这意味着在同一集群中不同节点之间的队列名是唯一的,在一个节点中可以操作另一个节点的队列数据。

3. 生产者连接节点一 `rabbit@clusterRabbit1`，发布`clusterRabbit1key`消息。【发布此节点队列消息】

   - 消费者一 成功接收到消息。

4. 生产者连接节点二 `rabbit@clusterRabbi2`，发布`clusterRabbit1key`消息。【发布非此节点队列消息】

   - 消费者一 成功接收到消息。

5. 生产者连接节点一 `rabbit@clusterRabbit1`，发布`clusterRabbitCommonKey`消息。【多个节点队列绑定相同消息】

   - 消费者二 成功接收到消息。
   - 消费者五 成功接收到消息。

6. 生产者连接节点一 `rabbit@clusterRabbit1`，发布`clusterRabbit2key`消息。【消费非此节点队列消息】

   - 消费三、四 轮询接收到消息。

7. 关闭节点二 `rabbit@clusterRabbit2`，连接节点一 `rabbit@clusterRabbit1`，发布`clusterRabbit2key`消息【投递消息给集群中意外退出的节点】

   - 连接节点二  `rabbit@clusterRabbit2`的消费者全部断开，消费者四并未断开。
   - 节点一`rabbit@clusterRabbit1` 晋升成为队列三、队列四的主节点。
   - 消费者四 可以继续成功接收到消息。
   - 再次启动节点二`rabbit@clusterRabbit2`，没有再次成为队列三、队列四的主节点，只是复制了队列三、队列四的镜像队列。

8. 修改消费者四，在`ack`之前`sleep(20)`，发布`clusterRabbit2key`消息，并立即关闭节点二 `rabbit@clusterRabbit2`

   - 和上面一样，消费者四并未断开。
   - 消费者收到两条相同的消息，说明节点二在收到 `ack`之前宕机后，此消息会被当做未消费的消息重新放入队列后消费。

