# iSCSI

```
参考
http://www.open-iscsi.org/
# Updates to Ceph tgt (iSCSI) support
http://ceph.com/dev-notes/updates-to-ceph-tgt-iscsi-support/
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

Details of the bs\_rbd backing-store driver, possible future work





