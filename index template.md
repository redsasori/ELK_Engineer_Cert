```
PUT /_index_template/metrics-template
{
  "index_patterns": ["metrics-cpu-*"],
  "data_stream": {},
  "priority": 500,
  "composed_of": [
    "metrics-settings"
  ],
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date_nanos" 
        },
        "cpu_percentage": {
          "type": "float"
        }
      }
    }
  }
}
```

