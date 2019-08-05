# 使用nrm管理npm的源

## 一、nrm的作用

    nrm是一个npm源管理器，允许你快速地在 npm 源间切换。

## 二、安装nrm

    全局安装nrm：npm install -g nrm

## 三、一些常用命令

    1、查看可用的源
    
    	nrm ls
    
    2、切换源
    
    	nrm use <registry>
    
    3、添加源
    
    	nrm add <registry> <url>
    
    	其中reigstry为源名，url为源的路径
    
    4、删除源
    
    	nrm del <registry>
    
    5、测试源的速度
    
    	nrm test <registry>