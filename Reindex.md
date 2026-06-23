The `_source` field contains the **exact, literal JSON payload** that you sent to Elasticsearch when you first indexed the document. It preserves your original formatting, field names, data types, and structural hierarchy completely intact.

To understand exactly what data it contains (and what it _does not_ contain), let's look at a concrete example.

###  A Real-World View: Metadata vs. `_source`

When you retrieve a document from Elasticsearch, the response is wrapped in a JSON object. The data is divided into two parts: **Cluster Metadata** and the **`_source` field**.

Imagine you index this document:

```
{
  "product_name": "Wireless Headphones",
  "price": 99.99,
  "tags": ["audio", "bluetooth"],
  "specs": { "battery_life": "24h", "waterproof": true }
}
```

When you search for it, Elasticsearch returns this complete structure:

```
{
#Metadata
  "_index": "store-products",
  "_id": "prod-101",
  "_version": 1,
  "_seq_no": 42,
  "_primary_term": 1,
  "found": true,
  "_score": 1.0,
  
  #Source/Actual Logs
  "_source": {
    "product_name": "Wireless Headphones",
    "price": 99.99,
    "tags": ["audio", "bluetooth"],
    "specs": {
      "battery_life": "24h",
      "waterproof": true
    }
  }
}
```

### Exactly What `_source` Contains:

1. **Your Original Fields & Keys:** Every key-value pair you defined (`product_name`, `price`).
    
2. **Original Data Types:** Strings stay strings, numbers stay numbers, booleans stay booleans.
    
3. **Complex Structures:** Arrays (`tags`) and nested inner objects (`specs`) retain their exact hierarchical layout.
    
4. **Original Formatting:** If you included specific whitespace or indentation in your raw text fields, it is preserved exactly as it was sent.

### What `_source` does NOT Contain:

On the exam, it is crucial to remember that **metadata is separate from source data**. The `_source` block does **not** contain:

- **The Document ID (`_id`):** The system identifier is stored separately in the cluster index metadata.
    
- **The Index Name (`_index`):** The target index name is part of the system tracking layer.
    
- **The Search Score (`_score`):** The relevance score is calculated dynamically at query-time and is never written to disk inside the source.
    
- **Tokens or Analyzed Terms:** If the string `"Wireless Headphones"` was broken down into `["wireless", "headphones"]` for search by an analyzer, those lowercase tokens live inside the **Inverted Index**, completely separate from the `_source`.    

### Summary Rule for the Exam

Think of the `_source` field as a **zipped backup copy of your original JSON input**. Elasticsearch uses the highly modified, unreadable Inverted Index to _find_ the document matching a query, but it opens up this `_source` zip file to _show_ you what the document originally looked like.

### Reindex Examples

```
POST /_reindex
{
  "source": {
    "index": "server-logs-v1",
  },
  "dest": {
    "index": "server-logs-errors-only"
  }
}
```

```
POST /_reindex
{
  "source": {
    "index": "server-logs-v1",
    "query": {
      "match": {
        "log_level": "ERROR"
      }
    }
  },
  "dest": {
    "index": "server-logs-errors-only"
  }
}
```

