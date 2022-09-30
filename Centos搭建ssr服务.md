# Centos搭建ssr服务

安装依赖包及拉取自动化部署脚本

```
[root@VM-0-16-centos ~]# yum install wget -y
[root@VM-0-16-centos ~]# wget -N –no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh
[root@VM-0-16-centos ~]# chmod +x ssr.sh 
[root@VM-0-16-centos ~]# bash ssr.sh 
```

选择1安装ssr

![image-20211130101719714](D:/Typora/img/image-20211130101719714.png)

设置端口及密码此处以5000端口为例

![image-20211130101901605](D:/Typora/img/image-20211130101901605.png)

配置加密方式，此处选择10

![image-20211130102002338](D:/Typora/img/image-20211130102002338.png)

插件协议选择默认，回车就可以，兼容原版这里选择兼容y

![image-20211130102126651](D:/Typora/img/image-20211130102126651.png)

混淆选择默认，以下均为默认

![image-20211130102341226](D:/Typora/img/image-20211130102341226.png)

开始安装依赖，按y开始自动部署

![image-20211130102413553](D:/Typora/img/image-20211130102413553.png)

部署完成

![image-20211130103036652](D:/Typora/img/image-20211130103036652.png)

**加速vps服务器**

```
[root@VM-0-16-centos ~]# wget –no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
[root@VM-0-16-centos ~]# chmod +x bbr.sh 
[root@VM-0-16-centos ~]# bash bbr.sh 
```

回车，卡住就回车就好

![image-20211130103203900](D:/Typora/img/image-20211130103203900.png)

部署完成重启服务器

![image-20211130103638141](D:/Typora/img/image-20211130103638141.png)

```
[root@VM-0-16-centos ~]# reboot
```

查看服务状态

![image-20211130103815124](D:/Typora/img/image-20211130103815124.png)

关闭firewalld防火墙

```
[root@VM-0-16-centos ~]# systemctl stop firewalld
[root@VM-0-16-centos ~]# systemctl disable firewalld
```

至此搭建完成下载ssr客户端验证

客户端下载地址：https://github.com/shadowsocksrr/shadowsocksr-csharp/releases/download/4.9.0/ShadowsocksR-win-4.9.0.zip

配置服务器

服务器ip: 为云服务器的公网ip地址

连接端口：5000

密码：为上面配置ssr密码。并非为服务器ssh密码

加密方法：aes-256-cfb

协议：auth_sha1_v4

混淆参数：plain

![image-20211130104624298](D:/Typora/img/image-20211130104624298.png)

![image-20211130104004059](D:/Typora/img/image-20211130104004059.png)

访问google测试

![image-20211130104032550](D:/Typora/img/image-20211130104032550.png)