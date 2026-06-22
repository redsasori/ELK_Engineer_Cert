```
PUT employee_data
{
    "mappings": {
        "properties": {
            "employee_id":
            {
                "type": "keyword"
            },
            "fullname" :{
                "type": "text",

            "fields": {
                "keyword": {
                    "type": "keyword",
                    "ignore_above": "256"
                }
            }
            }
        }
    }
}

```

```
POST employee_data/_bulk
{"index":{"_id": "1"}} 
{"employee_id": "143636","fullname": "Nilesh Sachin Mishra"}
{"index":{"_id": "2"}} 
{"employee_id": "68396376", "fullname": "Nilesh Kumar Sharma"}
```

