# Monitoring
---
## CloudWatch
![cloudwatch-grafana](https://user-images.githubusercontent.com/84123877/207511350-84eaeda9-efe8-4bde-9a3c-c19358421d3d.png)
- [CloudWatch + Grafana 모니터링 구축](https://github.com/chanW-pack/Monitoring/blob/main/CloudWatch%20%2B%20Grafana%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EA%B5%AC%EC%B6%95.md) </br>
CloudWatch -> Grafana 연동, 대시보드 출력

- [Grafana Slack 연동](https://github.com/chanW-pack/Monitoring/blob/main/Grafana%20Slack%20%EC%97%B0%EB%8F%99.md) </br>
grafana Alerting Slack 연동(grafana 경보알람이 Slack 메세지로 발송)   
하나의 grafana server에 여러개의 대시보드와 Contact points를 생성하였더니 충돌하는 이슈 발생 </br>
11.08 - 해결 완료: (contect point에 Custom Labels group을 지정하여 충돌 문제 해결) </br>
- [Grafana Image Renderer](https://github.com/chanW-pack/Monitoring/blob/main/Grafana%20Image%20Renderer.md) </br>
slack에 보내지는 경고 메세지를 커스텀마이징 </br>
임계치를 넘은 순간의 그래프 이미지를 메세지로 같이 보내는 기능 조사(Image Rendering 기능) </br>

- [Grafana Alert Templet](https://github.com/chanW-pack/Monitoring/blob/main/Grafana%20Alert%20Templet.md)  
grafana alert -> slack 메세지 템플릿 구현  
추가로, firing/resolved 상태 구분하여 메세지 전송 (if 문)   
![grafana-slack](https://user-images.githubusercontent.com/84123877/207511925-59a98426-4110-4929-905b-518f38d33eb6.png)

## Zabbix
![zabbix](https://user-images.githubusercontent.com/84123877/200781041-805cb412-bde1-4c94-9db2-d3754466b292.png)
- [ZABBIX 자빅스 모니터링 시스템 구축하기](https://github.com/chanW-pack/Monitoring/blob/main/Zabbix_%20%EC%9E%90%EB%B9%85%EC%8A%A4%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0.md)

## Prometheus
![promethus](https://user-images.githubusercontent.com/84123877/200781352-02720582-ad6a-4c9a-ab5e-8f6d1bb03c08.png)
- [Prometheus + Grafana 모니터링 구축](https://github.com/chanW-pack/Monitoring/blob/main/Prometheus%20%2B%20Grafana%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EA%B5%AC%EC%B6%95.md) </br>
Prometheus 및 Grafana, docker 활용하여 container 이미지로 구축. 

## Scouter

- [Scouter 설치 및 Agent 연동](https://github.com/chanW-pack/Monitoring/blob/main/Linux%20Scouter%20%EC%84%A4%EC%B9%98%20%EB%B0%8F%20Agent%20%EC%97%B0%EB%8F%99.md)

---
# 모니터링 시스템 백업 서버 구축 프로젝트
---
- 이슈
    
    server1 서버 (zabbix, grafana) A-service 대시보드 연결 이슈 발생  
    > 이슈 1. A-service 관련 슬랙 경보 알람이 마구잡이로 발송됨  
                > A-service 본 서버 확인하니 , 서버에는 이상이 없음.  
       이슈 2. 하지만, B-service ,C-server 대시보드 모두 연결 이상 x  
                > 즉, 모니터링 서버 그라파나도 이상은 없는 상태,  
       이슈 3. 모니터링 서버 **자빅스 mysql 이슈**  
                > 프로세스 작동은 확인되나, zabbix web 접속 불가. 오류 코드는 **mysql 접속 불가**  
                > mysql 접속 불가로 확인하니, mysql은 이상 없으며 **서버 디스크 용량이 full** 상태였음.  
    
- 해결  
    
    즉, 모니터링시, 서버에 파일이 남게되는데 서버 디스크 용량이 가득 차, 더 이상 모니터링이 송신되지 않는 이슈였음.  
    가장 오래된 파일을 수동으로 삭제하여 용량 확보하니, 모니터링 시스템 전체 정상화.  
    
- 조사가 필요한 사항  
    
    용량이슈로, 언젠가는 발생할 이슈였음. 해당 용량 관리의 자동화 방법을 물색해야 함  
    

---

### 모니터링 서버 Zabbix, Grafana 용량 확보 방안 조사

1. 이슈 발생 시, 그 동안에 데이터는 저장되지 않음 > 예비용 grafana 서버 구축  
2. 모니터링 서버 디스크 용량 확장  
3. zabbix 의 경우 mysql에 데이터 저장됨. 저장 기간을 자유롭게 설정 가능하며, 백업도 지원  
 s3에 백업 가능하지만, 현재 esxi 서버 디스크용량이 많기 때문에 esxi 서버에 백업용 서버 생성 고민  
 = esxi에 백업 서버 생성 후, NFS 로 백업 db data 저장  
 
 - [예비 Grafana 서버 구축(Server1 > Server2 마이그레이션)](https://github.com/chanW-pack/Monitoring/blob/main/process_to_container_Migration.md)   
 기존 서버1 grafana는 프로세스로 서비스하고 있다. 이를 new grafana server에는 docker container로 마이그레이션 한다.  
 - [모니터링 서버 디스크 용량 확장](https://github.com/chanW-pack/Monitoring/blob/main/Monitoring_server_disk_extend.md)  
 단순한 LVM 용량 확장이다.
 - [zabbix Mysql binlog 관리](https://github.com/chanW-pack/Monitoring/blob/main/Zabbix%20Binlog%20Management.md)
 - [Monitoring Backup Server 구축](https://github.com/chanW-pack/Monitoring/blob/main/Monitoring_backup_server_set.md)  
 esxi에 zabbix, grafana DB data를 저장할 백업 서버를 구축한다.  
 ---

