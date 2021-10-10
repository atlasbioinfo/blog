---
layout: post
title: windows的WSL2安装mariaDB
categories:
 - 生物信息
---

win10或11默认用的是wsl1.wsl2推出之后本来没有动力去升级，因为相当于开了个linux模拟器，不如wsl1来的快。但项目需要把原来的数据库换到mariaDB，所以还是需要wsl2（BTW，docker也能在wsl2上跑）。这个博客记录了安装mariadb全过程。
>* WSL2安装mariaDB
>* mariaDB的初始化
>* 用了HeidiSQL，从此不用再写SQL

***

写在前面：

* **数据库是啥？**
  * 数据库就是很多excel存着数据，并且查找速度还很快。
* **用哪个数据库？**
  * 之前我用sqlite，很方便；之后觉得效率太低换了MySQL；然后知道MySQL的主创团队更新了mariaDB，于是就换了这个。
* **需要学什么语言？**
  * SQL；如果懒一点但是会python，可以用sqlalchemy；如果再懒一点，可以用phpMyAdmin或者HeidiSQL这种界面化的可以直接点击管理数据库的软件。

## WSL2安装mariaDB

### 确认是否是wsl2

先确认一下自己是wsl1还是2。打开powershell，
```dos
#powershell中运行
wsl -l -v
```
![图 1](https://pic.atlasbioinfo.com/wsl2_pic_1633899125259_1633899132704.png)  

如果还没有升级，请参考：[microsoft官方文档如何安装wsl](https://docs.microsoft.com/en-us/windows/wsl/install)。

注意一点，如果之前安装过ubuntu，可能需要卸载再安装，可能。虽然理论上可以通过set-defalut-version改变，但是我没成功（可能后续更新后可行）。

### 安装mariadb

一系列操作如下：
打开ubuntu
```bash
sudo apt install software-properties-common apt-transport-https -y
curl -LsSO https://downloads.mariadb.com/MariaDB/mariadb-keyring-2019.gpg
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb-keyring-2019.gpg.sha256 | sha256sum -c
sudo mv mariadb-keyring-2019.gpg /etc/apt/trusted.gpg.d/ && chmod 644 /etc/apt/trusted.gpg.d/mariadb-keyring-2019.gpg

```
之后建立新文件``vim /etc/apt/sources.list.d/mariadb.list``, 内容是：
```bash
# MariaDB Server
# To use a different major version of the server, or to pin to a specific minor version, change URI below.
deb http://downloads.mariadb.com/MariaDB/mariadb-10.5/repo/ubuntu focal main
deb http://downloads.mariadb.com/MariaDB/mariadb-10.5/repo/ubuntu focal main/debug
```
之后继续用命令：
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install mariadb-server
sudo service mariadb start
```
安装结束。

## mariaDB初始化

所谓送佛送到西。第一次我跟着网上教程成功安装完成后，就结束了，完全一脸懵逼。

之后要干什么？什么创建用户？怎么链接数据库？所以我这里需要跟大家说一下：

```bash
#打开mariadb，虽然用的是mysql
sudo mysql
#之后进入命令行界面，开始数据mysql的命令
#更改root密码
ALTER USER root@localhost IDENTIFIED VIA mysql_native_password USING PASSWORD("你的root密码");
#ctrl-C 退出
```

既然有了密码，就可以用root用户正规进入了：
```bash
sudo mysql -p
#之后输入你刚才的密码

#添加用户
CREATE USER 'atlas'@'localhost' IDENTIFIED BY '用户的密码';
#给这个用户所有权限（注意，这一步之后这个用户类似root，不要随意给）
GRANT ALL PRIVILEGES ON *.* TO 'atlas'@'localhost' WITH GRANT OPTION;
#刷新一下
FLUSH PRIVILEGES;

```

设置完成。

是否还是有点懵逼，会弄用户了，但是怎么加数据？怎么查询？

如果不想学SQL，可以直接用各种软件。比如phpMyAdmin。我接下来需要推荐另一个win平台的开源软件，我觉得很好用，叫HeidiSQL

## 用了HeidiSQL，从此不用再写SQL

官网见：[hedisql.com](https://www.heidisql.com/)。可以点``Download``下载最新版本的。

我觉得这种软件挺不容易的。我试了很多软件，一些开源的软件，高级功能就需要付费，并且非常贵。我感觉最常用的数据库操作外，数据导入导出合并什么属于基本需求，但是很多开源软件都不行。这个软件全部免费，无付费版本，很不容易。作者辛苦了。

其实不是我不付费。而是刚开始我还在学怎么用数据库，什么概念都不知道你让我突然交个几千块我只能一脸懵逼。

![图 2](https://pic.atlasbioinfo.com/heidisql_pic_1633900410639_1633900417252.png)  

上图为heidiSQL的初始连接界面。进入之后：
```bash
127.0.0.1是本地地址，按默认走；
3306是默认端口号

其实需要改的就是用户名和密码，改成刚才设置的；
如果刚才没有添加用户，就直接用root和root的密码。
```

接下来导入数据、导出数据、查询都很简单，相信你点一点都会了。

![博主简介](https://pic.atlasbioinfo.com/logo.png)