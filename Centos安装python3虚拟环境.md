## 源码安装Python3

安装依赖包

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

退出虚拟环境命令

```
(py3) [root@i-ic6pktm9 ~]# deactivate
[root@i-ic6pktm9 ~]# 
```

pythone3虚拟环境搭建完成

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

