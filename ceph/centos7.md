1.安装git

```
yum install git
```

2.下载源码

```
git clone https://github.com/ceph/ceph.git
```

3.安装依赖（在master分支执行）

```
./install-deps.sh
```

4.查看branch和tag，并切换branch或tag

```
git branch -a
git tag -l
git checkout v12.0.0
```



