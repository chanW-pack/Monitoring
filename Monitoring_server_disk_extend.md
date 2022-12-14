### 2. 모니터링 서버 디스크 용량 확장

기존 (디스크1 /root:16G) > 변경 (디스크2 추가, /root:30G) 

16GB 디스크(sdb)를 추가 장착한 뒤, LVM extend로 용량을 증설하였다.

![Untitled 7](https://user-images.githubusercontent.com/84123877/207513281-147114c3-39c6-4b48-88db-3a57f464c3a3.png)


```bash
# LVM 생성
fdisk /dev/sdb

# PV 생성
pvcreate /dev/sdb1

# 기존 VG에 증설(추가)
vgextend ubuntu-vg /dev/sdb1

# lv 증설 (-r 옵션으로 파일시스템 rezise)
lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv

# 용량 확인
df -h | grep ubuntu
/dev/mapper/ubuntu--vg-ubuntu--lv   31G   14G   16G  47% /

기존 10G에서 31G로 증설되었다.
```

---
