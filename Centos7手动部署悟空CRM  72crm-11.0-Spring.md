## Centos7手动部署悟空CRM  72crm-11.0-Spring

#### 安装java

```
 yum -y install java-1.8.0-openjdk-devel
```

#### 安装配置Mysql数据库

```
rpm -ivh https://repo.mysql.com//yum/mysql-5.7-community/el/7/x86_64/mysql57-community-release-el7-10.noarch.rpm --nodeps --force
yum -y install mysql-community-server
systemctl start mysqld
systemctl enable mysqld
```

**获取Mysql的root密码**

```
grep "password" /var/log/mysqld.log
```

![image-20210907103713635](https://img.myhappiness.top/img/202109071037704.png)

**使用初始密码登录数据库，登录成功后需要设置root密码（修改的密码需要符合长度，且必须含有数字，小写或大写字母，特殊字符）如需设置简单的密码需要使用以下方法进行操作**

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

mysql> set password for 'root'@'localhost'=password('password');     #设置密码命令
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> grant all on *.* to 'root'@'%' identified by 'password';     #授权root用户
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;              #刷新权限表
Query OK, 0 rows affected (0.00 sec)

mysql> \q               #退出
Bye
```

**编辑配置文件添加以下配置**

```
vim /etc/my.cnf
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

![image-20211221155525740](https://img.myhappiness.top/img/image-20211221155525740.png)

**重启数据库**

```
systemctl restart mysqld
```

#### 安装redis

```
yum -y install epel-release
yum -y install redis
systemctl enable redis
#-- 修改redis密码为123456
yum -y install vim
vim /etc/redis.conf
#-- 在文件最下面追加一行
requirepass 123456
#-- 或者输入 / 搜索 # requirepass foobared
#-- 将前面的#删除，将foobared改为123456
#-- 修改完成之后 :wq 保存并退出,重启redis
service redis restart
```

####  安装maven环境

```
yum -y install maven
```

**配置使用国内镜像加速**

![image-20211221155321307](https://img.myhappiness.top/img/image-20211221155321307.png)

```
vim /etc/maven/settings.xml
  <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
   </mirror>
```

#### 安装elasticsearch

```
  #-- 在opt目录下安装
  cd /opt

  #-- 下载elasticsearch6.8.6
  wget https://file.72crm.com/static/elasticsearch-6.8.6.zip

  #-- 下载analysis-icu-6.8.6分词器插件
  wget https://file.72crm.com/static/analysis-icu-6.8.6.zip

  #--下载unzip
  yum -y install unzip

  #--解压刚才下载的elasticsearch
  unzip elasticsearch-6.8.6.zip
  
  #--将分词器插件解压到elasticsearch/plugins目录下
  unzip analysis-icu-6.8.6.zip -d elasticsearch-6.8.6/plugins/analysis-icu-6.8.6

  #--创建elasticsearch用户
  useradd elasticsearch

  #--赋予用户目录权限
  chown -R elasticsearch:elasticsearch elasticsearch-6.8.6

  #--切换至elasticsearch用户
  su elasticsearch

  #--启动
  cd elasticsearch-6.8.6/bin
  ./elasticsearch -d
  #--退出elasticsearch用户
  exit
```

####  下载悟空CRM源码

```
  #--下载git
  yum -y install git

  #--下载代码到/opt目录
  cd /opt
  git clone https://gitee.com/wukongcrm/crm_pro.git

  #-- 进入项目目录
  cd crm_pro

  #--打包，第一次打包会比较慢，请耐心等待
  mvn clean -Dmaven.test.skip=true package

  #-- 创建package文件夹
  mkdir /opt/package;

  #--将打包后代码复制到一个目录，方便统一管理
  cd /opt/crm_pro/
  cp admin/target/admin.zip /opt/package;
  cp authorization/target/authorization.zip /opt/package;
  cp crm/target/crm.zip /opt/package;
  cp gateway/target/gateway.zip /opt/package;
  cp bi/target/bi.zip /opt/package;
  cp examine/target/examine.zip /opt/package;
  cp oa/target/oa.zip /opt/package;
  cp work/target/work.zip /opt/package;
  cp job/target/job.zip /opt/package;
  cp hrm/target/hrm.zip /opt/package;

  #-- 解压对应文件
  cd /opt/package;
  unzip admin.zip -d admin;
  unzip authorization.zip -d authorization;
  unzip crm.zip -d crm;
  unzip gateway.zip  -d gateway;
  unzip bi.zip -d bi;
  unzip examine.zip -d examine;
  unzip oa.zip -d oa;
  unzip work.zip -d work;
  unzip job.zip -d job;
  unzip hrm.zip -d hrm;
```

#### 创建数据库

```
  #-- 创建nacos数据库
  create database nacos character set utf8mb4 collate utf8mb4_general_ci;
  use nacos;
  source /opt/crm_pro/DB/nacos.sql;

  #-- 创建seata数据库
  create database seata character set utf8mb4 collate utf8mb4_general_ci;
  use seata;
  source /opt/crm_pro/DB/seata.sql;

  #-- 创建xxl_job数据库
  create database xxl_job character set utf8mb4 collate utf8mb4_general_ci;
  use xxl_job;
  source /opt/crm_pro/DB/xxl_job.sql;

  #-- 创建crm数据库
  create database wk_crm_single character set utf8mb4 collate utf8mb4_general_ci;
  use wk_crm_single ;
  source /opt/crm_pro/DB/wk_crm_single.sql;
  
  #-- 创建hrm数据库
  create database wk_hrm_single character set utf8mb4 collate utf8mb4_general_ci;
  use wk_hrm_single ;
  source /opt/crm_pro/DB/wk_hrm_single.sql;
  
  #-- 退出数据库
  exit;
```

#### 安装配置nacos

```
  #--在opt目录下安装
  cd /opt
  #--下载nacos
  wget https://file.72crm.com/static/nacos-server-1.2.1.zip
  #--解压nacos到opt目录
  unzip nacos-server-1.2.1.zip -d /opt
  #--删除压缩包
  rm -rf nacos-server-1.2.1.zip
```

**配置nacos**

```
  cd /opt/nacos/conf
  vim application.properties   #取消以下行注释并按照以下内容进行配置
```

![image-20211221160841168](https://img.myhappiness.top/img/image-20211221160841168.png)

**启动nacos**

```
  cd /opt/nacos/bin
  sh startup.sh -m standalone
```

**查看端口是否正常访问及日志文件中是否有报错信息**

![image-20211221161206104](https://img.myhappiness.top/img/image-20211221161206104.png)

**浏览器访问http://服务器ip:8848/nacos/**

默认用户名：nacos 密码:nacos

![image-20211221161528524](https://img.myhappiness.top/img/image-20211221161528524.png)

**登录完成后查看到以下信息说明配置正确**

![image-20211221161429781](https://img.myhappiness.top/img/image-20211221161429781.png)

#### 配置安装seata

```
  #--下载seata
  wget https://file.72crm.com/static/seata-server-1.2.0.zip
  #--解压seata到opt目录
  unzip seata-server-1.2.0.zip -d /opt
  #--删除安装包
  rm -rf seata-server-1.2.0.zip
```

**配置seata持久化到nacos**

```
  cd /opt/seata/conf/
  mv registry.conf registry.conf.bak
  vim registry.conf
```

```
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    namespace = "public"
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = "public"
    group = "SEATA_GROUP"
    username = "nacos"
    password = "nacos"
  }
  file {
    name = "file.conf"
  }
}
```

**启动seata**

```
  cd /opt/seata/bin
  nohup sh seata-server.sh > log.out 2>&1 &
```

#### 安装sentinel

```
  #--下载sentinel
  wget https://file.72crm.com/static/sentinel-dashboard-1.7.2.jar
  #--将sentinel放到指定目录
  mkdir /opt/sentinel
  mv sentinel-dashboard-1.7.2.jar /opt/sentinel/sentinel.jar
```

**启动sentinel**

```
  #-- 启动sentinel
  cd /opt/sentinel
  nohup java -Xms128m -Xmx512m -Dserver.port=8079 -Dproject.name=sentinel-dashboard -jar sentinel.jar &
```

**浏览器访问查看**

默认用户名密码为：sentinel

![image-20211221163516054](https://img.myhappiness.top/img/image-20211221163516054.png)

#### 安装配置xxl-job 2.1.2

```
  #-- 下载xxl-job 2.1.2版本
  cd /opt
  git clone -b 2.1.2 https://gitee.com/xuxueli0323/xxl-job.git
  cd xxl-job;
  修改xxl-job的配置文件
  vim xxl-job-admin/src/main/resources/application.properties
  #-- 修改spring.datasource.url，spring.datasource.username,spring.datasource.password参数，如下图所示
```

![image-20211221162642904](https://img.myhappiness.top/img/image-20211221162642904.png)

**打包并启动服务**

```
  mvn clean -Dmaven.test.skip=true package
  #-- 启动xxl-job-admin
  mkdir /opt/xxl-job-admin
  cp xxl-job-admin/target/xxl-job-admin-2.1.2.jar /opt/xxl-job-admin
  cd /opt/xxl-job-admin/
  nohup java -Xms128m -Xmx512m -jar xxl-job-admin-2.1.2.jar> log.out 2>&1 &
```

#### 启动项目

```
  cd /opt/package/admin/ && sh 72crm.sh start;
  cd /opt/package/authorization/ && sh 72crm.sh start;
  cd /opt/package/crm/ && sh 72crm.sh start;
  cd /opt/package/gateway/ && sh 72crm.sh start;
  cd /opt/package/bi/ && sh 72crm.sh start;
  cd /opt/package/examine/ && sh 72crm.sh start;
  cd /opt/package/oa/ && sh 72crm.sh start;
  cd /opt/package/work/ && sh 72crm.sh start;
  cd /opt/package/job/ && sh 72crm.sh start;
  cd /opt/package/hrm/ && sh 72crm.sh start;
```

#### 测试访问

**浏览器访问：服务器ip:8443**

![image-20211210112113349](https://img.myhappiness.top/img/image-20211210112113349.png)

![image-20211221163305329](https://img.myhappiness.top/img/image-20211221163305329.png)

![image-20211221163401808](https://img.myhappiness.top/img/image-20211221163401808.png)

#### 配置使用nginx代理

```
[root@localhost ~]# yum -y install nginx
[root@localhost ~]# vim /etc/nginx/conf.d/wkcrm.conf
    server {
        listen       80;
        server_name  192.168.200.35;
        client_max_body_size 100M;
        location / {
                proxy_pass   http://192.168.200.35:8443/;
        }
        location /nacos/ {
                add_header Access-Control-Allow-Origin *;
                add_header Access-Control-Allow-Methods *;
                proxy_buffering off;
                proxy_redirect off;
                proxy_set_header Host $host:$server_port;
                proxy_set_header proxy_url "api";
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto  $scheme;
                proxy_connect_timeout 60;
                proxy_send_timeout 120;
                proxy_read_timeout 120;
                proxy_pass   http://192.168.200.35:8848/nacos/;
            }
        location /xxl-job-admin/ {
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods *;
            proxy_buffering off;
            proxy_redirect off;
            proxy_set_header Host $host:$server_port;
            proxy_set_header proxy_url "api";
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto  $scheme;
            proxy_connect_timeout 60;
            proxy_send_timeout 120;
            proxy_read_timeout 120;
            proxy_pass   http://192.168.200.35:8080/xxl-job-admin/;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
[root@localhost ~]# nginx -t
nginx: [warn] conflicting server name "localhost" on 0.0.0.0:80, ignored
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@localhost ~]# systemctl restart nginx
[root@localhost ~]# systemctl enable nginx
```

**测试使用80端口访问**

![image-20211221181118350](https://img.myhappiness.top/img/image-20211221181118350.png)