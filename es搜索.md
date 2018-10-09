### es对于范围的搜索

​	搜索姓氏为“smith”而且年龄大于30的人

```
es语句的搜索方式
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {						多个条件同时查询
            "must": {					必须满足的条件
                "match" : {             通过分析器解析（解析last_name属性）后进行搜索
                    "last_name" : "smith" 
                }
            },
            "filter": {					过滤条件
                "range" : {				比较的写法
                    "age" : { "gt" : 30 } 			gt大于，gte大于等于，lt小于，lte小于等于
                }
            }
        }
    }
}
java的写法
BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
boolQuery.must(QueryBuilders.matchQuery("last_name", "smith")).
          	filter(QueryBuilders.rangeQuery("age").gt(30));
SearchResponse res = client.prepareSearch(INDEX).    	INDEX索引名称
			setTypes(TYPE).setQuery(boolQuery).			TYPE类型名称				
			setSize(size).get();
```

### es对于短语的搜索

​	搜索包含rock climbing短语的文档

```
es语句的搜索方式
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {			必须满足rock climbing两个单词相连，不同于match的搜索方式		
            "about" : "rock climbing"
        }
    }
}
java的写法
BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();P
boolQuery.must(QueryBuilders.matchPhraseQuery("about", "rock climbing"));
SearchResponse res = client.prepareSearch(INDEX).    	INDEX索引名称
			setTypes(TYPE).setQuery(boolQuery).			TYPE类型名称				
			setSize(size).get();
```

### es搜索分析

```
对所有人进行兴趣爱好统计分析       interests兴趣爱好
GET /megacorp/employee/_search
{
  "aggs": {									固定写法
    "all_interests": {						固定写法
      "terms": { "field": "interests" }		搜索所有的兴趣及每个兴趣下的人数
    }
  }
}
对所有姓氏为"smith"的人做兴趣爱好统计分析
GET /megacorp/employee/_search
{
  "query": {
    "match": {					
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
查询所有兴趣爱好员工的平均年龄
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

### es集群健康查看

```
GET /_cluster/health
示例： http://192.168.8.106:9200/_cluster/health
返回结果：
	{
    	"cluster_name": "coron",
    	"status": "yellow",
    	"timed_out": false,
    	"number_of_nodes": 1,
    	"number_of_data_nodes": 1,
    	"active_primary_shards": 14,		   表示主分片的数量
        "active_shards": 14,				   表示分片的数量			
    	"relocating_shards": 0,
    	"initializing_shards": 0,
    	"unassigned_shards": 14,               没有被分配到任何节点的副本分片数量
    	"delayed_unassigned_shards": 0,
    	"number_of_pending_tasks": 0,
    	"number_of_in_flight_fetch": 0,
    	"task_max_waiting_in_queue_millis": 0,
    	"active_shards_percent_as_number": 50
	}
status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：
green：所有的主分片和副本分片都正常运行。
yellow：所有的主分片都正常运行，但不是所有的副本分片都正常运行。
red：有主分片没能正常运行。
```

### es创建小细节1-1

```
{
   "settings" : {
      "number_of_shards" : 3,			主分片数量
      "number_of_replicas" : 1			副本数量
   }
}
```

### 测试es是否启动成功

```
终端的写法：
curl 'http://localhost:9200/?pretty'

浏览器写法：
http://localhost:9200/?pretty

返回结果：
{
    "name": "coron-node",
    "cluster_name": "coron",					拥有相同的cluster_name可以一起工作并共享数据
    "cluster_uuid": "dXuWBE9wSMO9p8fdvf5lcw",
    "version": {
        "number": "5.5.0",
        "build_hash": "260387d",
        "build_date": "2017-06-30T23:16:05.735Z",
        "build_snapshot": false,
        "lucene_version": "6.6.0"
    },
    "tagline": "You Know, for Search"
}
```

### es文档的元数据

```
_index		表示文档在哪里存放
_type		文档表示的对象类别	
_id			文档唯一标识
```

