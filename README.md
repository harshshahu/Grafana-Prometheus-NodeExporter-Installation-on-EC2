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
