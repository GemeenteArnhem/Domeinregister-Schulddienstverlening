# Monitoring & Logging

Observability stack voor production Schulddienstverlening Register.

---

## Logging Architecture

### Log Sources

1. **Application Logs** (FastAPI / Node.js)
   - STDOUT/STDERR → Docker → Filebeat/Logstash

2. **Database Logs** (PostgreSQL)
   - Query logs, errors → log_collect

3. **API Access Logs** (NGINX)
   - HTTP requests → access.log → Filebeat

4. **Audit Trail** (Application)
   - Business events → PostgreSQL audit_log table

### ELK Stack (Elasticsearch, Logstash, Kibana)

```
┌──────────────┐      ┌──────────────┐     ┌───────────────┐
│  Docker      │ ──→  │  Filebeat    │ ──→ │ Elasticsearch │
│  Container   │      │              │     │               │
│  Logs        │      └──────────────┘     └───────────────┘
└──────────────┘                                   ▲
                                                   │
                      ┌──────────────────────────┘
                      │
                      ▼
            ┌──────────────────┐
            │    Kibana        │
            │  (Dashboard)     │
            └──────────────────┘
```

---

## Application Logging

### Log Format (Structured JSON)

```json
{
  "timestamp": "2026-05-26T13:20:14Z",
  "level": "INFO",
  "service": "schulddienstverlening-api",
  "message": "Dossier created",
  "request_id": "uuid-xxx",
  "user_id": "user-123",
  "action": "CREATE",
  "resource_type": "DOSSIER",
  "resource_id": "dossier-xyz",
  "duration_ms": 145,
  "status_code": 201,
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0...",
  "environment": "production"
}
```

### Log Levels

| Level | Use Case | Example |
|-------|----------|---------|
| DEBUG | Development only | Variable values, slow queries |
| INFO | Key events | Dossier created, user logged in |
| WARNING | Unexpected but handled | Retry attempt #3, deprecated endpoint |
| ERROR | Application error | Validation failed, DB connection lost |
| CRITICAL | System failure | Out of disk, auth service down |

### Python/FastAPI Example

```python
import logging
import json
from pythonjsonlogger import jsonlogger

logger = logging.getLogger()
logHandler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter()
logHandler.setFormatter(formatter)
logger.addHandler(logHandler)
logger.setLevel(logging.INFO)

@app.post("/schulddossiers")
async def create_dossier(payload: DossierCreate, request: Request):
    logger.info(
        "Dossier created",
        extra={
            "request_id": request.headers.get("X-Request-ID"),
            "user_id": get_current_user_id(),
            "action": "CREATE",
            "resource_type": "DOSSIER",
            "duration_ms": response.elapsed.total_seconds() * 1000
        }
    )
    return {"id": dossier.id}
```

---

## Metrics & Alerting

### Prometheus Metrics

Key metrics to expose:

```
# API Performance
http_request_duration_seconds{method="GET", endpoint="/schulddossiers", status="200"} 0.145
http_request_total{method="GET", endpoint="/schulddossiers", status="200"} 12345
http_request_size_bytes{method="POST", endpoint="/schulddossiers"} 1024

# Database
db_query_duration_seconds{query="SELECT * FROM schulddossiers"} 0.035
db_connection_pool_available{} 8
db_connection_pool_total{} 10
db_slow_query_total{} 3

# Business Metrics
dossier_create_total{status="success"} 145
dossier_create_total{status="error"} 2
schuld_bedrag_total{} 250000.00

# System
process_resident_memory_bytes{} 512000000
process_cpu_seconds_total{} 1234.56
```

### Prometheus Scrape Config

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'schulddienstverlening-api'
    static_configs:
      - targets: ['localhost:8000']
    metrics_path: '/metrics'
    scrape_interval: 15s
