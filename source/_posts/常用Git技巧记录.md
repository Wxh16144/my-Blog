---
title: 常用Git技巧记录
date: 2019-08-07 11:35:08
tags: ['GIT']
---

本文总结平常工作中可能会用的到的**GIT**技巧;

<!--more-->

## 快捷查看本地log

在**Git**中我们经常需要擦看`log`; 都使用:
```bash
  git log
  # or
  git log --pretty --oneline
```
再全局添加命令
```bash
  git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset - %C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
```
之后就可以使用 查看简洁的`log`了
```bash
git lg
```


## 情景一
> 有一个 `dev` 分支, 是我当前正在开发的分支;
  有一个 `test` 分支, 是提交测试的分支;
  我正在进行 `dev` 分支某个功能开发;
  突然测试组告诉我有个bug马上修改;
  **但是我当前分支的功能还没有完成**;

创建场景:
<video src="/images/常用Git技巧记录/init.mp4" poster="/images/常用Git技巧记录/loading.gif" controls><video>

> 创建场景步骤：直接复制到git就好了

```bash
  mkdir test && cd test # 创建目录并切换
  git init # 初始化
  touch index.js # 添加文件
  # vim index.js # 任意添加/修改内容
  git add . # 添加暂存区
  git commit -m 'm:1' # 提交版本
  git checkout -b dev # 创建并切换到分支
  # vim index.js # 任意添加/修改内容
  git add . # 添加暂存区
  git commit -m 'd:1' # 提交版本
  git checkout -b test # 创建并切换到分支
  # vim index.js # 任意添加/修改内容
  git add . # 添加暂存区
  git commit -m 't:1' # 提交版本
  git branch # 查看本地所有分支

```
当你看到本地三个分支的时候说明已经成功了(`master`和`dev`和`test`)
```bash
$ git branch
  dev
  master
* test
```

### 使用分支概念
> 注意当前我的工作还没有完成, 要去修改`test`分支的bug;

平常思路:
1. 提交本地更改到暂存区
2. 添加本地`commit`
3. 切换到`test`分支修改bug
4. 修改完成, 提交`test`分支修改
5. 切回来`dev`合并`test`, 然后继续开发

### 方法一

> 先提交本地未完成的代码, 切回去修改; 然后切回来继续开发

**(当前在`test`分支下)**

修改**index.js**内容
```javascript
  var branch = 'test';
```
提交修改
```bash
  git commit -am 't:2'
```
切换到 `dev` 分支
```bash
  git checkout dev
```
**这一步开始，表明你正在开发`dev`分支**
添加**index.js**内容
```javascript
  var dev;
```
**接到通知`test`分支bug了,需要修改;**
收起手里现在的活去修改bug
提交`dev`分支修改
```bash
  git commit -am 'd:2'
```
切换到 `test` 分支
```bash
  git checkout test
```
修改**index.js**内容
```javascript
  var branch = 'test-edit-fix-bug';
```
提交修改
```bash
  git commit -am 'fix:test-bug'
```
切换到 `dev` 分支
```bash
  git checkout dev
```
合并`test`分支修改到本地
```bash
git merge test
```
解决冲突
```bash
<<<<<<< HEAD
var dev;
=======
var branch = 'test-edit-fix-bug';
>>>>>>> test
```
```bash
git commit -am 'merge:test'
```
解决完冲突到这里就可以继续在`dev`上愉快的开发了;

到这里我们使用分支基本操作就走了一遍

#### 思考一下
  假设`test`分支又出现**bug**了.
  而此时你本地`dev`这一个功能还没开发好.
  你又要提交本地`dev`的改动, 去修改`test`分支的**bug**.
  然后切换回来, 合并`test`修改, 解决冲突, 继续开发**dev**;

这里重复了一下上面的操作，我给直接省略了;
查看一下本地log
```bash
$ git lg
* 4452de2 -  (HEAD -> dev, test) fix: test-bug-2 (39 seconds ago) <WuXiaohong>* b79ad4d -  d:3 (3 minutes ago) <WuXiaohong>
*   ae1ad9d -  merge:test (17 minutes ago) <WuXiaohong>
|\
| * 2fe424d -  fix:test-bug (20 minutes ago) <WuXiaohong>
| * 1a7abe6 -  t:2 (21 minutes ago) <WuXiaohong>
* | 721e6af -  d:2 (21 minutes ago) <WuXiaohong>
|/
* 1691485 -  (master) m:1 (37 minutes ago) <WuXiaohong>
```

可以看到我们提交了很多日志, 但是`dev`功能还没开发好, 这样就带来一个不太友好的地方, 给其他开发者阅读`commit`难免会有一些痛苦;

### 方法二

#### [git stash (储藏)](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%EF%BC%88Stashing%EF%BC%89)

> **储藏**可以获取你工作目录的中间状态——也就是你修改过的被追踪的文件和暂存的变更——并将它保存到一个未完结变更的堆栈中，随时可以重新应用。
他可以不提交本地修改, 将他**储藏**， 然后去修改**bug**; 最后再切换回来继续开发;

#### 具体操作：
**(当前在`test`分支下)**

追加**index.js**内容
```javascript
  var dev3;
```
**接到通知`test`出bug了**
**使用 `git stash`命令 储藏刚刚的追加**
```bash
  git stash
```
你会发现刚刚添加的内容被隐藏了; 这个时候我们就可以切换到`test`修改**bug**;
切换到`test`分支
```bash
  git checkout test
```
修改**bug**, 添加**index.js**内容
```javascript
  var branch3 = 'test-edit-fix-bug-3';
```
提交修改
```bash
  git commit -am 'fix: test-bug-3'
```
切换到`dev`继续开发
```bash
  git checkout dev
```
**这个时候回来发现`dev`之前的东西都没有**
我们可以使用命令查看本地隐藏列表
```bash
  git stash list
  # stash@{0}: WIP on dev: 4452de2 fix: test-bug-2
```
可以看到我们本地有一条储藏的工作;
切回来继续`dev`分支上的工作:
```bash
  git stash apply
```
如果你想应用更早的储藏，你可以通过名字指定它，像这样：git stash apply stash@{2}。如果你不指明，Git 默认使用最近的储藏并尝试应用它;

#### dev开发好了
假设你已经开发好了`dev`, 你就可以提交并合并`test`分支
```bash
  git commit -am 'd:finish' # 开发完成
  git merge test
```
解决完冲突; 查看`log`;
```bash
$ git lg
*   eb6a5a5 -  (HEAD -> dev) merge: test-1 (11 seconds ago) <WuXiaohong>
|\
| * 4ede3c2 -  (test) fix: test-bug-3 (18 minutes ago) <WuXiaohong>
* | 67efd39 -  d:finish (8 minutes ago) <WuXiaohong>
|/
* 4452de2 -  fix: test-bug-2 (35 minutes ago) <WuXiaohong>
* b79ad4d -  d:3 (37 minutes ago) <WuXiaohong>
*   ae1ad9d -  merge:test (51 minutes ago) <WuXiaohong>
|\
| * 2fe424d -  fix:test-bug (55 minutes ago) <WuXiaohong>
| * 1a7abe6 -  t:2 (56 minutes ago) <WuXiaohong>
* | 721e6af -  d:2 (55 minutes ago) <WuXiaohong>
|/
* 1691485 -  (master) m:1 (71 minutes ago) <WuXiaohong>
```



