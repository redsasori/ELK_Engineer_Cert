### 1. Aliases on Index

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "tech-blog-v1",
        "alias": "tech-blog"
      }
    }
  ]
}
```

### 2. Atomic Swap on a Reindexed Data (Zero Downtime)

This is the exact scenario you will face in production. You realize `tech-blog-v1` has a broken mapping. You create `tech-blog-v2`, fix the mapping, and use the `_reindex` API to move all the data over.

Now, you have to tell your frontend application to start using `v2` instead of `v1`. If you delete the alias and then recreate it, there is a split-second where searches will fail.

**The Fix:** You use an **Atomic Swap**. You put both the `remove` and `add` actions in the exact same array. Elasticsearch executes them simultaneously.

```
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "tech-blog-v1",
        "alias": "tech-blog"
      }
    },
    {
      "add": {
        "index": "tech-blog-v2",
        "alias": "tech-blog"
      }
    }
  ]
}
```

### 3. Filtered Aliases (Security & UX)

This is a brilliant trick for multi-tenant applications or strict data access controls.

**The Scenario:** You have a massive index called `global-sales` containing data for the US, UK, and Japan. The Japan team wants an endpoint they can query, but for security and performance reasons, you don't want them accidentally seeing US data.

You don't need to create a separate index! You can create a **Filtered Alias**.
```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "global-sales",
        "alias": "japan-sales",
        "filter": {
          "term": { "country.keyword": "Japan" }
        }
      }
    }
  ]
}
```

**How it works:** When the Japan team runs `GET /japan-sales/_search`, Elasticsearch intercepts the query, secretly injects that `"term"` filter into it, and only returns the Japanese documents. The team is completely isolated to their specific data.

### 4.  ==Is_write_true==

If you point an alias to multiple indices at the same time (e.g., `logs-monday` and `logs-tuesday` both share the alias `all-logs`), what happens when you try to index a _new_ document into `all-logs`?

Elasticsearch will crash and throw an error: _"Alias points to multiple indices. Which one do I write to?"_

If an alias points to multiple indices, you must explicitly tell Elasticsearch which index is the "active" one for new data by setting `"is_write_index": true`.

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-wednesday",
        "alias": "all-logs",
        "is_write_index": true
      }
    }
  ]
}
```

Now, searching `all-logs` reads from Monday, Tuesday, and Wednesday, but _writing_ to `all-logs` only goes to Wednesday!

### Note: We can do the same for Datastreams, If we want to cross query  along multiple data streams then we can assign the same alias. But we cannot define is_write_index while making an alias for a datastream.

Example: **Datastream 1** : kibana-web-dstream, **Datastream 2**:  kibana-ecomm-dstream have the same alias **"kibana-all-ds"** 