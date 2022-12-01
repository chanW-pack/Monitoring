# Grafana Alert Templet

---

![Untitled](Grafana%20Alert%20Templet%20094fb6e0924d4d7bb6a079171f636b9f/Untitled.png)

기존 기본 메시지 형식은 가독성이 매우 나빴다.

이를 개선하기 위해 두 가지의 방법을 생각했는데,

1. Slack api message templet 수정
2. Grafana alert templet 수정

중, 두 번째의 방법인 grafana alert 수정 방식으로 진행해보았다.

![Untitled](Grafana%20Alert%20Templet%20094fb6e0924d4d7bb6a079171f636b9f/Untitled%201.png)

기본적은 템플릿으로는 , 크게 Value, Labels, Annotations 로 나뉘게되는데

각각 이름 그대로 정보 값, 해당 알림의 라벨, 주석 정도로 생각하면 된다.

![Untitled](Grafana%20Alert%20Templet%20094fb6e0924d4d7bb6a079171f636b9f/Untitled%202.png)

일단, Value, Labels 를 제외한 Annotations 만 출력하고, 해당 Annotations에 내가 필요한 값들을 추가시켜 구현하였다.

여기서 추가로 해당 인스턴스 이름, 임계값의 수치, 임계값의 범위 등을 추가하는 방안을 조사해야 할 것 같다.

---

## Slack Firing, Resolved 메세지 분할

![Untitled](Grafana%20Alert%20Templet%20094fb6e0924d4d7bb6a079171f636b9f/Untitled%203.png)

현재, 메트릭값이 임계치를 넘어서게 되면, Firing 경고 알람이 발송되고, 메트릭값이 정상화가 되면 Resolved 메세지를 전송하여 정상화의 알람까지 구현하였다.

하지만 현재는 메세지 타이틀 부분에 Status와 메세지 색상만 구별될 뿐, 내용 자체는 동일한 메세지가 전송된다.

이를 수정하여, 경고 알람과 정상화 알람을 개별로 구현하겠다.

## 1차 템플릿 (기본 기능)

```go
{{ define "alert_severity_prefix_emoji" -}}
	{{- if ne .Status "firing" -}}
		:white_check_mark:
	{{- else if eq .CommonLabels.severity "critical" -}}
		:red_circle:
	{{- else if eq .CommonLabels.severity "warning" -}}
		:warning:
	{{- end -}}
{{- end -}}

{{ define "cwslack.title" -}}
	{{ template "alert_severity_prefix_emoji" . }} 
	[{{- .Status | toUpper -}}{{- if eq .Status "firing" }} x {{ .Alerts.Firing | len -}}{{- end }}  | {{ .CommonLabels.env | toUpper -}} ] ||  {{ .CommonLabels.alertname -}}
{{- end -}}

{{- define "cwslack.text" -}}
{{- range .Alerts -}}
{{ if gt (len .Annotations) 0 }}
*project*: {{ .Annotations.project }}
*summary*: {{ .Annotations.summary }}
*description*: {{ .Annotations.description }}
*StartsAt*: {{ .StartsAt }}
*URL*: {{ .Annotations.URL }}
Labels: 
{{ range .Labels.SortedPairs }}{{ if or (eq .Name "env") (eq .Name "instance") }}• {{ .Name }}: `{{ .Value }}`
{{ end }}{{ end }}
{{ end }}
{{ end }}
{{ end }}
```

grafana alerting의 contact point Message templates이다.

해당 템플릿으로 여러 Contact points의 메세지 전송을 통일 할 수 있다.

현재 메세지 아이콘을 나타내는 `alert_severity_prefix_emoji` 템플릿과

해당 템플릿을 사용하며, Alert의 Status를 가져오는 `cwsaclk.title` 템플릿, 메세지 내용을 가져오는 `cwslack.text` 템플릿이 존재한다.

`cwslack.text` 템플릿에서는 Annotations (주석) 값에 정의한 내용들을 가져오게 작성하였다.

메세지에 추가할 내용이 있다면, 해당 템플릿에 추가하면 된다.

적용시에는 해당 contact point에서 title, text body에 해당 템플릿을 정의해주면 정상 작동한다.

## 2차 템플릿 (if문)

