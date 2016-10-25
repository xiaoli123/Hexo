---
title: Centos下 Mysql-5.6的几种安装方式
date: 2016-10-09
tags:
  - mysql
categories: mysql
---

# Centos下 Mysql-5.6的安装 ##

- [mysql-官方文档](http://dev.mysql.com/doc/refman/5.6/en/installing.html)

----------

## rpm安装 ##

> 特点：
> - 安装简单
> - 安装路径不可更改
> - 一台机器只能安装一个mysql

 1.  下载rpm包 [MySQL-5.6.33-1.linux_glibc2.5.x86_64.rpm-bundle.tar 点击下载](http://dev.mysql.com/downloads/mysql/)，注意：默认的是5.7需要选择，如果闲麻烦请从我的百度云下载 [点击下载](http://pan.baidu.com/s/1pL3xjib) 密码：9vrn
 2. 上传到你的linux，当然你也可以用wget直接下载，我是用windows下载的，上传到了/root/mysql_install文件夹
 3. 解压压缩包，得到以下几个rpm包，而我们只需要两个：client、server
```
[root@localhost mysql_install]# mkdir mysql-rpm
[root@localhost mysql_install]# tar xvf MySQL-5.6.33-1.linux_glibc2.5.x86_64.rpm-bundle.tar -C mysql-rpm/
MySQL-embedded-5.6.33-1.linux_glibc2.5.x86_64.rpm
MySQL-server-5.6.33-1.linux_glibc2.5.x86_64.rpm
MySQL-shared-compat-5.6.33-1.linux_glibc2.5.x86_64.rpm
MySQL-shared-5.6.33-1.linux_glibc2.5.x86_64.rpm
MySQL-client-5.6.33-1.linux_glibc2.5.x86_64.rpm
MySQL-devel-5.6.33-1.linux_glibc2.5.x86_64.rpm
MySQL-test-5.6.33-1.linux_glibc2.5.x86_64.rpm
```

 4. 检查当前系统是否安装了mysql，一般centos会带一个mariadb的mysql。关于mariadb的介绍可以[点击了解](http://www.biaodianfu.com/mysql-percona-or-mariadb.html),我们安装的是官方版本的，所以将其卸载
```
[root@localhost mysql-rpm]# rpm -qa|grep -i mariadb
如果检测到了用以下命令卸载
[root@localhost mysql-rpm]# rpm -qa|grep -i mysql
如果检测到了用以下命令卸载
[root@localhost mysql-rpm]# rpm -e -nodeps 检测到的mysql的name
```
 5. 检查是否安装了Dumper（如果不安装这个，mysql安装过程中会出现FATAL ERROR: please install the following Perl modules before executing /usr/bin/mysql_install_db:
Data::Dumper 的错误，出现这种情况，最好的方法不是修复此错误而是卸载当前安装的mysql，卸载完成务必手动删除/var/lib/mysql,执行whereis mysql 删除查询出的目录后重新安装。因为你不知道安装工程中因为少了Dumper而出现了多少坑。
）
```
[root@localhost mysql]# yum search Dumper
[root@localhost mysql]# yum install perl-Data-Dumper.x86_64
```

 6. 判断是否可以安装，如下，就可以安装
```
[root@localhost mysql-rpm]# rpm -ivh --test MySQL-server-5.6.33-1.linux_glibc2.5.x86_64.rpm
警告：MySQL-server-5.6.33-1.linux_glibc2.5.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
```
 7. 安装服务端
```
    [root@localhost mysql-rpm]# rpm -ivh MySQL-server-5.6.33-1.linux_glibc2.5.x86_64.rpm
    安装日志中会出现以下说明(一个随机密码产生路径是/root/.mysql_secret),你初次登录的时候需要它.
    A random root password has been set. You will find it in '/root/.mysql_secret'.
```
 8. + mysql-server的安装过程中会自动调用mysql_install_db --random-passwords初始化数据库并生成一个密码到root目录。
  + mysql会在/etc/init.d即(etc/rc.d/init.d)中添加相应的服务，会在服务器启动的时候自动启动	
 8. 安装客户端
```
[root@localhost mysql-rpm]# rpm -ivh MySQL-client-5.6.33-1.linux_glibc2.5.x86_64.rpm
```
 9. 登录mysql,并查看当前用户
```
[root@localhost mysql-rpm]# cat /root/.mysql_secret
# The random password set for the root user at Sun Oct  9 23:20:07 2016 (local time): jHaEiCgxMFm1tSzC
如果是以root登录而且是本地登录直接mysql -p就可以,不需要指定-u、-h、-P
[root@localhost mysql-rpm]# mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
mysql>
mysql> use mysql
此时我们切换到mysql库提示我们需要设置密码
ERROR 1820 (HY000): You must SET PASSWORD before executing this statement
我们可以通过mysqladmin命令更改密码
[root@localhost mysql-rpm]# mysqladmin -uroot -p password 'root'
输入mysql生成的密码更改成功
Enter password:
mysql> use mysql
Database changed
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)
通过以上看，我们发现有个test的库这个使我们不需要的。
```
 10. 步骤9是否可以更简单点?当然可以，mysql为我们提供了一个命令mysql_secure_installation
 ```
   [root@localhost mysql-rpm]# mysql_secure_installation 

    NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
          SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

    In order to log into MySQL to secure it, we'll need the current
    password for the root user.  If you've just installed MySQL, and
    you haven't set the root password yet, the password will be blank,
    so you should just press enter here.

    Enter current password for root (enter for none): 
    OK, successfully used password, moving on...

    Setting the root password ensures that nobody can log into the MySQL
    root user without the proper authorisation.

    You already have a root password set, so you can safely answer 'n'.
	你可以在这里修改密码
    Change the root password? [Y/n] Y
    New password: 
    Re-enter new password: 
    Password updated successfully!
    Reloading privilege tables..
     ... Success!

    By default, a MySQL installation has an anonymous user, allowing anyone
    to log into MySQL without having to have a user account created for
    them.  This is intended only for testing, and to make the installation
    go a bit smoother.  You should remove them before moving into a
    production environment.
	删掉匿名用户
    Remove anonymous users? [Y/n] Y
     ... Success!

    Normally, root should only be allowed to connect from 'localhost'.  This
    ensures that someone cannot guess at the root password from the network.

    Disallow root login remotely? [Y/n] Y
     ... Success!

    By default, MySQL comes with a database named 'test' that anyone can
    access.  This is also intended only for testing, and should be removed
    before moving into a production environment.
	删掉test库
    Remove test database and access to it? [Y/n] Y      
     - Dropping test database...
     ... Success!
     - Removing privileges on test database...
     ... Success!
    Reloading the privilege tables will ensure that all changes made so far
    will take effect immediately.
	重载权限
    Reload privilege tables now? [Y/n] Y
     ... Success!

    All done!  If you've completed all of the above steps, your MySQL
    installation should now be secure.
    Thanks for using MySQL!
    Cleaning up...
```

 11. 到此安装完成，不过我们需要把配置文件拷贝到etc下。
```
[root@localhost mysql-rpm]# cp /usr/share/mysql/my-default.cnf /etc/my.cnf
```

 12. 安装完成了。。。


## 二进制安装（推荐）

> 特点：
> - 采用官方编译好的二进制文件安装
> - 安装简单
> - 可以安装到任何路径下，灵活性好
> -  一台服务器可以MySQL安装多个 MySQL

1.  下载rpm包 [mysql-test-5.7.15-linux-glibc2.5-x86_64.tar.gz 点击下载](http://dev.mysql.com/downloads/mysql/)，注意：默认的是5.7需要选择，如果闲麻烦请从我的百度云下载 [点击下载](http://pan.baidu.com/s/1pL3xjib) 密码：9vrn
2. 上传到你的linux，当然你也可以用wget直接下载，我是用windows下载的，上传到了/root/mysql_install文件夹
3. 如何卸载之前的mysql：
```
[root@localhost mysql-rpm]# rpm -qa|grep -i mysql
MySQL-server-5.6.33-1.linux_glibc2.5.x86_64
MySQL-client-5.6.33-1.linux_glibc2.5.x86_64
[root@localhost mysql-rpm]# rpm -e --nodeps MySQL-server-5.6.33-1.linux_glibc2.5.x86_64
[root@localhost mysql-rpm]# rpm -e --nodeps MySQL-client-5.6.33-1.linux_glibc2.5.x86_64
[root@localhost mysql-rpm]# rm -rf /etc/my.cnf /var/lib/mysql/ /usr/lib64/mysql/
[root@localhost mysql-rpm]# whereis mysql
mysql:[root@localhost mysql-rpm]#
```
4. 创建mysql用户和用户组（rpm安装方式会自动创建）
```
检查用户，下图所示表示mysql用户和用户组已经存在，因为我们rpm安装的时候生成了，卸载的时候并没有删除掉。
[root@localhost local]# cat /etc/passwd|grep mysql
mysql:x:998:997:MySQL server:/var/lib/mysql:/bin/bash
[root@localhost local]# cat /etc/group|grep mysql
mysql:x:997:
有另一种方式即id username
[root@localhost ~]# id mysql
uid=998(mysql) gid=1000(mysql) 组=1000(mysql)
我们这里先删除再创建
[root@localhost local]# userdel mysql
[root@localhost local]# groupdel mysql
[root@localhost local]# groupadd mysql
[root@localhost local]# useradd -r -g mysql mysql
```
5. 解压安装包
```
[root@localhost mysql_install]# tar zxvf mysql-5.6.33-linux-glibc2.5-x86_64.tar_6.gz -C /usr/local/
```
6. 为解压的文件夹创建软连接，也可以直接重命名。官方文档为：创建软连接
```
[root@localhost mysql_install]# cd /usr/local/
[root@localhost local]# ln -s mysql-5.6.33-linux-glibc2.5-x86_64/ mysql
```
7. 修改mysql文件夹的用户和用户组
```
[root@localhost local]# cd mysql
[root@localhost mysql]# chown -R mysql .
[root@localhost mysql]# chgrp -R mysql .
```
8. 其实现在就算安装完了，下面我们拷贝服务脚本到指定目录
```
[root@localhost mysql]# cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysql
```
9. 配置环境变量
```
[root@localhost mysql]# vim /etc/profile
改动如下：
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL
JAVA_HOME=/usr/java/jdk1.7.0_79
JRE_HOME=$JAVA_HOME/jre
MYSQL_HOME=/usr/local/mysql
CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$MYSQL_HOME/bin
export JAVA_HOME PATH CLASSPATH JRE_HOME MYSQL_HOME
使生效
[root@localhost ~]# source /etc/profile
```
10. 初始化数据库
```
[root@localhost mysql]# scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data/
```
11. 注意，在执行mysql_install_db的时候会再basedir生成my.cnf,如果此时basedir中已经有了此文件会生成一个my-new.cnf，我们将新生成的文件拷贝到etc下
```
[root@localhost mysql]# cp my-new.cnf /etc/my.cnf
编辑 my.cnf
[root@localhost mysql]# vim /etc/my.cnf
修改如下：
# These are commonly set, remove the # and set as required.
 basedir = /usr/local/mysql
 datadir = /usr/local/mysql/data
 character_set_server=utf8
# port = .....
# server_id = .....
# socket = .....
```

12. 启动并连接mysql
```
[root@localhost ~]# service mysql start
Starting MySQL. SUCCESS! 
[root@localhost ~]# mysql -p
此时mysql是没有密码的直接回车即可进入
Enter password: 
这是我们看看用户表
mysql> use mysql
mysql> select * from user\G;
我们会看到两种用户
-
密码是空的
Host: localhost
User: root
Password: 
-
这应该就是传说中的匿名用户
Host: localhost
User: 
Password: 
```
13. 使用mysql_secure_installation安全的初始化mysql。参考rpm安装
14. 安装完成了。。。

## 编译安装(编译时间太长，暂时不讲)
## 多实例安装
> 特点：
> - 管理方便
> - 不同实例数据文件存放到不同磁盘，分散IO
> - 存在cpu和内存争用
> - 适合资金紧张型公司
> - 适合并发量不是很大的业务

1. 1-9步同上面的二进制安装。
2. 创建每个实例的datadir，当然我是创建在一个硬盘上的，因为我只有一个硬盘。
```
[root@localhost tmp]# mkdir /usr/local/mysql/{data3306,data3406}
```
3. 初始化实例
```
ldata相当于datadir
[root@localhost mysql]# ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --ldata=/usr/local/mysql/data3306/
[root@localhost mysql]# ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --ldata=/usr/local/mysql/data3406/
```
4. 修改配置文件，改成如下格式
```
[root@localhost mysql]# vim /etc/my.cnf 
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html
[mysqld_multi]
mysqld=/usr/local/mysql/bin/mysqld_safe
mysqladmin=/usr/local/mysql/bin/mysqladmin
user=admin #此用户用来管理多实例
password=admin123
[mysqld3306]
socket=/tmp/mysql3306.sock
port=3306
pid-file=/usr/local/mysql/data3306/mysql3306.pid
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data3306
server-id=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
[mysqld3406]
socket=/tmp/mysql3406.sock
port=3406
pid-file=/usr/local/mysql/data3406/mysql3406.pid
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data3406
server-id=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```
5. 启动mysql实例，
我们可以指定端口启动mysqld_multi start 3306，
可以以逗号分开启动多实例mysqld_multi start 3306,3406
也可以指定范围启动多实例mysqld_multi start 3306-3706
```
[root@localhost mysql]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld3306 is not running
MySQL server from group: mysqld3406 is not running
[root@localhost mysql]# mysqld_multi start 3306
[root@localhost mysql]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3406 is not running
[root@localhost mysql]# mysqld_multi start 3406
[root@localhost mysql]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3406 is running
```
6. 创建管理mysql用户即上面的[mysqld]中我们配置的用户
7. 切换到tmp文件夹，我们通过指定sock来来接指定的用户
```
[root@localhost mysql]# cd /tmp/
直接回车，我们以匿名用户登录
[root@localhost tmp]# mysql -S mysql3306.sock
切换到mysql数据库，删除掉匿名用户，因为mysql_secure_installation不支持指定sock我们只能手删
mysql> use mysql
mysql> delete from user where user='';
Query OK, 2 rows affected (0.28 sec)
创建管理用户
mysql> grant shutdown on *.* to 'admin'@'localhost' identified by 'admin123';
Query OK, 0 rows affected (0.00 sec)
刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.28 sec)
修改3406，同上。。。
```

8. 现在我们在停止服务，停止服务类似于启动服务
```
[root@localhost tmp]# mysqld_multi stop 3306
[root@localhost tmp]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3406 is running
```
9. 看到上面的结果你一定震惊万分，怎么没有关掉？恭喜你，找到了mysql的一个bug，至于bug产生的原因可以参考我找到的博客 [点击探索](http://blog.csdn.net/zhengwei125/article/details/52413835) ,现在我们来解决掉这个bug。
```
[root@localhost tmp]# vim /usr/local/mysql/bin/mysqld_multi
将
my $com= join ' ', 'my_print_defaults', @defaults_options, $group;
加一个-s
my $com= join ' ', 'my_print_defaults -s', @defaults_options, $group;
```
10. 再次指定关闭，发现成功了。这bug没改估计是因为多实例的应用可能确实少。
11. 多实例的日志的位置：
```
[root@localhost tmp]# tail -f /usr/local/mysql/share/mysqld_multi.log 
```
12. 到这里多实例的安装算是完成了。
13. 现在的root用户时没有密码的，我们可以直接update用户表也可以用mysqladmin设置
```
[root@localhost tmp]# mysqladmin -uroot -p --socket mysql3306.sock password 'root'
```
14. 拓展：我们的配置写在一个配置文件中，其实也可以分散到不同的文件中，具体参考这个博客。 [点击探索](http://blog.csdn.net/leshami/article/details/40339295)