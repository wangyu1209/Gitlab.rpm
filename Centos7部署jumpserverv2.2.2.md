# Centos7安装jumPserver2.2.2

- 系统环境
- 系统版本：CentOS Linux release 7.6.1810 (Core) 
- Python: Python 3.6.8
- Docker：Docker version 20.10.11, build dea9396
- 数据库：5.5.68-MariaDB
- nginx:   nginx version: nginx/1.20.1
- jumpserver-v2.2.2

说明：jumpserver包官方地址为：https://github.com/jumpserver/jumpserver

本文档使用的的包信息为官方相应版本的clone版本，排查部署中遇到的问题，代码托付在国内码云上，只供学习使用，以便拉取包更方便

**安装python3环境**

```
yum install wget zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make zlib zlib-devel -y
```

配置国内pip镜像加速

```
mkdir ~/.pip
vim ~/.pip/pip.conf
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

下载安装包

```
cd /opt/
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tar.xz
```

解压编译Python包，在opt目录下进行解压

```
tar xf Python-3.6.8.tar.xz 
cd Python-3.6.8
./configure && make -j 4 && make install
```

建立python3虚拟环境

因为 CentOS 6/7 自带的是 Python2，而 yum 等工具依赖原来的 Python，为了不扰乱原来的环境我们来使用 Python 虚拟环境

```
cd /opt
python3 -m venv py3
[root@i-ic6pktm9 opt]# source /opt/py3/bin/activate          #以后使用python3时都需要执行source /opt/py3/bin/activate命令
(py3) [root@i-ic6pktm9 opt]# pip -V
pip 18.1 from /opt/py3/lib/python3.6/site-packages/pip (python 3.6)
(py3) [root@i-ic6pktm9 opt]# python -V
Python 3.6.8
```

升级pip版本

```
(py3) [root@i-ic6pktm9 opt]# python -V
Python 3.6.8
(py3) [root@i-ic6pktm9 opt]#  pip install --upgrade pip
Looking in indexes: https://mirrors.aliyun.com/pypi/simple/
Collecting pip
  Downloading https://mirrors.aliyun.com/pypi/packages/a4/6d/6463d49a933f547439d6b5b98b46af8742cc03ae83543e4d7688c2420f8b/pip-21.3.1-py3-none-any.whl (1.7MB)
    100% |████████████████████████████████| 1.7MB 3.9MB/s 
Installing collected packages: pip
  Found existing installation: pip 18.1
    Uninstalling pip-18.1:
      Successfully uninstalled pip-18.1
