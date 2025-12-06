# Troubleshooting SMon Prometheus Integration

## Common Issues and Solutions

### 1. Metrics Endpoint Not Accessible
**Problem**: Prometheus cannot scrape metrics from `/metrics` endpoint.

**Solutions**:
- Verify SMon is running: `pm2 status`
- Check if port 3000 is accessible: `curl http://localhost:3000/metrics`
- Ensure firewall allows port 3000
- Check application logs: `pm2 logs smon`

### 2. Authentication Errors
**Problem**: Getting 401 Unauthorized when accessing `/metrics`.

**Solution**: The `/metrics` endpoint should be publicly accessible. Verify the `requireAuth` middleware in `app.js` excludes `/metrics`:

```javascript
if (req.path === '/metrics') {
  return next();
}
```

### 3. Missing Metrics in Prometheus
**Problem**: Some metrics are not appearing in Prometheus.

**Solutions**:
- Check if monitoring data exists in SMon (devices, ping targets, etc.)
- Verify metrics format is correct
- Restart Prometheus after configuration changes
- Check Prometheus targets status in web UI

### 4. Grafana Dashboard Shows No Data
**Problem**: Grafana panels show "No data".

**Solutions**:
- Verify Prometheus data source is configured correctly
- Check query syntax in Grafana panels
- Ensure metric names match (case-sensitive)
- Check time range in Grafana

### 5. High Resource Usage
**Problem**: SMon consumes too much CPU/memory after enabling metrics.

**Solutions**:
- Increase scrape interval in Prometheus config
- Optimize metrics collection in SMon
- Monitor system resources during peak times

### 6. Alerting Not Working
**Problem**: Prometheus alerts are not firing.

**Solutions**:
- Verify alerting rules syntax
- Check Alertmanager configuration
- Test alert expressions in Prometheus web UI
- Ensure notification channels are configured

## Debug Commands

### Test Metrics Endpoint
```bash
curl -s http://localhost:3000/metrics | head -20
```

### Check Prometheus Targets
```bash
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.labels.job=="smon")'
```

### Validate Prometheus Config
```bash
promtool check config /path/to/prometheus.yml
```

### Check Grafana Data Source
Access Grafana → Configuration → Data Sources → Prometheus → Test

## Performance Tuning

### Prometheus Configuration
- Increase `scrape_interval` for less frequent polling
- Use `scrape_timeout` to prevent hanging scrapes
- Configure `evaluation_interval` for alert checking frequency

### SMon Optimization
- Cache expensive metric calculations
- Implement metric collection rate limiting
- Use background processing for heavy computations

## Support

If issues persist:
1. Check application logs for errors
2. Verify all configuration files are valid YAML/JSON
3. Test individual components (SMon, Prometheus, Grafana)
4. Review network connectivity between components