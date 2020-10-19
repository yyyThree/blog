## 一、目的

​	在使用 `RabbitMq`集群时可能会遇到集群中某个节点出现异常或者连接数过多的情况，这个时候与该节点连接的`Consumer`将会断开，`Publisher`也会无法将消息发送至集群。为了解决这些问题，本文中将使用 `HAProxy`来代理集群，实现多个节点的负载均衡以及在某个节点异常时自动将连接切换至其他正常节点等功能。

## 二、HAProxy安装配置（Centos 7）

1. 安装 `HAProxy`

   1. 下载最新稳定版`2.0.8`并解压

      ```
      // PWD：/opt/ci123/www/html/rabbitMq/
      wget https://www.haproxy.org/download/2.0/src/haproxy-2.0.8.tar.gz
      tar xf haproxy-2.0.8.tar.gz
      ```

   2. 查看系统内核版本来指定编译版本

      ```
      uname -r
      3.10.0-862.6.3.el7.x86_64
      ```

      版本参考：

      ```
      - linux22    for Linux 2.2
      - linux24    for Linux 2.4 and above (default)
      - linux24e    for Linux 2.4 with support for a working epoll (> 0.21)
      - linux26    for Linux 2.6 and above
      - linux2628  for Linux 2.6.28, 3.x, and above (enables splice and tproxy)
      - solaris    for Solaris 8 or 10 (others untested)
      - freebsd    for FreeBSD 5 to 10 (others untested)
      - netbsd      for NetBSD
      - osx        for Mac OS/X
      - openbsd    for OpenBSD 5.7 and above
      - aix51      for AIX 5.1
      - aix52      for AIX 5.2
      - cygwin      for Cygwin
      - haiku      for Haiku
      - generic    for any other OS or version.
      - custom      to manually adjust every setting
      ```

      根据版本参考，这里我们选择`linux2628`版本进行编译。

   3. 编译到指定目录

      ```
      cd haproxy-2.0.8
      make TARGET=linux2628 PREFIX=/opt/ci123/haproxy
      
      // 这里出现报错，从2.0版本开始linux2628已被废弃
      Target 'linux2628' was removed from HAProxy 2.0 due to being irrelevant and
      often wrong. Please use 'linux-glibc' instead or define your custom target
      by checking available options using 'make help TARGET=<your-target>'.
      
      // 根据提示修改参数后编译
      make TARGET=linux-glibc PREFIX=/opt/ci123/haproxy
      make install PREFIX=/opt/ci123/haproxy
      ```

