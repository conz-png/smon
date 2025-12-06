# SMon Prometheus Integration Documentation Index

## 📋 Overview

This documentation provides comprehensive guidance for integrating SMon with Prometheus and Grafana for advanced monitoring and alerting capabilities.

## 📚 Documentation Files

### Core Documentation
- **[README.md](README.md)** - Main setup guide and overview
- **[CHANGELOG.md](CHANGELOG.md)** - Version history and release notes
- **[LICENSE.md](LICENSE.md)** - Licensing information and terms

### Configuration Files
- **[prometheus.yml](prometheus.yml)** - Example Prometheus configuration
- **[grafana-dashboard.json](grafana-dashboard.json)** - Grafana dashboard template
- **[alerting-rules.yml](alerting-rules.yml)** - Prometheus alerting rules

### Guides and Tutorials
- **[promql-queries.md](promql-queries.md)** - Useful PromQL queries and examples
- **[troubleshooting.md](troubleshooting.md)** - Common issues and solutions
- **[deployment.md](deployment.md)** - Scaling and production deployment

## 🚀 Quick Start

1. **Setup SMon**: Ensure SMon is running with monitoring enabled
2. **Configure Prometheus**: Use `prometheus.yml` as a template
3. **Import Dashboard**: Load `grafana-dashboard.json` in Grafana
4. **Configure Alerts**: Add `alerting-rules.yml` to Prometheus
5. **Troubleshoot**: Refer to `troubleshooting.md` for issues

## 📊 Metrics Reference

### System Metrics
- CPU usage percentage
- Memory usage percentage
- System uptime

### Device Monitoring
- Device online/offline status
- Device-specific metrics

### Network Monitoring
- Ping response times
- Packet loss percentages
- Target availability

### Web Monitoring
- Website response times
- SSL certificate expiry
- HTTP status codes

### Domain Monitoring
- Domain expiry dates
- Auto-renewal status
- DNS resolution status

### Event Monitoring
- Total events by severity
- Event trends and patterns

## 🔧 Configuration Examples

### Basic Prometheus Setup
```yaml
scrape_configs:
  - job_name: 'smon'
    static_configs:
      - targets: ['localhost:3000']
    metrics_path: '/metrics'
```

### Grafana Data Source
```
Type: Prometheus
URL: http://localhost:9090
Access: Server
```

### Alertmanager Configuration
```yaml
route:
  group_by: ['alertname']
  receiver: 'email'
  routes:
  - match:
      severity: critical
    receiver: 'pagerduty'
```

## 📈 Dashboard Panels

### System Health
- CPU and memory usage graphs
- System uptime indicators

### Service Monitoring
- Device status overview
- Ping response times
- Website availability

### Security & Compliance
- SSL certificate expiry alerts
- Domain renewal notifications

### Alert Summary
- Active alerts by severity
- Alert trends over time

## 🔍 PromQL Query Examples

### Basic Queries
```
# Current CPU usage
smon_system_cpu_usage_percent

# Online devices count
count(smon_device_status == 1)

# Websites down
smon_website_status == 0
```

### Advanced Queries
```
# Service availability (last 24h)
avg_over_time(up[24h]) * 100

# Top 5 slowest websites
topk(5, smon_website_response_time_ms)

# Domains expiring soon
smon_domain_expiry_days < 30
```

## 🚨 Alerting Rules

### Critical Alerts
- Device offline
- Website down
- Domain expired

### Warning Alerts
- High resource usage
- SSL expiring soon
- Slow response times

### Info Alerts
- System notifications
- Maintenance reminders

## 🏗️ Deployment Options

### Single Server
- Basic setup for small environments
- All components on one server

### Multi-Server
- Distributed architecture
- Load balancing and high availability

### Containerized
- Docker and Kubernetes deployments
- Orchestrated scaling

### Cloud Deployments
- AWS, GCP, Azure configurations
- Managed services integration

## 🐛 Troubleshooting

### Common Issues
- Metrics endpoint not accessible
- Authentication problems
- Missing data in dashboards

### Diagnostic Tools
- Endpoint testing commands
- Log analysis
- Configuration validation

### Performance Issues
- High resource usage
- Slow query responses
- Alert storms

## 📞 Support

### Documentation Resources
- Detailed guides for each component
- Configuration examples
- Best practices

### Community Support
- GitHub issues and discussions
- User forums and communities
- Knowledge base articles

### Professional Services
- Commercial support options
- Consulting and training
- Custom development

## 🔗 Related Links

### External Resources
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [PromQL Reference](https://prometheus.io/docs/prometheus/latest/querying/basics/)

### SMon Resources
- Main SMon documentation
- API reference
- Configuration guides

## 📝 Contributing

### Documentation
- Report issues or inaccuracies
- Suggest improvements
- Submit pull requests

### Development
- Code contributions
- Feature requests
- Bug reports

## 📋 Checklist

### Setup Verification
- [ ] SMon installed and running
- [ ] `/metrics` endpoint accessible
- [ ] Prometheus configured and scraping
- [ ] Grafana dashboard imported
- [ ] Alerting rules loaded
- [ ] Notifications configured

### Monitoring Coverage
- [ ] System metrics collected
- [ ] Device monitoring active
- [ ] Network monitoring configured
- [ ] Web services monitored
- [ ] Domain expiry tracked
- [ ] Events being logged

### Alerting Setup
- [ ] Critical alerts configured
- [ ] Warning thresholds set
- [ ] Notification channels working
- [ ] Alert routing rules defined

---

## 📊 Metrics Summary

| Metric Type | Count | Description |
|-------------|-------|-------------|
| System | 3 | CPU, Memory, Uptime |
| Device | 1 | Online/Offline status |
| Ping | 3 | Response time, packet loss, status |
| Website | 3 | Response time, SSL expiry, status |
| Domain | 3 | Expiry days, auto-renew, status |
| Events | 1 | Total by severity |
| **Total** | **14** | Comprehensive monitoring coverage |

## 🎯 Success Metrics

- **Uptime**: >99.9% service availability
- **Latency**: <30s metrics collection
- **Coverage**: 100% of monitored services
- **Alerts**: <1% false positive rate

---

*This documentation is continuously updated. Check for the latest version and contribute improvements to help the community.*