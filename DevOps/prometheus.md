# Prometheus 📊

[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-red)](https://prometheus.io/)
[![Go](https://img.shields.io/badge/Go-1.19+-blue)](https://golang.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

> **Open Source Monitoring & Alerting Toolkit** - Prometheus is a powerful systems monitoring and alerting toolkit originally built at SoundCloud, designed for reliability and scalability.

## 🎯 Overview

Prometheus is a complete monitoring and alerting system that:
- **Collects Metrics** - Pulls metrics from configured targets
- **Stores Time Series Data** - Efficiently stores time-series data
- **Provides Querying** - Powerful query language (PromQL)
- **Sends Alerts** - Configurable alerting based on metrics
- **Integrates Seamlessly** - Works with Grafana, AlertManager, and more

## 🚀 Key Features

### Core Capabilities
- **Multi-dimensional Data Model** - Time series identified by metric name and key-value pairs
- **Flexible Query Language** - PromQL for powerful data analysis
- **Pull-based Architecture** - Reliable metric collection
- **Service Discovery** - Automatic target discovery
- **No Dependencies** - Self-contained monitoring system
- **High Availability** - Built-in clustering support

### Data Collection
- **Metrics Scraping** - HTTP endpoints for metric collection
- **Push Gateway** - Support for short-lived jobs
- **Exporters** - Pre-built integrations for popular services
- **Custom Metrics** - Easy integration with any application

## 🏗️ Architecture

### System Components
```
Prometheus Ecosystem
├── 🎯 Prometheus Server
│   ├── Time Series Database
│   ├── Query Engine
│   └── Web UI
├── 📊 Exporters
│   ├── Node Exporter
│   ├── cAdvisor
│   └── Custom Exporters
├── 🔔 AlertManager
│   ├── Alert Routing
│   ├── Grouping & Inhibition
│   └── Notification Channels
└── 📈 Visualization
    ├── Grafana
    ├── Prometheus Web UI
    └── Custom Dashboards
```

## 🛠️ Installation & Setup

### Docker Installation
```bash
# Create Prometheus configuration
mkdir prometheus
cd prometheus

# Create prometheus.yml
cat > prometheus.yml << EOF
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
EOF

# Run Prometheus
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

### Binary Installation
```bash
# Download and install
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64

# Start Prometheus
./prometheus --config.file=prometheus.yml
```

### Kubernetes Installation
```yaml
# prometheus-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
spec:
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
  type: LoadBalancer
```

## 📊 Configuration

### Basic Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s      # How often to scrape targets
  evaluation_interval: 15s  # How often to evaluate rules
  external_labels:
    cluster: 'production'
    replica: 'prometheus-1'

rule_files:
  - "alert_rules.yml"
  - "recording_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
    scrape_interval: 5s
    metrics_path: /metrics

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### Service Discovery
```yaml
# Kubernetes service discovery
- job_name: 'kubernetes-services'
  kubernetes_sd_configs:
    - role: service
  relabel_configs:
    - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
      action: keep
      regex: true
    - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
      action: replace
      target_label: __metrics_path__
      regex: (.+)

# Consul service discovery
- job_name: 'consul-services'
  consul_sd_configs:
    - server: 'consul:8500'
      services: ['web', 'api', 'database']
```

## 🔍 PromQL (Prometheus Query Language)

### Basic Queries
```promql
# Select all time series with metric name
http_requests_total

# Filter by label
http_requests_total{job="api-server"}

# Multiple label filters
http_requests_total{job="api-server", method="GET"}

# Range queries
http_requests_total[5m]

# Rate calculation
rate(http_requests_total[5m])

# Counter increase
increase(http_requests_total[1h])
```

### Aggregation Functions
```promql
# Sum by label
sum(http_requests_total) by (job)

# Average
avg(cpu_usage_percent)

# Maximum
max(memory_usage_bytes)

# Quantiles
quantile(0.95, response_time_seconds)

# Count
count(up)
```

### Advanced Queries
```promql
# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# CPU usage percentage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage percentage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100
```

## 📈 Recording Rules

### Performance Rules
```yaml
# recording_rules.yml
groups:
- name: performance
  rules:
  - record: instance:http_requests:rate5m
    expr: rate(http_requests_total[5m])
  
  - record: instance:http_requests:rate1h
    expr: rate(http_requests_total[1h])
  
  - record: instance:http_request_duration_seconds:rate5m
    expr: rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])
  
  - record: instance:http_request_duration_seconds:rate1h
    expr: rate(http_request_duration_seconds_sum[1h]) / rate(http_request_duration_seconds_count[1h])
```

### System Rules
```yaml
- name: system
  rules:
  - record: instance:node_cpu_usage:rate5m
    expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
  
  - record: instance:node_memory_usage:ratio
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))
  
  - record: instance:node_disk_usage:ratio
    expr: (1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes))
```

## 🚨 Alerting Rules

### Basic Alerts
```yaml
# alert_rules.yml
groups:
- name: infrastructure
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} has been down for more than 1 minute."

  - alert: HighCPUUsage
    expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is above 80% for more than 5 minutes."

  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) > 0.9
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High memory usage on {{ $labels.instance }}"
      description: "Memory usage is above 90% for more than 5 minutes."
```

### Application Alerts
```yaml
- name: application
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.1
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High error rate on {{ $labels.job }}"
      description: "Error rate is above 10% for more than 2 minutes."

  - alert: HighResponseTime
    expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High response time on {{ $labels.job }}"
      description: "95th percentile response time is above 1 second."
