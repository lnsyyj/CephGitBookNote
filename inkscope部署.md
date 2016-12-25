Inkscope是管理和监控Ceph的项目。它依赖于ceph提供的API。inkscope-admiviz使用python flask框架，使用mongoDB来存储sysprobe和cephprobe采集上来的监控数据。前台页面的展示会调用ceph自身的ceph-rest-api服务，周期性轮训调用。

sysprobe：负责采集物理机的状态信息。

cephprobe：负责采集ceph集群的状态信息。

项目地址：

```
https://github.com/inkscope/inkscope
```

在inkscope的github页中有3个项目

```
https://github.com/inkscope
```

1.inkscope，主项目，也就是我们介绍的inkscope的源码。

2.inkscope-packaging，发布的deb包和rpm包，在这里可以找到老版本的inkscope包。

3.collectd-ceph，这个项目是新版inkscope中添加的采集模块，把数据采集到influxdb中，前台调用influxdb的rest API展示图表。这个后续系列我会再写一篇文章。

**一、架构图**

![](/assets/inkscope.png)

架构比较清晰的告诉了我们哪些ceph节点需要部署inkscope的哪些组件。

**二、部署**

下面是我测试的部署步骤，期间也遇到了一些坑，在下一小节会给大家说明，当然时间有些长了，这些坑产生的现象可能记忆的不太清晰了，哈哈。

当然这些是测试环境，举例也是想说明不同组件可以分开部署，如果是生产环境还需要根据自己的情况重新设计一下。

系统版本：Ubuntu14.04 server

ceph版本：jewel 10.2.3

inkscope版本：1.4.0.2

机器名ceph-01，192.168.18.105部署了ceph-mon、ceph-osd、rados gateway

机器名ceph-02，192.168.18.106部署了mongodb、apache

**ceph-mon 节点**

```
（1）添加inkscope源
vi /etc/apt/sources.list.d/inkscope.list
添加
官方源： deb https://raw.githubusercontent.com/inkscope/inkscope-packaging/master/DEBS ./
# apt-get update
（2）安装依赖包
# apt-get install python-pip  python-dev  sysstat  lshw
# apt-get install inkscope-common inkscope-sysprobe inkscope-cephrestapi inkscope-cephprobe
# pip install psutil==2.1.3
# pip install pymongo==2.6.3
（3）修改ceph配置文件
# vi /etc/ceph/ceph.conf
添加
[client.restapi]
log_file = /var/log/ceph/ceph-rest-api.log
keyring = /etc/ceph/ceph.client.admin.keyring
（4）启动ceph-rest-api服务
# nohup ceph-rest-api -n client.admin &
（5）修改inkscope配置文件
# vi /opt/inkscope/etc/inkscope.conf
    "ceph_rest_api": "192.168.18.105:5000",
    "ceph_rest_api_subfolder": "",
    "mongodb_host" : "192.168.18.106",
    "mongodb_user":"ceph",
    "mongodb_passwd":"ceph",
（6）启动probe监控程序
/etc/init.d/sysprobe start
/etc/init.d/cephprobe start
```

**ceph-osd 节点**

```
（1）添加inkscope源
vi /etc/apt/sources.list.d/inkscope.list
添加
官方源： deb https://raw.githubusercontent.com/inkscope/inkscope-packaging/master/DEBS ./
# apt-get update
（2）安装依赖包
# apt-get install python-pip  python-dev  sysstat  lshw
# apt-get install inkscope-common  inkscope-sysprobe
# pip install psutil==2.1.3
# pip install pymongo==2.6.3
（3）修改inkscope配置文件
# vi /opt/inkscope/etc/inkscope.conf
    "ceph_rest_api": "192.168.18.105:5000",
    "ceph_rest_api_subfolder": "",
    "mongodb_host" : "192.168.18.106",
    "mongodb_user":"ceph",
    "mongodb_passwd":"ceph",
（4）启动probe监控程序
/etc/init.d/sysprobe start
```

**mongoDB节点**

```
（1）加源、安装mongodb
# sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
# echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
# sudo apt-get update
# sudo apt-get install -y mongodb-org
# pip install pymongo==2.6.3
（2）修改mongodb端口
vi /etc/mongod.conf
修改
bindIp: 0.0.0.0
（3）创建mongodb账号
# mongo
> use admin
> db.createUser(
...   {
...     user: "ceph",
...     pwd: "ceph",
...     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
...   }
... )
（4）重启服务
# service mongod stop
# service mongod start
```

