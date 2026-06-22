Component Template 1 (metrics-settings): Configure the indices to have 2 primary shards and 0 replicas (to save disk space).

Component Template 2 (metrics-mappings): * Map cpu_percentage as a float.

Map the required timestamp field, but because these are high-frequency metrics, you must map it as date_nanos (not the standard date).

```
POST _component_template/mtrcs-comp-settings
{
    "template": {
        "settings": {
            "index.number_of_shards": 2,
            "number_of_replicas": 0
        }
    }
}
```

```
POST _component_template/mtrcs-comp-mapping
{
    "template": {
        "mappings":{
            "properties":
            {
            "cpu_percentage":{
                "type": "float"
            },
            "@timestamp":{
                "type": "date_nanos"
            }
        }
    }
    }
}
```


Master Index Template (metrics-template):

Target any index starting with metrics-cpu-.
Compose both components.
Give it a priority of 200.

Enable the Data Stream functionality.
Initialization: Push a single test document to metrics-cpu-prod using the current timestamp and a CPU percentage of 84.5.

```
POST _index_template/mtrcs-idx-temp
{
    "index_patterns": ["metrics-cpu-*"],
    "composed_of": ["metrics-settings","metrics-mappings"],
    "data_stream": {},
    "priority": 500
}
```

```
POST /metrics-cpu-prod/_doc
{
  "@timestamp": "2026-06-22T11:46:28.000Z",
  "cpu_percentage": 85.5
}
```

### OR 

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

Here the mappings are defined in the Index Template itself instead of defining component template.