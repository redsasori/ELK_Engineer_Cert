## Standard Pagination (`from` and `size`)

This is your standard "Page 1, 2, 3" web navigation.

- **`size`:** How many results per page (Default is 10).
    
- **`from`:** How many results to skip before starting (Default is 0).
    

If you want to view **Page 3** of a web store showing 10 items per page, your query looks like this:
```
GET /products/_search
{
  "from": 20, 
  "size": 10,
  "query": {
    "match": { "status": "active" }
  }
}
```

### The Fatal Flaw: The 10,000 Limit

`from` and `size` work perfectly for the first few pages. But what if a user jumps to **Page 1,000**? (`"from": 10000, "size": 10`).

If you run this, Elasticsearch will throw an error and refuse to execute the search.

**Why?** Because Elasticsearch is distributed. To find items 10,000 to 10,010, the coordinating node must ask _every single shard_ for its top 10,010 items. It then holds all 10,000+ items in RAM, sorts them globally, drops the first 10,000, and returns the 10. If you do this across millions of documents, the server will run out of memory and crash.

To protect itself, Elasticsearch enforces a hard limit called `index.max_result_window` (set to `10000`). To go deeper, you must use `search_after`.

## Part 3: Deep Pagination (`search_after`)

When you need to export massive datasets, scrape an index, or allow users to scroll endlessly past 10,000 results, you must use the `search_after` API.

Think of `search_after` as a bookmark. Instead of calculating deep offsets, you tell Elasticsearch: _"Here is the exact sorting data of the very last document on Page 1. Just give me the next 10 documents that come after this."_

### Step 1: The Initial Query (The Tie-Breaker Rule)

To use `search_after`, your sort array **must contain a unique tie-breaker field**. If you only sort by `price`, and 50 items cost `$9.99`, Elasticsearch won't know which `$9.99` item you left off on. We almost always use the document `_id` as the tie-breaker.

```
GET /products/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "sort": [
    { "price": { "order": "desc" } },
    { "_id": { "order": "asc" } } 
  ]
}
```

### Step 2: Grab the Bookmark

When you get the results back, look at the very last document in the `hits` array. It will have a `"sort"` array attached to it. It will look like this:

```
"sort": [ 19.99, "prod-8472" ]
```

### Step 3: The Next Page

To get Page 2, you run the exact same query, but you inject that sort array into the `"search_after"` parameter.

```
GET /products/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "search_after": [ 19.99, "prod-8472" ],
  "sort": [
    { "price": { "order": "desc" } },
    { "_id": { "order": "asc" } }
  ]
}
```