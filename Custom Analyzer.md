https://www.elastic.co/docs/manage-data/data-store/text-analysis/create-custom-analyzer#_example_configuration

Think of an analyzer as a **3-step assembly line**. Text flows through it in this exact order:

### The 3-Step Pipeline

1. **Character Filters (0 or more):** Cleans the raw text _before_ chopping.
    
    - _Example:_ `html_strip` (removes `<p>` tags).
        
2. **Tokenizer (Exactly 1 required):** Chops the text into individual tokens (words).
    
    - _Example:_ `standard` (chops on spaces/punctuation).
        
3. **Token Filters (0 or more):** Modifies, adds, or deletes the chopped tokens.
    
    - _Example:_ `lowercase` or `stop` (removes "the", "and").
        

### Point to Remember

Always test your custom pipeline using the **`_analyze` API** before you index real data to ensure it chops the text exactly how you expect!

### Example

```
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
}
POST my-index-000001/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Is this <b>déjà vu</b>?"
}
```