2. 配置

   1. 复制 `haproxy`命令至全局变量

      ```
      cp /opt/ci123/haproxy/sbin/haproxy /usr/bin/
      ```

   2. 创建系统用户

      ```
      useradd -r haproxy
      ```

   3. 添加`haproxy`配置文件

      1. `haproxy`配置文件由五部分组成：
         - `global`： 参数是进程级的，通常和操作系统相关。这些参数一般只设置一次，如果配置无误，就不需要再次配置进行修改。

      - `default`：默认参数。
        - `frontend`：用于接收客户端请求的前端节点，可以设置相应的转发规则来指定使用哪个`backend`。
        - `backend`：后端服务器代理配置，可实现代理多台服务器实现负载均衡、为请求添加额外报文数据等功能。
        - `listen`：是`frontend`和`backend`的结合，通常只对`tcp`流量有用。

      2. 添加配置文件`/opt/ci123/haproxy/conf/haproxy.cfg`

         ```
         # 全局配置
         global
             log 127.0.0.1 local3 # 设置日志
             pidfile /opt/ci123/haproxy/logs/haproxy.pid
             maxconn 4000                 # 最大连接数
             user haproxy
             group haproxy
             daemon                       # 守护进程运行
         
         # 默认配置
         defaults
             log     global
             mode    tcp                 # 默认的模式mode { tcp|http|health }，tcp是4层，http是7层，health只会返回OK
             option  httplog              # http 日志格式，仅在http模式下可用
             option dontlognull           # 不记录健康检查日志信息；
             option  redispatch           # serverId对应的服务器挂掉后,强制定向到其他健康的服务器
             option http-server-close
             #option  abortonclose        # 当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接；
             #option  forwardfor          # 如果后端服务器需要获得客户端真实ip需要配置的参数，可以从Http Header中获得客户端ip；
             #option  httpclose           # 主动关闭http通道,每次请求完毕后主动关闭http通道,ha-proxy不支持keep-alive,只能模拟这种模式的实现;  
             balance roundrobin           # 负载均衡算法,轮询；
             retries 3                    # 重试次数；
         
             timeout http-request  10s    # 客户端建立连接但不请求数据时，关闭客户端连接；
             timeout queue         1m     # 高负载响应haproxy时，会把haproxy发送来的请求放进一个队列中，timeout queue定义放入这个队列的超时时间；
             timeout connect 10s          # 定义haproxy将客户端请求转发至后端服务器所等待的超时时间；
             timeout client 1m            # 客户端非活动状态的超长时间(默认毫秒)
             timeout server 1m            # 服务端与客户端非活动状态连接的超时时间。(默认毫秒)
             timeout http-keep-alive 10s  # 定义保持连接的超时时长；
             timeout check 10s            # 心跳检测超时；
             maxconn 3000                 # 每个server最大的连接数；
             
         #前端配置
         frontend rabbitmq_cluster_front
             bind 0.0.0.0:10000           # http请求的端口，会被转发到设置的ip及端口
             default_backend rabbitmq_cluster_back
         
         # 后端配置
         backend rabbitmq_cluster_back
             #roundrobin 轮询方式
             balance roundrobin           # 负载均衡的方式,轮询方式
             
             # 配置Rabbitmq连接负载均衡
             # 需要转发的ip及端口
             # inter 2000 健康检查时间间隔2秒
             # rise 3 检测多少次才认为是正常的
             # fall 3 失败多少次才认为是不可用的
             # weight 30 权重
             server clusterRabbit1 192.168.3.14:5672 check inter 2000 rise 3 fall 3 weight 30
             server clusterRabbit2 192.168.3.14:5673 check inter 2000 rise 3 fall 3 weight 30
             
         # 统计页面配置
         listen admin_stats  
             bind 0.0.0.0:10080           # 监听IP和端口，为了安全可以设置本机的局域网IP及端口；
             mode http
             option httplog               # 采用http日志格式  
             stats refresh 30s            # 统计页面自动刷新时间  
             stats uri /haproxy     # 状态管理页面，通过/haproxy来访问
             stats realm Haproxy Manager  # 统计页面密码框上提示文本  
             stats auth duomai:shijiemori@2012  # 统计页面用户名和密码设置  
             #stats hide-version          # 隐藏统计页面上HAProxy的版本信息
         ```

   4. 配置日志 `rsyslog`

      ```
      vim /etc/rsyslog.conf
      # 取消如下2行注释
      $ModLoad imudp
      $UDPServerRun 51
      
      # 新增配置（自定义的日志设备）
      local3.*  /opt/ci123/haproxy/logs/haproxy.log
      
      # 重启rsyslog服务
      systemctl restart rsyslog
      ```

   5. 启动 `haproxy`

      ```
      haproxy -f /opt/ci123/haproxy/conf/haproxy.cfg
      ```

      访问统计页面出现如下界面：

         ![haproxy管理](https://img2.ciurl.cn/flashsale/upload/xinfotek_upload/2019/11/01/1572597613365029.png)

## 三、集群测试

​	***沿用上一篇【RabbitMq 镜像队列集群搭建】中的集群测试环境*，在测试中将`Publiser`和`Consumer`的连接替换为`HAProxy`的地址`192.168.3.14:10000`。**

1. 测试环境

   1. 节点：
      1. 节点一：
         - `clusterRabbit1`
         - 端口：`192.168.3.14:5672`
      2. 节点二：
         - `clusterRabbit2`
         - 端口：`192.168.3.14:5673`
   2. 队列：
      3. 节点一 `rabbit@clusterRabbit1`：
         1. 队列一：
            - `name`：`clusterRabbit1Queue1`
            - `routing_key`：`clusterRabbit1key`
         2. 队列二：
            - `name`：`clusterRabbit1Queue2`
            - `routing_key`：`clusterRabbitCommonKey`
      4. 节点二 `rabbit@clusterRabbit2`：
         1. 队列三：
            - `name`：`clusterRabbit2Queue1`
            - `routing_key`：`clusterRabbit2key`
         2. 队列四：
            - `name`：`clusterRabbit2Queue2`
            - `routing_key`：`clusterRabbitCommonKey`（与队列二 相同）

2. 启动消费者

   1. 消费者

      1. 消费者一：
         - 连接节点：节点一 `rabbit@clusterRabbit1`
         - 消费队列：队列一 `clusterRabbit1Queue1`
      2. 消费者二：
         - 连接节点：节点一 `rabbit@clusterRabbit1`
         - 消费队列：队列二 `clusterRabbit1Queue2`
      3. 消费者三：
         - 连接节点：节点二 `rabbit@clusterRabbit2`
         - 消费队列：队列三 `clusterRabbit2Queue1`
      4. 消费者四：
         - 连接节点：节点二 `rabbit@clusterRabbit2`
         - 消费队列：队列四 `clusterRabbit2Queue2`

   2. 启动结果：

      启动成功，但是在一分钟后客户端异常退出，原因是`HAProxy`设置了`timeout client 1m 和 timeout server 1m`，消费者在一分钟内都没有接收到消息导致被判定为不活跃连接从而被删除。

      由于`HAProxy`默认不支持长连接，上述问题可以使用`pm2`管理消费者的方法来解决，消费者进程在不活跃退出后`pm2`将自动重启此进程。

3. 发布消息

   1. 发布`clusterRabbit1key`消息
      - 消费者一 成功收到消息。
   2. 发布`clusterRabbitCommonKey`消息
      - 消费者二/四 成功收到消息。

4. 关闭节点二`rabbit@clusterRabbit2`后再次发布消息

   - 连接节点二的消费者先退出后重新使用`HAProxy`成功连接。
   - 发布的消息均能被成功消费。

## 四、使用说明

1. `HAProxy`默认不支持`tcp` 长连接，需要使用`PM2`之类的守护进程管理工具或者长连接技术来实现`rabbitMq`客户端持续连接。
2.  `HAProxy`通过活跃检测机制来判定负载均衡中的节点是否可用， 当`rabbitMq`集群中某个节点不可用时，在经过一段时间的活跃检测之后，`HAProxy`将弃用该节点直至节点恢复。在这种情况下，`rabbitMq`消费者将断开连接后选择剩余可用的节点再次启动，客户端发布时也会自动选择剩余可用的节点。
3. 在`rabbitMq`集群中某个节点宕机之后，`HAProxy`会自动使用可用的集群节点，所以不会出现在`HAProxy`活跃检测期间发布消息出现一半成功一半失败的情况，所有的消息都将通过可用的集群节点发布至集群。