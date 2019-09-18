---
layout: post
category: "Git"
title:  "Git配置常见步骤"
tags: [工具]
summary: ""
---
**Windows环境下配置git bash的home默认路径**

打开 Git/etc/profile，找到

```
# normalize HOME to unix path
HOME="$(cd "$HOME" ; pwd)"
export PATH="$HOME/bin:$PATH"
```

增加两行，修改后的代码如下

```
# normalize HOME to unix path
HOME="/e/repository"
HOME="$(cd "$HOME" ; pwd)"
cd
export PATH="$HOME/bin:$PATH"
```

再次启动Git bash，就会自动进入新修改后的home路径了，之后新配置的.ssh文件夹也为在新路径中。

**创建ssh公钥**

在bash命令行里执行

```
zzc@zzc-PC:~$ ssh-keygen -t rsa -C 'xxx@qq.com'
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): ~/.ssh/id_rsa2 #这里输入一个新的ssh key文件名                           
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in ~/.ssh/id_rsa2.
Your public key has been saved in ~/.ssh/id_rsa2.pub.
The key fingerprint is:
3a:01:17:b3:f9:26:5b:53:b3:69:be:71:a8:66:f6:96 xxx@qq.com
The key's randomart image is:
+--[ RSA 2048]----+
|      o          |
|       =         |
|    . +   o      |
|     . . . +     |
|      o S +      |
|       B + .     |
|      +  .+ +    |
|       .E..+     |
|       +.oo      |
+-----------------+
```

~/.ssh/id_rsa2为新ssh key文件名，如果要创建多个key，根据实际情况修改，保证每次名称不一样即可。

把新产生的ssh key包含进ssh-agent

```
zzc@zzc-PC:~$ eval "$(ssh-agent -s)"
zzc@zzc-PC:~$ ssh-add ~/.ssh/id_rsa2
```

如果执行ssh-add时出现Could not open a connection to your authentication agent，则先执行如下命令即可：

```
zzc@zzc-PC:~$ ssh-agent bash
```

**多个github账号**

我们可以在~/.ssh目录下新建一个config文件配置一下，就可以解决问题，如下：

```
	# company
    Host 192.168.1.2
        HostName 192.168.1.2
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_rsa
        User sheshiji

    # github
    Host github.com
        HostName github.com
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_xingjiegaojue
        User xingjiegaojue
		
	Host github.com
        HostName github.com
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_rsa
        User sheshiji
```

在多个账户下，不同账户的repository，可能要以下操作后，才能push

```
git remote rm origin
git remote add origin XXX.git
```