### 1. 예비 Grafana 서버 구축 (Server1>ServerB 마이그레이션)

현재 `A-server-zabbix-server(172.16.0.67)` 에 grafana가 동작 중이며, 
예비 grafana 서버로 `B-service-zabbix-server(172.16.0.4)` 구축 진행

현재 해당 서버들은 zabbix만 실행중이므로, 리소스에 여유가 있음. 모니터링 서버를 통합함으로서 유지 관리에 유용함

![Untitled](%E1%84%87%E1%85%A9%E1%86%AB%E1%84%89%E1%85%A1%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%89%E1%85%B3%E1%84%90%E1%85%A6%E1%86%B7%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%2029f9574659c34441aebdd53006294b1f/Untitled.png)

기존 프로세스로 실행되던 grafana server를 예비 모니터링 서버에서는 container 로 동작하게 구축한다.

![Untitled](%E1%84%87%E1%85%A9%E1%86%AB%E1%84%89%E1%85%A1%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%89%E1%85%B3%E1%84%90%E1%85%A6%E1%86%B7%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%2029f9574659c34441aebdd53006294b1f/Untitled%201.png)

```bash
grafana의 경우 
# config file
/etc/grafana
# DB file
/var/lib/grafana

의 데이터를 백업하면 된다.

zabbix의 경우
mysql 
zabbix.history
zabbix.history_uint
zabbix.trends
zabbix.trends_uint
table을 mysql dump로 백업 가능하다.
```

![Untitled](%E1%84%87%E1%85%A9%E1%86%AB%E1%84%89%E1%85%A1%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%89%E1%85%B3%E1%84%90%E1%85%A6%E1%86%B7%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%2029f9574659c34441aebdd53006294b1f/Untitled%202.png)

해당 데이터 디렉터리들을 tar로 압축하고 예비 모니터링 서버로 복사(scp)한다.

예비 모니터링 서버(172.16.0.4)에서는 container로 grafana를 실행하는데, docker container mount 기능을 사용하여 해당 백업 데이터를 사용하게 만든다.

![Untitled](%E1%84%87%E1%85%A9%E1%86%AB%E1%84%89%E1%85%A1%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%89%E1%85%B3%E1%84%90%E1%85%A6%E1%86%B7%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%2029f9574659c34441aebdd53006294b1f/Untitled%203.png)

마치 심볼릭 링크처럼 컨테이너에서 원하는 저장 공간만을 공유해서 데이터를 읽고 사용할 수 있는 옵션을 제공한다.

볼륨 생성 방법과 바인드 마운트 방법이 있는데, 본인은 후자의 방법으로 진행하였다.

![Untitled](%E1%84%87%E1%85%A9%E1%86%AB%E1%84%89%E1%85%A1%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%89%E1%85%B3%E1%84%90%E1%85%A6%E1%86%B7%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%2029f9574659c34441aebdd53006294b1f/Untitled%204.png)

원본 서버에서 가져온 tar 파일을 압축 해제한다.  

/etc/grafana(config file) : grafana

/var/lib/grafana(DB file) : grafanaDB

해당 디렉터리들을 마운트 포인트로 사용하여, grafana를 container로 실행시킨다.

```bash
#docker 
docker run -d -v /root/grafana:/etc/grafana -v /root/grafanaDB:/var/lib/grafana -p 3333:3000 grafana/grafana:9.2.2-ubuntu
```

-v 옵션 : -volume 옵션으로도 사용할 수 있으며,   
형식은 -v [마운트 포인트] : [컨테이너 위치] 이다.

![Untitled](%E1%84%87%E1%85%A9%E1%86%AB%E1%84%89%E1%85%A1%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%89%E1%85%B3%E1%84%90%E1%85%A6%E1%86%B7%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%2029f9574659c34441aebdd53006294b1f/Untitled%205.png)

![Untitled](%E1%84%87%E1%85%A9%E1%86%AB%E1%84%89%E1%85%A1%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%89%E1%85%B3%E1%84%90%E1%85%A6%E1%86%B7%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%2029f9574659c34441aebdd53006294b1f/Untitled%206.png)

원본 서버와 동일한 내용의 grafana data가 그대로 구현되었다.

추가로 container로 동작하기 때문에 유지관리의 더욱 효과적이다.

---

### 2. 모니터링 서버 디스크 용량 확장

### 3. Monitoring-backup server 구축

```bash

#docker 
docker run -d -v /root/grafana:/etc/grafana -v /root/grafanaDB:/var/lib/grafana -p 3333:3000 grafana/grafana:9.2.2-ubuntu
```
