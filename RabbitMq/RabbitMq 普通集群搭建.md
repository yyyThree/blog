## 一、集群概念

​	`RabbitMQ`集群是一个或多个节点的逻辑分组，每个节点共享用户、虚拟主机、队列、交换、绑定路由、运行时参数和其他分布式状态。集群中的节点可以动态地添加/删除，`RabbitMQ`代理一开始都运行在单个节点上，可以将这些节点连接到集群中，然后再将其转换回各个代理。

1. 默认情况下，`RabbitMq`将复制除消息队列外的所有数据至集群中的每一个节点。而消息队列的完整数据只会存放于创建该队列的节点上，其余节点仅保存该队列的元数据和指针（类似于索引）。如果需要复制队列，则需要启用镜像队列集群。

2. 集群中每个节点是平等的，不存在主从和特殊的节点。

3. 集群中的节点通过`Erlang Cookie`相互通信，每个节点必须具有相同的`cookie`。

4. 节点分为磁盘节点和`RAM`节点，`RAM`节点只在`RAM`中存储内部数据库表，并不存储包括消息、消息存储索引、队列索引和其他节点状态等数据，`RAM`节点的性能更加高效，但是由于数据是非持久化的，一旦宕机将无法恢复。默认创建的都是磁盘节点。

5. 单节点拓扑图如下，集群的拓扑是基于多个Node节点的扩展。

   ![单节点拓扑图](https://img2.ciurl.cn/flashsale/upload/xinfotek_upload/2019/10/24/1571883872293962.png)

## 二、配置需求

1. 配置方式
   - `config` 文件配置
   - `rabbitmqctl`命令配置（下文中使用此方法配置）
2. 集群中的节点名必须是唯一的
   - 可以在启动时使用环境变量`RABBITMQ_NODENAME`设置
   - 节点名由`[节点名称]@[host]`组成
3. 各个节点的启动端口可以被成功连接
4. 节点之间通过节点名相互访问，要求各个节点之间的`host`可以相互进行`DNS`解析
5. 每个节点之间必须配置相同的`Erlang Cookie`(**多机环境需要额外配置**)

## 三、集群配置

1. 启动多个独立的节点

   ```
   RABBITMQ_NODE_PORT=5674 RABBITMQ_NODE_IPDDRESS=192.168.0.235  RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME="clusterRabbit1@VM235"  /usr/local/rabbitmq/3.1.3/sbin/rabbitmq-server -detached
   RABBITMQ_NODE_PORT=5675 RABBITMQ_NODE_IPDDRESS=192.168.0.235  RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15674}]" RABBITMQ_NODENAME="clusterRabbit2@VM235" /usr/local/rabbitmq/3.1.3/sbin/rabbitmq-server -detached
   ```

   执行完成后，分别创建了名称为`clusterRabbit1@VM235`,`clusterRabbit2@VM235`的两个节点。

   `VM235`需要提前配置`host`文件，确保各个节点之间的`host`可以相互进行`DNS`解析。

2. 创建集群

   1. 关闭节点`clusterRabbit1@VM235`

      ```
      ./rabbitmqctl -n clusterRabbit1@VM235 stop_app
      ```

   2. 重置节点`clusterRabbit1@VM235`

      ```
      ./rabbitmqctl -n clusterRabbit1@VM235 reset
      ```

      **必须重置节点才能加入现有集群，重置节点将删除该节点上以前存在的所有资源和数据。这意味着节点不能在成为集群的成员时保留其现有数据，节点中的数据需要进行额外的备份和恢复。**

   3. 将节点`clusterRabbit1@VM235`加入`clusterRabbit2@VM235`的集群

      ```
      ./rabbitmqctl -n clusterRabbit1@VM235 join_cluster clusterRabbit2@VM235
      ```

   4. 启动节点`clusterRabbit1@VM235`

      ```
      ./rabbitmqctl -n clusterRabbit1@VM235 start_app
      ```

   5. 查看集群信息

      ```
      ./rabbitmqctl -n clusterRabbit1@VM235 cluster_status
      结果：
      Cluster status of node clusterRabbit1@VM235 ...
      [{nodes,[{disc,[clusterRabbit1@VM235,clusterRabbit2@VM235]}]},
       {running_nodes,[clusterRabbit2@VM235,clusterRabbit1@VM235]},
       {partitions,[]}]
      ...done.
      ```

3. 集群中节点的关闭与重启

   1. 节点关闭并重启之后会选择一个在线的集群成员(只考虑磁盘节点)进行同步。在重新启动节点时，默认情况下将尝试与该成员联系10次，并有30秒的响应超时。如果该成员在时间间隔内可用则节点将成功启动，并与该成员同步所需内容后继续运行。如果该成员无法响应，则重新启动的节点将放弃同步数据并启动。
   2. 以下情况将导致节点无法重新加入集群：
      - 修改节点名/主机名，节点的数据目录路径会因此更改。
      - 重置节点数据/更换节点数据目录

4. 移除集群中的节点

   1. 关闭该节点
   2. 重置该节点
   3. 再次启动该节点

## 四、集群测试

1. 创建用户、`vhost`、`exchange`、`queue`，并启动消费者。【测试基础】
   1. 用户：`api_management`（`management`标签，开放虚拟主机`cluster`所有权限）
   2. `vhost`：`cluster`
   3. `exchange`：`cluster`（直连交换机）
   4. `queue`：
      1. 节点一 `clusterRabbit1`：
         1. 队列一：
            - `name`：`clusterRabbit1Queue1`
            - `routing_key`：`clusterRabbit1key`
         2. 队列二：
            - `name`：`clusterRabbit1Queue2`
            - `routing_key`：`clusterRabbitCommonKey`
      2. 节点二 `clusterRabbit2`：
         1. 队列三：
            - `name`：`clusterRabbit2Queue1`
            - `routing_key`：`clusterRabbit2key`
         2. 队列四：
            - `name`：`clusterRabbit2Queue2`
            - `routing_key`：`clusterRabbitCommonKey`（与队列二 相同）
   5. `consumer`：
      1. 消费者一：
         - 连接节点：节点一 `clusterRabbit1`
         - 消费队列：队列一 `clusterRabbit1Queue1`
      2. 消费者二：
         - 连接节点：节点一 `clusterRabbit1`
         - 消费队列：队列二 `clusterRabbit1Queue2`
      3. 消费者三（非此节点的队列）：
         - 连接节点：节点一 `clusterRabbit1`
         - 消费队列：队列三 `clusterRabbit2Queue1`
      4. 消费者四：
         - 连接节点：节点二 `clusterRabbit2`
         - 消费队列：队列四 `clusterRabbit2Queue2`
2. 连接节点二 `clusterRabbit2`,再次声明队列名和队列一同名的队列`clusterRabbit1Queue1`。【同名队列再次声明】
   - 结果：声明不成功，节点二中没有生成新的`clusterRabbit1Queue1`队列，而节点一中`clusterRabbit1Queue1`队列多出了新绑定的`routing_key`,这意味着在同一集群中不同节点之间的队列名是唯一的,在一个节点中可以操作另一个节点的队列数据。
3. 生产者连接节点一 `clusterRabbit1`，发布`clusterRabbit1key`消息。【发布此节点队列消息】
   - 消费者一 成功接收到消息
4. 生产者连接节点二 `clusterRabbit2`，发布`clusterRabbit1key`消息。【发布非此节点队列消息】
   - 消费者一 成功接收到消息
5. 生产者连接节点一 `clusterRabbit1`，发布`clusterRabbitCommonKey`消息。【多个节点队列绑定相同消息】
   - 消费者二 成功接收到消息
   - 消费者四 成功接收到消息
6. 生产者连接节点一 `clusterRabbit1`，发布`clusterRabbit2key`消息。【消费非此节点队列消息】
   - 消费三 成功接收到消息
7. 关闭节点二 `clusterRabbit2`，连接节点一，发布`clusterRabbit2key`消息【投递消息给集群中意外退出的节点】
   - 关闭节点二 `clusterRabbit2`之后，消费者三、四异常退出
   - 投递消息至集群成功
   - 再次启动节点二 `clusterRabbit2`以及消费者三、四，之前投递的消息没有成功接收

## 五、使用总结

1. 同一集群中不同节点之间的队列名是唯一的，在一个节点中可以操作另一个节点的队列数据。
2. 生产者可以发布集群中任一节点队列绑定的消息，集群将自动匹配出符合条件的节点队列，并投递给消费者进行消费。
3. 集群中不同节点的队列如果绑定了相同的`routing_key`，消息将投递到集群中所有符合路由匹配条件的节点队列中。
4. 消费者可以订阅集群中任意节点的队列。
5. 集群中某个节点异常退出后，生产者投递到集群中的消息将无法送达至该节点，但是不影响其他节点的接收。（解决这个问题需要使用镜像队列集群）

## 六、错误记录

1. 启动新的节点报错：`could_not_start,rabbitmq_management`

   解决：

   ​	`rabbitmq_management`插件默认使用的是`15672`端口，这个端口已被之前启动的节点占用，修改启动命名为	`rabbitmq_management`插件指定一个新的端口即可。

   ```
   RABBITMQ_NODE_PORT=5674 RABBITMQ_NODE_IPDDRESS=192.168.0.235  RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME="clusterRabbit1@VM235"  /usr/local/rabbitmq/3.1.3/sbin/rabbitmq-server -detached
   ```

2. 【认知错误】`rabbitmq_management`管理后台`Overview`中节点的`Memory`表示使用的内存，`Disk space`表示剩余可用的磁盘空间，不是已使用的磁盘空间。

3. 修改节点的`host`后启动失败:``ERROR: epmd error for host "VM235": nxdomain (non-existing domain)`

   解决：

   ​	`VM235`需要预先添加到 `HOST`文件，确保可以被正确解析。

4. 节点一  `clusterRabbit1` 退出集群后重新加入集群，再次声明之前的队列提示`routing_key`绑定不成功

   ```
   NOT_FOUND - no binding clusterRabbitCommonKey between exchange 'cluster' in vhost 'cluster' and queue 'clusterRabbit1Queue2' in vhost 'cluster'
   ```

   此问题是`rabbitMq`集群本身的问题，且未得到官方明确的解决方案。

   一些临时解放方案：

   - 修改节点一名称
   - 删除并重建交换机
   - 修改旧的队列名或者队列参数

   根本解决：

   ​	升级`rabbitMq`至`3.8.0`，参考[【RabbitMq 使用docker搭建集群篇】](./RabbitMq%20使用docker搭建集群.md)。
