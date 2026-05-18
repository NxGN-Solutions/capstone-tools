# Recipe: Set Up Data Source

> Configure an external data source for automated data ingestion.

## When to Use

- "Connect to our SQL database for automated data"
- "Set up an MQTT feed from our sensors"
- "Create a REST API integration"
- "How do I automate data collection?"
- "Configure a data source for [system name]"

## Required Context

Before starting, Claude should know:
- [ ] What type of data source (MQTT, SQL, REST)
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
| **MQTT** | Real-time sensor data, IoT devices | Factory floor sensors, building management systems |
| **SQL** | Scheduled database queries | ERP data, financial systems, HR databases |
| **REST** | HTTP API integrations | Third-party services, cloud platforms |
| **Manual Capture** | Human data entry (default) | Form-based data collection |

**Ask if not provided:**
- "What system are you connecting to?"
- "How does it expose data? (database, API, message broker)"

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
2. Factory MQTT (MQTT — sensor data)
3. ERP Database (SQL — financial data)

Does your new source overlap with any of these?
```

---

### Step 3: Gather Connection Details

**Purpose:** Collect the information needed for the data source configuration.

**For MQTT:**
- Broker host and port (e.g., `mqtt://broker.example.com:1883`)
- Topic pattern (e.g., `factory/+/sensors/#`)
- Authentication (username/password or certificate)
- QoS level (0, 1, or 2)

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
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "Factory MQTT Sensors",
  "description": "Real-time sensor data from factory floor"
}
EOF
```

> **Note:** The CLI creates the data source entity. Connection details (broker URLs, credentials, queries) are configured through the web UI or Integration Service configuration, not via CLI. The data source record serves as the reference that input metrics link to.

**What to look for:**
- Success response: `{ "success": true, "id": "<new-guid>" }`
- Record the ID — it's needed when linking metrics to this source

---

### Step 5: Link Metrics to the Data Source

**Purpose:** Configure input metrics to receive data from this source.

When creating or updating input metrics, set the `inputDataFeeds` array to reference the data source:

```bash
cat <<'EOF' | cap model inputs save --json
{
  "id": "<existing-input-metric-guid>",
  "name": "Electricity Consumption",
  "inputDataFeeds": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "name": "Factory MQTT Feed",
      "dataSource": { "id": "<data-source-guid>" },
      "dataSourceInstruction": "topic: factory/site-a/electricity",
      "orgNodeOverrides": []
    }
  ]
}
EOF
```

**Key fields:**
- `dataSource.id` — The data source created in Step 4
- `dataSourceInstruction` — Source-specific configuration (MQTT topic, SQL query, REST endpoint)
- `orgNodeOverrides` — Optional per-location customization of the data feed

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

Data Source: Factory MQTT Sensors (abc123...)
Linked Metrics:
- Electricity Consumption → topic: factory/site-a/electricity
- Gas Consumption → topic: factory/site-a/gas

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

**User:** "Pull revenue and cost data monthly from our Oracle ERP"

**Claude:**
```bash
cat <<'EOF' | cap masterdata data-sources create --json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "Oracle ERP - Financial Data",
  "description": "Monthly revenue and cost data from Oracle ERP"
}
EOF
```

Data source created! ID: `xyz789...`

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
    "dataSource": { "id": "<mqtt-source-guid>" },
    "dataSourceInstruction": "topic: sensors/electricity"
  },
  {
    "name": "Manual Override",
    "dataSource": { "id": "<manual-capture-guid>" },
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
    "orgNode": { "id": "<site-a-guid>" },
    "dataSourceInstruction": "topic: factory-a/electricity"
  },
  {
    "orgNode": { "id": "<site-b-guid>" },
    "dataSourceInstruction": "topic: factory-b/electricity"
  }
]
```

---

## Related Recipes

- [Create Metric Wizard](./create-metric.md) — Create metrics that use the data source ✅
- [Export to Excel](./export-to-excel.md) — Bulk manage data sources via Excel ✅
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — See existing configuration ✅
