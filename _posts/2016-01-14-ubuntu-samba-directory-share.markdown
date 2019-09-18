---
layout: post
category: "软件工具"
title:  "Ubuntu下配置samba实现文件夹共享"
tags: [工具]
summary: ""
---
参考：[http://www.cnblogs.com/phinecos/archive/2009/06/06/1497717.html](http://www.cnblogs.com/phinecos/archive/2009/06/06/1497717.html)

###一. samba的安装:

```
sudo apt-get insall samba
sudo apt-get install smbfs
```

###二. 创建共享目录:

```sh
mkdir /home/XXX/share
sodu chmod 777 /home/XXX/share
注意：XXX为用户名
```

###三. 创建Samba配置文件:

```sh
1.保存现有的配置文件
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak

2.修改现配置文件
sudo gedit /etc/samba/smb.conf

3.在smb.conf最后添加
[share]
　　path = /home/XXX/share
　　available = yes
　　browsealbe = yes
　　public = yes
　　writable = yes
```

###四. 创建samba帐户

```sh
sudo touch /etc/samba/smbpasswd
sudo smbpasswd -a XXX
然后会要求你输入samba帐户的密码
注意：如果没有第四步，当你登录时会提示 session setup failed: NT_STATUS_LOGON_FAILURE。
```

###五. 重启samba服务器

```sh
sudo /etc/init.d/samba restart
```

###六. 测试

```sh
smbclient -L //localhost/share
```

###七. 使用

```sh
可以到windows下输入ip使用了，在文件夹处输入 "\\" + "Ubuntu机器的ip或主机名" + "\\" + "share"。
```