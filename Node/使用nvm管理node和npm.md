# 使用nvm管理node和npm

## 一、nvm的作用

    不同项目需要依赖不同版的NodeJS 运行环境，nvm实现了不同版本的node包管理

## 二、卸载全局安装的 node/npm

## 三、Windows 安装nvm

    1. 下载地址：https://github.com/coreybutler/nvm-windows/releases
    
    	下载nvm-noinstall.zip
    
    2. 自动安装失败时，手动添加settings.txt文件
    
        root: D:\nvm  // nvm的存放地址
        path: D:\node // 存放指向node版本的快捷方式，使用nvm的过程中会自动生成。一般写的时候与nvm同级
        arch: 64
        proxy: none 
        node_mirror: http://npm.taobao.org/mirrors/node/
        npm_mirror: https://npm.taobao.org/mirrors/npm/
    
    3. 设置全局环境变量
    
        NVM_HOME: D:\nvm
        NVM_SYMLINK: D:\node
        PATH:%NVM_HOME%;%NVM_SYMLINK%(在PATH的最后添加）

## 四、安装多版本的node/npm

    1. 安装node
    
        nvm install [node版本]
    
    2. 安装完node之后自动会安装对应的npm
    
    3. 切记要使用安装完的包，使用后会自动设置node和npm的全局变量
    
        nvm use [version]

## 五、nvm的一些基本命令

    1. 查看已安装列表
    
    	nvm ls
    
    2. 切换版本
    
    	nvm use [node版本]
    
    3. 卸载
    
    	nvm uninstall [version]
