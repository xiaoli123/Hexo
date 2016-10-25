---
title: Mysql基本的命令
date: 2016-10-19
tags: 
	- mysql
categories: mysql
---

## Mysql基本的命令
###  mysql_install_db
> 此命令用来初始化MySql数据库。

#### 常用参数
+ datadir：数据文件目录
+ ldata：等同于datadir
+ basedir：mysql的安装目录
+ keep-my-cnf：执行mysql_install_db会在mysql的basedir目录下自动生成一个.cnf配置文件，如果此目录下已经存在此文件，会加-new，如my-new.cnf。
+ user：创建的文件所属的用户一般为mysql，如果不加，文件的user和group默认会为root。会出事情的！
+ random-passwords：在root目录下生成一个随机密码。如果已存在此文件会追加到文件末尾。

#### 举个栗子：
1. 我们现在有一个安装好的数据库，现在我们想重置数据库。
2. 

