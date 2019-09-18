---
layout: post
category: "Redmine"
title:  "在Windows Server 2008 R2系统上安装Redmine"
tags: [工具]
summary: ""
---
###准备工作

①系统：

```
Windows Server 2008 R2（64位）
```

②安装包：

```
rubyinstaller-2.2.4.exe
railsinstaller-3.2.0.exe
mysql-installer-community-5.7.11.0.msi
dotNetFx40_Full_x86_x64.exe
redmine-3.1.3
```

###安装

注意：以管理员身份安装软件。

①安装rubyinstaller。安装完成后，打开cmd命令行，输入

```
ruby -v
```

可以查看版本号。

②安装railsinstaller，在弹出安装目录界面时，可选安装Git，在这里取消安装Git。

③安装dotNetFx40_Full_x86_x64。

④安装mysql。

将Mysql添加到环境变量，我的目录地址为“C:\Program Files\MySQL\MySQL Server 5.7\bin”，将其添加进PATH中。打开cmd命令行，输入：

```
mysql -u root -p
```

然后输入密码，检查是否能登录成功。

⑤安装redmine（参考doc/INSTALL文件进行操作）。

```
a、数据库操作
打开root登录“MySQL 5.7 Command Line Client - Unicode”cmd命令行

# 创建数据库redmine
create database redmine character set utf8;

# 创建用户
create user 'redmine'@'%' identified by 'my_password';
grant all privileges on redmine.* to 'redmine'@'%';
# 对于mysql 5.0.2 的版本 跳过create user ,用这个代替grant all privileges on redmine.* to 'redmine'@'%' identified by 'my_password';

b、复制config/database.yml.example文件为database.yml，打开编辑，修改root为redmine，修改密码。

c、打开cmd命令行，进入redmine目录下（下面几个步骤可能需要翻墙）：
# 检查gem源
gem sources
# 如果不是http://rubygems.org/则执行下面语句，如果是则忽略
gem sources --remove xxx（前面那个源地址）
gem sources --add http://rubygems.org/
# 清空源缓存
gem sources -c
# 更新源缓存
gem sources -u

# 安装bundle
gem install bundler

# 这一条命令可能会报错，查看报错信息，按提示解决
bundle install --without development test rmagick

# 生成数据库会话
bundle exec rake generate_secret_token

# 创建数据库中需要的表
bundle exec rake db:migrate RAILS_ENV="production"

# 启动服务
ruby bin/rails server -e production

# 安装thin服务器
gem install thin
# 修改redmine目录下Gemfile文件，添加一行代码
gem "thin", :group => :production

# 启动服务
ruby bin/rails server thin -e production

# 修改端口
ruby bin/rails server thin -e production -p 3001
```

###配置邮箱

参考地址：[http://www.cnblogs.com/cs_net/p/5020813.html](http://www.cnblogs.com/cs_net/p/5020813.html)

注意：很多网上的资料都是使用163的smtp服务。实际上，要使用163邮箱的smtp服务得专门开通才行。开通服务在:设置->邮箱设置->POP3/SMTP/IMAP下。开通SMTP服务，163要求设置“客户端授权密码",这个密码是要用到redmine的邮件发送配置的，所以要注意，使用其他邮箱也是类似操作。

下面配置的是QQ邮箱：

```
email_delivery:
  delivery_method: :smtp
  smtp_settings:
    address: "smtp.qq.com"
    port: 25
    authentication: :login
    domain: "qq.com"
    user_name: "你的邮箱@qq.com"
    password: "客户端授权密码(不是邮箱登录密码)"
```
	
重启redmine的服务。

检查redmine的邮件发送功能。
在 "管理->配置->邮件通知"标签下：

```
"邮件发送人地址"
改成:
你的邮箱@qq.com
```

保存设置，再点右下角的"发送测试邮件"按键即可。

###配置局域网

①防火墙打开3000端口，具体参考[http://www.veryhuo.com/a/view/48280.html](http://www.veryhuo.com/a/view/48280.html)。

②redmine服务启动成功后，浏览器访问http://192.168.10.107:3000，无法访问到内容，在本地telnet 192.168.10.107 3000连接不上。处理办法为：

```
# 192.168.10.107是在一个局域网内部的IP，redmine启动默认绑定的locahost端口为3000：
a、检查防火墙设置，开放3000端口。
b、localhost多绑定一个hosts，编辑hosts即可。
127.0.0.1			localhost
192.168.10.107		localhost
```