Successfully installed pip-21.3.1
(py3) [root@i-ic6pktm9 opt]# pip -V
pip 21.3.1 from /opt/py3/lib/python3.6/site-packages/pip (python 3.6)
```

**拉取jumPserver安装包**

```
(py3) [root@kvm ~]# cd /opt/
(py3) [root@kvm opt]# yum -y install git
(py3) [root@kvm opt]# git clone  -b v2.2.2 https://gitee.com/wangyu1209/jumpserver.git && cd jumpserver/ 
```

**安装依赖rpm包**

```
(py3) [root@kvm jumpserver]# cd requirements/
(py3) [root@kvm requirements]# yum -y install $(cat rpm_requirements.txt)
```

**安装python库依赖**

```
(py3) [root@kvm requirements]# pip install -r requirements.txt
```

**安装redis**

```
(py3) [root@kvm requirements]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
(py3) [root@kvm requirements]# yum -y install redis
(py3) [root@kvm requirements]# systemctl enable redis;systemctl start redis
```

**安装并配置mysql数据库**

```
(py3) [root@kvm requirements]# yum  install mariadb mariadb-devel mariadb-server -y
(py3) [root@kvm requirements]# systemctl enable mariadb;systemctl start mariadb
(py3) [root@kvm requirements]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.4.22-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database jumpserver default charset 'utf8';
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> \q
Bye
```

**修改jumpserver配置文件**

```
(py3) [root@kvm requirements]# cd /opt/jumpserver/
(py3) [root@kvm jumpserver]# cp config_example.yml config.yml
(py3) [root@kvm jumpserver]# SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`
(py3) [root@kvm jumpserver]# echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc
(py3) [root@kvm jumpserver]# BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`
(py3) [root@kvm jumpserver]# echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc
(py3) [root@kvm jumpserver]# sed -i "s/SECRET_KEY:/SECRET_KEY: $SECRET_KEY/g" /opt/jumpserver/config.yml
(py3) [root@kvm jumpserver]# sed -i "s/BOOTSTRAP_TOKEN:/BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN/g" /opt/jumpserver/config.yml 
(py3) [root@kvm jumpserver]# sed -i "s/# DEBUG: true/DEBUG: false/g" /opt/jumpserver/config.yml 
(py3) [root@kvm jumpserver]# sed -i "s/# LOG_LEVEL: DEBUG/LOG_LEVEL: ERROR/g" /opt/jumpserver/config.yml 
(py3) [root@kvm jumpserver]# sed -i "s/# SESSION_EXPIRE_AT_BROWSER_CLOSE: false/SESSION_EXPIRE_AT_BROWSER_CLOSE: true/g" /opt/jumpserver/config.yml 
(py3) [root@kvm jumpserver]# sed -i "s/DB_PASSWORD: /DB_PASSWORD: \'123456\'/g" /opt/jumpserver/config.yml 
```

![image-20211205171106194](https://img.myhappiness.top/img/image-20211205171106194.png)

**运行jumpserver**

```
(py3) [root@kvm jumpserver]# ./jms start all -d
```

![image-20211205174415936](https://img.myhappiness.top/img/image-20211205174415936.png)

浏览器访问：http://192.168.201.139:8080

看到页面访问不正常，是由于Debug 配置了false，此处不用理会，后续使用nginx代理访问

![image-20211205162054870](https://img.myhappiness.top/img/image-20211205162054870.png)

**安装koko组件**

```
(py3) [root@kvm jumpserver]# cd /opt/
(py3) [root@kvm opt]# wget https://gitee.com/wangyu1209/koko/attach_files/902742/download/koko-v2.2.2-linux-amd64.tar.gz
(py3) [root@kvm opt]# tar -xf koko-v2.2.2-linux-amd64.tar.gz 
(py3) [root@kvm opt]# mv koko-v2.2.2-linux-amd64 koko
(py3) [root@kvm opt]# chown -R root:root koko
(py3) [root@kvm opt]# cd koko
(py3) [root@kvm koko]# mv kubectl /usr/local/bin/
(py3) [root@kvm koko]# wget https://gitee.com/wangyu1209/koko/attach_files/902829/download/kubectl.tar.gz
(py3) [root@kvm koko]# tar -xf kubectl.tar.gz
(py3) [root@kvm koko]# chmod 755 kubectl
(py3) [root@kvm koko]# mv kubectl /usr/local/bin/rawkubectl
(py3) [root@kvm koko]# rm -rf kubectl.tar.gz
(py3) [root@kvm koko]# cp config_example.yml config.yml
```

**修改配置文件**

```

(py3) [root@kvm koko]# TOKEN=`awk '/BOOTSTRAP_TOKEN/ {print $2}' /opt/jumpserver/config.yml`
(py3) [root@kvm koko]# sed -i " s/BOOTSTRAP_TOKEN: .*/BOOTSTRAP_TOKEN: ${TOKEN}/" config.yml
```

**启动koko**

```
(py3) [root@kvm koko]# ./koko -d
(py3) [root@kvm koko]# netstat -tlunp |grep koko
tcp6       0      0 :::5000                 :::*                    LISTEN      10383/./koko        
tcp6       0      0 :::2222                 :::*                    LISTEN      10383/./koko        
```

**部署guacamole**

docker方式部署

基于 HTML 5 和 JavaScript 的 VNC 查看器 建议使用 Docker 部署 Guacamole 组件 , 部分环境可能无法正常编译安装

**安装docker**

一，确保没有安装docker，使用以下命令移除docker

```
yum remove docker* -y
```

二，安装docker需要使用的依赖

```
yum install -y yum-utils
```

三，下载国内镜像加速docker，repo源

```
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

四，安装docker

```
yum install docker-ce docker-ce-cli containerd.io -y
```

五，启动docker并设为开机自启

```
[root@kvm ~]# systemctl start docker
[root@kvm ~]# systemctl enable docker
```

配置国内镜像加速

```
[root@kvm ~]# sudo tee /etc/docker/daemon.json <<-'EOF'
{
   "registry-mirrors": ["https://1me84w5g.mirror.aliyuncs.com"]
}
EOF
[root@kvm ~]# systemctl daemon-reload && systemctl restart docker
```

**安装示例**

```
docker run --name jms_guacamole -d 
  -p 127.0.0.1:8081:8080 
  -e JUMPSERVER_SERVER=http://<Jumpserver_url> 
  -e BOOTSTRAP_TOKEN=<g8N451h8LANTeREJ> 
  -e GUACAMOLE_LOG_LEVEL=ERROR 
  jumpserver/jms_guacamole:<Tag>
<Jumpserver_url> 为 JumpServer 的 url 地址, <Jumpserver_BOOTSTRAP_TOKEN> 需要从 jumpserver/config.yml 里面获取, 保证一致, <Tag> 是版本
```

