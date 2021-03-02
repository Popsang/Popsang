# 创建索引

```
PUT test_2 {    "mappings":{      "_doc":{        "properties":{            "timestamp":{                "type":"date"            },            "taskId":{                "type":"long",                "copy_to":"allTags"            },            "moninetObject":{                "type":"keyword",                "copy_to":"allTags"            },            "area":{                "type":"keyword",                "copy_to":"allTags"            },            "idc":{                "type":"keyword",                "copy_to":"allTags"            },            "isp":{                "type":"keyword",                "copy_to":"allTags"            },            "sourceIp":{                "type":"keyword",                "copy_to":"allTags"            },            "commandResult":{                "type":"object",                "enabled":false            },            "allTags":{                "type":"text"            }        }      }    } }
```



# 插入模板

```
PUT _template/template_1 {  "index_patterns": ["te*", "bar*"],  "settings": {    "number_of_shards": 1  },  "mappings": {    "_doc": {      "_source": {        "enabled": false      },      "properties": {        "host_name": {          "type": "keyword"        },        "created_at": {          "type": "date",          "format": "EEE MMM dd HH:mm:ss Z yyyy"        }      }    }  } }
```



# 插入

```
POST test_2/_doc {  "timestamp": "1605167869040",  "moninetObject": "www.baidu.com",  "taskId": 30,  "sourceIp": "127.0.0.1",  "area": "bj",  "idc": "bj01",  "isp":"yidong",  "commandReslut":[{"test":"test"},{"mtr":2}] }
```

# 查询

```
2.查询指定字段下的数据，直接用match GET base_prsync_log/_search {  "query": {    "match": {      "message": "sz*"          }  } }
```



```
3.当同一个属性满足逻辑或时的查询，用bool和should GET base_prsync_log/_search {  "query": {    "bool": {      "should": [        {          "match": {            "message": "sh*"          }        },        {          "match": {            "message": "sz*"          }        }      ]    }  } }
```



```
4.多条件查询 GET base_prsync_log/_search，用bool和must {  "query": {    "bool": {      "must": [        {          "match": {            "message": "sh*"          }        },        {          "match": {            "agent.hostname": "win08-core-8-16-2"          }        }      ]    }  } }
```



```
5.范围查询并进行排序，用range：gte和lte指定范围，再用sort：asc和desc排序 GET base_company_event*/_search {  "query": {    "range": {      "p_stock2201_f033n": {        "gte": 1000,        "lte": 2300      }    }  },  "sort": [    {      "p_stock2201_f033n": {        "order": "asc"      }    }  ] }
```



```
6.聚合查询 （1）固定用到size和aggs，field后面输入属性，from和to后边输入要分段的范围 GET base_company_event*/_search {  "size": 20,  "aggs": {    "p_stock2201_f033n": {      "range": {        "field": "p_stock2201_f033n",        "ranges": [          {            "from": 1000,            "to": 2000          },{            "from": 2000,            "to": 3000          },{            "from": 3000,            "to": 4000          }        ]      }    }  } }
```

# 额外扩展

## 取消type字段

In Elasticsearch 7.0, each API will support typeless requests, and specifying a type will produce a deprecation warning. Certain typeless APIs are also available in 6.7, to enable a smooth upgrade path to 7.0.



### Indices APIs

Index creation, index template, and mapping APIs support a new `include_type_name` URL parameter that specifies whether mapping definitions in requests and responses should contain the type name. The parameter defaults to `true` in version 6.7 to match the pre-7.0 behavior of using type names in mappings. It defaults to `false` in version 7.0 and will be removed in version 8.0.

It should be set explicitly in 6.7 to prepare to upgrade to 7.0. To avoid deprecation warnings in 6.7, the parameter can be set to either `true` or `false`. In 7.0, setting `include_type_name` at all will result in a deprecation warning.

See some examples of interactions with Elasticsearch with this option set to `false`:

```
PUT index?include_type_name=false {  "mappings": {    "properties": {       "foo": {        "type": "keyword"      }    }  } }
```



### index templates

It is recommended to make index templates typeless by re-adding them with `include_type_name` set to `false`. Under the hood, typeless templates will use the dummy type `_doc` when creating indices.

In case typeless templates are used with typed index creation calls or typed templates are used with typeless index creation calls, the template will still be applied but the index creation call decides whether there should be a type or not. For instance in the below example, `index-1-01` will have a type in spite of the fact that it matches a template that is typeless, and `index-2-01` will be typeless in spite of the fact that it matches a template that defines a type. Both `index-1-01` and `index-2-01` will inherit the `foo` field from the template that they match.

```
PUT _template/template1?include_type_name=false {  "index_patterns":[ "index-1-*" ],  "mappings": {    "properties": {      "foo": {        "type": "keyword"      }    }  } } 
```

```
PUT _template/template1?include_type_name=false {  "index_patterns":[ "index-1-*" ],  "mappings": {    "properties": {      "foo": {        "type": "keyword"      }    }  } } 
```