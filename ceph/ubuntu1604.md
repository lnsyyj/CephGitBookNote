# 一、GitHub下载源码方式

1.安装git

`apt-get install git`

2.下载源码

```
git clone https://github.com/ceph/ceph.git
git submodule update --init --recursive
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

5.生成Makefile

```
./run-make-check.sh
./do_cmake.sh
```

# **二、下载tarballs方式**

```
wget https://download.ceph.com/tarballs/ceph-12.0.0.tar.gz
tar zxvf ceph-12.0.0.tar.gz
./run-make-check.sh
./do_cmake.sh -j 4
cd build/
make -j 4
```



