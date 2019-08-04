---
title: webhooks+node+express实现自动部署个人博客
date: 2019-08-04 13:36:39
tags: ['node','javascript','nginx']
---

本地更新一篇文章, 使用`git push` 推送到 **GitHub**, 实现自动部署到服务器;

<!--more-->
### 需求分析

在 本地 [**hexo**](https://hexo.io/zh-cn/) 创建并写好一篇文章后, 使用`git push` 把代码推送到远程 [**GitHub**](https://github.com/Wxh16144/my-Blog) 仓库;

我只做这一步剩下的交给机器去完成: 

  1. GitHub 发送一条 POST 请求;
  2. 个人服务器接收到POST请求并作出操作
  3. 拉取远程仓库更新到服务器本地
  4. 服务器本地用hexo打包成静态文件资源
  5. 更新自动部署的README文档
  6. 推送更新


### 准备工作
1. 一台`CentOS7`服务器; (需要基础环境`nginx`, `node`);
2. 一个域名,可以做一个二级域名专门用于这个例子, **记得创建域名解析** (我使用了自己的二级域名 http://api.wxhboy.cn);

### 设置GitHub
在自己的个人博客仓库中设置`Webhooks`;
![设置Webhooks](/images/webhooks+node+express实现自动部署个人博客/temp.gif)

### 配置Nginx
配置nginx, 转发请求; 
参考文档:
  + [nginx location proxy_pass 后面的url 加与不加/的区别](https://blog.csdn.net/a12345678n/article/details/80747586);
  + [nginx proxy_set_header设置，自定义header](https://www.cnblogs.com/liuxia912/p/10943970.html);
```nginx
server {
    listen 80 ; #监听端口
    server_name 'api.wxhboy.cn';  #域名
    index index.html index.htm index.php;
    root /home/api; #站点目录
    error_page  404              /404.html;
    location = /404.html {
        root /usr/local/nginx/html;
    }
    # hexo自动部署请求接口
    location /hexo/ {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:3000/; # 本地端口
    }
}
```
配置个人博客地址
设置**root**为你hexo打包的静态资源目录就好了
```nginx
server {
    listen 80 ; #监听端口
    server_name 'blog.wxhboy.cn';  #域名
    index index.html index.htm index.php;
    root /home/deploy-myblog/myblog/public; #站点目录
    error_page  404              /404.html;
    location = /404.html {
          root /usr/local/nginx/html;
    }
}
```
#### 用到几个**nginx**命令
检查配置是否有误
```bash
nginx -t
```
重启
```bash
nginx -s reload
```

到这里准备工作就已经基本完成

### 开始构建Node.js代码
目录结构

+ logs <span style="font-size:.8em;color:green">  存放日志<span>
+ myblog <span style="font-size:.8em;color:green">  个人博客目录<span>
+ app.js <span style="font-size:.8em;color:green">  入口文件<span>
+ utils.js <span style="font-size:.8em;color:green">  工具文件<span>

安装依赖
```bash
  npm install body-parser express shelljs simple-git -D
```

引入 **exporess** 和 **body-parser** 搭建服务

```javascript
// 系统模块
const http = require("http");
//引入express模块
const app = require('express')();
// 获取请求信息
const bodyParser = require('body-parser');
// 使用
app.use(bodyParser.urlencoded({extended: false}));
app.use(bodyParser.json());

// 响应头
app.use("*", (request, response, next) => {
    response.writeHead(200, {
        "Content-Type": "application/json;charset=utf-8"
    });
    next();
});

// GET请求
app.get('/', (req, res) => {
    res.end(JSON.stringify({status: 200, msg: '请求成功'}))
});

//端口号
const PORT = 3000;
//开启服务
const server = http.createServer(app);
server.listen(PORT, err => {
    console.log(err ? 'Server Error' : `http://127.0.0.1:${PORT}`); //开启服务
});
```
#### 运行node:
使用 **nodemon** 热重启 [nodemon npm 地址](https://www.npmjs.com/package/nodemon);
```bash
  nodemon app.js
```
#### 打开浏览器[http://127.0.0.1:3000](http://127.0.0.1:3000); 出现下面效果表示运行成功:
```json
{
    "status": 200,
    "msg": "请求成功"
}
```

### 监听POST请求

```javascript
//获取POST请求
app.post('/', function (req, res) {
    try {
        // 请求参数
        const {query, body: params} = req;
        // 组合
        const obj = {query, params};
        // 个人博客拉取更新
        myblogGIT.pull('origin', 'master');
        // hexo 生成静态文件
        hexo_generator();

        // 返回数据
        res.end(JSON.stringify({
            status: 200,
            msg: '完成'
        }));
    } catch (err) {
        res.end(JSON.stringify({status: 500, msg: 'post服务器繁忙', err: JSON.stringify(err)}))
    }
});
```

#### 拉取远程代码到本地
使用`simple-git` 拉取代码到本地
引入包, 并执行; 
**注意带入参数是当前个人博客的地址** `path.join(__dirname, 'myblog')`
```javascript
  // git // 选择目录为myblog // path.join(__dirname, 'myblog')
  const myblogGIT = require('simple-git')(path.join(__dirname, 'myblog'));
  // 个人博客拉取更新
  myblogGIT.pull('origin', 'master');
```

#### 实现 hexo_generator
使用hexo g 命令生成静态文件[hexo 生成文件 文档](https://hexo.io/zh-cn/docs/generating);
需要引入依赖 **shell-js** [shell-js npm 地址](https://www.npmjs.com/package/shell-js);
```javascript
const shell = require('shelljs');
const hexo_generator = ()=>{
    // 找到个人博客目录
    const directory = path.join(__dirname,'./myblog');
    // 切换到目录
    shell.cd(directory);
    // 执行hexo命令
    const EXEC_RES = shell.exec('hexo g');
    if (EXEC_RES.code === 0) {
        console.log('hexo 打包成功');
    } else {
        console.log('hexo 打包失败');
    }
}
```
到这里我们的服务器本地的博客就更新了, 通过博客地址就可以访问了;

<br>
<br>
## 额外操作(可有可无)
> 大家可以参考下面的文档, 实现把打包后的静态资源文件推送到GitHub仓库;

### 保存日志
把每一次的请求写入到本地 **logs/**目录下
```javascript
const fs = require('fs');
const path = require('path');
const fileName = `${params.after || myCreateDAte()}.json`; // log文件名称
fs.writeFileSync(path.resolve(__dirname, 'logs/', fileName), JSON.stringify(obj));

/**
 * 根据时间生年-月-日-时-分-秒
 * 2019-10-15-12-05-56
 * @returns {string}
 */
const myCreateDAte = () => {
    const date = new Date();
    const year = date.getFullYear();
    const month = date.getMonth() + 1;
    const day = date.getDate();
    const hour = date.getHours();
    const minute = date.getMinutes();
    const second = date.getSeconds();
    return `${year}-${month}-${day}-${hour}-${minute}-${second}`;
};
```

### 把更新详情渲染到README.md 文档;

1. 在根目录下新建一个`template.md`文件, 作为模板;
2. 需要引入依赖 **underscore** [shell-js npm 地址](https://www.npmjs.com/package/underscore);
3. 把渲染好的`README.md`的文件推送到出库;

创建`template.md`模板
```html
### 个人博客自动更新到服务器

#### 更新内容

<% _.each(obj, function(item){ %>

---
+ 更新说明: [<%= item.message %>](<%= item.url %>)
+ 更新时间: <%= item.timestamp %>
+ 更新内容:
    <% if(item.added.length !== 0 ){ %>
     - 添加
        <% _.each(item.added, function(e){ %>+ [<%= e.title %>](<%= e.url %>)
        <% }) %><% } %><% if(item.removed.length !== 0 ){ %>
     - 删除    
         <% _.each(item.removed, function(e){ %>+ [<%= e.title %>](<%= e.url %>)
         <% }) %><% } %><% if(item.modified.length !== 0 ){ %>
     - 修改    
         <% _.each(item.modified, function(e){ %>+ [<%= e.title %>](<%= e.url %>)
         <% }) %><% } %>

+ 查看差异: [<%= item.compare %>](<%= item.compare %>)
+ 作者: <%= item.pusher.name %><<%= item.pusher.email %>>
---
 <% }) %>
 
### other
> 博客地址仓库:[https://github.com/Wxh16144/my-Blog](https://github.com/Wxh16144/my-Blog/)

> 博客地址:[http://blog.wxhboy.cn](http://blog.wxhboy.cn)
```

实现渲染`README.md`文档;
```javascript
const _ = require('underscore');
/**
 * 渲染README文档
 * @param params
 */
const handleREADME = (params = {}, CACHE) => {
    const { commits, repository, compare, pusher} = params;
    // 引入模板
    let content = fs.readFileSync(path.join(__dirname, './template.md'));
    let compiled = _.template(content.toString());
    //循环记录数量
    content = compiled(commits.map(item => {
            // 处理commit数据
            const { message = '', timestamp = '', url = '', modified = [], removed = [], added = [] } = item;
            return {
                message, url, pusher, compare, timestamp,
                added: createModified(added, repository.url + '/blob/master'),
                removed: createModified(removed, repository.url + '/blob/master'),
                modified: createModified(modified, repository.url + '/blob/master'),
            }
        });
    );
    // 写入README.md到本地
    fs.writeFileSync(path.join(__dirname, './README.md'), content, 'utf-8');
    // 如果有记录数
    if (commits.length) {
        // 提交到远程仓库
        handlerCommit(commits.length);
    };
};
// 把文件获取出来
const createModified = (modified = [], rootPath = '') => {
    if (modified.length) {
        return modified.map(file => {
            return {
                title: getFileName(file),
                url: rootPath + '/' + file
            }
        })
    } else {
        return modified
    }
};
```
### 实现提交功能

```javascript
const handlerCommit = async len => {
    const P = new Promise( resolve => {
        const GIT = require('simple-git')(); // git
        GIT.addConfig('user.name', 'Wuxiaohong(server)')
            .addConfig('user.email', 'wxh1220@gmail.com')
            .add('./README.md')
            .commit(`🔨自动更新：博客更新${len}条记录`)
            .push(['-u', 'origin', 'master'], resolve);
    });
    const res = await P;
};
```

查看效果
![预览效果图](/images/webhooks+node+express实现自动部署个人博客/preview.png)


完成;

在测试代码的时候, 大家不妨使用[花生壳内网穿透](https://www.oray.com/);
把**GitHub**的`webhooks`添加花生壳的域名;

本文全都代码我放到了 **GitHub仓库:**[https://github.com/Wxh16144/deploy-myblog](https://github.com/Wxh16144/deploy-myblog);


