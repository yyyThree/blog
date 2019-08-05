# vagrant简易教程

**vagrant简易安装教程：**https://github.com/astaxie/go-best-practice/blob/master/ebook/zh/01.0.md

**实际使用遇到的问题：**

----------


 - **CentOS 7 访问80端口失败**

    CentOS 7 默认没有使用iptables，所以通过编辑iptables的配置文件来开启80端口是不可以的，CentOS 7 采用了firewalld防火墙。

    1、查询80端口是否打开 `firewall-cmd --query-port=80/tcp`
    
    2、开启80端口 `firewall-cmd --add-port=80/tcp`
    
    

----------


 - **修改了一些配置文件，重启vagrant后失效**
使用vagrant up/reload会导致部分配置文件被恢复到初始状态，比如已经启动的进程消失，开启的80端口关闭。
解决方案：
**方案一**：使用vagrant休眠/唤醒代替关闭/启动（最简单）
1、休眠虚拟机：`vagrant suspend`
2、唤醒虚拟机：`vagrant resume`

   **方案二**：[使用vagrant预处理脚本][1]
1、修改Vagrantfile，在需要添加预处理脚本的虚拟机设置里增加一行
`config.vm.provision "shell", inline: "/bin/sh /home/cront/autoload.sh"`，*此处的config为虚拟机名*。
2、再次启动vagrant  `vagrant up --provision`

    *注意，由于文件权限的问题（改成root登录也会存在问题），此方案会导致部分脚本启动失败，建议使用方案三。*

   **方案三**：[设置linux开机启动项][2]

----------


 - **创建自己的vagrant box**
 1、生成box
 `vagrant package --output my_vagrant.box my_vagrant`
my_vagrant是虚拟机实例名称，my_vagrant.box是box名。在创建box的时候，如果虚拟机实例正在运行，vagrant会先halt，然后再export。
2、将box添加到本地仓库
`vagrant box add my_vagrant my_vagrant.box`
3、查看本地仓库的box列表：
`vagrant box list`
4、创建并且切换到用于测试的项目目录
`mkdir ./test_vagrant && cd ./test_vagrant`
5、创建Vagrantfile
`vagrant init "my_vagrant"`
6、启动虚拟机
`vagrant up`

----------

 - **配置虚拟机挂载**

    在实际操作中，我们必然希望能够在修改本地代码之后能够在虚拟机环境中实时运行修改的结果，如果直接在虚拟机中修改肯定不如本地IDE编辑来的便捷。如果使用版本控制，如git，也需要每次修改完本地push，虚拟机pull。时间久了肯定也会觉得麻烦，如果能够将本地文件挂载到虚拟机，一切都会变得更加便捷，下面展示下实现方法：

    1、在Vagrantfile中找到被注释的config.vm.synced_folder,这就是我们需要的挂载命令。
    2、实现挂载：`web.vm.synced_folder "D:/test/", "/vagrant"`
        上段命令的作用是将本地的`D:/test/`文件夹挂载至虚拟机内部的`/vagrant`文件夹下。其中第一个参数可以设置为相对路径，默认根目录为Vagrantfile所在的目录，也可以像上面一样设置绝对路径。
        *注意第一个路径不能使用\反斜杠，vagrant运行时会无法识别*
    

[1]: https://www.vagrantup.com/docs/provisioning/shell.html
[2]: https://www.zybuluo.com/mdeditor#1052444