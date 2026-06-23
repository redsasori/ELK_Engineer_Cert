
| SQL Concept                     | Elasticsearch DSL Equivalent                                |
| ------------------------------- | ----------------------------------------------------------- |
| `WHERE field = 'Exact String'`  | **`term`** (inside a `filter` block)                        |
| `WHERE field IN ('A', 'B')`     | **`terms`** (inside a `filter` block)                       |
| `WHERE age >= 18 AND age <= 30` | **`range`**                                                 |
| `WHERE x = 1 AND y = 2`         | **`bool`** -> **`must`** or **`filter`** (Array of queries) |
| `WHERE x = 1 OR y = 2`          | **`bool`** -> **`should`** (Array of queries)               |
| `WHERE NOT (x = 1)`             | **`bool`** -> **`must_not`**                                |
| `WHERE text LIKE '%fox%'`       | **`wildcard`** (Slow, avoid if possible!)                   |
| `MATCH(text) AGAINST('fox')`    | **`match`** (Fast, analyzed, ranked search)                 |

### Why use all three (`must`, `match`, and `term`) together?

You rarely use _just_ a `match` or _just_ a `term` in the real world. You usually need to combine them to build a highly specific search.

Remember our container logic: **`must`** is the logical `AND` container. You use it to wrap your `match` and `term` queries so Elasticsearch knows both conditions are mandatory.

**Real-World Example:** Imagine you are searching a company directory. You want to find an employee whose department is exactly "Engineering" (an exact keyword), AND whose personal bio mentions "machine learning" (a full-text search).

You use all three like this:

```
GET /employees/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "department.keyword": "Engineering" } }, 
        { "match": { "bio": "machine learning" } }
      ]
    }
  }
}
```

### 1. When to use JUST `match` (The "Google Search")

You use a standalone `match` query when you are building a human-facing search bar, and you want Elasticsearch to act purely as a relevance engine.

- **The Goal:** Full-text search where you need the BM25 algorithm to calculate a `_score` and return the "best" results at the top.
    
- **Real-World Example:** You are building the search bar for a recipe website. A user types in "spicy chicken tacos". You don't want exact matches; you want anything mentioning spice, chicken, or tacos, ranked by how relevant the recipe is to that phrase.
    

```
GET /recipes/_search
{
  "query": {
    "match": {
      "instructions": "spicy chicken tacos"
    }
  }
}
```

### 2. When to use JUST `term` (The "Database Lookup")

You use a standalone `term` query when you are treating Elasticsearch exactly like a traditional relational database lookup.

- **The Goal:** A strict, binary, exact-character match. You know exactly what you are looking for, and you don't need fuzzy matching or text analysis.
    
- **Real-World Example:** A customer calls support and gives you their exact Order ID. You don't need a "relevance score" for an Order ID—it either exists or it doesn't.
    

```
GET /orders/_search
{
  "query": {
    "term": {
      "order_id.keyword": "ORD-99482-XYZ"
    }
  }
}
```

_(Note: Even though this is standing alone, it does technically calculate a score of `1.0`. To optimize this, we usually wrap it in a filter, which brings us to the next point)._

### 3. When to use JUST `filter` (The "Analytics Dashboard")

You use a standalone `filter` block (inside a `bool` query) when you are running background jobs, generating reports, or populating Kibana dashboards where **relevance scoring is completely useless.**

- **The Goal:** Lightning-fast, cached data retrieval sorted by something other than relevance (usually sorted by time).
    
- **Real-World Example:** You are building a security dashboard that shows all "Failed Logins" from the last 24 hours. Because the dashboard sorts the logs chronologically (newest first), calculating a search `_score` for millions of logs is a massive waste of CPU. You use `filter` to grab the data instantly and skip the math.
    

```
GET /security-logs/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "event.action": "failed_login" } }
      ]
    }
  }
}
```

| **Concept**  | **Use it alone when...**                          | **Real-World Scenario**                           |
| ------------ | ------------------------------------------------- | ------------------------------------------------- |
| **`match`**  | You need to rank text by relevance.               | A user types into a website search bar.           |
| **`term`**   | You need an exact ID or exact keyword.            | Looking up a specific user ID or tracking number. |
| **`filter`** | You want data fast and don't care about `_score`. | Populating a time-based dashboard or chart.       |
