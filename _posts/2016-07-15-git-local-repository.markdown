---
layout: post
category: "Git"
title:  "创建本地安全的Git使用环境"
tags: [工具]
summary: ""
---
**生成本地一个纯净的git仓库**

```
cd ~/project/gittest
git init 
cd ..
git clone --bare gittest gittest.git
cp gittest.git ~/repository/ -r
rm -f -r gittest
```

这样，一个纯净的git仓库就已经在repository中创建。

**clone该项目**

```
cd ~/project
git clone file:///home/zzc/repository/gittest.git
```

**提交项目**

```
git remote add origin file:///home/zzc/repository/gittest.git
git commit
git push origin master 
```