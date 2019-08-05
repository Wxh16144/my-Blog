---
title: npm --dev --save-dev 区别
date: 2018-08-04 13:36:39
tags: ['node']
---

一目了然搞懂 `--dev` `--save` `-D` `-S` 区别;

<!-- more -->


> 在使用 `npm install` 安装 `npm` 包时，有四种命令参数可以把它们的信息写入 `package.json` 文件;

### 添加到 dependencies (生产环境)
> `devDependencies` 下列出的模块，是我们开发时用的，不会被部署到生产环境; 比如`babel-loader`, `css-loader`

```bash
  npm install jquery --save
```
or
```bash
  npm install jquery -S
```

---

### 添加到 devDependencies (开发环境)

> `dependencies` 下的模块，则是我们生产环境中需要的依赖。比如 `swiper`, `jquery`

```bash
  npm install jquery --save-dev
```
or
```bash
  npm install jquery -D
```

## 总结:
`--save` 和 `-S` 会把依赖包名称添加到 `package.json` 的 `dependencies` 下;

而`--save-dev` 和 `-D` 则会添加到  `package.json` 的 `devDependencies` 下;


