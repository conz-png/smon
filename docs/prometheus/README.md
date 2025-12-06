# SMon Prometheus Exporter Documentation

## Overview

SMon provides a built-in Prometheus exporter that exposes monitoring metrics for integration with Grafana and other monitoring systems.

## Endpoint

**URL:** `http://your-smon-server:3000/metrics`

**Method:** GET

**Authentication:** None required (public endpoint)

## Available Metrics

### System Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `smon_system_uptime_seconds` | gauge | System uptime in seconds | - |
| `smon_system_cpu_usage_percent` | gauge | CPU usage percentage | - |
| `smon_system_memory_used_bytes` | gauge | Memory used in bytes | - |
| `smon_system_memory_total_bytes` | gauge | Total memory in bytes | - |

### Device Monitoring Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `smon_device_status` | gauge | Device status (1=up, 0=down) | `device_id`, `device_name`, `device_host` |

### Ping Monitoring Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `smon_ping_target_status` | gauge | Ping target status (1=alive, 0=dead) | `target_id`, `target_name`, `target_host`, `group` |
| `smon_ping_target_latency_ms` | gauge | Ping target latency in milliseconds | `target_id`, `target_name`, `target_host`, `group` |
| `smon_ping_target_packet_loss_percent` | gauge | Ping target packet loss percentage | `target_id`, `target_name`, `target_host`, `group` |

### Website Monitoring Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `smon_website_status` | gauge | Website status (1=up, 0=down) | `target_id`, `target_name`, `target_url`, `group` |
| `smon_website_response_time_ms` | gauge | Website response time in milliseconds | `target_id`, `target_name`, `target_url`, `group` |
| `smon_website_ssl_expiry_days` | gauge | SSL certificate expiry days remaining | `target_id`, `target_name`, `target_url`, `group` |

### Domain Monitoring Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `smon_domain_status` | gauge | Domain status (1=active, 0=expired) | `domain_id`, `domain_name`, `registrar`, `group` |
| `smon_domain_expiry_days` | gauge | Domain expiry days remaining | `domain_id`, `domain_name`, `registrar`, `group` |
| `smon_domain_auto_renew` | gauge | Domain auto-renew status (1=enabled, 0=disabled) | `domain_id`, `domain_name`, `registrar`, `group` |

### Event Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `smon_events_total` | counter | Total number of events | - |
| `smon_events_by_severity` | gauge | Events by severity level | `severity` (critical, warning, error, info) |

## Prometheus Configuration

Add the following job to your `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'smon'
    static_configs:
      - targets: ['your-smon-server:3000']
    scrape_interval: 30s
    metrics_path: '/metrics'
```

## Grafana Dashboard Examples

### System Overview Panel

```json
{
  "title": "System CPU Usage",
  "type": "stat",
  "targets": [
    {
      "expr": "smon_system_cpu_usage_percent",
      "legendFormat": "CPU Usage"
    }
  ]
}
```

### Device Status Panel

```json
{
  "title": "Device Status",
  "type": "table",
  "targets": [
    {
      "expr": "smon_device_status",
      "legendFormat": "{{device_name}}"
    }
  ]
}
```

### Ping Latency Graph

```json
{
  "title": "Ping Latency",
  "type": "graph",
  "targets": [
    {
      "expr": "smon_ping_target_latency_ms{target_name=~\"$target\"}",
      "legendFormat": "{{target_name}}"
    }
  ]
}
```

### Website Response Time

```json
{
  "title": "Website Response Time",
  "type": "graph",
  "targets": [
    {
      "expr": "smon_website_response_time_ms",
      "legendFormat": "{{target_name}}"
    }
  ]
}
```

### Domain Expiry Alert

```json
{
  "title": "Domains Expiring Soon",
  "type": "table",
  "targets": [
    {
      "expr": "smon_domain_expiry_days < 30",
      "legendFormat": "{{domain_name}}"
    }
  ]
}
```

## Sample Queries

### System Health
```promql
# CPU Usage
smon_system_cpu_usage_percent

# Memory Usage Percentage
(smon_system_memory_used_bytes / smon_system_memory_total_bytes) * 100

# System Uptime in days
smon_system_uptime_seconds / 86400
```

### Device Monitoring
```promql
# Count of online devices
count(smon_device_status == 1)

# Device status overview
smon_device_status
```

### Network Monitoring
```promql
# Average ping latency by target
avg by (target_name) (smon_ping_target_latency_ms)

# Packet loss percentage
smon_ping_target_packet_loss_percent{target_name=~"$target"}

# Ping targets that are down
smon_ping_target_status == 0
```

### Website Monitoring
```promql
# Websites that are down
smon_website_status == 0

# SSL certificates expiring in 30 days
smon_website_ssl_expiry_days < 30

# Average response time
avg(smon_website_response_time_ms)
```

### Domain Monitoring
```promql
# Domains expiring in 30 days
smon_domain_expiry_days < 30

# Count of expired domains
count(smon_domain_status == 0)

# Domains without auto-renew
smon_domain_auto_renew == 0
```

## Alerting Rules

```yaml
groups:
  - name: smon_alerts
    rules:
      - alert: DeviceDown
        expr: smon_device_status == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Device {{ $labels.device_name }} is down"
          description: "Device {{ $labels.device_name }} ({{ $labels.device_host }}) has been down for 5 minutes"

      - alert: HighPingLatency
        expr: smon_ping_target_latency_ms > 100
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High ping latency for {{ $labels.target_name }}"
          description: "Ping latency to {{ $labels.target_name }} is {{ $value }}ms"

      - alert: WebsiteDown
        expr: smon_website_status == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Website {{ $labels.target_name }} is down"
          description: "Website {{ $labels.target_url }} is not responding"

      - alert: SSLCertificateExpiring
        expr: smon_website_ssl_expiry_days < 30
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "SSL certificate expiring soon"
          description: "SSL certificate for {{ $labels.target_name }} expires in {{ $value }} days"

      - alert: DomainExpiring
        expr: smon_domain_expiry_days < 30
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "Domain expiring soon"
          description: "Domain {{ $labels.domain_name }} expires in {{ $value }} days"
```

## Troubleshooting

### Metrics Not Appearing

1. Check if SMon is running: `pm2 status`
2. Verify endpoint accessibility: `curl http://localhost:3000/metrics`
3. Check Prometheus scrape configuration
4. Ensure no firewall blocking port 3000

### Missing Data

- Device metrics only appear for configured SNMP devices
- Ping metrics only appear for enabled ping targets with recent data
- Website metrics only appear for enabled website targets with recent checks
- Domain metrics only appear for enabled domain targets

### Performance Considerations

- Metrics are generated on-demand when scraped
- Large numbers of targets may impact response time
- Consider increasing scrape intervals for high-volume deployments</content>
<parameter name="filePath">/home/dionipe/graphts/docs/prometheus/README.md