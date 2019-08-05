# RabbitMq nack使用

## 一、整体说明

    1. 目前的项目中的MQ使用了自动ack机制，可能会造成消息没有到达或者实际上没有成功消费的情况
    
    2. 修改为手动ack/nack机制
    
    3. 为普通队列添加默认的死信队列，收集由于nack被发送到死信队列的消息、来源等数据
    
    4. 将收集到的消息保存一段时间，定期删除

## 二、现有机制测试

    1. 目前自动ack机制下，再进行手动ack会造成什么情况
    
        会导致消费者进程异常退出
    
    2. 消费者忘记ack
    
    	如果消费者设置的不自动ack，且没有对消息进行ack，之后的消息可以正常接收，但是没有ack的消息会一直存活在队列里。
    
        如果有新的消费者会把这个消息发给新消费者，直到消息被ack或者过期。
    
    3. nack和reject的使用
    
    	nack可以删除队列中的消息并发送到死信队列中，reject和nack在这方面的效果相同。

## 三、修改方案

    1. 升级Mq composer包
    
        a. 队列消息过期时间设置为86400 000 
    
        b. 消费队列设置默认不自动ack
    
        c. 自动检测消费者代码中是否包含ack方法
    
        d. 添加删除队列方法
    
    2. 所有消费者添加手动ack/nack代码
    
        a. 消费者在ack之前return，消息将一直残留在队列中，如需return，要再return之前调用nack
    
        b. 消费进程中不要写die，写return
    
    3. 所有队列重新声明
    
        a. 队列消息过期时间设置为86400 000 
    
        b. 队列默认添加一个死信队列
    
    4. 所有pm2常驻进程重启
    
        a. 消费者设置为不自动ack
    
    5. 将nack的消息进行收集，定期删除等
    
    	a. 添加pm2常驻进程，订阅死信队列，用于收集死信消息
    
        b. 获取死信消息
    
            $msgObj->get('application_headers')->getNativeData()['x-death'][0];
    
        c. 死信消息解析（x-death）
    
            exchange：来源交换机
    
            queue：来源队列名
    
            routing-keys：来源队列绑定的key数组
    
            reason：进入死信队列的原因
    
            time：消息的日期和时间
    
            original-expiration：消息的过期时间（消息发布时的expiration）
    
        d、获取源数据
    
            $msgObj->body
    
        e、采集内容（数据表）
    
            table:
    
                + mq_dlx_logs：死信队列记录表
    
                    id：主键id，自增
                    exchange：来源交换机
                    queue：来源队列名
                    routing_keys：来源队列绑定的key数组，json格式
                    data：源数据
                    time：进入死信队列的时间
                    reason：进入死信队列的原因
                    created_at：创建时间
    
        f. 定期删除记录
