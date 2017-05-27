参照《Ceph分布式存储实战》书籍

测试环境

Ubuntu1604

```
1.安装包
root@ubuntu:~# apt-get install qemu-system-x86 qemu-kvm qemu-utils libvirt-bin
2.是否支持rbd
root@ubuntu:~# qemu-img --help | grep rbd
3.复制ceph.conf和keyring到kvm节点


```



