```

GET /tech-orders/_search
{
    "size": 0,
    "aggs": {
        "total_revenue": {
            "sum":{
                "field": "price"
            }
        }
    }
}

```

### Bucket Aggregations

```
GET /tech-orders/_search
{
  "size": 0,
  "aggs": {
    "group_by_date": {
      "date_histogram": {
        "field": "purchase_date",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_total_revenue": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

### Sample Data

```
POST /tech-orders/_bulk
{"index":{"_id":"1"}}
{"category":"laptop","brand":"Dell","price":1200,"purchase_date":"2026-06-01"}
{"index":{"_id":"2"}}
{"category":"laptop","brand":"Apple","price":2000,"purchase_date":"2026-06-02"}
{"index":{"_id":"3"}}
{"category":"phone","brand":"Apple","price":1000,"purchase_date":"2026-06-01"}
{"index":{"_id":"4"}}
{"category":"phone","brand":"Samsung","price":900,"purchase_date":"2026-06-03"}
{"index":{"_id":"5"}}
{"category":"accessory","brand":"Apple","price":50,"purchase_date":"2026-06-03"}
{"index":{"_id":"6"}}
{"category":"laptop","brand":"Dell","price":1400,"purchase_date":"2026-06-01"}
```

### Types of Aggregation

|**Aggregation Type**|**What it does**|**When to use it**|**Output Type**|
|---|---|---|---|
|**Metric**|Math|When you need a calculation (Sum, Avg, Max).|A single calculated number.|
|**Bucket**|Grouping|When you need to sort data into categories or timelines.|A list of distinct bins and their counts.|
|**Nested**|Unpacking|When querying an array of objects mapped as `nested`.|Allows Metrics/Buckets to run inside arrays.|

