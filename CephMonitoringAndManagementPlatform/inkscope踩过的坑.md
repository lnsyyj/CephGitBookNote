问题：

1、rbd前台有问题报keyring权限问题，修改/etc/ceph/的keyring权限为644，chmod +r \*

2、mongo数据库中 "partition" : DBRef\("partitions", null\)

`vi /opt/inkscope/bin/cephprobe.py`

大概390行

`osddatapartition = db.partitions.find_one({"_id" : {'$regex' : osdhostid+"*:*"}, "mountpoint" : '/var/lib/ceph/osd/'+clusterName+'-'+str(osd["osd"])})`

3、J版返回json没有msdmap了，注释掉statusApp.js 的 

`$scope.mdsmap.up_standby = data.output.mdsmap["up:standby"];`

