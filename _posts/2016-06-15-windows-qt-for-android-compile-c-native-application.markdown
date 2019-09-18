---
layout: post
category: "Qt"
title:  "Qt Creator编译android系统下原生的C/C++可执行文件"
tags: [Android]
summary: ""
---

①构建Android平台的纯C/C++项目（注：此步骤后会自动生成Makefile文件）；

②修改Makefile文件：

- 将LFLAGS值中去掉-shared和-Wl,-soname,xxx.so（注：xxx为项目名称）；

- 将QMAKE_TARGET和TARGET后面的名称改为想要的可执行文件名；

- 将LIBS值中去掉-lgnustl_shared；

③执行mingw32-make；