# kibana 统计语法

就是 ES DSL 语法的一部分，一般最外面有个 query，对一些字段进行高级查询的时候也可能针对单个字段设置 query，简单的键值对匹配用不着 query
前匹配

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "query_string": {
            "analyze_wildcard": true,
            "query": "*"
          }
        },
        {
          "prefix": {
            "upstream_version": "appstore-xxx-svc"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": 1540861800463,
              "lte": 1540865400463,
              "format": "epoch_millis"
            }
          }
        }
      ],
      "must_not": []
    }
  },
  "size": 0,
  "aggs": {
    "3": {
      "terms": {
        "field": "upstream_version",
        "size": 2,
        "order": {
          "_count": "desc"
        }
      },
      "aggs": {
        "4": {
          "date_histogram": {
            "field": "@timestamp",
            "interval": "1m",
            "time_zone": "Asia/Shanghai",
            "min_doc_count": 1
          }
        }
      }
    }
  }
}
```

# kibana 查询语法：

包含

```json
{
  "query": {
    "wildcard": {
      "api.keyword": "*/payment*"
    }
  }
}
前匹配
{
  "query": {
    "prefix": {
      "api.keyword": "*/payment*"
    }
  }
}
区间：
{
  "range": {
    "statuscode": {
      "gte": 400,
      "lt": 599
    }
  }
}
```
