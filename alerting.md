# 📊 Monitoring Stack – Post Setup Guide

## 🚀 Components

* Prometheus → Metrics collection
* Node Exporter → System metrics
* Grafana → Visualization
* Alertmanager → Notifications

---

# ⚙️ 1. Prometheus Alerting Configuration

## 📌 Step 1: Create Alert Rules

```bash
sudo nano /etc/prometheus/alerts.yml
```

```yaml
groups:
- name: node_alerts
  rules:

  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance Down"

  - alert: HighCPUUsage
    expr: 100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High CPU Usage"

  - alert: LowMemory
    expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) < 0.2
    for: 2m
    labels:
      severity: critical

  - alert: DiskAlmostFull
    expr: (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes) < 0.2
    for: 5m
```

---

## 📌 Step 2: Update Prometheus Config as below

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alerts.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - "localhost:9093"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

---

## 📌 Step 3: Restart Prometheus

```bash
sudo systemctl restart prometheus
```

---

# 🚨 2. Alertmanager Configuration

```bash
sudo nano /etc/alertmanager/alertmanager.yml
```

```yaml
route:
  group_by: ['alertname']
  receiver: default
  routes:
    - match:
        severity: critical
      receiver: email
    - match:
        severity: warning
      receiver: slack

receivers:
- name: email
  email_configs:
  - to: "your@email.com"
    from: "alert@email.com"
    smarthost: "smtp.gmail.com:587"
    auth_username: "your@email.com"
    auth_password: "APP_PASSWORD"

- name: slack
  slack_configs:
  - api_url: "https://hooks.slack.com/services/XXXX"
```

---

# 📊 3. Grafana Dashboards

## 📌 Step 1: Add Prometheus Data Source

* URL: `http://localhost:9090`

---

## 📌 Step 2: Import Dashboard

| Dashboard           | ID    |
| ------------------- | ----- |
| Node Exporter Full  | 1860  |
| Node Exporter Quick | 11074 |
| Prometheus Stats    | 3662  |

---

## 📌 Step 3: Important Queries

```promql
# CPU Usage
100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)

# Disk Usage %
(1 - node_filesystem_avail_bytes/node_filesystem_size_bytes) * 100

# Load Average
node_load1
```

---

# ⚡ 4. Manual Load Testing (VERY IMPORTANT)

## 🔥 CPU Test

```bash
#!/bin/bash

echo "CPU Stress Test"
sudo apt install -y stress
stress --cpu $(nproc) --timeout 120
```

---

## 🧠 Memory Test

```bash
#!/bin/bash

echo "Memory Stress Test"
sudo apt install -y stress
stress --vm 2 --vm-bytes 80% --timeout 120
```

---

## 💽 Disk Test

```bash
#!/bin/bash

echo "Disk Stress Test"
fallocate -l 2G /tmp/testfile
sleep 120
rm -f /tmp/testfile
```

---

## 🌐 Instance Down Test

```bash
#!/bin/bash

echo "Stopping Node Exporter"
sudo systemctl stop node_exporter
sleep 120
sudo systemctl start node_exporter
```

---

# 📊 5. Verification Steps

## 🔹 Check Targets

```bash
curl http://localhost:9090/api/v1/targets
```

---

## 🔹 Check Alerts

```bash
curl http://localhost:9090/api/v1/alerts
```

---

## 🔹 Check Alertmanager

```bash
curl http://localhost:9093/api/v2/alerts
```

---

## 🔹 Check Node Exporter

```bash
curl http://localhost:9100/metrics | head
```

---

# 📜 6. Logs Monitoring

```bash
# Prometheus
journalctl -u prometheus -f

# Node Exporter
journalctl -u node_exporter -f

# Grafana
journalctl -u grafana-server -f
```

---

# ⚙️ 7. Health Check Script

```bash
#!/bin/bash

echo "Checking Services..."

systemctl is-active prometheus
systemctl is-active node_exporter
systemctl is-active grafana-server

echo "Checking Endpoints..."

curl -s localhost:9090/-/healthy
curl -s localhost:9100/metrics | head -n 5
curl -s localhost:3000/api/health
```

---

# 🔥 8. Alert Testing Flow

1. Run CPU / Memory / Disk script
2. Open Grafana dashboard
3. Check Prometheus alerts → `/alerts`
4. Wait 2–5 minutes (`for:` condition)
5. Verify:

   * Alert triggered
   * Alertmanager received
   * Email/Slack sent

---

# 📌 9. Logging Stack (Optional)

* Use **Grafana Loki** + Promtail
* Alternative: **ELK Stack**

---

# ✅ Final Checklist

* [ ] Prometheus targets UP
* [ ] Grafana dashboard working
* [ ] Alerts configured
* [ ] Alertmanager routing working
* [ ] CPU/Memory/Disk test verified
* [ ] Logs accessible

---
