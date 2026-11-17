# Recipe: Set Up Data Source

> Configure an external data source for automated data ingestion.

## When to Use

- "Connect to our SQL database for automated data"
- "Import readings from our plant database"
- "Create a REST API integration"
- "How do I automate data collection?"
- "Configure a data source for [system name]"

## Required Context

Before starting, Claude should know:
- [ ] What type of data source (SQL Server, PostgreSQL, Xero, Sage Payroll, REST)
- [ ] Connection details (host, port, credentials)
- [ ] Which metrics will receive data from this source

**If missing:** Claude will ask clarifying questions in Step 1.

---

## Step-by-Step

### Step 1: Determine Data Source Type

**Purpose:** Understand what kind of integration the user needs.

**Available types:**

| Type | Best For | Example |
|------|----------|---------|
| **SQL** | Scheduled database queries | ERP data, financial systems, HR databases |
| **REST** | HTTP API integrations | Third-party services, cloud platforms |
| **Manual Capture** | Human data entry (default) | Form-based data collection |

MQTT is not an import source. Capstone only publishes to MQTT, as a data export target.

**Ask if not provided:**
- "What system are you connecting to?"
- "How does it expose data? (database, accounting system, API)"

---

### Step 2: Check Existing Data Sources

**Purpose:** See what's already configured and avoid duplicates.

**Command:**
```bash
cap masterdata data-sources list --json
```

**Present to user:**
```
Existing data sources:
1. Manual Capture (default — used for human data entry)
2. Plant Historian (SQL — meter readings)
3. ERP Database (SQL — financial data)

Does your new source overlap with any of these?
```

---

### Step 3: Gather Connection Details

**Purpose:** Collect the information needed for the data source configuration.

**For SQL:**
- Connection string (e.g., `Host=db.example.com;Database=erp;Username=reader`)
- Query or stored procedure
- Schedule (how often to poll)

**For REST:**
- Base URL (e.g., `https://api.example.com/v1`)
- Authentication method (API key, OAuth, Basic)
- Endpoint path and parameters

---

### Step 4: Create the Data Source

**Purpose:** Register the data source in Capstone.

**Command:**
```bash
cat <<'EOF' | cap masterdata data-sources create --json
{
  "id": "<empty-id>",
  "name": "Plant Historian",
  "description": "Daily meter readings from the plant historian database"
}
EOF
```

> **Note:** The CLI creates the data source entity. Connection details (broker URLs, credentials, queries) are configured through the web UI or Integration Service configuration, not via CLI. The data source record serves as the reference that input metrics link to.

**What to look for:**
- Success response: `{ "success": true, "id": "<new-id>" }`
- Record the ID — it's needed when linking metrics to this source

---

### Step 5: Link Metrics to the Data Source

**Purpose:** Configure input metrics to receive data from this source.

When creating or updating input metrics, set the `inputDataFeeds` array to reference the data source:

```bash
cat <<'EOF' | cap model inputs save --json
{
  "id": "<existing-input-metric-id>",
  "name": "Electricity Consumption",
  "inputDataFeeds": [
    {
      "id": "<empty-id>",
      "name": "Plant Historian Feed",
      "dataSource": { "id": "<data-source-id>" },
      "dataSourceInstruction": "<SQL query returning OrgNodeId, TimePeriod, Value>",
      "orgNodeOverrides": []
    }
  ]
}
EOF
```

**Key fields:**
- `dataSource.id` — The data source created in Step 4
- `dataSourceInstruction` — Source-specific configuration (for SQL Server and PostgreSQL, the query)
- `orgNodeOverrides` — The sites the feed writes; leave it empty for every site

A SQL Server or PostgreSQL source can also feed narrative text: set
`narrativeDataFeeds` on the narrative with `cap model narratives save`. The query
returns `OrgNodeId`, `TimePeriod` and `Text`, one row per site and period, and a
narrative can have only one feed per data source (see the narrative definition
payload in the model building reference).

---

### Step 6: Verify Configuration

**Purpose:** Confirm the data source and metric linkage.

**Commands:**
```bash
# Verify data source exists
cap masterdata data-sources get <data-source-id> --json

# Verify metric is linked
cap model inputs get <metric-id> --json
# Check inputDataFeeds array includes the new source
```

**Present confirmation:**
```
Data source configured!

Data Source: Plant Historian (<id>)
Linked Metrics:
- Electricity Consumption → kWh readings query
- Gas Consumption → gas readings query

Next steps:
- Configure the Integration Service with connection credentials
- Test the data flow by checking input values after data arrives
- Monitor: cap data input-values list --template <id> --periods "today" --json
```

---

## Example Interaction

**User:** "I want to connect our ERP database to pull monthly financial data"

**Claude:** I'll help you set up a SQL data source. Let me first check what exists.

```bash
cap masterdata data-sources list --json
```

No existing SQL sources found. I'll create one.

**Claude:** What's the purpose of this connection?

**User:** "Pull revenue and cost data monthly from our Enterprise data warehouse"

**Claude:**
```bash
cat <<'EOF' | cap masterdata data-sources create --json
{
  "id": "<empty-id>",
  "name": "Enterprise data warehouse - Financial Data",
  "description": "Monthly revenue and cost data from Enterprise data warehouse"
}
EOF
```

Data source created! ID: `<id>`

Now you'll need to:
1. **Configure connection details** in the Integration Service (database host, credentials, query schedule) — this is done via the web UI or service configuration
2. **Link your metrics** to this data source by updating their `inputDataFeeds`

Would you like me to link specific metrics to this source?

---

## Variations

### Multiple Data Sources per Metric

A metric can receive data from multiple sources (e.g., automated + manual override):

```json
"inputDataFeeds": [
  {
    "name": "Automated Feed",
    "dataSource": { "id": "<sql-source-id>" },
    "dataSourceInstruction": "<SQL query>"
  },
  {
    "name": "Manual Override",
    "dataSource": { "id": "<manual-capture-id>" },
    "dataSourceInstruction": ""
  }
]
```

### Bulk Setup via Excel

For configuring many data sources at once:
```bash
# Export current data sources to get the format
cap masterdata data-sources download-excel -o data-sources.xlsx

# Add new rows in Excel, then import
cap masterdata data-sources upload-excel data-sources.xlsx --json
```

### Org Node Overrides

Different locations can have different source instructions:

```json
"orgNodeOverrides": [
  {
    "orgNode": { "id": "<site-a-id>" },
    "dataSourceInstruction": "<SQL query for site A>"
  },
  {
    "orgNode": { "id": "<site-b-id>" },
    "dataSourceInstruction": "<SQL query for site B>"
  }
]
```

---

## Related Recipes

- [Create Metric Wizard](./create-metric.md) — Create metrics that use the data source ✅
- [Export to Excel](./export-to-excel.md) — Bulk manage data sources via Excel ✅
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — See existing configuration ✅
