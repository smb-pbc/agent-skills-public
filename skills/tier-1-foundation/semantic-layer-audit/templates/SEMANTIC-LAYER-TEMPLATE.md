# Semantic Layer - Data Catalog

> Your AI agent's single source of truth for available data.

**Last Updated:** YYYY-MM-DD  
**GCP Project:** `your-project-id`

---

## Quick Reference

| Need | Source | Access |
|------|--------|--------|
| Sales data | `your_dataset.orders` | BigQuery |
| Customer info | `your_dataset.customers` | BigQuery |
| Real-time metrics | Service API | REST API |
| Credentials | Secret Manager | gcloud |

---

## BigQuery Datasets

### `your_dataset` (Primary)

Your main business data.

| Table | Description | Key Fields | Granularity | Rows |
|-------|-------------|------------|-------------|------|
| `orders` | All orders | order_id, customer_id, created_at, total | Per order | ~50k |
| `customers` | Customer profiles | customer_id, email, created_at | Per customer | ~10k |
| `daily_summary` | Aggregated metrics | date, revenue, order_count | Daily | ~1k |

**Common Queries:**

```sql
-- Daily revenue
SELECT 
  DATE(created_at) as date,
  SUM(total) as revenue,
  COUNT(*) as orders
FROM `project.dataset.orders`
WHERE created_at >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
GROUP BY 1
ORDER BY 1 DESC
```

### `analytics` (Optional)

Web/app analytics data.

| Table | Description | Key Fields | Granularity |
|-------|-------------|------------|-------------|
| `events` | User events | user_id, event_name, timestamp | Per event |
| `sessions` | User sessions | session_id, user_id, duration | Per session |

---

## API Integrations

### Service Name

- **Docs:** https://docs.example.com/api
- **Auth:** API key in Secret Manager (`service-api-key`)
- **Base URL:** `https://api.example.com/v1`
- **Rate Limits:** 100 req/min

**Key Endpoints:**
| Endpoint | Method | Use Case |
|----------|--------|----------|
| `/orders` | GET | Fetch orders |
| `/orders/{id}` | GET | Single order |
| `/customers` | GET | Customer list |

---

## Credentials Inventory

All credentials stored in GCP Secret Manager.

| Secret Name | Service | Notes |
|-------------|---------|-------|
| `service-api-key` | Main Service | Production API key |
| `oauth-client-id` | OAuth | Client credentials |
| `oauth-client-secret` | OAuth | Client credentials |

**Access pattern:**
```bash
gcloud secrets versions access latest --secret="secret-name" --project=your-project
```

---

## Service Accounts

| Email | Purpose | Key Permissions |
|-------|---------|-----------------|
| `bigquery-reader@project.iam.gserviceaccount.com` | BQ read access | BigQuery Data Viewer |
| `api-service@project.iam.gserviceaccount.com` | API integrations | Secret Accessor |

---

## Data Freshness

| Source | Update Frequency | Lag |
|--------|------------------|-----|
| `orders` | Real-time | < 1 min |
| `daily_summary` | Daily 2am UTC | ~24 hrs |
| `analytics.events` | Hourly | ~1 hr |

---

## Known Limitations

- **Historical data:** Orders before 2023 may be incomplete
- **Timezone:** All timestamps in UTC unless noted
- **PII:** Customer emails available but use carefully

---

## Discovery Log

Track new data patterns found during ad-hoc work.

| Date | Discovery | Source | Added |
|------|-----------|--------|-------|
| YYYY-MM-DD | Example pattern | Session | ✅ |

---

## Maintenance

- **Weekly:** Run `audit_infrastructure.py` to catch schema changes
- **Monthly:** Review and prune deprecated items
- **Quarterly:** Deep audit of all integrations
