本人是centos的服务器，下面教大家配置ngnix服务器并搭建php的运行环境  
## 1、安装ngnix  
    yum install nginx  
安装完成后可以启动nginx，在浏览器里面访问，查看nginx是否安装成功。端口默认为80。  

    systemctl start nginx  
    nginx中yum安装的默认网站根目录在/usr/share/nginx/html  

运行成功会出现一个欢迎界面，  表示已成功安装nginx.  

## 2、安装PHP和PHP-FPM  
    yum install php php-fpm  

    启动php-fpm  
    systemctl start php-fpm  

## 3、将PHP与mysql模块关联起来  

此处是mariadb数据库  

    安装  
    yum install mariadh mariadb-server  
    关联  
    yum install php-gd php-mysql php-mbstring php-xml php-mcrypt  php-imap php-odbc php-pear php -xmlrpc  
## 4、配置nginx与php一起工作  
打开nginx主配置文件。

vim /etc/nginx/nginx.conf  

    在http模块中添加配置：  
           location / {  
            root   /usr/share/nginx/html;  
               index  index.html index.htm index.php;  
            }  
    location ~ \.php$ {  
               root           html;  
               fastcgi_pass   127.0.0.1:9000;  
               fastcgi_index  index.php;  
               fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  
               include        fastcgi_params;  
           }  

改动nginx默认的fastcgiparams配置文件： vim /etc/nginx/fastcgi_params 在文件的最后增加两行： 

    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  
    fastcgi_param PATH_INFO                $fastcgi_script_name;  

然后重启一下服务：  

    service nginx restart
    
    service php-fpm restart

## 5、运行

在网站根目录创建一个index.php文件  

文件内容如下： 
 
    <?php    
    phpinfo();    
    ?>    

提示nginx中yum安装的默认网站根目录在/usr/share/nginx/html   

故此在此文件夹下新建文件  
正常情况下就可以运行并访问php文件了。