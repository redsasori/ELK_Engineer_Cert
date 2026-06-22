Sorting tells the difference between how Elasticsearch calculates relevance and how it retrieves exact values from disk.

When you run a standard `match` query, Elasticsearch automatically sorts the results by **Relevance Score (`_score`)**, from highest to lowest. But in the real world (and on the exam), users usually want to sort by "Price: Low to High" or "Newest First."

The moment you introduce a custom `sort` array, **Elasticsearch stops sorting by `_score` entirely.** Here is your comprehensive, exam-focused guide to **How** and **Why** sorting works, complete with the biggest traps to avoid.

## Concepts : Doc Values vs. Inverted Index

To master sorting, you must understand _where_ Elasticsearch is getting the data to sort.

- **Search (The Inverted Index):** Text fields are broken into tokens and stored in an inverted index (like an index at the back of a textbook). This is great for finding _which_ document contains a word, but terrible for sorting.
    
- **Sorting & Aggregations (Doc Values):** To sort efficiently, Elasticsearch uses a columnar data structure called **Doc Values**. Doc values are enabled by default on `keyword`, `date`, and numeric fields. They are **disabled** on `text` fields.
    

> 🚨 **The Ultimate Exam Trap:** If you try to sort using a standard `text` field (e.g., `"sort": [ { "message": "asc" } ]`), Elasticsearch will instantly throw an `illegal_argument_exception` because text fields do not have doc values. You **must** sort on the `.keyword` sub-field.

### Sort Example

**Single Field Sort**

```
GET /tech-orders/_search
{
    "query": {
        "match": {
            "category": "laptop"
        }
    },
    "sort":{
        "price":{
            "order": "asc"
        }
    }
}
```

**Multi-Field Sort**

```
GET /tech-orders/_search
{
    "query": {
        "match": {
            "category": "laptop"
        }
    },
    "sort":[{
        "purchase_date": {
            "order": "desc"
        },
        "price":{
            "order": "asc"
        }
    }]
}
```

**Sort by Strings (Keyword) Alphabetically**
```
GET /tech-orders/_search
{
  "sort": [
    {
      "brand.keyword": {
        "order": "asc"
      }
    }
  ]
}
```


**Sort using mode parameter**
### Sorting Arrays (The `mode` Parameter)

This is a high-difficulty exam curveball.

**Scenario:** You have a document where a single product has multiple prices because it comes in different sizes (e.g., `{"price": [50, 100, 150]}`). The exam asks you to sort products by price (Lowest to Highest), but for products with multiple prices, use their _average_ price to determine their sorted position.

```
GET /tech-orders/_search
{
  "sort": [
    {
      "price": {
        "order": "asc",
        "mode": "avg"
      }
    }
  ]
}
```

## Exam Tips

- When sorting an array of numbers, Elasticsearch doesn't know which number to use. By default, if sorting `asc`, it picks the `min` value. If sorting `desc`, it picks the `max` value.
    
- The `mode` parameter lets you override this. You can set it to `min`, `max`, `sum`, `avg`, or `median`.

