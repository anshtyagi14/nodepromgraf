# Installing and setting up Node Exporter, Prometheus, and Grafana for system log visualization

This guide will help you install and set up Node Exporter, Prometheus, and Grafana to visualize system logs on Grafana.

## Requirements

Linux-based operating system (Ubuntu, Debian, etc.)

| Command | Description |
| --- | --- |
| git status | List all new or modified files |
| git diff | Show file differences that haven't been staged |

## Initial setup

### Step 1: Update your system

```console
$ sudo apt update
$ sudo apt upgrade
```

### Step 2: Install Node Exporter

Download and extract Node Exporter

```console
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
$ tar xvzf node_exporter-1.6.1.linux-amd64.tar.gz
```

Copy the binary to /usr/local/bin/ and set permissions

```console
$ sudo cp node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
$ sudo useradd --no-create-home --shell /bin/false node_exporter
$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Create a new systemd service file for Node Exporter

```console
$ sudo vim /etc/systemd/system/node_exporter.service
```

Paste the following content into the file

```yml
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Reload systemd, enable, and start Node Exporter

```console
$ sudo systemctl daemon-reload
$ sudo systemctl enable node_exporter
$ sudo systemctl start node_exporter
$ systemctl status node_exporter
```

### Step 3: Install Prometheus

Download and extract Prometheus

```console
$ wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
$ tar xvzf prometheus-2.46.0.linux-amd64.tar.gz
```

Create necessary directories and copy files

```console
$ sudo mkdir /var/lib/prometheus
$ sudo mkdir /etc/prometheus
$ sudo mkdir /etc/prometheus/consoles
$ sudo mkdir /etc/prometheus/console_libraries

$ sudo cp prometheus-2.46.0.linux-amd64/prometheus /usr/local/bin/
$ sudo cp prometheus-2.46.0.linux-amd64/promtool /usr/local/bin/
$ sudo cp prometheus-2.46.0.linux-amd64/prometheus.yml /etc/prometheus/
$ sudo cp -r prometheus-2.46.0.linux-amd64/consoles /etc/prometheus
$ sudo cp -r prometheus-2.46.0.linux-amd64/console_libraries/ /etc/prometheus
```

Create a Prometheus user and set permissions

```console
$ sudo useradd --no-create-home --shell /bin/false prometheus

$ sudo chown prometheus:prometheus /var/lib/prometheus
$ sudo chown prometheus:prometheus /etc/prometheus
$ sudo chown -R prometheus:prometheus /etc/prometheus/consoles
$ sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

Modify the Prometheus configuration file

```console
$ sudo vim /etc/prometheus/prometheus.yml
```

Add the following content under the scrape_configs section

```yml
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

Create a new systemd service file for Prometheus

```console
$ sudo vim /etc/systemd/system/prometheus.service
```

Paste the following content into the file

```yml
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Reload systemd, enable, and start Prometheus

```console
$ sudo systemctl daemon-reload
$ sudo systemctl enable prometheus
$ sudo systemctl start prometheus
$ systemctl status prometheus
```

### Step 4: Install Grafana

Install Grafana dependencies

```console
$ sudo apt-get install -y adduser libfontconfig1
```

Download and install Grafana

```console
$ wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.0.3_amd64.deb
$ sudo dpkg -i grafana-enterprise_10.0.3_amd64.deb
```

Enable and start Grafana

```console
$ sudo systemctl enable grafana-server
$ sudo systemctl start grafana-server
$ systemctl status grafana-server
```

### Step 5: Configure Grafana to visualize system logs

1. Open Grafana in your browser at http://your-server-ip:3000 and log in with the default credentials (username: admin, password: admin).
2. Click on the Connections in the left sidebar, then Data sources, and click Add Data source.
3. Select Prometheus as the data source.
4. Under HTTP, set the URL to http://localhost:9090.
5. Click Save & test to save the configuration and test the connection.
6. To import a pre-built dashboard, click on Dashboards (four squares icon) in the left sidebar, then click New, and select Import.
7. Click on Import dashboard and enter 1860 in the Import via grafana.com field, then click Load.
8. Select Prometheus as the data source from the dropdown and click Import.

Now you have successfully installed Node Exporter, Prometheus, and Grafana and visualized system logs on Grafana using a prebuilt dashboard.
