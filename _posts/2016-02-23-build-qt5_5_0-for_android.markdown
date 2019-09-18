---
layout: post
category: "Qt"
title:  "Windows下源码编译Qt-Android"
tags: [Android]
summary: ""
---
**参考文章：**[Building qt-android on windows](https://wiki.qt.io/Building_qt-android_on_windows)

```
set ANDROID_API_VERSION=android-19
set ANDROID_SDK_ROOT=D:\android\sdk
set ANDROID_TARGET_ARCH=armeabi-v7a
set ANDROID_BUILD_TOOLS_REVISION=23.0.2
set ANDROID_NDK_PATH=D:\android\android-ndk-r10e
set ANDROID_NDK_ROOT=D:\android\android-ndk-r10e
set ANDROID_TOOLCHAIN_VERSION=4.9
set ANDROID_NDK_HOST=windows-x86_64

configure.bat -developer-build -platform win32-g++ -opengl es2 -xplatform android-g++ -android-ndk %ANDROID_NDK_PATH% -android-sdk %ANDROID_SDK_ROOT% -opensource -confirm-license -nomake tests -nomake examples

mingw32-make.exe -j2
```

编译报错：

```
  409: template <typename T>
  410: Q_INLINE_TEMPLATE void QList<T>::node_destruct(Node *from, Node *to)
  411: {
  412:    if (QTypeInfo<T>::isLarge || QTypeInfo<T>::isStatic)
  413:        while(from != to) --to, delete reinterpret_cast<T*>(to->v);
  414:    else if (QTypeInfo<T>::isComplex)
  415:        while (from != to) --to, reinterpret_cast<T*>(to)->~T();
  416: }

...when compiled for ARM, causes this warning (or error with -Werror):

  src/corelib/tools/qlist.h: In member function ‘void QList<T>::node_destruct(QList<T>::Node*, QList<T>::Node*) [with T = QVariant]’:
  src/corelib/tools/qlist.h:738:5:   instantiated from ‘void QList<T>::dealloc(QListData::Data*) [with T = QVariant]’
  src/corelib/tools/qlist.h:714:9:   instantiated from ‘QList<T>::~QList() [with T = QVariant]’
  src/corelib/statemachine/qstatemachine.h:81:59:   instantiated from here
  src/corelib/tools/qlist.h:415:28: error: cast from ‘QList<QVariant>::Node*’ to ‘QVariant*’ increases required alignment of target type [-Werror=cast-align]
```

解决方法：
因为在编译QTDIR\qtscript时报错，所以将文件夹下Miakefile里的-Wcast-align去掉，再编译就可以了。