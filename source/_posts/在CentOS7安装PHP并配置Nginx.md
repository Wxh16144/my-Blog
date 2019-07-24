---
title: 在CentOS7安装PHP并配置Nginx
date: 2019-07-24 13:37:36
tags: ['nginx','linux']
---

`Windows server`翻车后，我就安装了`CentOS7`，为了之前的某些**php**项目可以跑起来，我不得不去安装php环境和配置一些文件;
<!-- more -->


### 准备工作
+ **CentOS7**服务器；
+ 安装**Nginx**； [在CentOS7安装Nginx并配置nginx.conf]()；


### 1. 安装 **epel-release**
```bash
    yum -y install epel-release
```
<div style="width:100%; display:flex;">
    <img style="" src="/images/在CentOS7安装PHP并配置Nginx/1.png" alt="安装epel-release过程" />
    <img style="" src="/images/在CentOS7安装PHP并配置Nginx/2.png" alt="安装epel-release成功" />
</div>

### 2. 安装**PHP7**
首先我们先来获取PHP7.0的yum源，执行下面的指令：
```bash
    rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```
![获取PHP7.0的yum源](/images/在CentOS7安装PHP并配置Nginx/3.png)

获取成功后我们可通过下面的指令来查看php7.0的扩展名称，可以按照自己的要求安装;

查看扩展名指令：
```bash
    yum search php70w
```
![查看扩展名指令](/images/在CentOS7安装PHP并配置Nginx/4.png)

查看到扩展名称后可以选择自己的要求安装几个，想要什么扩展后期也可以安装上去不用担心，指令也是一样的:
```bash
    yum install { 写扩展名 }
```
下面我推荐安装这几个通用的安装指令：
```bash
yum install php70w php70w-fpm php70w-cli php70w-common php70w-devel php70w-gd php70w-pdo php70w-mysql php70w-mbstring php70w-bcmath
```
安装的时候有**两次要输入Y**回车，才会继续安装的，乖乖输入就好了。
<div style="width:100%; display:flex;">
    <img style="" src="/images/在CentOS7安装PHP并配置Nginx/5.png" alt="第一次输入指令y" />
    <img style="" src="/images/在CentOS7安装PHP并配置Nginx/6.png" alt="第二次输入指令y" />
    <img style="" src="/images/在CentOS7安装PHP并配置Nginx/7.png" alt="安装成功" />
</div>

安装成功了，我来查看以下是否安装成功了，输入下面的指令看出版本试试吧！
```bash
    php -v
```
能看到下图的内容就证明安装成功了！
![检查PHP版本](/images/在CentOS7安装PHP并配置Nginx/8.png)

### 3. 启动PHP
检查是否启动
```bash
    ps -ef|grep php
```
如果出现下图，表示未启动
![检查PHP是否启动](/images/在CentOS7安装PHP并配置Nginx/9.jpg)

查看php安装目录
```bash
    whereis php
```
![查看php安装目录](/images/在CentOS7安装PHP并配置Nginx/10.jpg)

开启php-fpm服务
```bash
   service php-fpm start
```
![开启php-fpm服务](/images/在CentOS7安装PHP并配置Nginx/11.jpg)

确认是否启动
```bash
    ps -ef|grep php && netstat -tunlp| grep 9000
```
如果出现下图，表示启动成功，并监听端口**9000**
![检查PHP是否启动](/images/在CentOS7安装PHP并配置Nginx/12.jpg)

关闭php-fpm服务
```bash
    pkill php-fpm
```

### 修改**www&#183;conf**配置文件
我自己使用了默认的配置，查找配置文件目录
```bash
    find / -name www.conf # 查看目录
    cat /etc/php-fpm.d/www.conf # 查看配置文件
```
![查看www.conf配置文件](/images/在CentOS7安装PHP并配置Nginx/13.jpg)
需要修改请参考文档：[linux下php7修改端口号](https://blog.csdn.net/jiangshan35/article/details/53611602)

### 配置**NGINX**
查看PHP的配置文件`www.conf`，获取到资源地址
```nginx
 server {
        listen 80 ; #监听端口
        server_name 'wxhboy.cn';  #域名
        index index.html index.htm index.php;
        root /home/www; #站点目录
        location / {

        }
        location ~ \.php$ {
            root           /var/www/html; # 自己资源地址
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /var/www/html/$fastcgi_script_name; # 这里也要改成自己的资源地址 {资源地址}$fastcgi_script_name
            include        fastcgi_params;
        }
    }
```
### 测试是否配置成功
在资源目录中创建`phpinfo.php`文件;
```php
    <?php
        phpinfo();
    ?>
```
在浏览器中输入地址 http://example.com/phpinfo.php；
出现下图表示服务启动成功；
![php运行成功](/images/在CentOS7安装PHP并配置Nginx/14.jpg)

### 参考文档
+ [Centos7安装PHP、安装MySQL、安装apache](https://www.cnblogs.com/shengChristine/p/9293996.html)
+ [linux下查看php-fpm是否开启以及如何开启](https://blog.csdn.net/eddy23513/article/details/82945365)
+ [linux下php7修改端口号](https://blog.csdn.net/jiangshan35/article/details/53611602)