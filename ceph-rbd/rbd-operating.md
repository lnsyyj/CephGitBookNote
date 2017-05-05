向RBD中读写数据

KRBD

```
1. 加载RBD内核模块
modprobe rbd
2. 查看RBD模块信息
modinfo rbd
3. 创建一个5G的块设备
rbd create test-pool-1 --size 5120
```



