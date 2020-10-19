***建议食用本文前请先阅读[【RabbitMq 普通集群搭建】](./RabbitMq%20普通集群搭建.md)篇***
## 一、RabbitMq镜像
1. 镜像： `rabbitmq:3.8-management`
2. 启动参数：
   - `--name`: 容器名称
   - `-h / --hostname`：`rabbitMq`默认节点的 `host`
   - `-v`：文件挂载映射
   - `-p`：端口映射
     - 5672：容器内默认的`rabbitMq` 启动端口
     - 15672：容器内默认的`rabbitMq` 管理插件启动端口
3. 环境变量：（支持所有`rabbitMq`环境变量）
   - `RABBITMQ_NODENAME`：节点名称，缺省为 `rabbit@[hostname]`
   - `RABBITMQ_DEFAULT_USER`：默认用户名，缺省为`guset`
   - `RABBITMQ_DEFAULT_PASS`：默认密码，缺省为 `guest`
   - `RABBITMQ_DEFAULT_VHOST`：默认虚拟主机，缺省为`/`
   - `RABBITMQ_ERLANG_COOKIE`：`erlang.cookie`值

## 二、搭建集群

1. 启动多份`rabbitMq`容器

   ```
   # 启动节点一
   docker run -d --hostname clusterRabbit1 --name clusterRabbit1 -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -v /opt/ci123/www/html/rabbitMq/clusterRabbit1:/var/lib/rabbitmq rabbitmq:3.8-management
   
   # 启动节点二
   docker run -d --hostname clusterRabbit2 --name clusterRabbit2 -p 5673:5672 --link clusterRabbit1:clusterRabbit1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -v /opt/ci123/www/html/rabbitMq/clusterRabbit2:/var/lib/rabbitmq rabbitmq:3.8-management
   ```

   - `/var/lib/rabbitmq` 是容器内部文件数据存放目录，使用`-v`进行文件目录挂载。
   - 多个容器之间使用`--link <name or id>:alias`连接，否则需要自行在各个容器添加供`rabbitMq` 相互访问的`host`。
   - `Erlang Cookie`值必须相同，`rabbitMQ`是通过`Erlang`实现的，`Erlang Cookie`相当于不同节点之间相互通讯的秘钥，`Erlang`节点通过交换`Erlang Cookie`获得认证。

2. 启动集群

   1. 设置节点二 `rabbit@clusterRabbit2`,加入节点一 `rabbit@clusterRabbit1`的集群

      ```
      rabbitmqctl stop_app
      rabbitmqctl reset
      rabbitmqctl join_cluster rabbit@clusterRabbit1
      rabbitmqctl start_app
      ```

   2. 查看`rabbitMq`管理后台集群数据

      ![rabbitmq管理后台](https://img2.ciurl.cn/flashsale/upload/xinfotek_upload/2019/10/29/1572337428567936.png)

3. 一键化启动脚本

   ```
   #!bin/sh
   # 启动节点一
   docker run -d --hostname clusterRabbit1 --name clusterRabbit1 -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -v /opt/ci123/www/html/rabbitMq/clusterRabbit1:/var/lib/rabbitmq rabbitmq:3.8-management
   
   # 启动节点二
   docker run -d --hostname clusterRabbit2 --name clusterRabbit2 -p 5673:5672 --link clusterRabbit1:clusterRabbit1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -v /opt/ci123/www/html/rabbitMq/clusterRabbit2:/var/lib/rabbitmq rabbitmq:3.8-management
   
   echo "即将开始初始化"
   sleep 5
   # 加入集群
   docker exec -it clusterRabbit2 sh -c 'rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster rabbit@clusterRabbit1 && rabbitmqctl start_app'
   
   # 添加虚拟主机
   docker exec -it clusterRabbit1 sh -c 'rabbitmqctl add_vhost cluster'
   
   # 添加用户
   docker exec -it clusterRabbit1 sh -c 'rabbitmqctl add_user api_management shijiemori2012'
   
   # 添加用户权限
   docker exec -it clusterRabbit1 sh -c 'rabbitmqctl set_permissions -p cluster api_management ".*" ".*" ".*"'
   
   # 设置用户标签
   docker exec -it clusterRabbit1 sh -c 'rabbitmqctl set_user_tags api_management administrator'
   ```

   