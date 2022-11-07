# CloudWatch + Grafana 모니터링 구축

---

### CloudWatch

[cloudwatch](https://github.com/chanW-pack/aws_study/blob/main/Week%203/3_5%20CloudWatch%20(AWS%20%EB%A6%AC%EC%86%8C%EC%8A%A4%20%EC%83%81%ED%83%9C%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81).md)

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

![Untitled](https://user-images.githubusercontent.com/84123877/200257481-8366b994-115b-45ae-959f-4ba73568b704.png)

현재 실습용 ec2 3대가 실행중인 상태이다.

cloudwatch 대시보드 모니터링 생성과 그라파나 모니터링 생성은 내용이 비슷하여, 

그라파나로 실습을 진행하도록 하겠다.

![Untitled 1](https://user-images.githubusercontent.com/84123877/200257452-38261bde-aa08-4bfa-be78-fd020def851a.png)

aws 계정에 액세스 키를 사용하여 그라파나와 연동한다.

![Untitled 2](https://user-images.githubusercontent.com/84123877/200257460-83810fee-2750-4b40-bf1f-a26607d554bd.png)

본인 개인 서버에 grafana를 올려놨다.

Configuration → Add data source → cloudwatch 를 추가한다.

![Untitled 3](https://user-images.githubusercontent.com/84123877/200257467-07def6d0-c344-46d9-bca9-9c0a7f779e68.png)

![Untitled 4](https://user-images.githubusercontent.com/84123877/200257471-d8ded04b-3015-474a-9047-f7fc0099f5e3.png)

save & test 를 선택하여 데이터 소스 활성을 확인한다.

![Untitled 5](https://user-images.githubusercontent.com/84123877/200257474-75ecc799-0cab-447d-a216-ab1d767a8a4b.png)

이후 대시보드를 추가한다.

cloudwatch에서 기본적으로 가져오는 지표값을 입력하면, 바로 그래프화된다.

![Untitled 6](https://user-images.githubusercontent.com/84123877/200257477-6dcbc44d-fd9c-46a5-8cd2-c19ce41bd6ac.png)

대략적으로 완성되었다.

cpu, network 등의 지표를 불러와, grafana에서 띄어준다.

---
