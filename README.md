# task19
1. Создал все файлы <br>
![image](https://github.com/user-attachments/assets/e9883c0e-3f7f-4f7c-9602-191b0826ac9c)

docker-compose.jml :<br>
```
version: '3.7'

services:
  prometheus:
    image: prom/prometheus:v3.4.2 
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
    ports:
      - "9090:9090"
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:v1.9.1 
    ports:
      - "9100:9100"
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:v0.28.1 
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/config.yml
    ports:
      - "9093:9093"
    command:
      - '--config.file=/etc/alertmanager/config.yml'
```

prometheus.yml : <br>
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
          - 'node-exporter:9100'
        labels:
          instance: 'Node Exporter on inst1/my PC'

alerting:
  alertmanagers:
    - scheme: http
      static_configs:
        - targets:
          - 'alertmanager:9093'

rule_files:
  - '/etc/prometheus/alerts.yml'
```

alertmanager.yml : <br>
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

alerts.yml : <br>
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
<img width="990" height="492" alt="image" src="https://github.com/user-attachments/assets/14edfbdc-0c4e-458f-9071-5dde00eb882f" />

<img width="1903" height="201" alt="image" src="https://github.com/user-attachments/assets/75fa04b4-de7d-41ff-a61e-d9889e4b334e" />

![image](https://github.com/user-attachments/assets/964a5e3a-654d-42ef-b651-63c558b60b5c)

![image](https://github.com/user-attachments/assets/b45a0472-49a3-4ef6-bd71-c8435344f787)

<img width="849" height="198" alt="image" src="https://github.com/user-attachments/assets/49b6cf3b-1cb4-428d-bf35-e4ac8080df9e" />

<img width="937" height="274" alt="image" src="https://github.com/user-attachments/assets/40df78ad-72ce-4cbf-98c6-294fa3b00404" />