```go
{{ define "alert_severity_prefix_emoji" -}}
	{{- if ne .Status "firing" -}}
		:white_check_mark:
	{{- else if eq .CommonLabels.severity "critical" -}}
		:red_circle:
	{{- else if eq .CommonLabels.severity "warning" -}}
		:warning:
	{{- end -}}
{{- end -}}

{{ define "cwslack.title" -}}
	{{ template "alert_severity_prefix_emoji" . }} 
	[{{- .Status | toUpper -}}{{- if eq .Status "firing" }} x {{ .Alerts.Firing | len -}}{{- end }}  | {{ .CommonLabels.env | toUpper -}} ] ||  {{ .CommonLabels.alertname -}}
{{- end -}}

{{- define "cwking1" -}}
{{- range .Alerts -}}
  {{ if gt (len .Alerts.Firing) 0 }}
    firing:
    *summary*: {{ .Annotations.summary }}
  {{ end }}

  {{ if gt (len .Alerts.Resolved) 0 }}
    resolved:
    *summary*: {{ .Annotations.summary2 }}
  {{ end }}
{{ end }}
{{ end }}

{{ define "cwking2" }}
  {{ if gt (len .Alerts.Firing) 0 }}
    {{ len .Alerts.Firing }} firing:
    {{ range .Alerts.Firing }} {{ template "cwslack123.text" .}} {{ end }}
  {{ end }}
  {{ if gt (len .Alerts.Resolved) 0 }}
    {{ len .Alerts.Resolved }} resolved:
    {{ range .Alerts.Resolved }} {{ template "cwslack456.text" .}} {{ end }}
  {{ end }}

{{- define "cwslack123.text" -}}
{{- range .Alerts -}}
{{ if gt (len .Annotations) 0 }}
*project*: {{ .Annotations.project }}
*summary*: {{ .Annotations.summary }}
*description*: {{ .Annotations.description }}
*StartsAt*: {{ .StartsAt }}
*URL*: {{ .Annotations.URL }}
Labels: 
{{ range .Labels.SortedPairs }}{{ if or (eq .Name "env") (eq .Name "instance") }}• {{ .Name }}: `{{ .Value }}`
{{ end }}{{ end }}
{{ end }}
{{ end }}
{{ end }}

{{- define "cwslack456.text" -}}
{{- range .Alerts -}}
{{ if gt (len .Annotations) 0 }}
*project*: {{ .Annotations.project }}
*summary*: {{ .Annotations.summary2 }}
*description*: {{ .Annotations.description }}
*StartsAt*: {{ .StartsAt }}
*URL*: {{ .Annotations.URL }}
Labels: 
{{ range .Labels.SortedPairs }}{{ if or (eq .Name "env") (eq .Name "instance") }}• {{ .Name }}: `{{ .Value }}`
{{ end }}{{ end }}
{{ end }}
{{ end }}
{{ end }}
```

이후, Firing과 resolved Status에 달라지는  메세지를 구현하기 위해, 해당 IF문을 사용하는 템플릿을 생성하였다.

해당 템플릿은, if문으로 alert의 Status를 읽어들여, firing 상태가 1일때, resolved 상태가 1일때 각각 메세지를 다른 템플릿으로 가져오게 작성하였다.

테스트를 위해 if문 템플릿은 `cwking1` `cwking2` 두 형태로 생성하였으며, 

두 방법 다 if문의 정상 동작은 확인하였다.

하지만, 결론적인 메세지를 나타내는 템플릿을 가져오지 못하는 이슈가 있었다.  

(메세지를 가져오지 못하거나, 아니면 전체 모든 메세지를 가져와버린다.)

## 최종 템플릿 (실 서비스)

```go
{{- define "myalert112" -}}
  [{{.Status}}] {{ .Labels.alertname }}

  Labels:
  {{- range .Labels.SortedPairs -}}
    {{ .Name }}: {{ .Value }}
  {{ end }}

  {{ if gt (len .Annotations) 0 }}
  {{- range .Alerts -}}
  Annotations:
  *project*: {{ .Annotations.project }}
  *summary*: {{ .Annotations.summary }}
  *description*: {{ .Annotations.description }}
  *StartsAt*: {{ .StartsAt }}
  *URL*: {{ .Annotations.URL }}
  Silence alert: {{ .SilenceURL }}
  Go to dashboard: {{ .DashboardURL }}

  {{ end }}
  {{ end }}
{{ end }}

{{- define "myalert113" -}}
  [{{.Status}}] {{ .Labels.alertname }}

  Labels:
  {{- range .Labels.SortedPairs -}}
    {{ .Name }}: {{ .Value }}
  {{ end }}

  {{ if gt (len .Annotations) 0 }}
  {{- range .Alerts -}}
  Annotations:
  *project*: {{ .Annotations.project }}
  *summary*: {{ .Annotations.resolved }}
  *description*: {{ .Annotations.description }}
  *StartsAt*: {{ .StartsAt }}
  *URL*: {{ .Annotations.URL }}
  Silence alert: {{ .SilenceURL }}
  Go to dashboard: {{ .DashboardURL }}
  {{ end }}
  {{ end }}
{{ end }}

{{ define "cwzzang" }}
  {{ if gt (len .Alerts.Firing) 0 }}
    {{ len .Alerts.Firing }} firing:
    {{ range .Alerts.Firing }} {{ template "myalert112" .}} {{ end }}
  {{ end }}
  {{ if gt (len .Alerts.Resolved) 0 }}
    {{ len .Alerts.Resolved }} resolved:
    {{ range .Alerts.Resolved }} {{ template "myalert113" .}} {{ end }}
  {{ end }}
{{ end }}
```

기존 템플릿에 쓸데없는 부분들을 모두 제거하고, 필요한 정보만 정리하였다.

메세지를 전송하는 두 템플릿을 구분하여, 각 if문에 맞는 상황에만 불러오도록 작성하였다.

메세지를 작성하는 myalert112 myalert113 템플릿을, cwzzang 템플릿이 if문으로 구분하여 나타내고, 

즉 contact point에서는 cwzzang 템플릿만 불러오면 정상 작동이 된다.

![Untitled](Grafana%20Alert%20Templet%20094fb6e0924d4d7bb6a079171f636b9f/Untitled%204.png)

작동을 확인하였다.

추후 추가할 기능 (알람 발생 시간, 메트릭 값, host name 등) 의 관한 내용들은

본인이 작성한 템플릿에서 메세지 전송 부분에 이어 추가한다면, 간편하게 구현이 가능하다.

---