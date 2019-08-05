# CentOS 7 系统相关配置

---

 - **修改系统编码**
   
    在centos7修改系统编码不再是/etc/sysconfig/i18n路径，而是/etc/locale.conf

----------


 - **修改系统时区**
 
1. 查看当前系统时间及时区
    
       ```
       timedatectl status  
       ```
    
    2. 修改时区为北京时间
    
       ```
       rm -rf /etc/localtime
       ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
       ```

----------


​        
 - **查看CPU信息** 

    `cat /proc/cpuinfo`
    
    - model name 型号
    - physical id 物理CPU个数
    - processor 逻辑CPU个数
    - cpu cores CPU核数

----------

 - **无法PING通外网的解决方案**
1. 在文件 /etc/sysconfig/network-scripts/ifcfg-enp0s8 中配置DNS1=8.8.8.8
   2. 重启网络服务：/etc/init.d/network restart

---------

 

 - **添加开机启动脚本**

    1. 打开linux开机启动设置文件
    
       ```
       vim /etc/rc.d/rc.local
       ```
    
    2. 添加开机启动脚本
    
       ```
       sh /home/cront/autoload.sh
       ```
    
    3. 设置文件权限
    
       ```
       chmod +x /etc/rc.d/rc.local
       ```

----------

 - **网卡配置**
 1. 配置文件地址 /etc/sysconfig/network-scripts/ifcfg-eth0
2. 配置项
   `TYPE=Ethernet    # 网卡类型：为以太网
   PROXY_METHOD=none    # 代理方式：关闭状态
   BROWSER_ONLY=no      # 只是浏览器：否
   BOOTPROTO=dhcp  #设置网卡获得ip地址的方式，可能的选项为static(静态)，dhcp(dhcp协议)或bootp(bootp协议).
   DEFROUTE=yes        # 默认路由：是, 不明白的可以百度关键词 `默认路由`
   IPV4_FAILURE_FATAL=no     # 是不开启IPV4致命错误检测：否
   IPV6INIT=yes         # IPV6是否自动初始化: 是[不会有任何影响, 现在还没用到IPV6]
   IPV6_AUTOCONF=yes    # IPV6是否自动配置：是[不会有任何影响, 现在还没用到IPV6]
   IPV6_DEFROUTE=yes     # IPV6是否可以为默认路由：是[不会有任何影响, 现在还没用到IPV6]
   IPV6_FAILURE_FATAL=no     # 是不开启IPV6致命错误检测：否
   IPV6_ADDR_GEN_MODE=stable-privacy   # IPV6地址生成模型：stable-privacy [这只一种生成IPV6的策略]
   NAME=ens34     # 网卡物理设备名称  
   UUID=8c75c2ba-d363-46d7-9a17-6719934267b7   # 通用唯一识别码，没事不要动它，否则你会后悔的。。
   DEVICE=ens34   # 网卡设备名称, 必须和 `NAME` 值一样
   ONBOOT=no #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备 
   IPADDR=192.168.103.203   #网卡对应的ip地址
   PREFIX=24             # 子网 24就是255.255.255.0
   GATEWAY=192.168.103.1    #网关  
   DNS1=114.114.114.114        # dns
   HWADDR=78:2B:CB:57:28:E5  # mac地址`

 
