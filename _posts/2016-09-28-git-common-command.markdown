---
layout: post
category: "Git"
title:  "GIT常用命令"
tags: [工具]
summary: ""
---
1、克隆git仓库中的一个分支

```
git clone -b <branch> <remote_repo>
```

2、删除远程

```
git remote rm origin
```

3、添加远程

```
git remote add origin https://github.com/sheshiji/sheshiji.github.io.git
```

4、推送到远程origin的master分支

```
git push -u origin master
```

5、添加文件夹下所有内容

```
git add .
```

6、提交修改

```
git commit -m "xxx"
```

7、Git创建基于master分支的develop分支

```
git checkout -b develop master
```

8、将develop分支发布到master分支的命令：

```
切换到master分支
git checkout master
对develop分支进行合并
git merge --no-ff develop
```

9、前面讲到版本库的两条主要分支：master和develop。前者用于正式发布，后者用于日常开发。其实，常设分支只需要这两条就够了，不需要其他了。但是，除了常设分支以外，还有一些临时性分支，用于应对一些特定目的的版本开发。临时性分支主要有三种：

- 功能（feature）分支
- 预发布（release）分支
- 修补bug（fixbug）分支

这三种分支都属于临时性需要，使用完以后，应该删除，使得代码库的常设分支始终只有master和develop。

创建一个功能分支：

```
git checkout -b feature-x develop

```

开发完成后，将功能分支合并到develop分支：

```
git checkout develop
git merge --no-ff feature-x
```

删除feature分支：

```
git branch -d feature-x
```

10、gitk看下版本树

11、列出文件的所有改动历史，注意，这里着眼于具体的一个文件，而不是git库

```
git log --pretty=oneline 文件名
```

12、列出git库当前改动历史

```
git diff
```

13、查看修改

```
git show可显示具体的某次的改动的修改
git show 356f6def9d3fb7f3b9032ff5aa4b9110d4cca87e
```

14、git clone出完整的版本库

```
"git checkout <SHA1 ID的前8位(如76bd774c)>"就可以把之前时间提交的版本checkout出来
如果要checkout仓库其他的分支, 先用”git branch -a“查看分支, 再用命令: "git checkout -b <new_branch_name> <remote_branch_name>" checkout出remote_branch_name这个分支出来。
```

15、Git删除远程仓库中误传的文件

```
git rm idea -r
git commit -am ‘remove idea’
git push -u origin master
```