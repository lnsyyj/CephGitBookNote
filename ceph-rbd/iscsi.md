# iSCSI

```
http://www.open-iscsi.org/

# Updates to Ceph tgt (iSCSI) support
http://ceph.com/dev-notes/updates-to-ceph-tgt-iscsi-support/
# tgt project
http://stgt.sourceforge.net/
https://github.com/fujita/tgt
```

#### Adding Support for RBD to stgt

```
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
```



