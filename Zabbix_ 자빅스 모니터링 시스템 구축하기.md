# [ZABBIX] 자빅스 모니터링 시스템 구축하기

---

## Zabbix 란?

---

Zabbix는 엔터프라이즈에 대응한 모니터링 솔루션이며, 오픈 소스로 배포된다.
다수의 네트워크 매개 변수 및 서버의 상태와 무결성을 모니터링하는 소프트웨어이며, 유연한 알림 메커니즘을 갖추고 있어 메일 기반의 통지를 하도록 설정할 수 있다.

즉, 이러한 기능을 통해 서버의 장애에 신속하게 대응할 수 있다.

<aside>
💡 Zabbix는 저장된 데이터를 바탕으로 보고서 및 그래프 표시 기능을 제공하며 폴링과 트래핑을 모두 지원함

</aside>

### 장점

- 강력한 모니터링 기능과 그래프를 하나의 도구로 결합시킴
- 알림에 대해 구성을 고도화할 수 있음
- 기본적으로 30초마다 지표를 수집하며, 인터벌 조정도 가능
- 빠른 웹 인터페이스
- 시스템에 대한 사용자 권한 설정으로 특정 사용자를 특정 뷰에 한정
- 수집된 데이터를 유연하지 못한 RRD 파일 대신 MySQL 등의 데이터베이스에 저장
- 데이터 저장 기간을 자유롭게 구성 가능 (DB 백업 기능 지원)
- 쉘 스크립트를 통해 알림을 쉽게 스크립팅

### 단점

- 알림 설정 부분에서 다량의 임계치 설정이 필요함
- 웹 인터페이스 기능이 과다하고 복잡함
- 아이템 당 하나의 값만을 리턴받을 수 있음
- 같은 종류의 개별 자산을 모니터링할 때 템플릿이 적용되지 않아 일일이 트리거 설정이 필요함

## 환경 구성

---

환경 구성은 Zabbix Server, Linux Client, Windows Client로 구성했다.

![Untitled](https://user-images.githubusercontent.com/84123877/178692043-cd2d9d55-43f7-4903-b20b-3a8b89696fa0.png)

> Server 설정이 완료된 후 Client들에게 Agent 설치를 진행하기 위해 AWS를 이용해서 총 3대로 구성하였다.
> 

![Untitled 1](https://user-images.githubusercontent.com/84123877/178692009-96ff0eb7-3eb0-4835-980e-b8494cbbf3c6.png)

## Zabbix Server 설치

---

Zabbix를 구성하기 위해서는 기본적인 설정이 필요하다.

- 방화벽 오픈 정보

```bash
apt install -y apache2 // 아파치 데몬 설치
service apache2 start // 서비스 시작
```

- Server

```bash
10050 inbound source CLIENT_IP
```

- Client

```bash


1. Apache 설치

```bash
inbound open 10051 zabbix server port
inbound open 10050 zabbix agent port
```

1. ntp 설치 및 활성화

```bash
apt install -y ntp // 서버 동기화를 위한 ntp 설치
vi /etc/ntp.conf // ntp.conf 수정
```

![Untitled 2](https://user-images.githubusercontent.com/84123877/178692012-a8cb8f69-8698-4861-a64d-c6d2033ba76a.png)

> 기존의 22~25 라인 주석 처리하고 26번 라인에 국내 서버인 time.bora.net을 추가 후 저장
> 

```bash
service ntp restart
ntpq -p // ntp 동작 확인
```

![Untitled 3](https://user-images.githubusercontent.com/84123877/178692016-b46624f0-a325-43f4-9c2d-51514ce67f41.png)

## Zabbix 설치

---

Zabbix 구성을 위한 기본 설정이 완료되었다.

이제부터는 Zabbix 설치를 진행하겠다.

```bash
$ sudo wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
$ sudo dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb

$ sudo apt update

$ sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

설치 후 Mysql을 설치한다.

```bash
$ sudo apt-get install mysql-server
$ sudo service mysql start
```

- Zabbix 관련 DB 설정

![Untitled 4](https://user-images.githubusercontent.com/84123877/178692019-acfc3f83-2f01-4c55-a94d-892bb9a3eee5.png)

![Untitled 5](https://user-images.githubusercontent.com/84123877/178692025-821b2444-e5f3-4473-af9d-b076ed94873e.png)

```sql
CREATE DATABASE zabbix CHARACTER SET utf8 collate utf8_bin; // utf 8 설정
GRANT ALL PRIVILEGES ON zabbix.* to zabbix@'localhost' IDENTIFIED BY '1234'; 
                      생성한 DB명   DB 유저                          패스워드
grant all privileges on zabbix.* to zabbix@localhost; // 설정 저장
quit;
```

```bash
$ sudo mysql -uroot -p

mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by '<설정할 password>';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

DB 스키마 셋팅

```bash
$ sudo zcat /usr/share/doc/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix

=> mysql password 입력
```

Zabbix config 셋팅

```bash
$ sudo vi /etc/zabbix/zabbix_server.conf

**- zabbix_server.conf 아래 내용 추가 (맨 밑..)**
DBPassword=<Mysql 설정한 패스워드>

$ sudo systemctl restart zabbix-server zabbix-agent apache2
$ sudo systemctl enable zabbix-server zabbix-agent apache2
```

<aside>
💡 접속주소

http://server_ip_or_name/zabbix

</aside>

![Untitled 6](https://user-images.githubusercontent.com/84123877/178692030-0c158d70-98cb-435c-88a9-8c3104ef4398.png)

## 모니터링 할 서버에 Agent 설치

---

```bash
$ sudo wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
$ sudo dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb

$ sudo apt update
$ sudo apt install zabbix-agent

$ sudo vi /etc/zabbix/zabbix_agentd.conf

- 아래 내용 추가
Server=172.0.0.1 <위에 Zabbix를 설치한 서버 주소>
ServerActive=172.0.0.1 <위에 Zabbix를 설치한 서버 주소>
HOSTNAME= <hostname 설정>

$ sudo service zabbix-agent restart
$ sudo systemctl enable zabbix-agent
```

> 여기까지만 셋팅하더라도 zabbix default 설정으로 서버의 기본적인 모니터링은 가능하다
> 

---

## Zabbix 한글화 방법

---

Zabbix 설치 후 처음엔 한글이 비활성화 되어있다. 아래의 설정 후 한글화 선택 가능하다

```bash
$ sudo locale-gen ko_KR.UTF-8
$ vi /etc/default/locale

- 아래 내용 추가
LANG=ko_KR.UTF-8
LANGUAGE="ko_KR:ko:en_US:en"
```

![Untitled 7](https://user-images.githubusercontent.com/84123877/178692034-7575bf65-fc30-4340-852d-e4e684501054.png)

![Untitled 8](https://user-images.githubusercontent.com/84123877/178692037-2f8946c2-21af-4158-a22b-6c6ec00017ec.png)

> ZBX 에 초록불이 들어오기까지 시간이 좀 걸린다…
> 

![Untitled 9](https://user-images.githubusercontent.com/84123877/178692039-dc30e5b7-dc4a-4baa-9913-c6dfaf59fe3c.png)

> 이렇게 메모리를 과부하시키니 바로 알림이 생성된다.
> 

다음은 zabbix를 Grafana와 연동을 진행해보겠다.

---
