# Centos部署Harbor

环境准备

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
[root@kvm ~]#  sudo tee /etc/docker/daemon.json <<-'EOF'
{
   "registry-mirrors": ["https://1me84w5g.mirror.aliyuncs.com"]
}
EOF
[root@kvm ~]# systemctl daemon-reload && systemctl restart docker
```

**安装docker composer**

可以在github上查找想要安装的版本，并替换URL中的1.29.2

```
[root@kvm ~]# curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
[root@kvm ~]# chmod +x /usr/local/bin/docker-compose
[root@kvm ~]# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

**下载harbor安装包**

此处为下载离线安装包为例子

```
[root@kvm ~]# wget https://github.com/goharbor/harbor/releases/download/v2.3.3/harbor-offline-installer-v2.3.3.tgz
[root@kvm ~]# tar -xvf harbor-offline-installer-v2.3.3.tgz
```

配置harbor （老版本解压后会有一个harbor.cfg文件配置，直接配置harbor.cfg文件即可）

```
[root@kvm ~]# cd harbor
[root@kvm harbor]# cp harbor.yml.tmpl harbor.yml
[root@kvm harbor]# vim harbor.yml
```

改：hosname为服务器ip，实验环境注释https协议13行到18行，只启用http协议，如需要指定访问的端口修改80为想要指定的端口即可

![image-20211130160424814](https://img.myhappiness.top/img/image-20211130160424814.png)

为：

![image-20211130161900707](https://img.myhappiness.top/img/image-20211130161900707.png)

修改完配置文件后在当前目录下执行安装部署命令

```
[root@kvm harbor]# ./install.sh
```

部署完成后使用docker ps查看相关服务

```
docker ps
```

![](https://img.myhappiness.top/img/image-20211130163336167.png)

然后在浏览器上，输入服务器ip地址，打开harbor登录界面。默认用户名是admin，密码就是在harbor.yml里配置的密码默认的为Harbor12345

![image-20211130163515429](https://img.myhappiness.top/img/image-20211130163515429.png)

![image-20211130163558745](https://img.myhappiness.top/img/image-20211130163558745.png)

至此harbor搭建完成

**停止与启动Harbor**

因为Harbor是基于docker-compose服务编排的，所以通过 docker-compose启动或者关闭Harbor

在Harbor目录下面可以通过执行以下命令，进行关闭和启动Harbor

```
docker-compose down #关闭harbor服务
docker-compose up -d #启动harbor服务
```

关闭harbor服务

![image-20211130164014633](https://img.myhappiness.top/img/image-20211130164014633.png)

开始harbor服务

![image-20211130164109032](https://img.myhappiness.top/img/image-20211130164109032.png)

**配置开机后自动启动harbor服务**

```
[root@kvm ~]# vim /etc/rc.local  #添加以下内容
/usr/local/bin/docker-compose -f /root/harbor/docker-compose.yml up -d
```

**删除harbor**

如需删除harbor则执行以下命令

```
docker stop `docker ps -qa` 停止所有容器
docker rm `docker ps -qa` 删除所有容器
docker rmi -f $(docker images -qa) #强制删除所有镜像
```