```

## 🔌 Exporters

### Node Exporter
```bash
# Install Node Exporter
docker run -d \
  --name node-exporter \
  -p 9100:9100 \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /:/rootfs:ro \
  prom/node-exporter \
  --path.procfs=/host/proc \
  --path.sysfs=/host/sys \
  --collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($|/)
```

### cAdvisor
```bash
# Install cAdvisor
docker run -d \
  --name cadvisor \
  -p 8080:8080 \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -v /sys:/sys:ro \
  -v /var/lib/docker/:/var/lib/docker:ro \
  -v /dev/disk/:/dev/disk:ro \
  gcr.io/cadvisor/cadvisor:latest
```

### Custom Exporter
```python
# custom_exporter.py
from prometheus_client import start_http_server, Counter, Gauge, Histogram
import time
import random

# Define metrics
REQUEST_COUNT = Counter('app_requests_total', 'Total requests', ['method', 'endpoint'])
REQUEST_DURATION = Histogram('app_request_duration_seconds', 'Request duration')
ACTIVE_CONNECTIONS = Gauge('app_active_connections', 'Active connections')

def process_request():
    """Simulate request processing"""
    start_time = time.time()
    
    # Simulate work
    time.sleep(random.uniform(0.1, 0.5))
    
    # Record metrics
    REQUEST_COUNT.labels(method='GET', endpoint='/api').inc()
    REQUEST_DURATION.observe(time.time() - start_time)

if __name__ == '__main__':
    # Start metrics server
    start_http_server(8000)
    
    # Simulate requests
    while True:
        process_request()
        ACTIVE_CONNECTIONS.set(random.randint(10, 100))
        time.sleep(1)
```

## 📊 Grafana Integration

### Dashboard Configuration
```json
{
  "dashboard": {
    "title": "System Overview",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "CPU Usage %"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
            "legendFormat": "Memory Usage %"
          }
        ]
      }
    ]
  }
}
```

## 🔧 Best Practices

### Configuration
1. **Use Service Discovery** - Automate target discovery
2. **Set Appropriate Intervals** - Balance between detail and performance
3. **Organize Rules** - Group related rules together
4. **Use External Labels** - Identify Prometheus instances
5. **Validate Configurations** - Test before deploying

### Monitoring
1. **Monitor Prometheus Itself** - Track Prometheus performance
2. **Set Up Alerting** - Configure meaningful alerts
3. **Use Recording Rules** - Pre-compute expensive queries
4. **Regular Backups** - Backup configuration and data
5. **Capacity Planning** - Monitor storage and memory usage

### Querying
1. **Use Range Queries** - For time-series analysis
2. **Avoid High Cardinality** - Limit label combinations
3. **Use Recording Rules** - For complex calculations
4. **Test Queries** - Validate before using in alerts
5. **Document Queries** - Maintain query documentation

## 🚀 Advanced Features

### Federation
```yaml
# federate.yml
scrape_configs:
- job_name: 'federate'
  scrape_interval: 15s
  honor_labels: true
  metrics_path: '/federate'
  params:
    'match[]':
      - '{job=~"prometheus"}'
      - '{__name__=~"job:.*"}'
  static_configs:
    - targets:
      - 'prometheus-1:9090'
      - 'prometheus-2:9090'
```

### Remote Storage
```yaml
# prometheus.yml with remote storage
global:
  external_labels:
    cluster: 'production'
    replica: 'prometheus-1'

remote_write:
  - url: "http://cortex:9009/api/prom/push"
    queue_config:
      max_samples_per_send: 1000
      batch_send_deadline: 5s

remote_read:
  - url: "http://cortex:9009/api/prom/read"
```

## 🐛 Troubleshooting

### Common Issues
1. **High Memory Usage** - Check cardinality and retention
2. **Slow Queries** - Use recording rules and optimize queries
3. **Missing Metrics** - Verify scrape configuration
4. **Alert Fatigue** - Tune alert thresholds and grouping
5. **Storage Issues** - Monitor disk usage and retention

### Debugging Tools
```bash
# Check configuration
promtool check config prometheus.yml

# Test rules
promtool check rules alert_rules.yml

# Query metrics
curl 'http://localhost:9090/api/v1/query?query=up'

# Check targets
curl 'http://localhost:9090/api/v1/targets'
```

## 📚 Learning Resources

### Official Documentation
- [Prometheus Documentation](https://prometheus.io/docs/)
- [PromQL Guide](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Configuration Reference](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)

### Community Resources
- [GitHub Repository](https://github.com/prometheus/prometheus)
- [Community Forum](https://groups.google.com/forum/#!forum/prometheus-users)
- [Slack Channel](https://cloud-native.slack.com/messages/prometheus)

## 🔄 Updates and Roadmap

### Recent Updates
- **v2.45.0** - Improved query performance and memory usage
- **v2.44.0** - Enhanced service discovery and federation
- **v2.43.0** - Better error handling and logging

### Upcoming Features
- **v2.46.0** - Advanced query optimization
- **v2.47.0** - Enhanced remote storage integration
- **v2.48.0** - Improved alerting capabilities

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow Go coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/prometheus/prometheus/blob/main/LICENSE) file for details.

## 🙏 Acknowledgments

- **SoundCloud** for originally creating Prometheus
- **CNCF** for maintaining and evolving the project
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#prometheus-)

</div>

