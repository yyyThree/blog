# RabbitMq账户权限升级

## 一、说明

    RabbitMq支持多种用户标签，分别代表不同的用户权限。
    
    现有Mq系统中所有的用户都是administrator最高权限，其实是不合理的。
    
    应当建立不同的用户来实现管理、监控、业务等不同功能对应的账户权限切割。

## 二、用户权限

    1. 用户权限分为configure、write、read三种，用于控制用户对exchange，queue的操作权限。
    
        用户权限对应不同的vhost
    
    2. configure：exchange，queue的声明和删除
    
        exchange.declare
    
        exchange.delete
    
        queue.declare
    
        queue.delete
    
    3. write：exchange，queue的routing_key绑定、消息发布
    
        exchange.bind
    
        exchange.unbind
    
        queue.bind
    
        queue.unbind
    
        basic.publish
    
    4. read：读取消息进行消费
    
        basic.get
    
        basic.consume
    
        queue.purge

## 三、五种不同的用户标签（用于Management管理插件）

    1. 无标签
    
    	仅用于业务系统，无法登录管理后台以及调用api
    
    2. management
    
        a. 列出可以通过AMQP登录的虚拟主机
    
        b. 查看虚拟主机中的所有队列、交换器和绑定key
    
        c. 查看并关闭它们自己的通道和连接
    
    	d. 查看覆盖所有虚拟主机的“全局”统计数据，包括其中其他用户的活动
    
    3. policymaker （目前系统使用的MQ 3.1.3 版本，没有此标签）
    
        a. management用户所有权限
    
        b. 查看、创建和删除可通过AMQP登录的虚拟主机的策略和参数
    
    4. monitoring
    
        a. management用户所有权限
    
        b. 列出所有虚拟主机，包括无法使用消息传递协议访问的主机
    
        c. 查看其他用户的连接和通道
    
        d. 查看节点级数据，如内存使用和集群
    
        e. 查看所有虚拟主机的真实全局统计数据
    
    5. administrator（最高级）
    
        a. management用户 + monitoring用户 所有权限
    
        b. 创建和删除虚拟主机
    
        c. 查看、创建和删除用户
    
        d. 查看、创建和删除权限
    
        e. 关闭其他用户的连接

## 四、修改方案

    1. 统一使用四类用户账号
    
    	a. 超级管理员账号：对全部 `vhost` 开放任意权限 + `administrator` 标签
    	
    		用于统一管理所有的MQ 账号、`vhost`、节点信息、集群策略等
    
    	b. 监控账号：对全部 `vhost` 开放任意权限  + `monitoring` 标签
    
    		用于统一的监控报警系统使用
    
    	c. `vhost` 管理员账号：对应 `vhost` 开放所有权限 + `policymaker`（无 `policymaker` 使用 `management` 代替） 标签
    
    		用于管理固定 `vhost` 使用，声明队列、交换机
    
    	d. `vhost` 业务使用账号：对应 `vhost` 开放 `read`、`write` 权限 + `none` 标签
    
    		用于业务系统发布消息、订阅队列
    
    2. 项目中使用的交换机、队列、routing_key统一写入配置文件
    
    3. 将队列、交换机的声明和业务系统解耦
    
    	a. 需要声明的交换机写入配置文件
    
    		读取配置文件，声明所有交换机，新交换机的声明走单独脚本
    
    	b. 需要声明的队列和routing_keys写入配置文件
    
    		读取配置文件，声明所有的队列并绑定routing_keys,新队列的声明走单独脚本
    
    4. 修改旧业务系统使用，只允许发布消息、订阅消息，不允许操作交换机、队列等。
    
    5. 修改MQ sdk，将交换机的操作和发布、订阅消息解耦


​        

​        