```

### Alert Rules (AlertManager)

```yaml
# alerts.yml
groups:
  - name: schulddienstverlening
    rules:
      # API Health
      - alert: HighErrorRate
        expr: rate(http_request_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        annotations:
          summary: "{{ $labels.endpoint }} has high error rate"

      # Database Health
      - alert: DatabaseDown
        expr: up{job="postgres"} == 0
        for: 1m
        annotations:
          summary: "PostgreSQL database is down"

      # Performance
      - alert: SlowQueries
        expr: histogram_quantile(0.95, db_query_duration_seconds) > 1.0
        for: 5m
        annotations:
          summary: "95th percentile DB query > 1 second"

      # Capacity
      - alert: DiskSpaceRunningOut
        expr: (node_filesystem_avail / node_filesystem_size) < 0.2
        for: 10m
        annotations:
          summary: "Disk < 20% available"

      # Security
      - alert: FailedLoginAttempts
        expr: rate(failed_login_total[5m]) > 10
        for: 1m
        annotations:
          summary: "High rate of failed logins"
```

---

## Audit Logging

### Immutable Audit Trail

Business events logged to PostgreSQL (GDPR, BIO2):

```sql
-- Query: All actions by user
SELECT * FROM audit_log 
WHERE actor_id = 'user-123'
ORDER BY timestamp DESC;

-- Query: All changes to dossier
SELECT * FROM audit_log
WHERE entity_type = 'DOSSIER' AND entity_id = 'dossier-xyz'
ORDER BY timestamp DESC;

-- Query: Export data for subject (GDPR Art. 15)
SELECT * FROM audit_log
WHERE actor_id = 'burger-123' OR entity_id IN (
  SELECT id FROM schulddossier WHERE burger_id = '123456789'
)
ORDER BY timestamp;
```

---

## Uptime & SLA Monitoring

### Synthetic Monitoring

Regular tests to ensure availability:

```python
# tests/synthetic_test.py
import requests
from datetime import datetime

def test_api_health():
    """Synthetic test run every 5 minutes"""
    response = requests.get('https://api.schulddienstverlening.nl/health')
    assert response.status_code == 200, f"API down: {response.text}"
    
    # Log result
    log_metric("synthetic_test", response.elapsed.total_seconds())

def test_critical_endpoints():
    """Test core functionality"""
    response = requests.get(
        'https://api.schulddienstverlening.nl/v1/schulddossiers',
        headers={'Authorization': f'Bearer {TEST_TOKEN}'}
    )
    assert response.status_code == 200
```

### Uptime Dashboard

Target: **99.5% SLA** (monthly: max 3.6 hours downtime)

```
• Last 24 hours: 99.95% ✓
• Last 7 days: 99.87% ✓
• Last 30 days: 99.52% ✓
• Last 90 days: 99.48% ✗ (33 minutes downtime)
```

---

## Dashboards (Grafana)

### Key Dashboards

1. **Health Overview**
   - Uptime %
   - API response time (p50, p95, p99)
   - Error rate %
   - Active users

2. **Database Performance**
   - Query latency distribution
   - Slow query log
   - Connection pool usage
   - Storage size

3. **Business Metrics**
   - Dossiers created (24h)
   - Success rate of maatregelen
   - Average schuldbedrag

4. **Security**
   - Failed login attempts
   - Unusual API patterns
   - Data export requests

---

## Log Retention & Archival

### Policy

| Log Type | Hot Storage | Warm Archive | Retention |
|----------|-------------|-----------------|-----------|
| Application | 30 days | S3 | 90 days |
| Access logs | 14 days | S3 | 1 year |
| Audit trail | 7 years (DB) | Backup | 10 years |
| Metrics | 15 days (Prometheus) | Thanos | 1 year |

### Cleanup Script

```bash
#!/bin/bash
# cleanup-logs.sh - Run daily

# Archive old logs
find /var/log/schulddienstverlening -mtime +30 | xargs \
  tar czf /archive/logs-$(date +%Y%m%d).tar.gz && \
  rm -rf /var/log/schulddienstverlening/*

# Delete old archives
find /archive -mtime +90 -delete

# Compress backups for cost
find /backups -mtime +7 ! -name "*.gz" | xargs gzip

echo "Archived and cleaned logs at $(date)"
```

---

## On-Call Runbooks

### If API is Down

1. Check status dashboard: https://status.schulddienstverlening.nl
2. SSH to prod: `ssh deploy@api.schulddienstverlening.nl`
3. Check logs: `docker logs -f schulddienstverlening-api`
4. Check DB: `docker exec db pg_isready -U schulddienstverlening`
5. Restart: `docker-compose restart api`
6. Verify: `curl https://api.schulddienstverlening.nl/health`
7. Notify stakeholders

### If Database is Slow

1. Check active query count: `SELECT count(*) FROM pg_stat_activity WHERE state='active'`
2. Kill long-running queries: `SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE duration > 300000`
3. Run VACUUM: `VACUUM ANALYZE schulddossier`
4. Check indexes: `SELECT * FROM pg_stat_user_indexes WHERE idx_scan < 100`
5. Scale read replicas

---

**Document version**: 1.0 | **Last update**: Mei 2026
