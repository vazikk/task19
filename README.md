# task19
1. Создал все файлы <br>
![image](https://github.com/user-attachments/assets/e9883c0e-3f7f-4f7c-9602-191b0826ac9c)

docker-compose.jml :<br>
```
  version: '3.7'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
    ports:
      - "9090:9090"

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"

  alertmanager:
    image: prom/alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/config.yml
    ports:
      - "9093:9093"
    command:
      - '--config.file=/etc/alertmanager/config.yml'
```

prometheus.jml : <br>
```
  global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
        - 'node-exporter:9100'

alerting:
  alertmanagers:
    - scheme: http
      static_configs:
        - targets:
          - 'alertmanager:9093'

rule_files:
  - '/etc/prometheus/alerts.yml'

```

alertmanager.jml : <br>
```
  global:
  resolve_timeout: 1m

route:
  group_by: ['alertname']
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/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'
        channel: '#all-stazha'
        title: '{{ .CommonLabels.alertname }}'
        text: '{{ .CommonAnnotations.summary }}\n{{ .CommonAnnotations.description }}'
```

alerts.jml : <br>
```
groups:
  - name: cpu-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 * (1 - avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[1m])) > 0.1)
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% (over 10% for 1 minute)"

```

ИТОГ: <br>
(Для проверки я поставил превышение - 10% )
Запустил команду для нагрузки проца на 2 мин на все ядра
```
stress --cpu $(nproc) --timeout 120 
```
![image](https://github.com/user-attachments/assets/a9e06dff-1b4b-48a8-ab3f-8f5472cf420a)

![image](https://github.com/user-attachments/assets/524e2837-7359-46e2-84e1-50c6699097b2)

![image](https://github.com/user-attachments/assets/964a5e3a-654d-42ef-b651-63c558b60b5c)

![image](https://github.com/user-attachments/assets/b45a0472-49a3-4ef6-bd71-c8435344f787)

![image](https://github.com/user-attachments/assets/1fc95c19-2200-4215-8fc5-caaf147b56b1)

![image](https://github.com/user-attachments/assets/2df4b12c-c78f-4ae8-b4a9-69f5ca128ee7)














