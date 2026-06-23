### Day 1: Index Management & Templates

Focus on setting up the structural foundations of Elasticsearch.

- **Define an index:** Know how to set explicit settings (e.g., `number_of_shards`, `number_of_replicas`).
    
	- **Dynamic Templates:** Master how to control unmapped fields based on their data type or field name (using `match_mapping_type` or `match`). [[Doubt 1 MM and Match]]
    
- **Index Templates & Data Streams:** Practice creating [[component templates]](`_component_template`) and combining them into an [[index template]] (`_index_template`) configured for a data stream (requires a matching `data_stream: {}` object and a `timestamp` field).
### Day 2: Data Processing & Ingest Pipelines

Learn how to manipulate data before and after it hits the disk.

- **Mappings & Multi-fields:** Practice creating [[explicit mappings]]. Know how to index a field as both `text` (for full-text search) and `keyword` (for [[aggregations]]/[[sorting]]) using `fields`.
    
- **[[Ingest Pipelines]]:** Get comfortable with processors like `grok`, `set`, `rename`, `convert`, and `script`. Know how to test them using the `_simulate` API.

- **[[Reindex]] & Update By Query:** Understand how to change mappings by reindexing data into a new index, and how to apply pipeline updates to existing documents. 

### Day 3: Core Search, Sort, & Pagination

The exam will require you to write complex queries. Drill down on precision here.

- **[[Queries vs. Filters]]:** Master the `bool` query. Remember that `must` and `should` contribute to the relevance score (`_score`), while `filter` and `must_not` run in a filter context (no scoring, heavily cached).
    
- **Match vs. Term:** Use `match` for analyzed text fields and `term` for exact match keyword/numeric fields.
    
- **Sorting & Pagination:** Practice sorting by multiple fields and implementing pagination using `from`/`size` or `search_after` for large datasets.
    
- **Aliases:** Know how to create, swap, and filter index aliases seamlessly.

### Day 4: Aggregations & Asynchronous Search

This is where many people lose time. Speed up your aggregation nesting.

- **Metrics vs. Buckets:** Practice combining bucket aggregations (like `terms`, `date_histogram`, or `range`) with metric aggregations (like `avg`, `sum`, or `max`).
    
- **Sub-aggregations:** Get used to nesting aggregations inside buckets to generate multi-layered analytical reports.
    
- **Asynchronous Search:** Learn to use `_async_search` to run long-running queries in the background and retrieve partial results.

### Day 5: Runtime Fields & Cross-Cluster Ops

Handling dynamic data properties and multi-cluster patterns.

- **Runtime Fields:** Know both methods: defining them on the fly inside a search request (`runtime_mappings`), and defining them permanently in the index mapping. Practice simple Painless scripts to emit values (e.g., `emit(doc['field'].value * 2)`).
    
- **Cross-Cluster Search (CCS):** Practice executing queries across local and remote clusters using the `cluster_name:index_name` syntax.

### Day 6: Cluster Management & Resilience

Infrastructure, backup, and health optimization.

- **Shard Diagnostics & Health:** Understand why a cluster might turn `yellow` (unassigned replica shards) or `red` (unassigned primary shards). Practice using `_cat/allocation`, `_cat/shards`, and the `_cluster/allocation/explain` API to find and fix routing issues.
    
- **Snapshot Lifecycle Management (SLM) & Backups:** Register a snapshot repository (usually type `fs` on the exam) and write an SLM policy to automate backups.
    
- **Searchable Snapshots & CCR:** Know how to restore a snapshot as a searchable index (fully or partially mounted). Set up Cross-Cluster Replication (CCR) by configuring a follower index to track a leader index from a remote cluster.

### Day 7: Index Lifecycle Management (ILM) & Mock Practice

Tie it all together.

- **ILM Policies:** Build a multi-phase policy (Hot, Warm, Cold, Delete). Practice configuring a rollover action in the Hot phase, and moving indices to different data tiers or shrinking/force-merging them in later phases.
    
- **Speed Run:** Spend the last hours mimicking exam conditions. Pick documentation links instead of typing commands from memory to ensure your searching skills are sharp.

## 💡 Top 3 Pro-Tips from a 3x Certified Engineer

1. **Bookmark the Docs Strategically:** You are allowed to use the official Elastic Documentation during the exam. Create a clean browser bookmark folder with direct links to the **Request Examples** of complex APIs (e.g., ILM policy JSON, Ingest Pipeline setup, Reindex structure, and Dynamic Templates). Copying and pasting boilerplate code saves critical minutes.
    
2. **The `_cluster/allocation/explain` API is Your Best Friend:** If the exam asks you to diagnose why a cluster is red or yellow, run this API immediately. It will explicitly tell you the exact node block, disk threshold breach, or shard lock preventing allocation.
    
3. **Validate Your Work:** Never assume a command worked because you got a `{"acknowledged": true}` response. If you create a pipeline, use `_simulate`. If you build an ILM policy, verify that the indices are actually inheriting it using `GET /<index>/_ilm/explain`.