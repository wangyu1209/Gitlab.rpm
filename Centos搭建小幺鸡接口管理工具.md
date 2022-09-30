# Centos搭建小幺鸡接口管理工具

**准备环境**

- apache-tomcat-9.0.58
- jdk-8u251-linux-x64
- apiManager-master
- xiaoyaoji-2.1.7.1
- Mysql5.7

**系统版本**

- Centos7.6

小幺鸡软件包下载地址https://gitee.com/zhoujingjie/apiManager

## 一，下载软件包上传至服务器

```
[root@localhost ~]# ls
jdk-8u251-linux-x64.tar.gz 
apache-tomcat-9.0.58.tar.gz          
apiManager-master.zip       
xiaoyaoji-2.1.7.1.zip
```

## 二，部署java环境

```
[root@localhost ~]# tar zxf jdk-8u251-linux-x64.tar.gz 
[root@localhost ~]# mv jdk1.8.0_251/ /usr/local/java
[root@localhost ~]# cat >> /etc/profile << EOF
export JAVA_HOME=/usr/local/java
export PATH=$PATH:/usr/local/java/bin
EOF
[root@localhost ~]# source /etc/profile
[root@localhost ~]# java -version
java version "1.8.0_251"
Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.251-b08, mixed mode)
```

## 三，安装tomcat

```
[root@localhost ~]# tar zxf apache-tomcat-9.0.58.tar.gz           #解压压缩包
[root@localhost ~]# mv apache-tomcat-9.0.58 /usr/local/tomcat   
[root@localhost ~]# vim /etc/init.d/tomcat                    #编辑启动脚本
#!/bin/bash  
#  
# tomcat startup script for the Tomcat server  
#  
# chkconfig: 345 80 20  
# description: start the tomcat deamon  
#  
# Source function library  
JAVA_HOME=/usr/local/java
export JAVA_HOME  
CATALANA_HOME=/usr/local/tomcat
export CATALINA_HOME  
case "$1" in
start)
    echo "Starting Tomcat..."  
    $CATALANA_HOME/bin/startup.sh
    ;;
stop)
    echo "Stopping Tomcat..."  
    $CATALANA_HOME/bin/shutdown.sh
    ;;
restart)
    echo "Stopping Tomcat..."  
    $CATALANA_HOME/bin/shutdown.sh
    sleep 2
    echo  
    echo "Starting Tomcat..."  
    $CATALANA_HOME/bin/startup.sh
    ;;
*)
    echo "Usage: $prog {start|stop|restart}"  
    ;;
esac
exit 0
[root@localhost ~]# chmod +x /etc/init.d/tomcat              #添加执行权限
[root@localhost ~]# service tomcat start                     #启动服务
[root@controller ~]# service tomcat start                    #Centos7也可使用systemctl进行管理
Starting Tomcat...
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/java
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
[root@localhost ~]# chkconfig --add tomcat                   #添加服务到系统启动项
[root@localhost ~]# chkconfig tomcat on                      #设置开机自启
[root@localhost ~]# netstat -tlunp |grep 8080                 #查看端口是否监听
tcp6       0      0 :::8080                 :::*                    LISTEN      16847/java 
```

## 四，安装配置mysql数据库

安装Mysql之前需要检测是否安装了MariaDB数据库，如已安装了MariaDB数据库使用以下命令进行移除

```
 yum -y remove MariaDB* mariadb*
```

添加国内Mysql5.7yum源

```
cat >> /etc/yum.repos.d/mysql-community.repo  << EOF
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=https://mirrors.cloud.tencent.com/mysql/yum/mysql-5.7-community-el7-x86_64/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
EOF
```

通过命令安装Mysql5.7

```
yum -y install mysql-community-server
```

安装完成后启动数据库，并设为开机自启

```
systemctl start mysqld
systemctl enable mysqld
```

获取Mysql的root密码

```
grep "password" /var/log/mysqld.log
```

![image-202109071037704](https://img.myhappiness.top/img/202109071037704.png)

使用初始密码登录数据库，登录成功后需要设置root密码（修改的密码需要符合长度，且必须含有数字，小写或大写字母，特殊字符）如需设置简单的密码需要使用以下方法进行操作

```
[root@localhost ~]# mysql -uroot -p
Enter password:           #输入刚获取到的初始密码
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.7.35

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set global validate_password_policy=0;      #定义复杂度
Query OK, 0 rows affected (0.00 sec) 

mysql> set global validate_password_length=1;       #定义长度，默认是8
Query OK, 0 rows affected (0.00 sec)

mysql> set password for 'root'@'localhost'=password('Changeme@123');     #设置密码命令
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> grant all privileges on *.*  to 'root'@'%' identified by 'Changeme@123';    #授权root用户可以远程登录
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;              #刷新权限表
Query OK, 0 rows affected (0.00 sec)

mysql> \q               #退出
Bye
```

## 五，配置项目包

将/usr/local/tomcat/webapps/ROOT/下文件清空 然后将小幺鸡项目包解压到这个目录下；

```
[root@localhost ~]# rm -rf /usr/local/tomcat/webapps/ROOT/*
[root@localhost ~]# unzip xiaoyaoji-2.1.7.1.zip -d /usr/local/tomcat/webapps/ROOT/
```

## 六，导入mysql数据库

启动musql 创建数据库xiaoyaoji，并执行数据库脚本，脚本在另一个压缩包中

```
[root@localhost ~]# unzip apiManager-master.zip -d /tmp/
[root@localhost ~]# ls /tmp/apiManager-master/doc/
readme.md  xiaoyaoji.sql
[root@localhost ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 43
Server version: 5.7.37 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database xiaoyaoji;
Query OK, 1 row affected (0.00 sec)

mysql> use xiaoyaoji;
Database changed
mysql> source /tmp/apiManager-master/doc/xiaoyaoji.sql
.....
Query OK, 0 rows affected, 1 warning (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

mysql> \q
Bye
```

## 七，修改小幺鸡配置文件

修改数据库的用户名和密码信息即可

```
[root@localhost ~]# vim /usr/local/tomcat/webapps/ROOT/WEB-INF/classes/config.properties
```

![image-20220225165017056](https://img.myhappiness.top/img/image-20220225165017056.png)

## 八，重新启动tomcat不然会无法连接数据库

```
[root@localhost ~]# /usr/local/tomcat/bin/shutdown.sh 
[root@localhost ~]# /usr/local/tomcat/bin/startup.sh 
[root@localhost ~]# netstat -tlunp |grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      28069/java
```

## 九，访问测试

浏览器访问服务器ip:8080

![image-20220225165337194](https://img.myhappiness.top/img/image-20220225165337194.png)