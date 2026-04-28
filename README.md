## 🔹 Step 1: Update EC2 System

```bash
sudo yum update -y
```

---

## 🔹 Step 2: Install Grafana (Enterprise Edition)

### Install Grafana RPM

```bash
sudo yum update -y
sudo yum install wget tar -y
sudo yum install make -y
sudo yum install -y https://dl.grafana.com/grafana-enterprise/release/12.2.1/grafana-enterprise_12.2.1_18655849634_linux_amd64.rpm
```

### Verify Grafana Version

```bash
grafana-server --version
```

---

## 🔹 Step 3: Start & Enable Grafana Service

```bash
sudo systemctl start grafana-server
sudo systemctl enable  grafana-server
sudo systemctl status grafana-server
```

✅ **Grafana Web UI**

```
http://<EC2-PUBLIC-IP>:3000
```

**Default Credentials**

* Username: `admin`
* Password: `admin`

---

## 🔹 Step 4: Install WinSCP and Download Prometheus Setup | LTS/Linux
## WinSCP  >> Install 
https://winscp.net/eng/download.php

## Download Prometheus Setup | LTS/Linux
https://prometheus.io/download/


## 🔹 Step 5: Copy Prometheus Binary to EC2

### SCP from Local Machine (macOS/Linux)

```bash
scp -i ansible.pem prometheus-3.5.3.linux-amd64.tar.gz \
ec2-user@3.7.66.154:/home/ec2-user/
```

---

## 🔹 Step 6: Move & Extract Prometheus

login to your ec2 and type below:

```bash
sudo mv prometheus-3.5.1.linux-amd64.tar.gz /opt
cd /opt
sudo tar -xvf prometheus-3.5.1.linux-amd64.tar.gz
sudo mv prometheus-3.5.1.linux-amd64 prometheus
sudo rm prometheus-3.5.1.linux-amd64.tar.gz
```

---

## 🔹 Step 7: Create Prometheus System User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

---

## 🔹 Step 8: Configure Prometheus Directories

```bash
cd /opt/prometheus

sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo cp prometheus.yml /etc/prometheus/
```

### Set Permissions

```bash
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

---

## 🔹 Step 9: Create Prometheus systemd Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

### 📄 Paste the Following Configuration

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

---

## 🔹 Step 10: Update Prometheus Config

edit config file 
```
sudo nano /etc/prometheus/prometheus.yml
```
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

Start Prometheus Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

✅ **Prometheus Web UI**

```
http://<EC2-PUBLIC-IP>:9090
```
---


## 🔹 Step 11: Add Data Source in Grafana

```
1. go to grafana weblink http://public-ip:3000/
2. click on 'data sources'
3. click on 'prometheus'
4. Prometheus server URL: http://localhost:9090
5. save and test
6. you will see: Successfully queried the Prometheus API.
```

---

## 🔹 Step 12: Download Node Exporter & Extract

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64
```

---

## 🔹 Step 13: Run Manually (Test)

```bash
./node_exporter
```

👉 Access in browser:

```
http://<Public-IP>:9100/metrics
```

you will see 'Node Exporter' Page but the problem is when I type: Ctrl+c, node exporter will also stoped.
So we will make a service so it runs continuously.

---

## 🔹 Step 14: Install Binary & Create User

```bash
sudo cp node_exporter /usr/local/bin
sudo useradd node_exporter --no-create-home --shell /bin/false
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

---

## 🔹 Step 15: Create Systemd Service

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

### Add below content:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

---

## 🔹 Step 16: Start & Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

---

## 🔹 Step 17: Verify

Open browser:

```
http://<SERVER-IP>:9100/metrics
```

✔ You should see system metrics output

---
