```
POST /_ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "My Exam Practice Pipeline",
    "processors": [
      {
        "grok": {
          "field": "message",
          "patterns": [
            "%{IP:client_ip} - %{LOGLEVEL:log_level} \\[%{WORD:service}\\] %{WORD:error_type} %{NUMBER:failure_count}"
          ]
        }
      },
      {
        "rename": {
          "field": "service",
          "target_field": "service_name"
        }
      },
      {
        "convert": {
          "field": "failure_count",
          "type": "integer"
        }
      },
      {
        "set": {
          "field": "environment",
          "value": "production"
        }
      },
      {
        "script": {
          "source": "if (ctx.failure_count >= 5) {
            ctx.threat_score = 100;
            } else {
                ctx.threat_score = ctx.failure_count * 10;
                }"
        }
      }
    ]
  },
  "docs": [
    {
      "_source": {
        "message": "192.168.1.50 - ERROR [auth_service] failed_login 5"
      }
    }
  ]
}
```