**docker安装运行guacamole**

```
docker run --name jms_guacamole -d \
  -p 127.0.0.1:8081:8080  \
  -e JUMPSERVER_SERVER=http://192.168.201.139:8080 \
  -e BOOTSTRAP_TOKEN=8XpS55jBbPkewdaq \
  -e GUACAMOLE_LOG_LEVEL=ERROR \
  jumpserver/jms_guacamole:v2.2.2
```

**部署lina和luna组件**

我nginx用root启动的，所以赋权root

**部署luna**

```
(py3) [root@kvm koko]# cd /opt/
(py3) [root@kvm opt]# wget https://gitee.com/wangyu1209/luna/attach_files/902737/download/luna-v2.2.2.tar.gz
(py3) [root@kvm opt]# tar -xf luna-v2.2.2.tar.gz 
(py3) [root@kvm opt]# mv luna-v2.2.2 luna
(py3) [root@kvm opt]# chown -R root:root luna
```

**lina部署**

```
(py3) [root@kvm opt]# cd /opt/
(py3) [root@kvm opt]# wget https://gitee.com/wangyu1209/lina/attach_files/902741/download/lina-v2.2.2.tar.gz
(py3) [root@kvm opt]# tar -xf lina-v2.2.2.tar.gz 
(py3) [root@kvm opt]# mv lina-v2.2.2 lina
(py3) [root@kvm opt]# chown -R root:root lina
```

**配置nginx**

```
(py3) [root@kvm opt]# yum -y install nginx
(py3) [root@kvm opt]# vim /etc/nginx/conf.d/jumpserver.conf
server {
    listen 80;
    client_max_body_size 100m;  # 录像及文件上传大小限制
    location /ui/ {
        try_files $uri / /index.html;
        alias /opt/lina/;
    }

    location /luna/ {
        try_files $uri / /index.html;
        alias /opt/luna/;  # luna 路径, 如果修改安装目录, 此处需要修改
    }

    location /media/ {
        add_header Content-Encoding gzip;
        root /opt/jumpserver/data/;  # 录像位置, 如果修改安装目录, 此处需要修改
    }
    location /static/ {
        root /opt/jumpserver/data/;  # 静态资源, 如果修改安装目录, 此处需要修改
    }

    location /koko/ {
        proxy_pass       http://localhost:5000;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location /guacamole/ {
        proxy_pass       http://localhost:8081/;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location /ws/ {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:8070;
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    location /api/ {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /core/ {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location / {
        rewrite ^/(.*)$ /ui/$1 last;
    }
}
(py3) [root@kvm opt]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
(py3) [root@kvm opt]# systemctl start nginx && systemctl enable nginx

```

浏览器访问http://192.168.201.139  用户名:admin 密码：admin，不要使用8080访问

![image-20211205165550923](https://img.myhappiness.top/img/image-20211205165550923.png)

登录成功后重新设定密码，然后重新登录

![image-20211201170720765](https://img.myhappiness.top/img/image-20211201170720765.png)

![image-20211205165734467](https://img.myhappiness.top/img/image-20211205165734467.png)

**进行系统设置**

![image-20211206112551431](https://img.myhappiness.top/img/image-20211206112551431.png)

**配置邮箱**

![image-20211206113403563](https://img.myhappiness.top/img/image-20211206113403563.png)

**配置完成后测试链接收到邮件后点击提交**

![image-20211206113514809](https://img.myhappiness.top/img/image-20211206113514809.png)![image-20211206113540614](https://img.myhappiness.top/img/image-20211206113540614.png)

**测试终端登录**

密码为刚重置的密码

```
ssh admin@192.168.3.9 -p2222
```

![image-20211205165826368](https://img.myhappiness.top/img/image-20211205165826368.png)

# 配置自动载入Python3虚拟环境

**配置自动载入python3虚拟环境**

```
[root@kvm ~]# git clone git://github.com/inishchith/autoenv.git ~/.autoenv
[root@kvm ~]# echo 'source ~/.autoenv/activate.sh' >> ~/.bashrc
[root@kvm ~]# source ~/.bashrc
[root@kvm ~]# echo "source /opt/py3/bin/activate" > /opt/.env
```

**测试使用**

第一次进入需要输入y进行确认

```
[root@kvm ~]# cd /opt/
autoenv:
autoenv: WARNING:
autoenv: This is the first time you are about to source /opt/.env:
autoenv:
autoenv:   --- (begin contents) ---------------------------------------
autoenv:     source /opt/py3/bin/activate$
autoenv:
autoenv:   --- (end contents) -----------------------------------------
autoenv:
autoenv: Are you sure you want to allow this? (y/N) y
(py3) [root@kvm opt]# 
```



