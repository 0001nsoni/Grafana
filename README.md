# Setting Up Prometheus, Node Exporter, Blackbox Exporter, and Grafana for Monitoring

## Install Prometheus

### Step 1: Create a User and Download Prometheus
```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
```

### Step 2: Extract and Move Files
```bash
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

### Step 3: Set Ownership
```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

### Step 4: Create a Systemd Service for Prometheus
```bash
sudo vim /etc/systemd/system/prometheus.service
```

#### Add the following content:
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

### Step 5: Enable and Start Prometheus
```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

### Step 6: Verify Prometheus Status
```bash
sudo systemctl status prometheus
```

### Step 7: Access Prometheus
Open a browser and navigate to:
```
http://<your-server-ip>:9090
```

---

## Install Node Exporter

### Step 1: Create a User and Download Node Exporter
```bash
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

### Step 2: Extract and Move Files
```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

### Step 3: Create a Systemd Service for Node Exporter
```bash
sudo vim /etc/systemd/system/node_exporter.service
```

#### Add the following content:
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
```

### Step 4: Enable and Start Node Exporter
```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### Step 5: Verify Node Exporter Status
```bash
sudo systemctl status node_exporter
```

#### Add Node Exporter to Prometheus Configuration:
```yaml
- job_name: "node_exporter"
  static_configs:
    - targets: ["<IP>:9100"]
```

---

## Install Blackbox Exporter

### Step 1: Download and Setup Blackbox Exporter
```bash
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.19.0/blackbox_exporter-0.19.0.linux-amd64.tar.gz
mv blackbox_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false blackbox_exporter
sudo chown blackbox_exporter:blackbox_exporter /usr/local/bin/blackbox_exporter
sudo mkdir /etc/blackbox_exporter
sudo chown blackbox_exporter:blackbox_exporter /etc/blackbox_exporter
```

### Step 2: Configure Blackbox Exporter
```bash
sudo vim /etc/blackbox_exporter/blackbox.yml
```
#### Add the following content:
```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:      
      valid_status_codes: []
      method: GET
```
```bash
sudo chown blackbox_exporter:blackbox_exporter /etc/blackbox_exporter/blackbox.yml
```

### Step 3: Create a Systemd Service for Blackbox Exporter
```bash
sudo vim /etc/systemd/system/blackbox_exporter.service
```

#### Add the following content:
```
[Unit]
Description=Blackbox Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox_exporter
Group=blackbox_exporter
Type=simple
ExecStart=/usr/local/bin/blackbox_exporter --config.file /etc/blackbox_exporter/blackbox.yml

[Install]
WantedBy=multi-user.target
```

### Step 4: Enable and Start Blackbox Exporter
```bash
sudo systemctl daemon-reload
sudo systemctl enable blackbox_exporter
sudo systemctl start blackbox_exporter
sudo systemctl status blackbox_exporter
```

#### Add Blackbox Exporter to Prometheus Configuration:
```yaml
- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
      - http://prometheus.io
      - https://prometheus.io
      - http://example.com:8080
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: localhost:9115
```

```bash
systemctl restart prometheus.service
```

---

## Install and Configure Grafana

### Step 1: Install Dependencies
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
```

### Step 2: Add Grafana Repository
```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### Step 3: Install Grafana
```bash
sudo apt-get update
sudo apt-get -y install grafana
```

### Step 4: Enable and Start Grafana
```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### Step 5: Access Grafana
```
http://<your-server-ip>:3000
```
Default credentials:
- **Username:** admin
- **Password:** admin (change upon first login)

