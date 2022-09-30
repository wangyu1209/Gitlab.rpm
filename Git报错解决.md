Git操作push时报错

```
git push -u origin main
```

![image-20220930171558527](https://img.myhappiness.top/img/image-20220930171558527.png)

问题分析

需要将远程仓库和本地进行合并

使用命令进行合并

```
git pull --rebase origin main
```

然后进行push成功

![image-20220930171908863](https://img.myhappiness.top/img/image-20220930171908863.png)