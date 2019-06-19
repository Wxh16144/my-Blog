---
title: 'node-js获取本机ip'
date: 2019-06-17 17:18:25
tags: ['nodejs','javascript']
categories: javascript
summary: 使用node.js的os模块获取本地ip地址
---

使用node.js获取本地IPv4地址
<!--more-->

#### 项目需求

在公司开发项目过程中, 使用`Vue`项目配置本地host的时候;
每个人从`SVN`仓库pull下来的代码, 安装依赖后, 无法正常运行, 需要修改ip为自己本地IP?(不知何用意);
索性每次都要改, 每个开发者都麻烦, 所以自己写了一个方法来获取IP地址;

#### 解决方法

```javascript
const os = require('os');
const getLocalIP = () => {
  //所有的网卡
  const ifaces = os.networkInterfaces();
  let network = [];
  //移除loopback,没多大意义
  Reflect.ownKeys(ifaces).forEach(key => {
    if (!/loopback/ig.test(key)) {
      network = [...network, ...ifaces[key]];
    };
  });
  return network.reduce((arr, { address, family }) => {
    const ip = (/^IPv4$/ig.test(family) ? [address] : []);
    return [...arr, ...ip];
  }, []);
};

// exports.getLocalIP = getLocalIP
const a = getLocalIP()
console.log(a);
```

#### 参考文档

1. [Nodejs获取本机地址](https://blog.csdn.net/spy19881201/article/details/13394933)
2. [Nodejs中os.networkInterfaces()](http://nodejs.cn/api/os.html#os_os_networkinterfaces)