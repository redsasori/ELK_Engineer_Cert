### 1. Matching by Data Type (`match_mapping_type`)

This looks at the incoming JSON data type and overrides how Elasticsearch assigns the final field type.

- **Supported JSON Types:** `string`, `long`, `double`, `boolean`, `date`, `object`.
    
- **The Blueprint:** _"Any tidme a new field arrives that looks like a **JSON Type**, map it as an **Elasticsearch Type**."_
#### Example:

By default, all numbers without decimals are mapped as `long`. To save space, you want to force all newly discovered integers to be mapped as a compact `integer`.

```
PUT /orders-index
{
  "mappings": {
    "dynamic_templates": [
      {
        "integers_as_compact": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      }
    ]
  }
}
```

### 2. Matching by Field Name (`match`)

This completely ignores the JSON data type and looks strictly at the **literal string name** of the field key using wildcards (`*`).

- **Key Parameters:** `match` (the pattern to include), `unmatch` (patterns to exclude).
    
- **The Blueprint:** _"I don't care what type of data it is; if the field **name** matches this pattern, map it like this."
#### Example:

Any newly discovered field that ends with the suffix `_code` must be mapped strictly as a non-analyzed `keyword`.

JSON

```
PUT /products-index
{
  "mappings": {
    "dynamic_templates": [
      {
        "codes_as_keywords": {
          "match": "*_code",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```

### 3. Combining Both (The Exam Power Move)

The exam will often require precision. If you use `match: "*_ip"` but a user accidentally sends an integer, your index mapping might break. Combining both parameters acts as an **AND** condition.

JSON

```
PUT /telemetry-index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_ending_in_ip": {
          "match_mapping_type": "string",
          "match": "*_ip",
          "mapping": {
            "type": "ip"
          }
        }
      }
    ]
  }
}
```

_> **Condition:** The field must be a JSON string **AND** its name must end with `_ip` to be mapped as an `ip` type._


