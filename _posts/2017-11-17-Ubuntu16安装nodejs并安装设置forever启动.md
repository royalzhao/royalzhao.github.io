在公司原有tomcat环境下想安装nodejs环境进行开发，因为本人是前端工程师，故此参考一些别人的博客简书写下这篇文章，原文链接如下  
[Ubuntu16.04安装最新版nodejs ](http://www.jianshu.com/p/2b24cd430a7d)     
[forever让nodejs应用后台执行 ](http://cnodejs.org/topic/5021c2cff767cc9a51e684e3)  

# 安装最新版nodejs
### 更新ubuntu软件源

    sudo apt-get update
    sudo apt-get install -y python-software-properties software-properties-common
    sudo add-apt-repository ppa:chris-lea/node.js
    sudo apt-get update


安装nodejs

    sudo apt-get install nodejs
    sudo apt install nodejs-legacy
    sudo apt install npm

更新npm的包镜像源，方便快速下载

    sudo npm config set registry https://registry.npm.taobao.org
    sudo npm config list

全局安装n管理器(用于管理nodejs版本)

    sudo npm install n -g

安装最新的nodejs（stable版本）

    sudo n stable
    sudo node -v

可以通过以下命令检查node是否安装成功  

    node -v

# 安装forever  
    $ sudo npm install forever -g   #安装
    $ forever start app.js  #启动
    $ forever stop app.js   #关闭
    $ forever start -l forever.log -o out.log -e err.log app.js   #输出日志和错误

如有问题请留言！