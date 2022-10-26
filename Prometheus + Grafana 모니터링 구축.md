# Prometheus + Grafana 모니터링 구축

---

### docker 설치 (ubuntu)

```bash
$ apt-get update -y

# 도커 설치 확인
$ docker ps

# 도커 설치
$ apt install docker.io -y

# 도커 실행
$ systemctl start docker
$ systemctl enable docker
```

### Prometheus 설치

[https://prometheus.io/download/](https://prometheus.io/download/) // 프로메테우스 설치 파일 공식 홈페이지

```bash
# 설치
$ sudo apt-get install -y prometheus prometheus-node-exporter prometheus-pushgateway
prometheus-alertmanager

# 설치 및 기동 확인
$ ps -ef | grep prometheus

---------------------------
# tar 파일의 경우
$ wget https://github.com/prometheus/prometheus/releases/download/v2.39.1/
prometheus-2.39.1.linux-amd64.tar.gz
$ tar zxvf prometheus-2.37.1.linux-amd64.tar.gz
$ cd /[tar 압축해제한 위치]
$ vi prometheus.yaml

# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate 'rules' every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "MS"
    static_configs:
      - targets: ['172.16.0.79:9100']

# promethus run
$ ./prometheus --config.file=prometheus.yml
```

실행 후 브라우저에서 접속 `http://localhost:9090`

### Node-exporter 설치

docker container 위에서 run한다.

```bash
# container image pull
$ docker pull bitnami/node-exporter

# docker background run
$ docker run -d --name exporter -p 9100:9100 bitnami/node-exporter

# docker container check
$ docker container ls
```

### 그라파나 설치

docker container 위에서 run한다.

```bash
# grafana container image pull and run
$ docker run -d --name=grafana -p 3000:3000 grafana/grafana

# 이후 프로메테우스 재시작
$ systemctl restart prometheus
```

### 프로메테우스 포트 변경

```bash
$ vim /etc/systemd/system/prometheus.service

Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=prometheus
Restart=on-failure

ExecStart=/home/prometheus/prometheus/prometheus \
  --config.file=/home/prometheus/prometheus/prometheus.yml \
  --storage.tsdb.path=/home/prometheus/prometheus/data
  --web.listen-address=:172.16.0.30:9999

[Install]
WantedBy=multi-user.target
```

기타/ 메트릭 make code

cpu : 

`100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`

`(1 - avg(irate(node_cpu_seconds_total{mode="idle"}[10m])) by (instance)) * 100`

mem:

`100 * (1 - ((avg_over_time(node_memory_MemFree_bytes[10m]) + avg_over_time(node_memory_Cached_bytes[10m]) + avg_over_time(node_memory_Buffers_bytes[10m])) / avg_over_time(node_memory_MemTotal_bytes[10m])))`

과부화 테스트

```bash
$ apt install epel-release
$ apt install stress

$ stress -c 4 --vm 3 --vm-bytes 2048m --hdd 2 --hdd-bytes 1024m --timeout 30s
```