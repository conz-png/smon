# Deployment and Scaling Guide for SMon Prometheus Integration

## Single Server Deployment

### Basic Setup
1. Install SMon on your monitoring server
2. Configure Prometheus to scrape the `/metrics` endpoint
3. Set up Grafana with the provided dashboard

### Directory Structure
```
/opt/smon/
├── app.js
├── package.json
├── docs/
│   └── prometheus/
│       ├── README.md
│       ├── prometheus.yml
│       ├── grafana-dashboard.json
│       ├── alerting-rules.yml
│       ├── troubleshooting.md
│       ├── promql-queries.md
│       └── deployment.md
```

## Multi-Server Deployment

### Architecture Overview
```
[ SMon Server 1 ]     [ SMon Server 2 ]     [ SMon Server 3 ]
       |                       |                       |
       └───────────────────────┼───────────────────────┘
                               |
                       [ Prometheus Server ]
                               |
                       [ Grafana Server ]
```

### Configuration for Multiple Instances
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'smon-cluster'
    static_configs:
      - targets:
          - 'smon-01.company.com:3000'
          - 'smon-02.company.com:3000'
          - 'smon-03.company.com:3000'
    scrape_interval: 30s
    metrics_path: '/metrics'
    labels:
      cluster: 'production'
```

## Docker Deployment

### Dockerfile for SMon
```dockerfile
FROM node:16-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
EXPOSE 3000

CMD ["npm", "start"]
```

### Docker Compose Setup
```yaml
version: '3.8'
services:
  smon:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - ./data:/app/data
    restart: unless-stopped

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

## Kubernetes Deployment

### SMon Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: smon
spec:
  replicas: 3
  selector:
    matchLabels:
      app: smon
  template:
    metadata:
      labels:
        app: smon
    spec:
      containers:
      - name: smon
        image: your-registry/smon:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Service for SMon
```yaml
apiVersion: v1
kind: Service
metadata:
  name: smon-service
spec:
  selector:
    app: smon
  ports:
  - port: 3000
    targetPort: 3000
  type: ClusterIP
```

### Prometheus ServiceMonitor
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: smon-monitor
  labels:
    team: backend
spec:
  selector:
    matchLabels:
      app: smon
  endpoints:
  - port: web
    path: /metrics
    interval: 30s
```

## Load Balancing

### Nginx Configuration
```nginx
upstream smon_backend {
    server smon-01:3000;
    server smon-02:3000;
    server smon-03:3000;
}

server {
    listen 80;
    server_name monitoring.company.com;

    location / {
        proxy_pass http://smon_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /metrics {
        proxy_pass http://smon_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## High Availability Setup

### Prometheus Federation
For large-scale deployments, use Prometheus federation:

```yaml
# Global Prometheus (federation receiver)
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job="smon"}'
    static_configs:
      - targets:
        - 'prometheus-region-1:9090'
        - 'prometheus-region-2:9090'
```

### Alertmanager Clustering
```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.company.com:587'
  smtp_from: 'alerts@company.com'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'email'
  routes:
  - match:
      severity: critical
    receiver: 'pagerduty'

receivers:
- name: 'email'
  email_configs:
  - to: 'ops@company.com'
- name: 'pagerduty'
  pagerduty_configs:
  - service_key: 'your-pagerduty-key'
```

## Scaling Considerations

### Vertical Scaling
- Increase CPU/memory limits based on monitoring load
- Use SSD storage for better I/O performance
- Monitor resource usage and adjust accordingly

### Horizontal Scaling
- Deploy multiple SMon instances behind load balancer
- Use shared database for configuration synchronization
- Implement proper session management

### Database Scaling
- Use connection pooling
- Implement read replicas for high read loads
- Consider database sharding for very large deployments

## Monitoring the Monitoring Stack

### Prometheus Metrics
Monitor Prometheus itself:
```
prometheus_tsdb_head_series
prometheus_target_scrapes_sample_limit
prometheus_target_interval_length_seconds
```

### Grafana Metrics
Monitor Grafana performance:
```
grafana_stat_totals_dashboard
grafana_stat_totals_datasource
```

### SMon Metrics
Monitor SMon performance:
```
smon_system_cpu_usage_percent
smon_system_memory_usage_percent
process_resident_memory_bytes{job="smon"}
```

## Backup and Recovery

### Configuration Backup
```bash
# Backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf smon_config_$DATE.tar.gz /opt/smon/data/
tar -czf prometheus_config_$DATE.tar.gz /etc/prometheus/
tar -czf grafana_config_$DATE.tar.gz /var/lib/grafana/
```

### Database Backup
```bash
# MongoDB backup (if using MongoDB)
mongodump --db smon --out /backup/smon_$DATE

# PostgreSQL backup (if using PostgreSQL)
pg_dump smon > /backup/smon_$DATE.sql
```

### Recovery Procedure
1. Restore configuration files
2. Restore database
3. Restart services in order: database → SMon → Prometheus → Grafana
4. Verify metrics collection and alerting

## Security Considerations

### Network Security
- Use HTTPS for all external access
- Implement proper firewall rules
- Use VPN for internal communication

### Authentication
- Enable authentication for Grafana
- Use OAuth/SSO integration
- Implement role-based access control

### Data Encryption
- Encrypt sensitive configuration data
- Use TLS for database connections
- Implement proper secrets management

## Performance Optimization

### Prometheus Tuning
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Storage settings
storage:
  tsdb:
    retention: 30d
    max_block_duration: 2h
    min_block_duration: 2h
```

### SMon Optimization
- Implement caching for expensive operations
- Use background workers for heavy tasks
- Optimize database queries
- Implement rate limiting for API endpoints

## Troubleshooting Scaling Issues

### Common Problems
1. **High latency**: Check network connectivity, increase timeouts
2. **Memory issues**: Monitor heap usage, increase memory limits
3. **Database bottlenecks**: Optimize queries, add indexes
4. **Alert storms**: Tune alerting rules, implement alert aggregation

### Debug Commands
```bash
# Check cluster status
kubectl get pods -l app=smon

# Monitor resource usage
kubectl top pods -l app=smon

# Check logs
kubectl logs -f deployment/smon

# Prometheus query debugging
curl "http://prometheus:9090/api/v1/query?query=up"
```