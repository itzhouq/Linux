# Centos6.5安装MySQL5.6
## 1. 查看系统状态
```
[root@itzhouq32 tools]# cat /etc/issue
CentOS release 6.5 (Final)
Kernel \r on an \m

[root@itzhouq32 tools]# uname -a
Linux itzhouq32 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux

```
## 2.创建需要下载rpm软件包的目录
 ```
[root@itzhouq32 /]# mkdir soft
[root@itzhouq32 /]# cd soft
[root@itzhouq32 soft]# ls
[root@itzhouq32 soft]#
 ```
## 3. 查看系统中是否有已经存在的MySQL
如果有需要卸载
用这个查找
 ```
# rpm -qa | grep mysql
 ```
用这个卸载
``` 
yum -y remove mysql-libs-5.1*
```
## 4. 下载MySQL-5.6安装包
分别执行下面的三条命令
``` 
# wget http://dev.mysql.com/Downloads/MySQL-5.6/MySQL-server-5.6.21-1.rhel5.x86_64.rpm
# wget http://dev.mysql.com/Downloads/MySQL-5.6/MySQL-devel-5.6.21-1.rhel5.x86_64.rpm
# wget http://dev.mysql.com/Downloads/MySQL-5.6/MySQL-client-5.6.21-1.rhel5.x86_64.rpm
```
如果下载的速度太慢，也可以使用window下载后传输到虚拟机的这个目录，然后安装。

## 5. 安装
``` 
# rpm -ivh MySQL-server-5.6.21-1.rhel5.x86_64.rpm
```
如果有以下报错信息，需要安装依赖包。
```sh
error: Failed dependencies:

        libaio.so.1()(64bit) is needed by MySQL-server-5.6.21-1.rhel5.x86_64

        libaio.so.1(LIBAIO_0.1)(64bit) is needed by MySQL-server-5.6.21-1.rhel5.x86_64

        libaio.so.1(LIBAIO_0.4)(64bit) is needed by MySQL-server-5.6.21-1.rhel5.x86_64
```
执行：
```
# yum install -y libaio
```
再次安装
```
# rpm -ivh MySQL-server-5.6.21-1.rhel5.x86_64.rpm
```
```
# rpm -ivh MySQL-client-5.6.21-1.rhel5.x86_64.rpm 
Preparing...                ########################################### [100%]
   1:MySQL-client           ########################################### [100%]
```

## 6. 修改配置文件位置
```
# cp /usr/share/mysql/my-default.cnf /etc/my.cnf
```

## 7. 初始化MySQL
```
# /usr/bin/mysql_install_db
# /etc/init.d/mysql start
# ps -ef | grep mysql
# netstat -anpt | grep 3306
# more /root/.mysql_secret
```
然后会显示你的数据库随机码
```
# The random password set for the root user at Fri Mar 15 01:43:14 2019 (local time): BKEIavP3r8C8wNoB
```
## 8. 登录
```
# mysql -uroot -pBKEIavP3r8C8wNoB
```
登录成功后会显示版本信息
```
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.21

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
## 9. 设置新密码
```
mysql> SET PASSWORD = PASSWORD('2626');
Query OK, 0 rows affected (0.00 sec)

mysql> EXIT
Bye
```
我设置了密码为2626，并退出MySQL
## 10. 设置MySQL服务开机自启动
```
# chkconfig mysql on
# chkconfig mysql --list
mysql          	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
```

## 11. 检查服务状态
```
# service mysql status
 SUCCESS! MySQL running (2447)
```
## 12. 开启MySQL的远程登录
默认情况下MySQL为安全起见，不支持远程登录MySQL，所以需要设置开启登录MySQL的权限。登录MySQL后输入命令：
```
mysql> grant all privileges on *.* to 'root' @'%' identified by '2626';
Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
其中2626是用于远程访问的密码，可以与root密码不一样。这样别的用户远程访问你的数据库时，可以使用root账号，密码2626登录。

## 13. 开放Linux的对外访问的端口
```
[root@itzhouq32 bin]# /sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
[root@itzhouq32 bin]# /etc/rc.d/init.d/iptables save
iptables：将防火墙规则保存到 /etc/sysconfig/iptables：     [确定]
[root@itzhouq32 bin]# 
```

OK，打完收工了！