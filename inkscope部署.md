Inkscope是管理和监控Ceph的项目。它依赖于ceph提供的API。使用python flask框架，使用mongoDB来存储sysprobe和cephprobe采集上来的监控数据。

sysprobe：负责采集物理机的状态信息。

cephprobe：负责采集ceph集群的状态信息，主要是调用ceph自身的ceph-rest-api服务。

项目地址：

```
https://github.com/inkscope/inkscope
```

**一、架构图**

![](/assets/inkscope.png)

**二、部署**

**ceph-mon 节点**

```
（1）
vi /etc/apt/sources.list.d/inkscope.list
添加
官方源： deb https://raw.githubusercontent.com/inkscope/inkscope-packaging/master/DEBS ./
# apt-get update
（2）
# apt-get install python-pip  python-dev  sysstat  lshw
# apt-get install inkscope-common inkscope-sysprobe inkscope-cephrestapi inkscope-cephprobe
# pip install psutil==2.1.3
# pip install pymongo==2.6.3
（3）
# vi /etc/ceph/ceph.conf
添加
[client.restapi]
log_file = /var/log/ceph/ceph-rest-api.log
keyring = /etc/ceph/ceph.client.admin.keyring
（4）
启动ceph-rest-api服务
# nohup ceph-rest-api -n client.admin &
（5）
# vi /opt/inkscope/etc/inkscope.conf
    "ceph_rest_api": "192.168.18.105:5000",
    "ceph_rest_api_subfolder": "",
    "mongodb_host" : "192.168.18.106",
    "mongodb_user":"ceph",
    "mongodb_passwd":"ceph",
（6）
/etc/init.d/sysprobe start
/etc/init.d/cephprobe start
```

**ceph-osd 节点**

```
vi /etc/apt/sources.list.d/inkscope.list
添加
官方源： deb https://raw.githubusercontent.com/inkscope/inkscope-packaging/master/DEBS ./
# apt-get update
（2）
# apt-get install python-pip  python-dev  sysstat  lshw
# apt-get install inkscope-common  inkscope-sysprobe
# pip install psutil==2.1.3
# pip install pymongo==2.6.3
（3）
# vi /opt/inkscope/etc/inkscope.conf
    "ceph_rest_api": "192.168.18.105:5000",
    "ceph_rest_api_subfolder": "",
    "mongodb_host" : "192.168.18.106",
    "mongodb_user":"ceph",
    "mongodb_passwd":"ceph",
（4）
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
vi /etc/apt/sources.list.d/inkscope.list
添加
官方源： deb https://raw.githubusercontent.com/inkscope/inkscope-packaging/master/DEBS ./
# apt-get update
（2）
# apt-get install inkscope-common   inkscope-admviz  apache2 
# pip install flask-login simple-json
# apt-get install libapache2-mod-wsgi
# pip install pymongo==2.6.3
（3）
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
    ProxyPass /ceph-rest-api/ http://192.168.18.101:5000/api/v0.1/
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

**各节点全部部署完后**

Browser访问[http://192.168.18.106:8080/](http://192.168.18.106:8080/)

