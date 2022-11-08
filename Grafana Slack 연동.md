# Grafana Slack 연동

---

### Slack app 구축

---

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled.png)

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%201.png)

slack api에 접속하여, create new app을 진행한다.

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%202.png)

from scartch을 선택한다. 이후 app name을 설정하고 연결할 slack workspace를 선택한다.

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%203.png)

좌측 Features 탭에서 OAuth & Permissions을 선택한다.

![2022-11-07 18 36 34.png](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/2022-11-07_18_36_34.png)

설치를 진행한다. 관리자 권한이 필요하다.

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%204.png)

이후 슬랙 api를 통해 메세지와 그래프이미지 등을 전송할 것이기 때문에, 

files:wirte , chat:wirte 권한을 추가한다.

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%205.png)

이후, 생성된 토큰값을 복사하여 

grafana → alerting → contact point를 생성한다.

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%206.png)

생성된 contact point에 대한 라우팅 설정을 진행해야 하는데,

notification policles 에서 정책을 추가한다.

root policy의 경우 모든 알림에 대한 내용이 전달되고,

본인의 경우처럼 한 grafana 서버에 여러 모니터링 툴의 정보를 가져와서 나타낸다면,

**Specific routing** 에서 구분한 labels 별로 알람 전송을 제어할 수 있다.

 ****

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%207.png)

본인은 알람 대시보드를 group 틀로 나누어 name을 설정하였다.

이제 해당 group에 포함한 알람 내용만 설정한 contact point에 전송될 것이다.

 

![Untitled](Grafana%20Slack%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%83%E1%85%A9%E1%86%BC%20f58db8c1ef1c446aaeed182eacdb7b9b/Untitled%208.png)

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