monitor管理

```
1.启动monitor
sudo systemctl start ceph-mon.target
2.停止monitor
sudo systemctl stop ceph-mon.target
```

Pool管理

```
1.创建pool
ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] [crush-ruleset-name] [expected-num-objects]
2.删除pool
需要添加配置文件 mon_allow_pool_delete = true
ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
3.调整Pool副本数
ceph osd pool set {poolname} size {num-replicas}
```

移除OSD

```
1.登录到osd所在节点
ceph osd out osd.{osd-num}
sudo systemctl stop ceph-osd@{osd-num}
ceph osd crush remove {osd-name.osd-num}
ceph auth del osd.{osd-num}
ceph osd rm {osd-num}
```