**inkscope前后台节点**

```
（1）添加inkscope源
vi /etc/apt/sources.list.d/inkscope.list
添加
官方源： deb https://raw.githubusercontent.com/inkscope/inkscope-packaging/master/DEBS ./
# apt-get update
（2）安装依赖包
# apt-get install inkscope-common   inkscope-admviz  apache2 
# pip install flask-login simple-json
# apt-get install libapache2-mod-wsgi
# pip install pymongo==2.6.3
（3）修改apache配置文件
# vi /etc/apache2/ports.conf
添加：
Listen 8080
# vi /etc/apache2/apache2.conf
添加：
ServerName ceph-01
# vi /etc/apache2/sites-available/inkScope.conf
修改： 
<VirtualHost *:8080>
    ServerName  ceph-01
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/inkscope
    <Directory "/var/www/inkscope">
        Options All
        AllowOverride All
    </Directory>
    ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
    <Directory "/usr/lib/cgi-bin">
        AllowOverride None
        Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
        Order allow,deny
        Allow from all
    </Directory>
    WSGIScriptAlias /inkscopeCtrl /var/www/inkscope/inkscopeCtrl/inkscopeCtrl.wsgi
    <Directory "/var/www/inkscope/inkscopeCtrl">
        Order allow,deny
        Allow from all
    </Directory>
    WSGIScriptAlias /ceph_rest_api /var/www/inkscope/inkscopeCtrl/ceph-rest-api.wsgi
    <Directory "/var/www/inkscope/inkscopeCtrl">
         Require all granted
    </Directory>
    # Possible values include: debug, info, notice, warn, error, crit,
    # alert, emerg.
    LogLevel warn
    ProxyRequests Off
    ProxyPass /ceph-rest-api/ http://192.168.18.105:5000/api/v0.1/
    ErrorLog /var/log/inkscope/webserver_error.log
    CustomLog /var/log/inkscope/webserver_access.log common
</VirtualHost>
（4）
# sudo a2enmod proxy_http
# sudo a2enmod rewrite
# sudo a2ensite inkScope
（5）
# vi /opt/inkscope/etc/inkscope.conf
    "ceph_rest_api": "192.168.18.105:5000",
    "ceph_rest_api_subfolder": "",
    "mongodb_host" : "192.168.18.106",
    "mongodb_user":"ceph",
    "mongodb_passwd":"ceph",
    "radosgw_url": "http://192.168.18.105:80",
    "radosgw_admin": "admin",
    "radosgw_key": "U78Y9WSV96QT9SOUT4S6",
    "radosgw_secret": "tgwkKsvMfdsw47mk1a96iMvFDMTga4xHkDmCZ34b"
（6）
# service apache2 restart
```

配置rados gateway时，需要手动创建一个账号，并赋予相应的caps。

```
radosgw-admin user create --uid=admin --display-name="Admin" --email=lnsyyj@hotmail.com
radosgw-admin caps add --uid=admin --caps="users=*"
radosgw-admin caps add --uid=admin --caps="buckets=*"
radosgw-admin caps add --uid=admin --caps="metadata=*"
radosgw-admin caps add --uid=admin --caps="usage=*"
radosgw-admin caps add --uid=admin --caps="zone=*"
```

**各节点全部部署完后**

打开浏览器，访问[http://192.168.18.106:8080/](http://192.168.18.106:8080/)

三、部署中遇到的问题

问题：

1、rbd前台页面报500，keyring权限问题，修改/etc/ceph/的keyring权限为644，chmod +r \*

2、inkscope的正则无法匹配机器名，导致前台无法正常显示物理机的信息。这个问题一般不会遇到，所以不需要修改，我们只是偶然遇到的。

mongo数据库中 "partition" : DBRef\("partitions", null\)

`vi /opt/inkscope/bin/cephprobe.py`

大概390行

`osddatapartition = db.partitions.find_one({"_id" : {'$regex' : osdhostid+"*:*"}, "mountpoint" : '/var/lib/ceph/osd/'+clusterName+'-'+str(osd["osd"])})`

3、ceph jewel版修改了mds，而inkscope项目没有更新，会导致前台主页面全部变灰，无法填充cluster health、placement groups

、osd等图表。

J版返回json没有msdmap了，注释掉statusApp.js 的

`$scope.mdsmap.up_standby = data.output.mdsmap["up:standby"];`

