问题：

1、rbd前台有问题报keyring权限问题，修改/etc/ceph/的keyring权限为644，chmod +r \*

2、链接错误，有一行代码，访问的url为//带有双斜杠了 （

ceph\_rest\_api\_subfolder

）

vi    /var/www/inkscope/inkscopeCtrl/poolsCtrl.py

class PoolsCtrl:

    def \_\_init\_\_\(self,conf\):

        \#self.cephRestApiUrl = "http://"+conf.get\("ceph\_rest\_api", ""\)+"/"+conf.get\("ceph\_rest\_api\_subfolder", ""\)+"/api/v0.1/"

        self.cephRestApiUrl = "http://"+conf.get\("ceph\_rest\_api", ""\)+"/api/v0.1/"

        pass

3、mongo数据库中      "partition" : DBRef\("partitions", null\)

vi /opt/inkscope/bin/cephprobe.py

大概390行

osddatapartition = db.partitions.find\_one\({"\_id" : {'$regex' : osdhostid+"\*:\*"}, "mountpoint" : '/var/lib/ceph/osd/'+clusterName+'-'+str\(osd\["osd"\]\)}\)

4、J版返回json没有msdmap了，

注释掉statusApp.js 的 $scope.mdsmap.up\_standby = data.output.mdsmap\["up:standby"\];

