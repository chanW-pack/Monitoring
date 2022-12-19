### Zabbix Binlog Management  
zabbix mysql binlog 관리를 진행한다.
![Untitled 8](https://user-images.githubusercontent.com/84123877/208335015-bd625973-ce00-4b0b-84f9-b585228102a9.png)
binlog(바이너리 로그)란 crate,drop과 같은 DDL 문, insert, delete와 같은 DML문을 통해 데이터의 변화가 발생할 경우 해당 이벤트들을 기록하는 로그 파일이다.    

mysql 내부에서 해당 명령을 통해 로그를 조회할 수 있다.  

```bash
# binlog 조회
mysql> show binary logs;
]+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000019 |  35384453 | No        |
| binlog.000020 |  40343089 | No        |
| binlog.000021 |  40367412 | No        |
| binlog.000022 |  40225781 | No        |
| binlog.000023 |  40124397 | No        |
| binlog.000024 |  39889005 | No        |
| binlog.000025 |  36267031 | No        |
| binlog.000026 |  37923004 | No        |
| binlog.000027 |  39885421 | No        |
| binlog.000028 |  39886883 | No        |
| binlog.000029 |  39939786 | No        |
| binlog.000030 |  39915514 | No        |
| binlog.000031 |  47161506 | No        |
| binlog.000032 |  49962260 | No        |
| binlog.000033 |  49937213 | No        |
| binlog.000034 |  49922017 | No        |
| binlog.000035 |  49964647 | No        |
| binlog.000036 |  49776693 | No        |
| binlog.000037 |  49741473 | No        |
| binlog.000038 |  49714634 | No        |
| binlog.000039 |  49697893 | No        |
| binlog.000040 |  50412279 | No        |
| binlog.000041 |  51224687 | No        |
| binlog.000042 |  51267841 | No        |
| binlog.000043 |  51280307 | No        |
| binlog.000044 |  59405384 | No        |
| binlog.000045 |   2833711 | No        |
| binlog.000046 |  50100776 | No        |
| binlog.000047 |  52637962 | No        |
| binlog.000048 |  52737484 | No        |
| binlog.000049 |  52734922 | No        |
| binlog.000050 |  52702013 | No        |
| binlog.000051 |   3413917 | No        |
+---------------+-----------+-----------+
33 rows in set (0.33 sec)

# binlog 삭제 (이전 데이터도 한번에 삭제)
mysql> purge master logs to 'binlog.000036';
Query OK, 0 rows affected (0.05 sec)

mysql> show binary logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000036 |  49776693 | No        |
| binlog.000037 |  49741473 | No        |
| binlog.000038 |  49714634 | No        |
| binlog.000039 |  49697893 | No        |
| binlog.000040 |  50412279 | No        |
| binlog.000041 |  51224687 | No        |
| binlog.000042 |  51267841 | No        |
| binlog.000043 |  51280307 | No        |
| binlog.000044 |  59405384 | No        |
| binlog.000045 |   2833711 | No        |
| binlog.000046 |  50100776 | No        |
| binlog.000047 |  52637962 | No        |
| binlog.000048 |  52737484 | No        |
| binlog.000049 |  52734922 | No        |
| binlog.000050 |  52702013 | No        |
| binlog.000051 |   3470476 | No        |
+---------------+-----------+-----------+
16 rows in set (0.00 sec)
```

하지만 매번 삭제하는 방법은 비효율적이다.  

따라서 보관 기간을 설정하여 해당 기간까지만 binlog가 남도록 설정한다.  

```bash
# binlog 보관 기간 조회
mysql > show global variables like 'binlog_expire_logs_seconds';
+----------------------------+---------+
| Variable_name              | Value   |
+----------------------------+---------+
| binlog_expire_logs_seconds | 2592000 |
+----------------------------+---------+
1 row in set (0.01 sec)

# binlog 보관 기간 설정 (30일 > 7일) 
mysql> set global binlog_expire_logs_seconds=604800;
mysql> show global variables like 'binlog_expire_logs_seconds';
+----------------------------+--------+
| Variable_name              | Value  |
+----------------------------+--------+
| binlog_expire_logs_seconds | 604800 |
+----------------------------+--------+
1 row in set (0.00 sec)

# 86,400초(1일) * 7일 = 259,200 초
```

- MySQL Version에 따라서 기존에는 expire_logs_days를 사용하는 부분도 있으나, MySQL 8.x Version부터는 binlog_expire_logs_seconds를 사용한다.  

---
