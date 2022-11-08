# Grafana Slack 연동

---

### Slack app 구축

---

![Untitled](https://user-images.githubusercontent.com/84123877/200572948-c5de12b9-fd04-4cf6-bc27-1e5b232405df.png)

![Untitled 1](https://user-images.githubusercontent.com/84123877/200572952-37b4b48f-dac5-4120-822c-9fedaf3b6c5e.png)

slack api에 접속하여, create new app을 진행한다.

![Untitled 2](https://user-images.githubusercontent.com/84123877/200572954-f9d3891c-8e1c-402e-9680-ed1845ea8a72.png)

from scartch을 선택한다. 이후 app name을 설정하고 연결할 slack workspace를 선택한다.

![Untitled 3](https://user-images.githubusercontent.com/84123877/200572955-25f60854-5b76-4527-aa02-68f06b3ed1f0.png)

좌측 Features 탭에서 OAuth & Permissions을 선택한다.

![2022-11-07_18_36_34](https://user-images.githubusercontent.com/84123877/200572950-975a9de9-f4e6-4ad7-b9a0-f110dc67888d.png)

설치를 진행한다. 관리자 권한이 필요하다.

![Untitled 4](https://user-images.githubusercontent.com/84123877/200572928-a3ec36b7-75e0-489a-bad0-6ab81287b10b.png)

이후 슬랙 api를 통해 메세지와 그래프이미지 등을 전송할 것이기 때문에, 

files:wirte , chat:wirte 권한을 추가한다.

![Untitled 5](https://user-images.githubusercontent.com/84123877/200572937-4ee2fb35-b656-4f8c-80eb-fb91f730087a.png)

이후, 생성된 토큰값을 복사하여 

grafana → alerting → contact point를 생성한다.

![Untitled 6](https://user-images.githubusercontent.com/84123877/200572940-49358c4d-54d9-4ba7-a031-8b62c09316ea.png)

생성된 contact point에 대한 라우팅 설정을 진행해야 하는데,

notification policles 에서 정책을 추가한다.

root policy의 경우 모든 알림에 대한 내용이 전달되고,

본인의 경우처럼 한 grafana 서버에 여러 모니터링 툴의 정보를 가져와서 나타낸다면,

**Specific routing** 에서 구분한 labels 별로 알람 전송을 제어할 수 있다.

 ****

![Untitled 7](https://user-images.githubusercontent.com/84123877/200572944-8065b1ec-8942-4cb1-af35-2f6e9fd2a2f8.png)

본인은 알람 대시보드를 group 틀로 나누어 name을 설정하였다.

이제 해당 group에 포함한 알람 내용만 설정한 contact point에 전송될 것이다.

 

![Untitled 8](https://user-images.githubusercontent.com/84123877/200572945-a7ea44f0-37fb-4ca4-a1b2-6d3f3f089fb8.png)

### 여러 서비스 slack 연결 충돌 이슈

---

현재 한 서버에 grafana를 구축하여, 여러 monitoring 툴에서 각 대시보드로 구성하여 운영중이다.

zabbix, clouewatch 등의 서비스를 이용중이였는데, 그라파나에서 한번에 관리하게 함으로서, 

알람 기능도 통합하여 그라파나 알람으로 진행하였다.

그러나 각 서비스의 notification policles에 대한 labels를 설정 없이 진행하여

(기본 값은 모든 알람 전송) 모든 알람이 충돌되어 한 point에만 집중되어 전송되었다.

(server 1, 2, 3가 각 서버의 slack으로 전송되는것이 아닌, server 1 slack으로 모든 알람이 전송되었다.)

이는 , notification policles 를 수정하여 대시보드 별 알람을 group labels로 묶어, 본인 point에 해당하는 group 만 가져오게 설정하여 해결하였다.

---
