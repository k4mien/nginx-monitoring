# nginx-monitoring
Monitoring stack for nginx ssl web server

## Table of contents
* [Demo](#demo)
* [Tech stack](#tech-stack)
* [Features](#features)
* [To do](#to-do)
* [Installation](#installation)
* [More](#more)

## Demo
![Screenshot 2024-07-09 005607](https://github.com/k4mien/nginx-monitoring/assets/56881087/e79b94d0-48a7-40ba-b5a9-1cc7249cdfa1)
![Screenshot 2024-07-09 005620](https://github.com/k4mien/nginx-monitoring/assets/56881087/e2dfe713-6f38-48c5-be7a-02b39ba64611)

## Tech stack
- Docker
- Nginx
- Grafana
- Prometheus
- Node-Exporter
- Cadvisor
- Nginx-Prom-Exporter

## Features
- pre-configured, only cert/key and dhparam are needed to generate
- clean dashboard
- selfsigned ssl for web server
- useful metrics

## To do
- more complex metrics

## Installation

1. Git clone this repo
```bash
git clone https://github.com/k4mien/nginx-monitoring.git
```
2. Change directory to project
```bash
cd nginx-monitoring
```
3. Generate cert/key and dhparam
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./selfsigned.key -out ./selfsigned.crt -subj "/CN=localhost"
```

```bash
sudo openssl dhparam -out ./dhparam.pem 2048
```
4. Start docker compose
```bash
docker compose up -d
```

## More
By default I kept node-exporter in docker bridge network. It should be running on ```network_mode=host``` or host.
To get network metrics from host you have to change following config:
```yaml
node_exporter:
  image: prom/node-exporter
  container_name: node-exporter
  command:
    - "--path.rootfs=/host"
  pid: host
  network_mode: host
  volumes:
    - "/:/host:ro"
```

Now your node-exporter wont be in docker compose network but host. Then in ```prometheus.yml``` point to host ip rather than docker dns.
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: nginx-prom
    static_configs:
      - targets:
          - nginx-prom-export:9113

  - job_name: node-exporter
    static_configs:
      - targets:
          - <host-ip>:9100

  - job_name: cadvisor
    static_configs:
      - targets:
          - cadvisor:8080

```
