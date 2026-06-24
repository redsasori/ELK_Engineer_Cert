### Types of Aggregation

| **Aggregation Type** | **What it does** | **When to use it**                                       | **Output Type**                              |
| -------------------- | ---------------- | -------------------------------------------------------- | -------------------------------------------- |
| **Metric**           | Math             | When you need a calculation (Sum, Avg, Max).             | A single calculated number.                  |
| **Bucket**           | Grouping         | When you need to sort data into categories or timelines. | A list of distinct bins and their counts.    |
| **Nested**           | Unpacking        | When querying an array of objects mapped as `nested`.    | Allows Metrics/Buckets to run inside arrays. |
### 1. Combining Metrics and Buckets 

**The Concept:** You use a Bucket to group documents into bins, and a Metric to perform a mathematical calculation on the documents inside each specific bin.

**Exam Scenario:** "Group our products by price ranges, and tell me the highest (max) review score inside each price bracket."

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_brackets": {
      "range": {                               <-- BUCKET (The Organizer)
        "field": "price",
        "ranges": [
          { "to": 50 },
          { "from": 50, "to": 100 },
          { "from": 100 }
        ]
      },
      "aggs": {
        "highest_review": {
          "max": {                             <-- METRIC (The Calculator)
            "field": "review_score"
          }
        }
      }
    }
  }
}
```

### 2. Multi-Layered Sub-Aggregations

**The Concept:** You don't just stop at one bucket. You put a bucket _inside_ a bucket, and then put a metric inside of _that_. This creates a multi-dimensional analytical report.

**Exam Scenario:** "Show me the top 3 selling countries. For each country, break down the sales by month. For each month, calculate the total revenue."

```
GET /global-sales/_search
{
  "size": 0,
  "aggs": {
    "by_country": {
      "terms": {                               <-- LAYER 1: BUCKET (Country)
        "field": "country.keyword",
        "size": 3
      },
      "aggs": {
        "by_month": {
          "date_histogram": {                  <-- LAYER 2: BUCKET (Month)
            "field": "sale_date",
            "calendar_interval": "month"
          },
          "aggs": {
            "monthly_revenue": {
              "sum": {                         <-- LAYER 3: METRIC (Revenue)
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}
```

### 3. Asynchronous Search (`_async_search`)

**The Concept:** When a query is so massive it will time out your connection, you send it to the background. You get an ID, check on it later, and delete it when you are done.

**The 3-Step Exam Workflow:**

**Step 1: Submit the heavy query (Tell it to run in the background after 1 second).**

```
POST /massive-logs-index/_async_search?wait_for_completion_timeout=1s
{
  "size": 0,
  "aggs": {
    "heavy_math": { "cardinality": { "field": "ip_address" } }
  }
}
```

_(Elasticsearch replies with `"id": "FmRldE8..."` and `"is_running": true`)_

**Step 2: Check the progress using the ID.**

```
GET /_async_search/FmRldE8zREVEU...
```

_(If it is still running, you will see partial results. If it is done, you will see the final answer)._

**Step 3: Delete the task from cluster memory.**
```
DELETE /_async_search/FmRldE8zREVEU...
```

### Simple Examples

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

