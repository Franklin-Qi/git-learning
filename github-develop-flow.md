# github 开发分支流程

Table of Contents
=================

   * [github 开发分支流程](#github-开发分支流程)
   * [Table of Contents](#table-of-contents)
      * [一、远端及本地仓库准备](#一远端及本地仓库准备)
      * [二、feature/bug issue提出。](#二featurebug-issue提出)
      * [三、基于最新的master创建研发分支](#三基于最新的master创建研发分支)
      * [四、代码提交](#四代码提交)
      * [五、创建pr](#五创建pr)
      * [六、代码评审，pr合并](#六代码评审pr合并)
      * [七、相关issue状态更新](#七相关issue状态更新)
      * [八、参考资料](#八参考资料)

## 一、远端及本地仓库准备

* fork上游仓库到个人名下，
* clone个人名下的仓库到本地
* 为自己的本地仓库配置远端仓库源
   * 默认的origin为个人名下仓库的地址
   * 添加上游仓库地址
   ```
   eg: git remote add <起个名做标示> <上游仓库的地址>
   ```

## 二、feature/bug issue提出。

issue的三个常用的属性
* label：标签，可以用来设置issue的类型、重要程度、目前的处理状态。[issue 标签的设置规范](https://github.com/ukui/community/blob/master/zh_CN/issue_manage.md)
* Milestone：里程碑，可以用来设置研发项目的不同阶段，明确任务的优先级与截止日期。
* Assignee：负责人，明确issue解决者，相关issue的变动，负责人会收到邮件通知。

对于团队而言，issue可以看作是一个轻量级的协作系统
* 产品同学用来做产品需求的反馈，讨论。
* 项目同学通过拆分项目任务到issue，设置里程碑，跟踪issue的进展等完成项目的过程管理。
* 研发同学根据issue，明确研发任务，将自己的代码提交，与issue做关联，明确每个提交的意义，方便追溯以及版本回退。
* 测试同学用来做测试问题的反馈，与研发者、需求方进行沟通，记录相关讨论与解决的过程。

在github仓库的setting标签中，为issue设置模版。常用的两种issue：feature类型和bug类型
* feature类型的模版

**功能需求** 
描述需求功能点

**设计方案** 
描述该需求的设计方案。

**补充说明** 
需要补充说明的内容。


* bug类型的模版

**ISO版本:** ISO 的版本号

**测试环境:** 描述测试的基础环境，cpu架构、机型等

**测试步骤:** 复现问题的步骤
1. xxx
2. xxx
3. xxxx

**预期结果:** 测试步骤正常应该出现的结果.

**实际结果:** 描述测试过程中出现的问题。

**重现概率:** 必现 or 偶发

**截图：**
辅助问题说明的截图

**补充说明:** 需要补充说明的内容.

## 三、基于最新的master创建研发分支

* 更新本地master分支，与上游的master分支做同步。
```
git fetch upstream master
git rebase upstream/master # rebase 可避免再次增加一个merge记录

或者
git fetch upstream master
git merge --no-ff upstream/master # 默认快进ff，改为--no-ff比较好

或者

# Tips: 当上游分支仓库设置为pr提交者的地址，那么也可用于审核者在本地的分支，同步pr提交者的修改，然后在本地编译看差别
# git pull --rebase pr master 
git pull --rebase upstream master 


```

* 基于最新的master，创建研发分支。
```
# 分支名参考命名规则： 
# 从类型开始，例如feature或bug，然后是供参考的问题编号，以及描述问题的文本。
# feature/945-state-diagrams
# bug/12425-add-search-function

git checkout -b <研发分支名>
```
* 在研发分支，进行编码。

## 四、代码提交

* 查看分支状态，查看文件变更，确认提交的内容。（将不需要提交的文件类型，配置到.gitignore文件，减少diff出不必要的文件信息）
* 代码commit之前，应该完成静态代码检测，可通过配置pre-commit脚本(位于本地仓库.git/hooks目录下)完成。
* 规范的commit message是版本维护以及代码合并的基础。
```
commit msg 组成
<type>: <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
1. type：指定当前commit的类型
   * feat: 新功能
   * fix: 修复问题
   * docs: 修改文档
   * style: 修改代码格式，不影响代码逻辑
   * refactor: 重构代码，理论上不影响现有功能
   * perf: 提升性能
   * test: 增加修改测试用例
   * chore: 修改工具相关（包括但不限于文档、代码生成等）
   * deps: 升级依赖
2. subject：简述本次提交做了什么。
3. body：补充说明本地提交的内容，没有则省略
4. footer：
   * 注明本次修改的约束条件，没有则省略。
   * issue ref：对应解决的issue的连接
   * doc ref：涉及到的文档连接
```
* 本地代码提交到远端。此处，直接提交到本人名下fork的仓库。后续通过pr合并到上游仓库。
```
git pull --rebase origin <branch_name> // 如果有其他小伙伴同步在该分支上做开发，合并之前先更新本研发分支
git pull --rebase upstream master // 研发分支与上游master分支同步。自检一下有没有冲突。
git push origin <branch_name>
```

## 五、创建pr

在个人名下仓库的github页面，新建pull request：
* 检查：目标仓库为上游仓库，分支为master分支；
* 检查：即将被合入的仓库为名下fork的仓库，分支为开发分支；
* 关于pr message，模版同commit message
```
只有一个commit的pr，会拿当前commit的message作为pr的message。
其中，commit-message的header就是pr的title，commit-message的body和footer，会直接作为pr message的主体。

当pr中commit的个数多于一个，pr的title会是当前开发分支的分支名，message的内容不会自动填充。此时需要手写。
```
* 创建pr时会自动检测当前pr包含的提交内容是否可有正常合并。如有冲突，需要基于研发分支处理冲突，直接提交即可，pr会自动更新。
* 指定pr的Reviewers，来通知干系人，进行代码评审。

## 六、代码评审，pr合并

* pr干系人，查看pr提交的内容，就代码风格、逻辑等进行同级评审。通过这个方式进一步降低出bug的概率，同时也是一个相互讨论和学习的过程。
* 评审出的问题，直接在研发分支，修改提交，pr内容会自动更新。
* 最终pr同级评审通过后，合并到master分支。

```
经过测试修改和代码评审，最后pr通过了，在合并之前发现commit list中含有很多临时提交，这些提交对版本信息无意义，建议将其，精简提交信息。
```

## 七、相关issue状态更新

pr合并后，对应issue更改为状态对应的label。比如从status/confirmed，改为status/resloved。

## 八、参考资料

+ [github 分支开发流程](https://docs.github.com/zh/get-started/quickstart/github-flow)
+ [mermaid 开发贡献流程](https://mermaid.js.org/community/development.html)


