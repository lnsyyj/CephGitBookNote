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

2013年2月中旬以来，添加bs\_rbd path被接受到mainline repository，有非常简短的README信息来告诉你如何使用它

tgtd为了支持rbd必须使用CEPH\_RBD flag来构建，您可能需要从源代码自己构建它

