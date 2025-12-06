# Useful PromQL Queries for SMon Metrics

## System Metrics

### Current CPU Usage
```
smon_system_cpu_usage_percent
```

### Current Memory Usage
```
smon_system_memory_usage_percent
```

### System Uptime
```
smon_system_uptime_seconds / 3600  # Convert to hours
```

## Device Monitoring

### Online Devices Count
```
count(smon_device_status == 1)
```

### Offline Devices
```
smon_device_status == 0
```

### Device Status by ID
```
smon_device_status{device_id="device123"}
```

## Ping Monitoring

### Average Ping Response Time
```
avg(smon_ping_response_time_ms)
```

### Ping Targets Down
```
smon_ping_status == 0
```

### Packet Loss Percentage
```
smon_ping_packet_loss_percent
```

### Ping Response Time by Target
```
smon_ping_response_time_ms{target="8.8.8.8"}
```

## Website Monitoring

### Website Response Time Average
```
avg(smon_website_response_time_ms)
```

### Websites Down
```
smon_website_status == 0
```

### SSL Certificates Expiring Soon (< 30 days)
```
smon_website_ssl_expiry_days < 30
```

### Website Status by Target
```
smon_website_status{target="https://example.com"}
```

## Domain Monitoring

### Domains Expiring Soon (< 30 days)
```
smon_domain_expiry_days < 30
```

### Expired Domains
```
smon_domain_expiry_days < 0
```

### Auto-renew Status
```
smon_domain_auto_renew
```

### Domain Status by Name
```
smon_domain_status{domain="example.com"}
```

## Event Monitoring

### Total Events
```
smon_events_total
```

### Events by Severity
```
smon_events_total{severity="error"}
smon_events_total{severity="warning"}
smon_events_total{severity="info"}
```

### Recent Events (last 1 hour)
```
increase(smon_events_total[1h])
```

## Advanced Queries

### Device Availability Percentage (last 24h)
```
avg_over_time(up(smon_device_status)[24h]) * 100
```

### Ping Success Rate (last 1h)
```
rate(smon_ping_status == 1)[1h]) * 100
```

### Website Uptime Percentage (last 24h)
```
(avg_over_time(smon_website_status[24h]) * 100)
```

### Top 5 Slowest Websites
```
topk(5, avg_over_time(smon_website_response_time_ms[1h]))
```

### Domains Expiring in Next 7 Days
```
smon_domain_expiry_days < 7 and smon_domain_expiry_days > 0
```

### System Resource Trends (last 24h)
```
avg_over_time(smon_system_cpu_usage_percent[24h])
avg_over_time(smon_system_memory_usage_percent[24h])
```

## Alerting Queries

### Devices Offline for > 5 minutes
```
smon_device_status == 0 and on() absent_over_time(smon_device_status == 1 [5m])
```

### Websites Down for > 2 minutes
```
smon_website_status == 0 and on() absent_over_time(smon_website_status == 1 [2m])
```

### High CPU Usage Trend
```
predict_linear(smon_system_cpu_usage_percent[1h], 3600) > 90
```

## Dashboard Queries

### Service Health Overview
```
up{job="smon"}
```

### All Monitoring Targets Status
```
count by (job, instance) (up)
```

### Error Rate by Service
```
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```

## Custom Calculations

### Calculate MTTR (Mean Time To Recovery)
```
# Calculate downtime periods and average recovery time
# This requires custom recording rules in Prometheus
```

### Service Availability SLA
```
# 99.9% uptime calculation
(avg_over_time(up[30d]) * 100) >= 99.9
```

## Query Best Practices

1. **Use appropriate time ranges**: `[5m]`, `[1h]`, `[24h]`, `[7d]`
2. **Filter by labels**: `{job="smon"}`, `{instance="localhost:3000"}`
3. **Use aggregation functions**: `avg()`, `sum()`, `count()`, `max()`, `min()`
4. **Rate calculations**: `rate(metric[5m])`, `increase(metric[1h])`
5. **Time-based aggregations**: `avg_over_time(metric[1h])`
6. **Top/bottom queries**: `topk(5, metric)`, `bottomk(3, metric)`

## Grafana Variables

Create dashboard variables for dynamic filtering:

- `device_id`: `label_values(smon_device_status, device_id)`
- `target`: `label_values(smon_ping_response_time_ms, target)`
- `domain`: `label_values(smon_domain_expiry_days, domain)`

## Recording Rules

Add these to your Prometheus configuration for better performance:

```yaml
groups:
  - name: smon_recording_rules
    rules:
      - record: smon:devices_online:count
        expr: count(smon_device_status == 1)

      - record: smon:websites_up:percent
        expr: (count(smon_website_status == 1) / count(smon_website_status)) * 100

      - record: smon:avg_response_time:ms
        expr: avg(smon_website_response_time_ms)
```