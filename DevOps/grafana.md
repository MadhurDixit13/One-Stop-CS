# Grafana 📊

[![Grafana](https://img.shields.io/badge/Grafana-Visualization-orange)](https://grafana.com/)
[![Go](https://img.shields.io/badge/Go-1.19+-blue)](https://golang.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

> **Open Source Analytics & Monitoring Platform** - Grafana is a multi-platform open source analytics and interactive visualization web application that provides charts, graphs, and alerts when connected to supported data sources.

## 🎯 Overview

Grafana is a powerful visualization platform that:
- **Creates Dashboards** - Interactive dashboards for data visualization
- **Connects Data Sources** - Supports 100+ data sources
- **Provides Alerting** - Real-time alerting and notifications
- **Enables Exploration** - Ad-hoc data exploration and analysis
- **Supports Collaboration** - Team-based dashboard sharing
- **Offers Flexibility** - Highly customizable and extensible

## 🚀 Key Features

### Core Capabilities
- **Multi-Data Source Support** - Prometheus, InfluxDB, MySQL, PostgreSQL, and more
- **Rich Visualizations** - 50+ panel types and visualization options
- **Real-time Monitoring** - Live data streaming and updates
- **Advanced Alerting** - Sophisticated alert rules and notifications
- **User Management** - Role-based access control and permissions
- **Plugin Ecosystem** - Extensible through plugins and custom panels

### Visualization Types
- **Time Series** - Line, area, and bar charts for time-based data
- **Stat Panels** - Single value displays with thresholds
- **Tables** - Structured data presentation
- **Heatmaps** - Density and distribution visualization
- **Geomaps** - Geographic data visualization
- **Custom Panels** - Plugin-based custom visualizations

## 🏗️ Architecture

### System Components
```
Grafana Platform
├── 🎨 Frontend (React/TypeScript)
│   ├── Dashboard Editor
│   ├── Panel Library
│   └── User Interface
├── 🔧 Backend (Go)
│   ├── API Server
│   ├── Data Source Proxies
│   └── Alert Engine
├── 💾 Data Layer
│   ├── SQLite/PostgreSQL
│   ├── Redis (Caching)
│   └── File System
└── 🔌 Data Sources
    ├── Prometheus
    ├── InfluxDB
    ├── MySQL/PostgreSQL
    └── Custom APIs
```

## 🛠️ Installation & Setup

### Docker Installation
```bash
# Create Grafana configuration
mkdir grafana
cd grafana

# Create docker-compose.yml
cat > docker-compose.yml << EOF
version: '3.8'
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    volumes:
      - grafana-storage:/var/lib/grafana
      - ./grafana.ini:/etc/grafana/grafana.ini
    networks:
      - monitoring

volumes:
  grafana-storage:

networks:
  monitoring:
    driver: bridge
EOF

# Start Grafana
docker-compose up -d
```

### Binary Installation
```bash
# Download and install
wget https://dl.grafana.com/oss/release/grafana-10.0.0.linux-amd64.tar.gz
tar -zxvf grafana-10.0.0.linux-amd64.tar.gz
cd grafana-10.0.0

# Start Grafana
./bin/grafana-server
```

### Kubernetes Installation
```yaml
# grafana-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: "admin"
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
      volumes:
      - name: grafana-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
  type: LoadBalancer
```

## ⚙️ Configuration

### Basic Configuration
```ini
# grafana.ini
[server]
http_port = 3000
domain = localhost
root_url = http://localhost:3000/

[database]
type = sqlite3
path = grafana.db

[security]
admin_user = admin
admin_password = admin
secret_key = your-secret-key

[users]
allow_sign_up = true
auto_assign_org = true
auto_assign_org_role = Viewer

[log]
mode = console
level = info

[auth.anonymous]
enabled = false

[auth.basic]
enabled = true
```

### Data Source Configuration
```yaml
# prometheus-datasource.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-datasource
data:
  prometheus.yaml: |
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      access: proxy
      url: http://prometheus:9090
      isDefault: true
      editable: true
```

## 📊 Dashboard Creation

### Basic Dashboard
```json
{
  "dashboard": {
    "id": null,
    "title": "System Overview",
    "tags": ["system", "monitoring"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "CPU Usage %",
            "refId": "A"
          }
        ],
        "yAxes": [
          {
            "label": "Percentage",
            "min": 0,
            "max": 100
          }
        ],
        "xAxis": {
          "mode": "time"
        },
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 0,
          "y": 0
        }
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "5s"
  }
}
```

### Advanced Dashboard
```json
{
  "dashboard": {
    "title": "Application Performance",
    "panels": [
      {
        "title": "Request Rate",
        "type": "stat",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "reqps",
            "color": {
              "mode": "thresholds"
            },
            "thresholds": {
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 100},
                {"color": "red", "value": 500}
              ]
            }
          }
        }
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile",
            "refId": "A"
          },
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "50th percentile",
            "refId": "B"
          }
        ]
      }
    ]
  }
}
```

## 🔔 Alerting

### Alert Rules
```json
{
  "alert": {
    "name": "High CPU Usage",
    "message": "CPU usage is above 80%",
    "frequency": "10s",
    "conditions": [
      {
        "evaluator": {
          "params": [80],
          "type": "gt"
        },
        "operator": {
          "type": "and"
        },
        "query": {
          "params": ["A", "5m", "now"]
        },
        "reducer": {
          "params": [],
          "type": "last"
        },
        "type": "query"
      }
    ],
    "executionErrorState": "alerting",
    "for": "5m",
    "noDataState": "no_data",
    "handler": 1
  }
}
```

### Notification Channels
```json
{
  "notification": {
    "name": "Slack Alerts",
    "type": "slack",
    "settings": {
      "url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
      "channel": "#alerts",
      "title": "Grafana Alert",
      "text": "{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}"
    }
  }
}
```

## 🔌 Data Sources

### Prometheus Integration
```yaml
# prometheus-datasource.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-datasource
data:
  prometheus.yaml: |
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      access: proxy
      url: http://prometheus:9090
      isDefault: true
      editable: true
      jsonData:
        httpMethod: POST
        manageAlerts: true
        prometheusType: Prometheus
        prometheusVersion: 2.40.0
        cacheLevel: 'High'
        disableRecordingRules: false
        incrementalQueryOverlapWindow: 10m
```

### InfluxDB Integration
```yaml
# influxdb-datasource.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: influxdb-datasource
data:
  influxdb.yaml: |
    apiVersion: 1
    datasources:
    - name: InfluxDB
      type: influxdb
      access: proxy
      url: http://influxdb:8086
      database: telegraf
      user: grafana
      password: grafana
      isDefault: false
      editable: true
```

### MySQL Integration
```yaml
# mysql-datasource.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-datasource
data:
  mysql.yaml: |
    apiVersion: 1
    datasources:
    - name: MySQL
      type: mysql
      access: proxy
      url: mysql:3306
      database: monitoring
      user: grafana
      password: grafana
      isDefault: false
      editable: true
```

## 🎨 Panel Types

### Time Series Panel
```json
{
  "title": "Time Series",
  "type": "timeseries",
  "targets": [
    {
      "expr": "rate(http_requests_total[5m])",
      "refId": "A"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "short",
      "color": {
        "mode": "palette-classic"
      }
    }
  }
}
```

### Stat Panel
```json
{
  "title": "Total Requests",
  "type": "stat",
  "targets": [
    {
      "expr": "sum(http_requests_total)",
      "refId": "A"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "short",
      "color": {
        "mode": "thresholds"
      },
      "thresholds": {
        "steps": [
          {"color": "green", "value": null},
          {"color": "yellow", "value": 1000},
          {"color": "red", "value": 5000}
        ]
      }
    }
  }
}
```

### Table Panel
```json
{
  "title": "Top Endpoints",
  "type": "table",
  "targets": [
    {
      "expr": "topk(10, sum(rate(http_requests_total[5m])) by (endpoint))",
      "refId": "A"
    }
  ],
  "transformations": [
    {
      "id": "organize",
      "options": {
        "excludeByName": {
          "Time": true
        },
        "indexByName": {},
        "renameByName": {
          "Value": "Requests/sec"
        }
      }
    }
  ]
}
```

## 🔧 Advanced Features

### Variables and Templating
```json
{
  "templating": {
    "list": [
      {
        "name": "instance",
        "type": "query",
        "query": "label_values(up, instance)",
        "refresh": 1,
        "includeAll": true,
        "multi": true
      },
      {
        "name": "job",
        "type": "query",
        "query": "label_values(up, job)",
        "refresh": 1,
        "includeAll": true,
        "multi": true
      }
    ]
  }
}
```

### Annotations
```json
{
  "annotations": {
    "list": [
      {
        "name": "Deployments",
        "datasource": "Prometheus",
        "expr": "increase(deployment_events_total[1m])",
        "iconColor": "rgba(0, 211, 255, 1)",
        "enable": true,
        "hide": false
      }
    ]
  }
}
```

### Dashboard Links
```json
{
  "links": [
    {
      "title": "Grafana Documentation",
      "url": "https://grafana.com/docs/",
      "type": "link",
      "icon": "external link"
    },
    {
      "title": "Prometheus Targets",
      "url": "http://prometheus:9090/targets",
      "type": "link",
      "icon": "external link"
    }
  ]
}
```

## 🔌 Plugins and Extensions

### Installing Plugins
```bash
# Install plugin via Docker
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -e GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel \
  grafana/grafana:latest

# Install plugin via CLI
grafana-cli plugins install grafana-piechart-panel
grafana-cli plugins install grafana-worldmap-panel
```

### Custom Panel Plugin
```typescript
// custom-panel.tsx
import React from 'react';
import { PanelProps } from '@grafana/data';
import { SimpleOptions } from 'types';

interface Props extends PanelProps<SimpleOptions> {}

export const SimplePanel: React.FC<Props> = ({ options, data, width, height }) => {
  return (
    <div style={{ width, height }}>
      <h3>{options.text}</h3>
      <div>
        {data.series.map((series, index) => (
          <div key={index}>
            {series.name}: {series.fields[0].values.get(0)}
          </div>
        ))}
      </div>
    </div>
  );
};
```

## 🔧 Best Practices

### Dashboard Design
1. **Keep It Simple** - Focus on key metrics
2. **Use Consistent Colors** - Establish color coding standards
3. **Group Related Metrics** - Organize panels logically
4. **Add Context** - Include descriptions and units
5. **Test on Different Screens** - Ensure mobile compatibility

### Performance Optimization
1. **Limit Data Points** - Use appropriate time ranges
2. **Optimize Queries** - Use efficient data source queries
3. **Cache Dashboards** - Enable dashboard caching
4. **Use Variables** - Reduce dashboard duplication
5. **Monitor Performance** - Track dashboard load times

### Security
1. **Use HTTPS** - Enable SSL/TLS encryption
2. **Implement RBAC** - Role-based access control
3. **Secure Data Sources** - Use credentials management
4. **Regular Updates** - Keep Grafana updated
5. **Audit Logs** - Monitor user activities

## 🚀 Advanced Use Cases

### Multi-Tenant Setup
```yaml
# multi-tenant.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-multitenant
data:
  grafana.ini: |
    [users]
    allow_sign_up = false
    auto_assign_org = true
    auto_assign_org_role = Viewer
    
    [auth.anonymous]
    enabled = false
    
    [auth.proxy]
    enabled = true
    header_name = X-WEBAUTH-USER
    header_property = username
    auto_sign_up = true
```

### High Availability
```yaml
# ha-setup.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 3
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        env:
        - name: GF_DATABASE_TYPE
          value: "postgres"
        - name: GF_DATABASE_HOST
          value: "postgres:5432"
        - name: GF_DATABASE_NAME
          value: "grafana"
        - name: GF_DATABASE_USER
          value: "grafana"
        - name: GF_DATABASE_PASSWORD
          value: "grafana"
```

## 🐛 Troubleshooting

### Common Issues
1. **Dashboard Not Loading** - Check data source connectivity
2. **Slow Performance** - Optimize queries and reduce data points
3. **Authentication Issues** - Verify user permissions and settings
4. **Plugin Errors** - Check plugin compatibility and logs
5. **Memory Issues** - Monitor resource usage and limits

### Debugging Tools
```bash
# Check Grafana logs
docker logs grafana

# Test data source connectivity
curl -u admin:admin http://localhost:3000/api/datasources

# Verify plugin installation
grafana-cli plugins ls

# Check configuration
grafana-cli admin reset-admin-password admin
```

## 📚 Learning Resources

### Official Documentation
- [Grafana Documentation](https://grafana.com/docs/)
- [Dashboard Tutorials](https://grafana.com/tutorials/)
- [Plugin Development](https://grafana.com/docs/grafana/latest/developers/plugins/)

### Community Resources
- [GitHub Repository](https://github.com/grafana/grafana)
- [Community Forum](https://community.grafana.com/)
- [Slack Channel](https://grafana.slack.com/)

## 🔄 Updates and Roadmap

### Recent Updates
- **v10.0.0** - New panel library and improved performance
- **v9.5.0** - Enhanced alerting and notification features
- **v9.0.0** - New visualization types and data source improvements

### Upcoming Features
- **v10.1.0** - Advanced query builder
- **v10.2.0** - Enhanced collaboration features
- **v10.3.0** - New panel types and visualizations

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow TypeScript/React coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/grafana/grafana/blob/main/LICENSE) file for details.

## 🙏 Acknowledgments

- **Grafana Labs** for creating and maintaining Grafana
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements
- **Plugin Developers** for extending Grafana's capabilities

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#grafana-)

</div>

