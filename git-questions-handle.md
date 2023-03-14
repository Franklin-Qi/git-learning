## 记录一次git 操作冲突的解决过程

**背景：本地不知情的情况下，创建了一个和远端同名的branch**

push到远端时，报错信息如下：

```
liuxiran:settings liuxiran$ git branch
* issue_2_add_monitor
  master
liuxiran:settings liuxiran$ git push origin issue_2_add_monitor
To github.com:liuxiran/ukui-settings-daemon.git
 ! [rejected]        issue_2_add_monitor -> issue_2_add_monitor (non-fast-forward)
error: failed to push some refs to 'git@github.com:liuxiran/ukui-settings-daemon.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
liuxiran:settings liuxiran$ 
```
**原因分析：**
本质上来说是本地issue_2_add_monitor分支与origin/issue_2_add_monitor分支不一致导致了上述问题的发生。

**解决方案：**
1. 如果目前要提交的这个分支完成的内容与已有分支完成内容不同，则重命名该分支后提交
```
git branch -m issue_2_add_monitor issue_2_add_monitor_new
git push origin issue_2_add_monitor_new
```
2. 如果目前要提交的这个分支与远端分支是协作关系，则需要同步远端分支后再提交。
```
git pull --rebase origin issue_2_add_monitor
rebase 的过程中，遇到冲突的，按照git的错误提示，解决文件内的冲突
git add <解决冲突的文件> // 标记冲突
git rebase --continue // 继续rebase的过程
git push origin issue_2_add_monitor

```
**细心的小伙伴可能已经发现，周五演示的过程中，按照方案2做过最基本的问题处理，但是当时没做成功，为啥呢？**
原因是：我本地的issue_2_add_monitor分支，开发之前，基于upstream/master做过rebase，所以本地的根基是commit
```
commit b03c3181fdaa3a6f7991aa5fdaf7242058b19887 (upstream/master)
Merge: 5376a56 a60f9e3
Author: tong2357 <18001212351@163.com>
Date:   Fri Mar 20 20:26:32 2020 +0800

    Merge pull request #20 from dingjingmaster/master
    
    delete binary file
```
而远端对应分支根基是commit
```
commit 4066c278e0f8c1b33e0b5401025ec3aad6af4538
Merge: 7391b21 a8e2e77
Author: liutong <18001212351@163.com>
Date:   Wed Mar 18 17:18:21 2020 +0800

    Merge branch 'common'
```
对比可以发现，我本地分支的根基比远程分支的根基要更新一些，所以当我在本地issue_2_add_monitor基于origin/issue_2_add_monitor做rebase的时候，大量的冲突出现了🤦‍♀️，最关键的是，这些冲突源于基础代码，而非我引入的新代码。
怎么办呢？
```
git checkout -b 'issue_2_add_monitor_for_update' origin/issue_2_add_monitor // 把远端同名分支下载到本地
git pull --rebase upstream master // 跟上游maste分支同步
git push origin issue_2_add_monitor_new:issue_2_add_monitor -f // 确认无误，就需要强行更新远端的
git checkout issue_2_add_monitor
git pull --rebase origin issue_2_add_monitor
git push origin issue_2_add_monitor
```

**后记**
1. 问题解决的过程，我的git pull 后面加了rebase的参数，为啥呢？这里就涉及到到底是git merge还是git rebase呢？于我个人而言，最主要是因为师傅当年是这么教我的🤦‍♀️，个人感悟就是git rebase解决冲突更容易，并且不会产生冗余的commit message。如果要深究：[请参阅该文章](https://www.cnblogs.com/FraserYu/p/11192840.html)
2. git的有点就是他的多分支管理，同时多分支往往也是引入问题的源头。为了减少提交时的代码冲突，一个好的习惯挺重要的。比如：
   * 创建分支前先看看已经有哪些分支了？`git branch -r // 列出本地的，已经配置的各个远端的都有哪些分支`
   * 新建的研发分支，开发之前，先与origin/<研发分支>同步，再于upstream/master分支同步。至少你开工前，代码是干净的。
   * 提交代码之前，重复两步同步过程，有冲突及时处理，这样提交pr时，是不存在冲突的。
   