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

	