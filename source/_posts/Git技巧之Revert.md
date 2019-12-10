---
title: Git技巧之Revert
date: 2019-12-10 13:36:02
tags: ['GIT']
---

使用**(Revert)** 重新操作

<!--more-->

### 假设场景
正在开发一个项目，然后产品告诉你前面一个正常的功能，现在因为一些原因准备下线了；

然后我把他下线了，继续开发其他功能；

但是突然又有一天他回来告诉你前面下线的功能，可能还需要继续使用一段时间；需要把他原封不动的上线；


### 这个场景我们就可以使用 **revert** 来完成

### 具体操作

1. 创建一个空的txt文件, 并提交

```bash
  git init
  touch a.txt
  git add .
  git commit -m 'init'
```

2. 添加第一条记录

a.txt：

```txt
  一、功能一

```

command：

```bash
  git commit -am 'feat:1'
```

3. 再添加一条记录
 
a.txt：

```txt
  一、功能一
  二、功能二
    1. 功能二-1小功能
    2. 功能二-2小功能
  
```

command：

```bash
  git commit -am 'feat:2'
```


4. 再添加一条记录
 
a.txt：

```txt
  一、功能一
  二、功能二
    1. 功能二-1小功能
    2. 功能二-2小功能
  三、功能三

```

command：

```bash
  git commit -am 'feat:3'
```

### 这个时候产品通知你 功能二的第二个小功能需要你下线；

1. ok我们操作一下

a.txt：

```txt
  一、功能一
  二、功能二
    1. 功能二-1小功能
    ---2. 功能二-2小功能--- // 下线这个功能
  三、功能三

```

command：

```bash
  git commit -am 'chore: 下线功能'
```

2. 继续开发

a.txt：

```txt
  一、功能一
  二、功能二
    1. 功能二-1小功能
    ---2. 功能二-2小功能--- // 下线这个功能
  三、功能三
  四、功能四

```

command：

```bash
  git commit -am 'feat:4'
```

### 这个时候产品告诉你需要你把前面下掉的功能原封不动的还原回来
我已经开发了第四个功能，现在又要回去把之前下线的功能还原回来

先看一下前面我们提交记录

```bash
  git log --onelie
```

```log
  e40b773 (HEAD -> master) feat:4
  cb3f831 chore: 下线功能
  d2f8575 feat:3
  a97e68d feat:2
  6557f01 feat:1
  4ffa71b init
```

我们需要把 `cb3f831` 记录还原回来；

这里假设下线的功能涉及到好几个文件，并且我们第四个功能也开发涉及到很多个文件；

这个时候我们如果使用 **reset** 回到 `cb3f831` 会把第四个功能的开发丢弃，回到当前，手动修改；

思考: 有没有更好办法去实现方法；

这个时候我们就可以用到前面提到的命令 **revert**

```bash 
  git revert -n 9a9b92d
```

这样就把下线前的代码暂存到git中了，打开 **a.txt** 查看文件

```txt
  一、功能一
  二、功能二
      1. 功能二-1小功能
      2. 功能二-2小功能
  三、功能三
  四、功能四

```

我们发现之前下线的功能二的第2个小功能已经回来了；并且已经添加到暂存中了；
(这里可能会有冲突，解决一下冲突就好了)

然后我们就可以提交`commit`了

comcommand：
``` bash
  git commit -m 'feat:上线之前下线的功能'
```

ok,到这里恭喜你又学会一个技巧了；

```log
  43a40b4 (HEAD -> master) feat:上线之前下线的功能
  e40b773 feat:4
  cb3f831 chore: 下线功能
  d2f8575 feat:3
  a97e68d feat:2
  6557f01 feat:1
  4ffa71b init
```

### 查看结果

![查看结果](/images/Git技巧之Revert/demo.gif)

### 总结
  + 这里我只用了一个文件 **a.txt** 来代替开发环境，只是做了一个模拟场景；工作项目中一个功能可能就会涉及到许多个文件，修改起来也比较麻烦；
  + 在执行一个自己不是很熟悉的命令时候，切记要新建一个分支来测试，避免造成不可挽回的损失；
### 参考文档
  + [Git恢复之前版本的两种方法reset、revert（图文详解）](https://blog.csdn.net/yxlshk/article/details/79944535)
  + [https://git-scm.com/docs/git-revert](https://git-scm.com/docs/git-revert)