KRBD

Linux ubuntu 4.4.0-75-generic

```
1、安装KVM虚拟化工具
root@ubuntu:~# apt-get install qemu-system-x86 qemu-kvm qemu-utils libvirt-bin

2、查看qemu是否支持rbd块存储
root@ubuntu:~# qemu-img --help | grep rbd
```

向RBD中读写数据

KRBD

```
1. 加载RBD内核模块
modprobe rbd
2. 查看RBD模块信息
modinfo rbd
3. 创建一个5G的块设备,有些操作系统kernel不支持format 2格式某些新特性，需要关掉
rbd create test-krbd --size 5120 --image-feature layering
4.查看创建的块设备
rbd list
5.查看RBD信息
rbd info test-krbd
6.
rbd map test-krbd

7.
8.
9.
10.
```



