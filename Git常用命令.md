# Git常用命令

```
#提交代码到github上
echo "# Gitlab.rpm" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/wangyu1209/Gitlab.rpm.git
git push -u origin main


#远程删除仓库指定文件
git rm 文件名
git commit -m "删除某某文件"
git push -u origin main

完全卸载gitlab
gitlab-ctl stop
 rpm -e gitlab-ce
 ps -aux |grep gitlab
root     11948  0.0  0.0   4380   100 ?        Ss   21:06   0:00 runsvdir -P /opt/gitlab/service log: ...
kill -9 11948
find / -name gitlab |xargs rm -rf

github不允许上传100M文件解决方法
安装lfs:
yum install git-lfs
git lfs install 


Updated git hooks.
Git LFS initialized.
git lfs track “包名”
git add .gitattributes
 git add “包名”
git commit -m “”
git push -u origin master
```

