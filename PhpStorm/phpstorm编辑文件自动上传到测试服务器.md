# phpstorm编辑文件自动上传到测试服务器

1. **背景**
---------

    在测试机开发的时候，经常会直接登录到测试机上使用vim进行代码编辑
    在vim的简单模式使用的时候，经常会写一些非常低级的错误代码


2. **目的**
---------

    测试机开发的时候，也能使用IDE编码
    提升效率、降低各种低级错误

3. **实现**
---------

    测试机开启ftp服务
    将html目录作为ftp服务根节点
    将整个html节点的用户归属改为userftp（权限问题）
    phpstorm设置 tools deploment 
    参考 https://www.cnblogs.com/jikey/p/3486621.html
    配置 tools => deploment => configuration
    connection中，选择ftp，192.168.0.235，账号密码 userftp fuyuan1906
    测试连接，假如连接不上请选择Advanced options的被动模式Passive mode
    mapping中例如：
    local path: /Users/yuanliang/project/corp/api_yl
    server path: /apibeta_shop/yl
    web path： /
    配置 tools => deploment => options
    关闭preserve files timestamps
    upload change files automatically to the default server，选择保存的时候触发
    开启 stop operation on the first error

4. **效果**
---------

    修改local path中对应的文件
    phpstorm 自动上传代码到测试机
    查看修改后的效果