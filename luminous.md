CEPH V12.2.0 LUMINOUS中文release-notes

```
http://mp.weixin.qq.com/s/PyhpKoUneiNdtr9JggWXdw
```

ubuntu1604源安装

```
yujiang@ubuntu001:~$ lsb_release --all
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.1 LTS
Release:	16.04
Codename:	xenial

yujiang@ubuntu001:~$ wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
yujiang@ubuntu001:~$ echo deb https://download.ceph.com/debian/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
yujiang@ubuntu001:~$ echo deb https://download.ceph.com/debian-luminous/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
yujiang@ubuntu001:~$ sudo apt update
yujiang@ubuntu001:~$ sudo apt-get install ceph
yujiang@ubuntu001:~$ ceph -v
ceph version 12.2.0 (32ce2a3ae5239ee33d6150705cdb24d43bab910c) luminous (rc)

```



