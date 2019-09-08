---
title: Git 修改历史Commit作者和邮箱
date: 2019-09-02 16:36:02
tags: ['GIT']
---

使用**变基(rebase)** 修改已提交**Commit**的 **作者**(`author`) 和 **邮箱**(`email`);

<!--more-->

在朋友的电脑上拉取了自己的仓库，然后修改了代码，提交Commit就会使用计算机全局的配置；

这里需要把作者和邮箱添加到当前项目中；

## 如何添加作者和邮箱

> 当前仓库中添加

```git
  git config user.name “userName”
  git config user.email "test@xx.com"
```
> 全局中添加

```git
  git config --global user.name “userName”
  git config --global user.email "test@xx.com"
```

查看配置文件
```git
  git config --list
```

<br/>
<br/>

但是，我们可能已经添加了多个`commit`，并且推送到远程分支了！怎么办

![已经提交错误Commit示例](/images/Git修改历史Commit作者和邮箱/error_demo.jpg)


### 使用REBASE变基

>  **rebase** 到你要修改的那一条 `commit`

```git
    git rebase -i <commit_hashcode>
```

 > 这个时候会出现一个文本编辑界面内容大概如下, 把你要改的 `commit` 前面的 **pick** 替换成 **edit**，然后输入 **wq** 退出

```git
pick <hashcode1>  <commit_message1>
pick <hashcode2>  <commit_message2>
pick <hashcode3>  <commit_message3>
pick <hashcode4>  <commit_message4>
```

> 使用 `--amend` 修改 **author**

```git
 git commit --amend --author='userName <xxxx@xx.com>'
```
>输入 `git rebase --continue` 结束修改

```git
  git rebase --continue
```

<br/>

### 具体操作

#### 1. 查看提交记录
<span style="color:red;font-size:12px">这里使用的全局命令<span style="color:black"> git lg</span> 在文章中提及过：[常用Git技巧记录](https://blog.wxhboy.cn/2019/08/07/%E5%B8%B8%E7%94%A8Git%E6%8A%80%E5%B7%A7%E8%AE%B0%E5%BD%95/)</span>

![查看错误的提交记录](/images/Git修改历史Commit作者和邮箱/error_screenshot.jpg)

#### 2. **rebase** 变基操作

```git
  # git rebase -i <要开始编辑的commit hash 上一个> # 注释
  git rebase -i  97eea04
```
把需要修改的记录前面的 **pick** 替换成 **edit**，然后输入 **wq** 退出；

![pick替换成 edit](/images/Git修改历史Commit作者和邮箱/edit_error.jpg)

![替换结果](/images/Git修改历史Commit作者和邮箱/guocheng.jpg)


#### 3.  修改 **autho** r和 **email**
+ 修改第一条

```git
   git commit --amend --author='userName <xxxx@xx.com>'
```

  + 完成第一条修改

```git
   git rebase --continue
```
重复上面的操作，得到下图：

![修改完成结果](/images/Git修改历史Commit作者和邮箱/rebase.jpg)


#### 4.  检查修改

查看修改结果是否满足自己的需求

```git
  git log
```

![检查修改结果](/images/Git修改历史Commit作者和邮箱/log.jpg)

### 推送

如果你之前错误用户名和邮箱已经提交了，你是无法提交的；
因为远端的库和本地库历史不一致，那么你需要使用`-f`强制推送；

```git
git push origin master -f
```

其实在推送时，尽量避免 `git push -f`的操作，或者说`git push -f`是一个需要谨慎的操作;
它是将本地历史覆盖到远端仓库的行为。

### 参考文档
+ [修改git commit信息中的author](https://blog.csdn.net/liang890806/article/details/46813039)
+ [git配置用户名邮箱，全局配置/单仓库配置](https://www.cnblogs.com/xxoome/p/9183515.html)
