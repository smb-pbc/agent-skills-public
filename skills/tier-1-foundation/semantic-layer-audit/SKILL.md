---
name: semantic-layer-audit
description: Audit and maintain a data semantic layer for AI agents. Discovers what data infrastructure exists (BigQuery, Snowflake, Redshift, Postgres, etc.) and keeps your data catalog current. Essential for any agent that needs to know "what data do I have access to?"
---

# Semantic Layer Audit

> **Why this matters:** AI agents can only use data they know about. This skill maintains a living data catalog so you never lose track of available datasets, APIs, or credentials.

## Step 1: Discover Data Infrastructure

**Before running any scripts, ask the user:**

> "Where does your data live? Do you have a data warehouse or database I should know about?"

Common setups:
| Platform | Signs to Look For |
|----------|-------------------|
| **BigQuery** | GCP project, `bq` CLI, `GOOGLE_APPLICATION_CREDENTIALS` |
| **Snowflake** | `snowsql` CLI, `SNOWFLAKE_ACCOUNT` env var |
| **Redshift** | AWS account, `psql` with Redshift endpoint |
| **Databricks** | Databricks workspace, Unity Catalog |
| **PostgreSQL** | `psql`, `DATABASE_URL`, connection strings |
| **None yet** | Help them set one up or document raw sources |

**If they don't know:** Check for existing credentials, connection strings, or environment variables that hint at a data warehouse.

## Step 2: Run Platform-Specific Audit

### For GCP/BigQuery Users

**Prerequisites:**
- gcloud CLI authenticated (`gcloud auth login`)
- `GCP_PROJECT_ID` environment variable set

```bash
export GCP_PROJECT_ID="your-project-id"
python3 scripts/audit_infrastructure.py > audit-results.json
```

### For Other Platforms

The included script is GCP-focused. For other platforms:

**Snowflake:**
```sql
-- List databases and schemas
SHOW DATABASES;
SHOW SCHEMAS IN DATABASE your_db;
SHOW TABLES IN SCHEMA your_db.your_schema;
```

**PostgreSQL/Redshift:**
```sql
-- List schemas and tables
SELECT schemaname, tablename FROM pg_tables WHERE schemaname NOT IN ('pg_catalog', 'information_schema');
```

**Databricks:**
```sql
-- Unity Catalog
SHOW CATALOGS;
SHOW SCHEMAS IN catalog_name;
SHOW TABLES IN catalog_name.schema_name;
```

Document findings manually in your SEMANTIC-LAYER.md using the template.

## Step 3: Document Everything

## When to Run

| Trigger | Action |
|---------|--------|
| "What data do we have?" | Full audit |
| "Audit data sources" | Full audit |
| After new integrations | Infrastructure scan |
| Weekly maintenance | Cron: catch schema changes |
| Before complex analysis | Verify sources exist |

## Audit Process

### 1. Infrastructure Scan

```bash
python3 scripts/audit_infrastructure.py
```

The script scans:
- **BigQuery:** All datasets, tables, and row counts
- **Secrets:** All Secret Manager secrets (credential inventory)
- **APIs:** All enabled Google Cloud APIs
- **Service Accounts:** All service accounts in the project

Output is JSON for easy processing.

### 2. Memory Pattern Discovery

Search your agent's memory for data access patterns not yet documented:

```bash
# Find BigQuery queries in recent sessions
memory_search "BigQuery query SELECT FROM"

# Find API usage patterns  
memory_search "API endpoint curl"

# Find undocumented data sources
memory_search "gcloud bq data"
```

Look for:
- Tables queried but not in your semantic layer doc
- API endpoints used successfully
- Join patterns between datasets
- Calculated fields or derived metrics

### 3. Gap Analysis

Compare audit results against your `SEMANTIC-LAYER.md`:

- [ ] New BigQuery tables not documented
- [ ] New secrets/credentials not listed
- [ ] APIs enabled but not documented
- [ ] Data sources in memory but not in catalog
- [ ] Deprecated tables still listed

### 4. Update Semantic Layer

Update your data catalog with:
- New tables (descriptions, key fields, granularity)
- New API integrations
- New credentials
- Deprecated items (mark or remove)
- Discovered query patterns

### 5. Track Discoveries

Log ad-hoc discoveries for future audits:

```markdown
## Discovery Log

| Date | Discovery | Source | Added |
|------|-----------|--------|-------|
| 2024-01-15 | New sales_daily table | Session | ✅ |
```

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GCP_PROJECT_ID` | Yes | Your GCP project ID |
| `SEMANTIC_LAYER_PATH` | No | Path to your semantic layer doc (default: `docs/SEMANTIC-LAYER.md`) |

### Customizing the Script

Edit `scripts/audit_infrastructure.py` to:
- Skip certain datasets (add to `SKIP_DATASETS`)
- Add custom API checks
- Adjust row count timeout

## Files

| File | Purpose |
|------|---------|
| `scripts/audit_infrastructure.py` | Infrastructure scanner |
| `templates/SEMANTIC-LAYER-TEMPLATE.md` | Starter template for your data catalog |

## Cron Schedule (Recommended)

Weekly audit to catch schema changes:

```
0 6 * * 0
```

Text: "Run semantic layer audit - check for new BigQuery tables, API changes, and undocumented data patterns"

## Example Output

```json
{
  "project_id": "your-project",
  "audit_time": "2024-01-15T10:30:00",
  "bigquery": {
    "datasets": [
      {
        "dataset_id": "sales_data",
        "tables": [
          {"table_id": "orders", "type": "TABLE", "row_count": 50000},
          {"table_id": "daily_summary", "type": "VIEW"}
        ]
      }
    ]
  },
  "secrets": ["api-key-service-a", "oauth-token-service-b"],
  "apis": ["bigquery.googleapis.com", "secretmanager.googleapis.com"]
}
```

## Post-Audit Checklist

- [ ] Semantic layer doc updated
- [ ] Deprecated items marked/removed
- [ ] Discovery log updated
- [ ] Changes committed to git
- [ ] User notified of significant changes
