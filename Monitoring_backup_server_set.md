### 3. Monitoring-backup server 구축

esxi에 DB data backup server를 구축한다.

```bash
서버 정보
CPU: 1 core
MEM: 2 GB
DISK: 16GB /sda
      200GB /sdb
      200GB /sdc

/sdb 디스크를 /cwNFS 디렉터리와 마운트시켜 NFS로 공유한다.

vi /etc/exports
/cwNFS *(rw,sync,no_root_squash)

exporfs -ar
exporfs -v
/cwNFS          
<world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)

showmount -e
Export list for mo-bg:
/cwNFS *

기본 NFS port는 2049이다.
```

```bash

# A-server NFS mount
mkdir /monitoring-bak
vi /etc/fstab
172.16.0.189:/cwNFS /monitoring-bak nfs defaults 0 0

mount -a
df -h | grep cwNFS
172.16.0.189:/cwNFS               205309952       0 194808064   0% /monitoring-bak

# B-server NFS mount
mkdir minitoring-bak2
vi /etc/fstab
172.16.0.189:/cwNFS /monitoring-bak2 nfs defaults 0 0

mount -a
df -h | grep cwNFS
172.16.0.189:/cwNFS               205309952        0 194808064   0% /monitoring-bak2
```

![Untitled 8](https://user-images.githubusercontent.com/84123877/207513286-08d263ff-6e48-4f49-b1fd-2480feec93ef.png)


200GB의 공유 디렉터리를 생성하였고, A/B server에서 crontab으로 DB 백업 데이터를 전송하면 된다.

---
