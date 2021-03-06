# iSCSI

```
博客
http://www.sysnote.org/2017/04/21/three-iscsitargets-io-flow/
http://blog.csdn.net/vah101/article/details/6300197
参考
http://www.open-iscsi.org/
# tgt project
http://stgt.sourceforge.net/
https://github.com/fujita/tgt
```

#### Adding Support for RBD to stgt

```
原文
http://ceph.com/dev-notes/adding-support-for-rbd-to-stgt/
```

可以使用RADOS block device \(rbd\) image作为iSCSI target device的存储后端

##### The bs\_rbd backing-store driver

2013年2月中旬以来，添加bs\_rbd path被接受到mainline repository，有非常简短的README信息来告诉你如何使用它

tgtd为了支持rbd必须使用CEPH\_RBD flag来构建，您可能需要从源代码自己构建它

```
您可以使用命令检查是否存在支持
tgtadm --lld iscsi --mode system --op show
tgtd由tgtadm命令配置，需要选择一个RBD image作为tgtd instance的后端存储，您可以使用：
--bstype <type>，type指定rbd选项来告诉tgtd应该使用bs_rbd访问存储
--backing-store <path>，选项以常用的Ceph语法选择rbd image，例如：--backing-store [pool/]image[@snap]来选择一个名为"image"的rbd image，pool可选，snapshot可选
您可以使用rbd command-line tool创建image
你必须给device创建一个名字，典型的名称格式为：iqn.<year>-<month>.<domain>:<domain-specified-string>，这个格式不是必须的，可根据自己的习惯
```

##### Using bs\_rbd with tgtd

使用手动命令：

```
1、首先，在Ceph集群中创建一个image（一个500MB的image，名称为iscsi-image）
rbd create iscsi-image --size 500
tgtadm或tgtd将通过默认的Ceph配置文件(/etc/ceph/$cluster.conf,$cluster默认为ceph)提供的配置访问Ceph集群，或通过CEPH_CONF environment variable，确保您的配置可通过其中一个设置访问
2、接下来，为tgtd守护进程创建一个新的target来模拟（截至2013年7月16日，由于不清楚的原因，你不能使用tid 0）
tgtadm --lld iscsi --mode target --op new --tid 1 --targetname rbd
3、创建一个LUN在这个target上并且绑定到rbd image
tgtadm --lld iscsi --mode logicalunit --op new --tid 1 --lun 1 --backing-store iscsi-image --bstype rbd
4、允许访问LUN
tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL
5、验证image可以被本地iscsi initiator看到
iscsiadm -m discovery -t st -p localhost
6、登录节点，这将创建一个/dev/sdX块设备
iscsiadm -m node --login
7、现在您可以使用iSCSI访问本地/dev/sdX设备了
您可以从不同的网络主机执行5和6步, 只需要修改 -p <tgtd-hostname> 参数
8、完成后，您可以终止会话并删除device
iscsiadm -m node --logout
```

##### Details of the bs\_rbd backing-store driver, possible future work

driver与librbd和librados相连，必须安装librbd和librados库，必须选择配置选项CEPH\_RBD

```
例如：
make CEPH_RBD=1
```

#### Updates to Ceph tgt \(iSCSI\) support

```
原文
http://ceph.com/dev-notes/updates-to-ceph-tgt-iscsi-support/
```

tgt-admin可以与rbd后端bs\_rbd一起使用

tgt-admin用于从target-configuration file设置tgtd，通常在引导时使用，所以这使得在主机上映射持久性targets 更为方便

tgtadm 接受一个新的参数--bsopts &lt;bs options&gt;为每个映射的image去设置bs\_rbd 选项

```
conf=<path-to-ceph.conf> 允许您为每个image引用不同的ceph集群（每个image都有自己的集群连接）
id=<client-id> 允许每个image使用不同的ceph 客户端id，这允许为每个image配置不同的客户端（包括permissions、log settings、rbd cache settings等），一般完整的客户端名字是"client.<client-id>"(默认id是admin,默认的客户端名称为"client.admin")
例如：
tgtadm --lld iscsi --mode logicalunit --op new --tid 1 --lun 1 --bstype rbd --backing-store public-image --bsopts "conf=/etc/ceph/pubcluster.conf;id=public"
你可以在"pubcluster"集群为名字是"public-image"的image创建target，ceph auth list中的名字是"client.public"
```

您可以从以下网站获取支持Ceph rbd的Debian和RPM 包

```
http://ceph.com/packages/ceph-extras
```

#### Ubuntu1604测试tgt iscsi

```
1.源安装
root@ubuntu:~# apt-get install tgt
root@ubuntu:~# apt-get install tgt-rbd 
root@ubuntu1604:~# tgtadm --help
Linux SCSI Target administration utility, version 1.0.63
```

实验步骤：

