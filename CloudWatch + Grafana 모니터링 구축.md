# CloudWatch + Grafana 모니터링 구축

---

### CloudWatch

(워치 링크)

본인이 학습한 cloudwatch 개념 정리된 링크이다.

현재 aws ec2에 서버가 있고, 해당 서버의 기본 리소스(cpu,disk R/W 등)를 grafana로 모니터링 및 slack 알람까지 구현하겠다.

```
# CloudWatch 기본 개념
기본적인 CPU, disk 읽기/쓰기, network 등은 권한만 있다면 cloudwatch에서 기본적으로 
모니터링이 가능하지만,
EC2는 OS 수준의 memory사용량, 디스크 사용량 등의 지표는 제공하지 않는다.

이는 cloudwatch Agent를 설치하여 해결할 수 있다.
(그 외에 온프레미스 서버에서도 cloudwatch agent 사용하여 가능)

cloudwatch 관련한 내용은 따로 학습하여 작성하겠다.
```

![Untitled](CloudWatch%20+%20Grafana%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%20fe8fe11cf25f41d9a458240aea53787d/Untitled.png)

현재 실습용 ec2 3대가 실행중인 상태이다.

cloudwatch 대시보드 모니터링 생성과 그라파나 모니터링 생성은 내용이 비슷하여, 

그라파나로 실습을 진행하도록 하겠다.

![Untitled](CloudWatch%20+%20Grafana%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%20fe8fe11cf25f41d9a458240aea53787d/Untitled%201.png)

aws 계정에 액세스 키를 사용하여 그라파나와 연동한다.

![Untitled](CloudWatch%20+%20Grafana%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%20fe8fe11cf25f41d9a458240aea53787d/Untitled%202.png)

본인 개인 서버에 grafana를 올려놨다.

Configuration → Add data source → cloudwatch 를 추가한다.

![Untitled](CloudWatch%20+%20Grafana%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%20fe8fe11cf25f41d9a458240aea53787d/Untitled%203.png)

![Untitled](CloudWatch%20+%20Grafana%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%20fe8fe11cf25f41d9a458240aea53787d/Untitled%204.png)

save & test 를 선택하여 데이터 소스 활성을 확인한다.

![Untitled](CloudWatch%20+%20Grafana%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%20fe8fe11cf25f41d9a458240aea53787d/Untitled%205.png)

이후 대시보드를 추가한다.

cloudwatch에서 기본적으로 가져오는 지표값을 입력하면, 바로 그래프화된다.

![Untitled](CloudWatch%20+%20Grafana%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%20fe8fe11cf25f41d9a458240aea53787d/Untitled%206.png)

대략적으로 완성되었다.

cpu, network 등의 지표를 불러와, grafana에서 띄어준다.

---