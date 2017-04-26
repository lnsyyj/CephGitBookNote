KRBD

Linux ubuntu 4.4.0-75-generic

```
1、安装KVM虚拟化工具
root@ubuntu:~# apt-get install qemu-system-x86 qemu-kvm qemu-utils libvirt-bin

2、查看qemu是否支持rbd块存储
root@ubuntu:~# qemu-img --help | grep rbd


```



