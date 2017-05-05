1.启动monitor

```
sudo systemctl start ceph-mon.target
```

Pool管理

1、调整Pool副本数

```
ceph osd pool set {poolname} size {num-replicas}
```

移除OSD

```
1.登录到osd所在节点
ceph osd out osd.{osd-num}
sudo systemctl stop ceph-osd@{osd-num}

```



