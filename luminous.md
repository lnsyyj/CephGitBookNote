CEPH V12.2.0 LUMINOUS中文release-notes

```
http://mp.weixin.qq.com/s/PyhpKoUneiNdtr9JggWXdw
```

参考文章

```
http://ceph.com/planet/ceph-luminous-%E6%96%B0%E5%8A%9F%E8%83%BD%E4%B9%8B%E5%86%85%E7%BD%AEdashboard/
```

ubuntu1604源安装

```
yujiang@ubuntu001:~$ lsb_release --all
No LSB modules are available.
Distributor ID:    Ubuntu
Description:    Ubuntu 16.04.1 LTS
Release:    16.04
Codename:    xenial

yujiang@ubuntu001:~$ wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
yujiang@ubuntu001:~$ echo deb https://download.ceph.com/debian/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
yujiang@ubuntu001:~$ echo deb https://download.ceph.com/debian-luminous/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
yujiang@ubuntu001:~$ sudo apt update
yujiang@ubuntu001:~$ sudo apt-get install ceph
yujiang@ubuntu001:~$ ceph -v
ceph version 12.2.0 (32ce2a3ae5239ee33d6150705cdb24d43bab910c) luminous (rc)

yujiang@ubuntu001:~/ceph-deploy$ ceph-deploy  new --public-network 192.168.30.134/24 --cluster-network 192.168.30.134/24 ubuntu001
yujiang@ubuntu001:~/ceph-deploy$ ceph-deploy mon create-initial
yujiang@ubuntu001:~/ceph-deploy$ ceph-deploy mon create

yujiang@ubuntu001:~$ lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0                        2:0    1    4K  0 disk 
sda                        8:0    0   60G  0 disk 
├─sda1                     8:1    0  487M  0 part /boot
├─sda2                     8:2    0    1K  0 part 
└─sda5                     8:5    0 59.5G  0 part 
  ├─ubuntu001--vg-root   252:0    0 57.5G  0 lvm  /
  └─ubuntu001--vg-swap_1 252:1    0    2G  0 lvm  [SWAP]
sdb                        8:16   0   15G  0 disk 
sdc                        8:32   0   15G  0 disk 
sr0                       11:0    1 1024M  0 rom
yujiang@ubuntu001:~/ceph-deploy$ ceph-deploy osd prepare ubuntu001:/dev/sdb ubuntu001:/dev/sdc
yujiang@ubuntu001:~/ceph-deploy$ sudo cp *.keyring /etc/ceph/

L版默认不创建rbd pool，手动创建一个
yujiang@ubuntu001:~$ sudo ceph osd pool create rbd 128
yujiang@ubuntu001:~$ sudo ceph osd pool set rbd size 1
yujiang@ubuntu001:~$ sudo ceph -s
  cluster:
    id:     928739d7-df46-43c8-8488-2bd35bacedfb
    health: HEALTH_WARN
            no active mgr

  services:
    mon: 1 daemons, quorum ubuntu001
    mgr: no daemons active
    osd: 2 osds: 2 up, 2 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   0 kB used, 0 kB / 0 kB avail
    pgs:

yujiang@ubuntu001:~/ceph-deploy$ ceph-deploy mgr create ubuntu001:yujiangmgr
yujiang@ubuntu001:~/ceph-deploy$ sudo cp *.keyring /etc/ceph/
yujiang@ubuntu001:~/ceph-deploy$ ceph mgr module enable dashboard
yujiang@ubuntu001:~/ceph-deploy$ sudo ceph -s
  cluster:
    id:     928739d7-df46-43c8-8488-2bd35bacedfb
    health: HEALTH_OK

  services:
    mon: 1 daemons, quorum ubuntu001
    mgr: yujiangmgr(active)
    osd: 2 osds: 2 up, 2 in

  data:
    pools:   1 pools, 128 pgs
    objects: 0 objects, 0 bytes
    usage:   2110 MB used, 28407 MB / 30517 MB avail
    pgs:     128 active+clean
    
yujiang@ubuntu001:~/ceph-deploy$ sudo ceph osd tree
ID CLASS WEIGHT  TYPE NAME          STATUS REWEIGHT PRI-AFF 
-1       0.02917 root default                               
-3       0.02917     host ubuntu001                         
 0   hdd 0.01459         osd.0          up  1.00000 1.00000 
 1   hdd 0.01459         osd.1          up  1.00000 1.00000 
yujiang@ubuntu001:~/ceph-deploy$ sudo ceph df
GLOBAL:
    SIZE       AVAIL      RAW USED     %RAW USED 
    30517M     28407M        2110M          6.92 
POOLS:
    NAME     ID     USED     %USED     MAX AVAIL     OBJECTS 
    rbd      1         0         0        26881M           0 

```



