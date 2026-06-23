- **`match` is for Full-Text Search:** It is "smart." It analyzes your search words (lowercasing them, removing punctuation, etc.) to find similar data.
    
- **`term` is for Exact Lookups:** It is "strict." It acts exactly like an `=` operator in SQL. It does absolutely no analysis and looks for a 100% identical, case-sensitive character match.

### The Practical Scenario: The "Fox" Problem

Imagine you index a single product into your database:
```
{
  "title": "The Quick Brown Fox",       // Mapped as 'text'
  "status": "In-Stock"                  // Mapped as 'keyword'
}
```

Because `title` is a `text` field, Elasticsearch's analyzer chops it up and saves it in the hidden inverted index as lowercase tokens: `["the", "quick", "brown", "fox"]`.

Because `status` is a `keyword` field, Elasticsearch skips the analyzer and saves it exactly as typed: `["In-Stock"]`

Let's see what happens when we query them.

#### Scenario 1: Using `term` on a text field (The Trap)
You want to find the fox, so you run an exact `term` query:

```
GET /products/_search
{
  "query": {
    "term": {
      "title": "Quick"
    }
  }
}
```

**Result: 0 Hits.**

_Why?_ The `term` query is strict. It marched into the database looking for a capital **"Q"**. But remember, the analyzer lowercased the database tokens. Since `"Quick"` does not exactly match `"quick"`, it fails silently.

#### Scenario 2: Using `match` on a text field (The Solution)

You change your query to `match`:
```
GET /products/_search
{
  "query": {
    "match": {
      "title": "Quick"
    }
  }
}
```

**Result: 1 Hit.**

_Why?_ The `match` query is smart. Before it searches, it says, "Oh, I see the user capitalized _Quick_. Let me pass that through the analyzer first." It lowercases your search to `"quick"`, compares it to the database's `"quick"`, and finds a perfect match.

#### Scenario 3: Using `match` on a keyword field (The Mistake)

Now you want to find in-stock items, so you use `match`:

JSON

```
GET /products/_search
{
  "query": {
    "match": {
      "status": "in-stock"
    }
  }
}
```

**Result: 0 Hits.**

_Why?_ The `match` query analyzed your input, perhaps stripping the hyphen and looking for `["in", "stock"]`. But the database strictly saved the exact string `["In-Stock"]`. They don't match.

#### Scenario 4: Using `term` on a keyword field (The Perfect Pair)

You run a strict `term` query against the strict `keyword` field:

JSON

```
GET /products/_search
{
  "query": {
    "term": {
      "status": "In-Stock"
    }
  }
}
```

_Why?_ Exact query meets exact data. Perfection.
### The Ultimate Cheat Sheet

If you tape this to your monitor for the exam, you will never get a search question wrong:

|**Query Type**|**Field Target**|**Behavior**|**Best Used For**|
|---|---|---|---|
|**`match`**|`text` fields|Analyzes your input before searching.|Search bars, descriptions, blogs, human input.|
|**`term`**|`keyword` (or numbers/dates)|Bypasses analysis for a strict, exact match.|IDs, status tags, email addresses, exact categories.|
