**Monitoring in Inkscope with collectd and influxDB**

**CollectD installation**

```
# apt-get install collectd
```

**collectd-ceph installation**

```
# wget https://github.com/inkscope/collectd-ceph/archive/master.zip
# unzip master.zip
# mkdir -p /usr/lib/collectd/plugins/ceph
# cp -p  collectd-ceph-master/plugins/* /usr/lib/collectd/plugins/ceph
```

**在/etc/collectd/collectd.conf.d/目录下添加一个名为ceph\_plugins.conf配置文件，文件内容如下**

```
<LoadPlugin "python">
    Globals true
</LoadPlugin>

<Plugin "python">
    ModulePath "/usr/lib/collectd/plugins/ceph"
    #Import "getsigchld"  #uncomment for centos
    Import "ceph_latency_plugin"

    <Module "ceph_latency_plugin">
        Verbose "True"
        Cluster "ceph"
        Interval "60"
    </Module>

    Import "ceph_monitor_plugin"

    <Module "ceph_monitor_plugin">
        Verbose "True"
        Cluster "ceph"
        Interval "60"
    </Module>

    Import "ceph_osd_plugin"

    <Module "ceph_osd_plugin">
        Verbose "True"
        Cluster "ceph"
        Interval "60"
    </Module>

    Import "ceph_pg_plugin"

    <Module "ceph_pg_plugin">
        Verbose "True"
        Cluster "ceph"
        Interval "60"
    </Module>

    Import "ceph_pool_plugin"

    <Module "ceph_pool_plugin">
        Verbose "True"
        Cluster "ceph"
        Interval "60"
        TestPool "test"
    </Module>
</Plugin>
```

**InfluxDB installation**

```
# wget http://s3.amazonaws.com/influxdb/influxdb_0.11.0-1_amd64.deb
# dpkg -i influxdb_0.11.0-1_amd64.deb
# service influxdb start

```

**Connection between collectd - influxDB 0.11**

```
在/etc/influxdb/influxdb.conf，
加入
[collectd]
  enabled = true
  bind-address = ":8096"
  database = "collectd"
  typesdb = "/usr/share/collectd/types.db"

restart influxdb:
# service influxdb restart
```

**create collectd database in InfluxDB:**

```
/etc/collectd/collectd.conf
修改
Loadplugin network
.
.
.
<Plugin "network">
  Server "influxdb-host-name" "8096"
</Plugin>

# service collectd restart

log在/var/log/syslog
如果出现error，创建data pool
```

**inkscope configuration to use collectD/influxDB data**

```
修改
/opt/inkscope/etc/inkscope.conf
"influxdb_endpoint":"http://<influxdb-host-name>:<influxdb-port>",
# service apache2 restart
```



