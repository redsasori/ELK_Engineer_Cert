## `_update_by_query` (Applying Pipelines to Old Data)

**The Scenario:** You just built a brilliant Ingest Pipeline named `extract-ip-pipeline` that parses an IP address out of your log messages. You attached it as the `default_pipeline` to your index, so all _new_ logs are getting parsed perfectly.

However, you have 100,000 existing logs sitting in `app-logs` that missed out on the pipeline.

### The Query

You use `_update_by_query` and explicitly attach the `pipeline` parameter to the URL.

JSON

```
POST /app-logs/_update_by_query?pipeline=extract-ip-pipeline
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "extracted_ip"
        }
      }
    }
  }
}
```

### Exam Pro-Tips for Update By Query:

- **The Query Filter is Critical:** Notice that I added a `must_not` exists query? If you don't do this, Elasticsearch will pull all 100,000 documents off the disk, run them through the pipeline again, and rewrite them, wasting massive amounts of CPU and I/O. **Always filter your updates to target only the documents that actually need it.**
    
- **Version Conflicts (`conflicts: proceed`):** If a document is updated by a user at the exact millisecond your `_update_by_query` touches it, Elasticsearch will throw a version conflict and abort the entire job. On the exam, if you are updating massive indices, add `?conflicts=proceed` to the URL so it skips the locked document but finishes the rest of the job.
    

### Master Cheat Sheet Comparison

|**API**|**What it does**|**When to use it on the Exam**|
|---|---|---|
|**`_reindex`**|Moves data from Index A to Index B.|When you need to change a field's mapping type, or combine two daily indices into one monthly index.|
|**`_update_by_query`**|Modifies data in-place inside Index A.|When you need to run existing data through a new pipeline, or execute a Painless script to fix typos.|