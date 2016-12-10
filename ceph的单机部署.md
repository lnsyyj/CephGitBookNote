在刚接触Ceph或者经常使用Ceph做测试的时候，需要部署一个或多个Ceph集群。然而有些公司没有那么多服务器，只能在自己的虚拟机中搭建测试环境。这篇笔记就是记录怎样在单台虚拟机中搭建Ceph集群。

### 使用VMware部署单机版的Ceph

在这个例子中，虚拟机挂载三块硬盘，每块硬盘大小**不小于**10G，因为journal需要5G分区，如果小于等于5G第二个分区会建不出来。

系统是ubuntu-14.04.1-server-amd64

两块挂载网卡、三块硬盘

主机名叫ceph-01

1.安装ceph-deploy

```
sudo apt-get update && apt-get install python-pip && pip install ceph-deploy
```

2.修改hosts文件

```
vi /etc/hosts
添加
192.168.18.101    ceph-01
```

3.配置静态ip

```
# The primary network interface
auto eth0
iface eth0 inet static
address 192.168.18.101
netmask 255.255.255.0
gateway 192.168.18.2
dns-nameservers 192.168.18.2

auto eth1
iface eth1 inet static
address 192.168.180.201
netmask 255.255.255.0
gateway 192.168.18.2
dns-nameservers 192.168.18.2
```

4.做ssh互信

```
ssh-keygen
ssh-copy-id ceph-01
```

5.添加发布密钥

```
wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -
```

6.使用ceph-deploy安装Ceph（这里我使用的是国内的163源）

```
ceph-deploy install ceph-01 --release jewel --repo-url http://mirrors.163.com/ceph/debian-jewel/
```

7.创建一个目录（ceph-deploy会生成一些配置文件，方便管理和查看，所以ceph-deploy操作都在这个目录下执行）

```
mkdir ceph-deploy
cd ceph-deploy
```

8.使用ceph-deploy创建一个mon

```
ceph-deploy --cluster ceph001 new --public-network 192.168.18.0/24 --cluster-network 192.168.180.0/24 ceph-001
```

9.初始监控节点并收集密钥

```
ceph-deploy mon create-initial
```

10.创建monitor

```
ceph-deploy mon create
```

11.添加osd

```
ceph-deploy osd prepare ceph-01:/dev/sdb ceph-01:/dev/sdc ceph-01:/dev/sdd
```

12.这时用ceph命令就可以看到集群运行了，但是ERROR状态。原因是Ceph默认3副本，隔离域是host，默认会创建一个rbd的pool。

```
ceph -s
```

13.修改pool的副本数为1



