# 部署悟空crm-v11.3.3

**补充：v11.2.0版本默认用户名为：18888888888序列号：000000；新版本序列号则按照提示的步骤在个人中心进行复制粘贴。**

建议安装最新版本v11.3.3，目前发现v11.2.0部署后部分服务启动有问题

**拉取项目包**

```
[root@kvm ~]# cd /opt/
[root@kvm opt]# git clone --depth 1 https://gitee.com/wukongcrm/crm_pro.git
```

**安装docker及docker-composer**

**以下安装方式根据实际情况选择**

- **方式一**

**使用产品包脚本安装docker及composer**

此脚本会升级你的系统版本所以可忽略此步骤手动安装docker及docker-composer

```
[root@kvm opt]# cd crm_pro/docker/
[root@kvm docker]# chmod +x docker-install.sh 
[root@kvm docker]# ./docker-install.sh 
```

- **方式二**

**手动安装docker**

一，确保没有安装docker，使用以下命令移除docker

```
yum remove docker* -y
```

二，安装docker需要使用的依赖

```
yum install -y yum-utils device-mapper-persistent-data lvm2
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
systemctl start docker
```

```
systemctl enable docker
```

**手动安装docker composer**

可以在github上查找想要安装的版本，并替换URL中的1.29.2

```
[root@kvm ~]# curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
[root@kvm ~]# chmod +x /usr/local/bin/docker-compose
[root@kvm ~]# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

## 开始部署crm

```
[root@kvm docker]# chmod +x start.sh 
[root@kvm docker]# ./start.sh 
```

**待服务器启动完成后访问**

![image-20211210112113349](https://img.myhappiness.top/img/image-20211210112113349.png)



**使用账号和密码登录系统**

![image-20211210112146723](https://img.myhappiness.top/img/image-20211210112146723.png)

**进入系统查看**

![image-20211209143029131](https://img.myhappiness.top/img/image-20211209143029131.png)

