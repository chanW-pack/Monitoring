# Grafana Image Renderer

---

그라파나에서 대시보드 패널이미지를 포함시키기 위해선 그라파나 이미지 랜더러가 필요하다.

### 그라파나 이미지 렌더러(Grafana Image Renderer)

그래프 이미지화를 위해 해당 플로그린을 설치하거나 독립 실행형으로 배포할 수 있다.

이미지 렌더러 문서(링크) 에서 자세한 설치방법을 확인할 수 있다.

### 1. **그라파나 cli를 사용한 설치**

현재 그라파나를 직접 설치해서 사용하고 있다. 이 같은 경우 cli를 통한 설치가 가능하다.

설치과정에서 크로미움 관련 라이브러리 종속성이 있다.

설치 후, 그라파나 재시작이 필요하기 때문에 docker로 돌리고 있다면 이후 설명하는 독립실행형으로 설치 또는 이미지 렌더러를 포함한 이미지를 사용해야 한다.

```bash
$ grafana-cli plugins install grafana-image-renderer
```

![Untitled](https://user-images.githubusercontent.com/84123877/200779834-78329b6a-92dc-4ed5-8bf4-75bc5d764984.png)

### 2. 독립실행형으로 설치

환경에 따라 설정 값을 다르게 해줘야 한다.

- **로컬환경에서 도커를 사용하는 경우**

```bash
#그라파나 이미지 렌더러 실행
docker run -d \
-p 8081:8081 \
--name=grafana-image-renderer \
-e "IGNORE_HTTPS_ERRORS=true" \
grafana/grafana-image-renderer
```

그라파나 실행

```bash
docker run -d \
-p 3000:3000 \
--name=grafana \
-e "GF_RENDERING_CALLBACK_URL: http://localhost:3000/" \
-e "GF_RENDERING_SERVER_URL: http://localhost:8081/render" \
grafana/grafana
```

- **쿠버네티스 그라파나 오퍼레이터의 경우**

그라파나 오퍼레이터에 사이드카러 이미지렌더러를 추가한다.

`사이드카의 경우 그라파나 디플로이먼트가 늘어날때마다 같이 늘어나기 때문에 한 개의 디플로이먼트일때만 추천`

같은 팟에 있기 때문에 localhost로 통신이 가능하며 그라파나는 3000포트 그라파나 이미지 렌더러는 8081이 기본값이다.

```bash
---
apiVersion: v1
data:
  GF_RENDERING_CALLBACK_URL: http://localhost:3000/
  GF_RENDERING_SERVER_URL: http://localhost:8081/render
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: grafana-image-renderer
---
apiVersion: integreatly.org/v1alpha1
kind: Grafana
metadata:
  name: grafana
spec:
  deployment:
    envFrom:
    - configMapRef:
        name: grafana-image-renderer
  containers:
  - name: image-renderer
    image: grafana/grafana-image-renderer
    env:
    - name: IGNORE_HTTPS_ERRORS
      value: "true"
  #... 이하 생략
  ...
```

- **쿠버네티스 그라파나의 경우**

아래와 같이 사이드카를 사용하여 로컬호스트 통신이 가능하다.

단순 테스트용이나 작은 서비스가 아닐 경우 그라파나와 이미지 렌더러를 따로 띄워줘야한다.

grafana-image-renderer의 서비스 주소를 `GF_RENDERING_SERVER_URL`에 넣어주고 
grafana 서비스의 주소를 `GF_RENDERING_CALLBACK_URL`에 넣어준다.

서비스의 주소는 `{ServiceName}.{Namespace}.svc.cluster.local` 양식을 가진다.

사이드카 사용시

```bash
---
apiVersion: v1
data:
  GF_RENDERING_CALLBACK_URL: http://localhost:3000/
  GF_RENDERING_SERVER_URL: http://localhost:8081/render
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: grafana-image-renderer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      name: grafana
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        envFrom:
        - configMapRef:
          name: grafana-image-renderer
        #이하 설정 생략 ...
      - name: grafana-image-renderer
        image: grafana/grafana-image-renderer:latest
        env:
        - name: IGNORE_HTTPS_ERRORS
          value: "true"
```

각각 디플로이먼트 사용시

```bash
---
apiVersion: v1
data:
  GF_RENDERING_CALLBACK_URL: http://grafana-service.grafana.svc:3000/
  GF_RENDERING_SERVER_URL: http://grafana-image-renderer.grafana.svc:8081/render
kind: ConfigMap
metadata:
  name: grafana-image-renderer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      name: grafana
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        envFrom:
        - configMapRef:
          name: grafana-image-renderer
        #이하 설정 pvc grafana server config 등 생략 ...
---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: grafana-image-renderer
  name: grafana-image-renderer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana-image-renderer
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: grafana-image-renderer
    spec:
      containers:
      - image: grafana/grafana-image-renderer
        name: grafana-image-renderer
        env:
        - name: IGNORE_HTTPS_ERRORS
          value: "true"
        resources: {}
        ports:
        - containerPort: 8081
          name: http
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  ports:
  - name: grafana
    port: 3000
    protocol: TCP
    targetPort: grafana-http
  selector:
    app: grafana
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-image-renderer
spec:
  ports:
  - name: grafana-image-renderer
    port: 8081
    protocol: TCP
    targetPort: http
  selector:
    app: grafana-image-renderer
  sessionAffinity: None
  type: ClusterIP
```

---

## 이미지 렌더러 설치 검증 및 슬랙 채널 추가

---

그라파나 7.1.1 기준으로,

1. 그라파나 Alerting → Notification channerls → add channel
2. 이미지처럼 입력

![Untitled 1](https://user-images.githubusercontent.com/84123877/200779827-f79b09fa-b92c-412d-a310-73da9c03d432.png)

1. Name: 자유롭게 입력Type: SlackDefault: true로 키게되면 알럿할 채널의 기본값이 된다.Include Image: 이미지 전송을 위해 켜야한다.경고문구 없이 켜져야 렌더러 설치와 설정이 잘된것이다.Slack SettingUrl: https://slack.com/api/chat.postMessageRecipient: #grafana-alertToken: 슬랙설정에서 발급받은 토큰 입력
2. Send Test로 메세지가 오는지 확인한다.

메세지가 슬랙으로 전송된다면 세이브를 눌러서 추가시켜준다.

---

### 문제 해결

1. 해당 기능은 현재(2022.11.09) 그라파나 버전에 맞지 않는다. 현재 grafana ver.9 에서는 이미지 전송 옵션이 존재하지 않는다. (추후 추가 예정이라함)
2. 다른 방법으로는 캡처한 이미지를 S3, google drive 등 저장소에 저장 후 메세지 발송시 불러오는 방법이 있다.

---
