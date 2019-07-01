---
title: 记一次WindowServer2012转CentOS7
date: 2019-06-29 12:26:35
tags: ['linux']
---

在**Windows server 2012**上安装了一个 `Nodepad++` 然后服务器中病毒了???

<!--more-->
<div style="width:100%;text-align:center">
    <img style="width:30%; " src="/images/记一次WindowServer2012转CentOS7/what.jpg" />
</div>

### 为什么用WINDOWS!!!

很早之前, 我以学生的身份买了一台 [腾讯云服务器](https://buy.cloud.tencent.com/cvm?tab=lite&loginSet=SET_PASSWORD) ;然后安装了:`Windows Server 2012 R2`;
傻瓜式安装, 安装了`WampServe`、`Node`、`Nginx`、`Java` 等等一堆东西, 把之前云虚拟机上的资源文件搬到了服务器上面, Nginx 配置一下, Wamp配置一下就跑起来;

### 服务器有些什么东西???

一些平常自己写的小游戏, 平常学习写的小Demo, 个人生活照片(拿得出手的那种)图片资源, 有趣的~~小视频~~ , 音乐, 一些常用的资源 `Css`、`Js` 等; 
直到毕业的前一个月部署了自己的个人博客，都在这个服务器上面。 不过自己把博客资源放在的Github, 这一点减少了我的一点损失;


### 然后BOMM!!!

平静的岁月还是被打破了, 今天天气炎热, 朋友黑鬼借我服务器玩什么 `Webhook` 运行 Java ; 我也不知道发生了什么, 下午我引用服务器资源 [Mreset.css](http://wxhboy.cn/css/Mreset); 前面还好好的, 突然资源404?
我链接到服务器查看, 懵逼了, 怎么还重启了, 问题不大,  重新运行 `WampServe`  !!  ~~草~~ 怎么回事, 运行不起来了。

我当然要去找我朋友问问, 他说什么都没做??? 好吧, 借出去谁都不能保证, 我就自己去查看错误原因把。 东摸摸西摸摸, 发现文件全 加了一个后缀 `.feenikss`; 百度关键字也没找到原因, Google 搜索也全是英文, 好吧, 放弃了;


<div style="width:100%; display:flex;">
    <img style="" src="/images/记一次WindowServer2012转CentOS7/error1.png" />
    <img style="" src="/images/记一次WindowServer2012转CentOS7/error2.png" />
    <img style="" src="/images/记一次WindowServer2012转CentOS7/error3.png" />
</div>

### 怀疑问题!!!

出问题, 我也想知道问题原因, 我找了公司同事问问, 有没有类似问题, 然后给我发来一个链接 [勒索病毒_百度百科](https://baike.baidu.com/item/%E5%8B%92%E7%B4%A2%E7%97%85%E6%AF%92/16623990?fr=aladdin) 
这他妈是啥? 我发现我的 `C` 盘文件全部修改了后缀 (除了没有权限的文件); 
我之前为了方便修改代码, 于是安装了 `Visual Studio Code` :(windows记事本你懂的);
后面 VScode 太卡了, 我就想安装 `Nodepad++`; 把 VScode 卸载了。(所以我强行甩锅Nodepad++);

### 有没有补救???

有没有补救, 专业点的肯定是可以, 问题就是我不专业, 同事推荐我在服务器上面安装 360 , 好吧安装了扫描了, 也修复不了文件了; 
原本想把自己 `www` 目录下的文件拷贝到本地, `nginx.conf` 配置拷贝下来, 文件都 ~~TMD~~ 加密了; 没得办法了呀!!!;
但是最重要的个人博客文章我都放在了Github, 减少了我的损失;

## 老子不要数据了, 从头来过!
> 我再也不用 `Windows server` 了; 服务器重新安装操作系统 CentOS7;

之后我会贴出来如何安装CentOS7;