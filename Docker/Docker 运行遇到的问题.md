# Docker 运行遇到的问题

 - **在容器内部无法使用网络** 

    + `vim /usr/lib/sysctl.d/00-system.conf`
    添加如下代码：
    + `net.ipv4.ip_forward=1`
    重启network服务
    + `systemctl restart network`
      



----------


 - **删除docker文件夹后无法启动docker容器**
`/usr/bin/docker-current: Error response from daemon: error creating overlay mount to /var/lib/docker/overlay2/66e8df15fe16a7977a70c032460a096e82677885f608f3028437f2f82ff05bd2/merged: invalid argument.
See '/usr/bin/docker-current run --help'.`

    这个是因为用的overlay2文件系统，而系统默认只能识别overlay文件系统
    所以我们就要更新文件系统了。
   
    **解决方案：**
   
   
   - `systemctl stop docker      //停掉docker服务`
   - `rm -rf /var/lib/docker     //注意会清掉docker images的镜像`
   - `vi /etc/sysconfig/docker-storage       //将文件里的overlay2改成overlay即可`
   - `vi /etc/sysconfig/docker         //去掉option后面的--selinux-enabled`
   - 重启docker服务