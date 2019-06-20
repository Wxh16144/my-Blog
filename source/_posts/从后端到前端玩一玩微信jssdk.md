---

title: 从后端到前端玩一玩微信jssdk

date: 2018-09-26 20:26:07

tags: ['nginx','javascript','php']

summary: 安装并配置微信开发授权，使用JSSDK
---

因为平常在公司经常需要使用微信的各种api , 本地测试页比较麻烦 , 笔者索性自己配一个 , 方便自己本地调试的同时 , 也借这个机会学习一下 ,;

<!--more-->

下面就和大家 分享一下自己从零使用微信公众号并配置到云服务器的奇妙之旅. 

#### 准备工作

一个完成备案的**域名**;
一台**云服务器**;
注册一个**微信公众号** (可以是个人订阅号);
准备 **微信web开发者工具**[下载地址](https://mp.weixin.qq.com/cgi-bin/safecenterstatus?action=devlist&token=918564786&lang=zh_CN)  **wampServe**[下载地址](https://sourceforge.net/projects/wampserver/) **nginx**[下载地址](http://nginx.org/en/)



#### 写在前面

笔者在[阿里云](www.net.cn)购买域名  [腾讯云](https://cloud.tencent.com)购买服务器 ;
域名备案大家可以网上先查找文档 ; (笔者后期更新) 
腾讯云服务器如何登录也建议大家先查看文档  (笔者后期更新) ;
  安装 wampServe
  安装 nginx 
笔者假定大家都具备以上这些条件;
折腾完了 , 有大佬给我推荐了 [easywechat](https://www.easywechat.com/) 感兴趣的可以看这个 , 下面的文章比较麻烦



#### 域名解析

添加一个二级域名 : **api.example.com** 和 一级域名 : **examople.com**

解析到自己购买的服务器公网地址

  ![域名解析](/images/从后端到前端玩一玩微信jssdk/2018092601.png)



#### 云服务器安装 wampServe (未提供)

大家网上查找资料 , 和平常配置一样 ;
注意 : wamp安装后端口默认**80** , 笔者设置为**8080** ; 下文请大家注意 



#### 云服务器 Nginx 配置

如果大家`nginx`配置了 **ssl证书** 建议先关闭.

```nginx
# 一级域名 
server{
       listen  80; # 监听80端口
       server_name wxhboy.cn; # 域名
       server_name_in_redirect off;
       location / {
            proxy_pass http://127.0.0.1:8080/; # 代理到wampServe本地 根目录
       }
    }
# 二级域名
 server{
       listen  80;
       server_name api.wxhboy.cn;
       server_name_in_redirect off;
       location / {
           proxy_pass http://127.0.0.1:8080/wxjssdk/; # 代理到wampserver /wxjssdk目录
       }
    }
```

访问自己域名 example.com 出现图片完成
 ![域名解析](/images/从后端到前端玩一玩微信jssdk/2018092602.png)
如果大家和我一样配置 基本上不会出现问题 , 有问题也请大家上网查看资料 , 或联系笔者

#### 下载php 和 demo文件

[Github下载地址](https://github.com/loo2k/wxjssdk-signature)
[笔者整理的文件](http://www.wxhboy.cn/download/wxjssdk+php.zip)

#### 修改配置文件 

**wxjssdk**文件夹打开 `index.php`  ; **github**下载打开 `signature.php`

  修改获取请求  **设置公众号开发信息**

    ![](/images/从后端到前端玩一玩微信jssdk/2018092604.png)

  ```php
  // $url = $_GET['url']; //默认
  $url = isset($_REQUEST['url']) ? $_REQUEST['url'] : '';  // 笔者修改后
  $jssdk = new JSSDK('开发者ID(AppID)', '开发者密码(AppSecret)'); //  根据你公众号的开发信息填写
  ```

  添加跨域请求头 **注意修改成自己的域名**  **端口号也要加上 注意是前端的端口号!!!**

  ```php
  // ...
  function setHeader(){
  	$origin = isset($_SERVER['HTTP_ORIGIN'])? $_SERVER['HTTP_ORIGIN'] :'';
  	$allow_origin = array(
  			'http://wxhboy.cn:8080', // 跨域 前端的端口号 !!!!!
          // 这里的端口号是你本地测试的端口号
          // 一定要注意的地方
  	);
  	if(in_array($origin, $allow_origin)){
  		header("Access-Control-Allow-Origin: ".$origin);
  		header('Access-Control-Allow-Credentials: true');
  	}
  }
  setHeader();
  // ...
  ```

**wxjssdk/cache** 目录下有三个文件夹

  `.htaccess` 、 `access_token.json` 、`jsapi_ticket.json`

  因为代码中使用的是文件缓存可能导致 `access_token` 泄露，所以请保证 `/cache` 文件夹不可被 HTTP 直接访问，所以需要对 `/cache` 文件夹做权限的限制。

  如果使用wamp直接默认 

  **access_token.json**

    ``` javascript
    {
      "expire_time": 1537936345, // 过期时间戳
      "access_token": "...." // access_token
    }
    ```

  **jsapi_ticket.json**
    ```javascript
    {
      "expire_time": 1537936345,  // 过期时间戳
      "jsapi_ticket": "..." // access_token
    }
    ```

查看[微信官方文档 access_token](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)我们可以知道他的有效时间是 7200秒 而且每天的请求次数有上限 , 所以使用了文件缓存 ;  **这也是后面调试的时候需要注意的地方(如果你修改了你的 `AppID` 和 `AppSecret` ) 很有必要把过期时间设置为 0 或者删除文件**

上面代码就完成了基本配置 , 把文件夹所有内容拷贝到 wamp根目录即可; 通过域名 examople.com 来进行测试

![](/images/从后端到前端玩一玩微信jssdk/2018092603.png)



####  配置 token.php 文件

笔者参考[文档地址](https://blog.csdn.net/Aaroun/article/details/80887137) , 可以先了解一下 ;
打开文件 修改你的**token** 就好了 (记住你填写的内容) 下面会用到

```php
// 其他部分不修改
define('TOKEN','你的token');  // 输入您的tokne 微信官方文档要求必须为英文或数字，长度为3-32字符。
// 其他部分不修改
```

**修改好文件后 把文件拷贝到之前配置的二级域名目录下** 笔者是wamp根目录下的`wxjssdk`文件夹下



#### 设置公众开发信息Ip白名单

如果需要在本地开发需要把自己的**本地IP**添加  获取[本地访问IP](http://ip.qq.com/)
添加服务器**公网IP**

![](/images/从后端到前端玩一玩微信jssdk/2018092605.png)



####  设置服务器配置


**Url** 就是我们之前配置的二级域名 api.examople.com/touke.php

**Token** 就是上面 配置**token.php**文件时填写的`token`内容

**EncodingAESKey** 点击随机生成

**消息加解密方式** 笔者默认明文模式 安全模式 大家先自行研究

![](/images/从后端到前端玩一玩微信jssdk/2018092606.png)



#### 如果你看到这里 基本上已经完成了

**无需公众帐号、快速申请接口测试号 , 直接体验和测试公众平台所有高级接口;**

[微信公众平台接口测试帐号申请](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login&token=918564786&lang=zh_CN)
如果要添加测试账号 配置步骤一样 你可以在一级域名下新建一个文件夹**test** 再配置一次测试公众号
 测试账号设置 **JS接口安全域名** 添加自己的**一级域名** `example.com` 

![](/images/从后端到前端玩一玩微信jssdk/2018092612.jpg)

## 测试自己的配置

测试文件(demo) 笔者也分享到了之前下载的文件夹中  
笔者这里测试使用了**本地测试**

#### 修改本地host文件

window目录: `C:\Windows\System32\drivers\etc`
修改**127.0.0.1** 为 自己的一级域名 **example.com**

![](/images/从后端到前端玩一玩微信jssdk/2018092607.png)

#### 修改demo/index.html文件

修改请求地址为自己的二级域名地址
![](/images/从后端到前端玩一玩微信jssdk/2018092608.png)





#### 开启8080端口测试

用微信web开发者助手进行测试

  ![](/images/从后端到前端玩一玩微信jssdk/2018092609.png)

##### 恭喜配置成功!!!

---



## 遇到的问题

nginx 和 wampserver 问题笔者这里不过多阐述了

**token 验证失败**

   检查自己输入的信息时候正确
   二级域名是否可以访问
   自己的二级域名`api.example.com` 是否正确添加**token.php**文件? 

  ![](/images/从后端到前端玩一玩微信jssdk/2018092610.png)

**跨域阻塞问题**
  检查前面设置**跨域资源头**是否正确添加
  一级域名` example.com` 能否访问
  **本地host文件**是否修改
  **本地端口和服务器设置的端口是否对应( 最最最关键)**

  ![](/images/从后端到前端玩一玩微信jssdk/2018092611.png)

笔者基本上遇到最难解决的问题上述清楚了 ,  如果大家再捣鼓过程中遇到问题 ,  最好的解决方法还是自己看官方文档检查 

文章开头安装 **nginx** 和 **wampserver**  都是笔者资质尚浅,怕误人子弟,  没有过多描述 , .也是第一次写分享类文章 , 写的不好还请大家多多包含 , 后续我会分享具体微信Api , 需要注意的地方 !!

Hexo个人博客贴一个: [http://www.wxhboy.cn](http://www.wxhboy.cn)