```
1、创建两个测试pool
root@ubuntu:~# ceph osd pool create iscsi-test-pool-1 64 64
root@ubuntu:~# ceph osd pool create iscsi-test-pool-2 64 64
2、在iscsi-test-pool-1这个pool中创建两个rbd image
root@ubuntu:~# rbd create iscsi-test-pool-1/iscsi-rbd-image-1 --size 5
root@ubuntu:~# rbd create iscsi-test-pool-1/iscsi-rbd-image-2 --size 5
3、创建一个target
root@ubuntu:~# tgtadm --lld iscsi --mode target --op new --tid 1 --targetname iscsi-rbd-target-1
4、检查是否支持rbd，不支持安装tgt-rbd
root@ubuntu:~# tgtadm --lld iscsi --mode system --op show
5、查询target信息
root@ubuntu:~# tgtadm --lld iscsi --mode target --op show
6、创建一个LUN(logicalunit)在这个target上并且绑定到rbd image
root@ubuntu:~# tgtadm --lld iscsi --mode logicalunit --op new --tid 1 --lun 1 --backing-store iscsi-test-pool-1/iscsi-rbd-image-1 --bstype rbd
7、允许访问LUN
root@ubuntu:~# tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL
8、验证image可以被本地iscsi initiator看到
root@ubuntu:~# iscsiadm -m discovery -t st -p localhost
9、登录节点，这将创建一个/dev/sdX块设备
root@ubuntu:~# iscsiadm -m node --login
10、这时，在ubuntu下多了一块sdc
root@ubuntu:~# ll /dev/sd
sda   sda1  sda2  sda5  sdb   sdb1  sdb2  sdc
11、
mkfs.ext4 /dev/sdh
12、
mount /dev/sdh /mnt/yujiang/
13、
umount /mnt/yujiang
14、logout
iscsiadm --mode node --targetname target-test --portal 192.168.1.200:3260 --logout
```

多HOST实验步骤：

```
参考文章：http://www.bkjia.com/yjs/1018700.html

server node (192.168.30.128)
1.添加源并安装tgt
root@ubuntu:~# echo "deb http://ceph.com/packages/ceph-extras/debian $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/ceph-extras.list
root@ubuntu:~# apt-get update 
root@ubuntu:~# apt-get install tgt
root@ubuntu:~# apt-get install tgt-rbd
2.确认是否支持rbd
root@ubuntu:~# tgtadm --lld iscsi --op show --mode system | grep rbd
    rbd (bsoflags sync:direct)
3.创建rbd并确认是否创建成功
root@ubuntu:~# rbd create rbd/test_image --size 1024 --image-format 2
root@ubuntu:~# rados lspools 
rbd
root@ubuntu:~# rbd ls rbd
image-0
image-1
test_image
4.在tgt服务中注册刚才创建好的image
root@ubuntu:~# cat /etc/tgt/targets.conf 
# Empty targets configuration file -- please see the package
# documentation directory for an example.
#
# You can drop individual config snippets into /etc/tgt/conf.d
include /etc/tgt/conf.d/*.conf

root@ubuntu:~# vi /etc/tgt/conf.d/ceph.conf
添加
<target iqn.2014-04.rbdstore.example.com:iscsi>
    driver iscsi
    bs-type rbd
    backing-store rbd/test_image        # Format is <iscsi-pool>/<iscsi-rbd-image>
    initiator-address 192.168.30.129    #client address allowed to map the address
</target>
5.重新加载或重新启动tgt服务
root@ubuntu:~# service tgt reload
root@ubuntu:~# service tgt restart
6.关闭rbd cache
root@ubuntu:~# vim /etc/ceph/ceph.conf
[client]
rbd_cache = false


linux client node (192.168.30.129)
1.安装open-iscsi
root@ubuntu1604fio:~# apt-get install  open-iscsi
2.重启open-iscsi服务
root@ubuntu1604fio:~# service open-iscsi restart
3.发现目标设备
root@ubuntu1604fio:~# iscsiadm -m discovery -t st -p 192.168.30.128
192.168.30.128:3260,1 iqn.2014-04.rbdstore.example.com:iscsi
4.挂载目标设备
root@ubuntu1604fio:~# iscsiadm -m node --login
Logging in to [iface: default, target: iqn.2014-04.rbdstore.example.com:iscsi, portal: 192.168.30.128,3260] (multiple)
Login to [iface: default, target: iqn.2014-04.rbdstore.example.com:iscsi, portal: 192.168.30.128,3260] successful.
5.确认目标设备已挂载（sdc）
root@ubuntu1604fio:~# lsblk
NAME                         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0                            2:0    1    4K  0 disk 
sda                            8:0    0   20G  0 disk 
├─sda1                         8:1    0  487M  0 part /boot
├─sda2                         8:2    0    1K  0 part 
└─sda5                         8:5    0 19.5G  0 part 
  ├─ubuntu1604fio--vg-root   252:0    0 17.5G  0 lvm  /
  └─ubuntu1604fio--vg-swap_1 252:1    0    2G  0 lvm  [SWAP]
sdb                            8:16   0   10G  0 disk 
├─sdb1                         8:17   0    5G  0 part /var/lib/ceph/osd/ceph-1
└─sdb2                         8:18   0    5G  0 part 
sdc                            8:32   0    1G  0 disk 
sr0                           11:0    1  667M  0 rom


windows client node
http://www.jb51.net/os/windows/89053.html
http://blog.csdn.net/bearcatfly/article/details/70242369
http://www.xiazaijidi.com/jc/3072.html
http://blog.csdn.net/mj404/article/details/42321291
开启Windows的Virtula Disk服务
```



