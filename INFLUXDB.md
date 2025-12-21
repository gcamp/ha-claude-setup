# InfluxDB Access

Home Assistant logs sensor data to an InfluxDB instance for historical analysis and predictions.

## Connection Details

- **URL:** `http://10.0.0.10:8086`
- **Token:** Stored in `influxdb_credentials` (gitignored)
- **Access:** Direct HTTP access from local machine

## Common Operations

**List buckets:**
```bash
source influxdb_credentials
curl -s -H "Authorization: Token $INFLUXDB_TOKEN" $INFLUXDB_URL/api/v2/buckets
```

**Query data (Flux query language):**
```bash
source influxdb_credentials
curl -s -H "Authorization: Token $INFLUXDB_TOKEN" \
  -H "Accept: application/csv" \
  -H "Content-type: application/vnd.flux" \
  -d 'from(bucket:"homeassistant") |> range(start: -7d)' \
  $INFLUXDB_URL/api/v2/query
```

## Python Usage

**Load credentials:**
```python
import os

# Parse credentials file
with open('influxdb_credentials') as f:
    for line in f:
        if line.startswith('INFLUXDB_'):
            key, val = line.strip().split('=', 1)
            os.environ[key] = val

url = os.environ['INFLUXDB_URL']
token = os.environ['INFLUXDB_TOKEN']
```

**Query with influxdb-client:**
```python
from influxdb_client import InfluxDBClient

client = InfluxDBClient(url=url, token=token, org="homeassistant")
query_api = client.query_api()

query = '''
from(bucket: "homeassistant")
  |> range(start: -7d)
  |> filter(fn: (r) => r["entity_id"] == "sensor.example")
'''

tables = query_api.query(query)
for table in tables:
    for record in table.records:
        print(record.get_time(), record.get_value())
```

## Notes

- Bucket name is typically `homeassistant`
- Sensor entities are stored with their full `entity_id` (e.g., `sensor.montreal_temperature`)
- Weather entities stored as `weather.<name>` (e.g., `weather.montreal_forecast`)
- Use Flux query language for data extraction and aggregation
