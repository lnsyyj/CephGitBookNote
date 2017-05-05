1.启动monitor

```
sudo systemctl start ceph-mon.target
```

Pool管理

1、调整Pool副本数

```
ceph osd pool set {poolname} size {num-replicas}
```



