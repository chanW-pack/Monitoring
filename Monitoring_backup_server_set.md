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

```bash
# 백업 디렉터리 생성
mkdir -p /monitoring-bak2/data_backup/db

# mysql DB 백업 스크립트 작성
vi /home/mysql_Backup/db_backup.sh
#! /bin/bash
DATE=$(date +%Y%m%d%H%M%S)
BACKUP_DIR=/monitoring-bak2/data_backup/db

mysqldump -u root -pghddlr3839!  --databases zabbix  > $BACKUP_DIR"backup_"$DATE.sql

# 전체 DB를 백업할 경우
mysqldump -u root -p디비패스워드 --all-databases > $BACKUP_DIR"backup_"$DATE.sql

find $BACKUP_DIR -ctime +7 -exec rm -f {} \;

# crontab 스케쥴링 설정
00 06 * * * /home/mysql_Backup/db_backup.sh
# 매일 새벽 6시에 백업 진행
```
백업이 정상적으로 진행되는지 테스트한다.  
직접 스크립트 실행  
![Untitled 10](https://user-images.githubusercontent.com/84123877/208339040-0711022c-c6fd-44f7-9e73-07b8e07c5767.png)  
crontab 스케쥴링 테스트  </br>
![Untitled 11](https://user-images.githubusercontent.com/84123877/208339043-b851a55d-b2db-47b0-b5f6-874f0d571f88.png)  
![Untitled 12](https://user-images.githubusercontent.com/84123877/208339044-bd0f6139-5da9-4be8-a761-8476f33b5898.png)  

---
