ES 操作命令

**增加**

```
# 单个增加
$ curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
# 批量增加， accounts.json包含1,000个documents
curl -H "Content-Type: application/json" -X POST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@/local-file-path/accounts.json"

```

> 批量操作只有index，create，update，delete

**查找**

```
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'

```

> - `took` – how long it took Elasticsearch to run the query, in milliseconds
>
> - `timed_out` – whether or not the search request timed out
>
> - `_shards` – how many shards were searched and a breakdown of how many shards succeeded, failed, or were skipped.
>
> - `max_score` – the score of the most relevant document found
>
> - `hits.total.value` - how many matching documents were found
>
> - `hits.sort` - the document’s sort position (when not sorting by relevance score)
>
> - `hits._score` - the document’s relevance score (not applicable when using `match_all`)
>
> - 获取超过1w条数据 需要加上  "track_total_hits":true ，不然只能显示出9999条
>
>   `GET "localhost:9200/bank/_search?track_total_hits=true&size=0`

**更改**

```
curl -X POST "localhost:9200/bank/_doc/8/_update?pretty" -H 'Content-Type: application/json' -d'
{
   "doc":{
      "account_number" : 8,
      "firstname" : "刘",
      "lastname" : "索隆",
      "gender" : "M",
      "address" : "699 Visitation Place",
      "employer" : "Glasstep",
      "email" : "janburns@glasstep.com",
      "city" : "Wakulla",
      "state" : "AZ"
   }
}
'

curl -X POST "localhost:9200/seats/_update_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "row": {
              "lte": 3
            }
          }
        },
        {
          "match": {
            "sold": false
          }
        }
      ]
    }
  },
  "script": {
    "source": "ctx._source.cost -= params.discount",
    "lang": "painless",
    "params": {
      "discount": 2
    }
  }
}
'

```

> post /索引/类型/1/_update
>
> {
>
>  “doc”:{
>
>  列名：值   //精准修改其中某个列
>
>  }
>
> }

**删除**

```
curl -X DELETE "localhost:9200/bank/_doc/8/
```



**mapping API**

```
# create a new index with an explicit mapping
$ curl -X PUT "localhost:9200/my-index-000001?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}
'
# use the put mapping API to add one or more new fields to an existing index.
$ curl -X PUT "localhost:9200/my-index-000001/_mapping?pretty" -H 'Content-Type: application/json' -d'
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}
'
#  get mapping API
curl -X GET "localhost:9200/my-index-000001/_mapping?pretty"
#  get field mapping API.
curl -X GET "localhost:9200/my-index-000001/_mapping/field/employee-id?pretty"

```

**Document  APIs**

```
# add 
$ curl -X PUT "localhost:9200/my-index-000001/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "foo": "baz"
}
'

# get and delete APIs use the path {index}/_doc/{id}
$ curl -X GET "localhost:9200/my-index-000001/_doc/1?pretty"
 
# update
$ curl -X POST "localhost:9200/my-index-000001/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "doc" : {
    "foo" : "qux"
  }
}
'
$ curl -X GET "localhost:9200/my-index-000001/_source/1?pretty"

```

**index templates**

```
curl -X PUT "localhost:9200/_template/template1?pretty" -H 'Content-Type: application/json' -d'
{
  "index_patterns":[ "index-1-*" ],
  "mappings": {
    "properties": {
      "foo": {
        "type": "keyword"
      }
    }
  }
}
'
curl -X PUT "localhost:9200/_template/template2?include_type_name=true&pretty" -H 'Content-Type: application/json' -d'
{
  "index_patterns":[ "index-2-*" ],
  "mappings": {
    "type": {
      "properties": {
        "foo": {
          "type": "keyword"
        }
      }
    }
  }
}
'
# 可以根据模板匹配创建新的index
# 以下分别创建了新的index：index-1-01，index-2-01。并且新index中包含模板中的mappings
curl -X PUT "localhost:9200/index-1-01?include_type_name=true&pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "type": {
      "properties": {
        "bar": {
          "type": "long"
        }
      }
    }
  }
}
'
curl -X PUT "localhost:9200/index-2-01?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "bar": {
        "type": "long"
      }
    }
  }
}
'

```

> 在 ElasticSearch 7.x版本中创建mapping并指定type，则需要在URL后面添加参数：`include_type_name=true`
>
> index模板创建好之后，然后在向一个不存在的index添加文档时，如果能找到合适的模板，则自动创建index，否则抛出index不存在
>
> ```
> PUT /_template/template_1
> {
>  "index_patterns" : ["*"],
>  "order" : 0,
>  "settings" : {
>      "number_of_shards" : 1
>  },
>  "mappings" : {
>      "_doc" : {
>          "_source" : { "enabled" : false }
>      }
>  }
> }
> 
> PUT /_template/template_2
> {
>  "index_patterns" : ["te*"],
>  "order" : 1,
>  "settings" : {
>      "number_of_shards" : 1
>  },
>  "mappings" : {
>      "_doc" : {
>          "_source" : { "enabled" : true }
>      }
>  }
> }
> # 首先从order=0进行匹配，由于其表达式为*，则默认会全匹配，故首先尝试使用该模板，然后再遍历下一个模板，也就是order=1的模板，如果匹配，则使用第二个模板的配置，如果不匹配，则使用第一个模板的配置，依次类推。
> ```
>
> 



SQL Access

ES SQL 操作仅支持'SELECT', 'SHOW'操作。

```
$ curl -X POST "localhost:9200/_sql?format=txt&pretty" -H 'Content-Type: application/json' -d'
{
  "query": "SELECT * FROM bank WHERE balance > 30000"
}
'

$ curl -X POST "localhost:9200/_sql?format=txt&pretty" -H 'Content-Type: application/json' -d'
{
  "query": "SELECT firstname AS name FROM bank WHERE balance > 40000 GROUP BY firstname"
}
'
# ORDER BY expression [ ASC | DESC ] [, ...] 按照expression进行排序，默认为ASC(升序)。
$ curl -X POST "localhost:9200/_sql?format=txt&pretty" -H 'Content-Type: application/json' -d'
{
  "query": "SELECT age a, COUNT(*) C FROM bank WHERE balance > 40000 GROUP BY age ORDER BY age ASC LIMIT 5"
}
'
# HAVING condition 满足condition的数据则返回
$ curl -X POST "localhost:9200/_sql?format=txt&pretty" -H 'Content-Type: application/json' -d'
{
  "query": "SELECT age AS a, COUNT(*) AS c FROM bank GROUP BY a HAVING c BETWEEN 15 AND 20"
  }
'
# Order By Score  字符串可以用params传入。/u0027是单引号。
curl -X POST "localhost:9200/_sql?format=txt&pretty" -H 'Content-Type: application/json' -d'
{
  "query": "SELECT SCORE(), * FROM library WHERE MATCH(name, ?) ORDER BY SCORE() DESC",
  "params": ["dune"]
  }
'

```

> COUNT() 计数
>
> ROUND(x, d) x指要处理的数，d是指保留几位小数 
>
> MAX()
>
> MIN()
>
> AVG()

**查看es的状态**

```
curl -XGET 'http://localhost:9200/_cluster/health?pretty=true'
```

**查看index状态** 

```
curl "localhost:9200/_cat/indices?v"
```



While the text/plain format is nice for humans, computers prefer something more structured. You can replace the value of format with: -json aka application/json - yaml aka application/yaml - smileaka application/smile - cbor aka application/cbor - txt aka text/plain - csv aka text/csv - tsv aka text/tab-separated-values


https://www.elastic.co/guide/en/elasticsearch/reference/7.2/getting-started-search.html

```
"settings" : {
        "index" : {
            "max_inner_result_window" :"10000"
  }
}
```

max_result_window是分页返回的最大数值，因为ES是分布式的，数据散落在各个节点，当你查询分页数据的时候，比如每页500条数据，那么需要在每个Shard先进行排序，取前500条数据，最后把每个Shard的数据在JVM中汇总，再次排序，最后取前500。max_result_window本身是对JVM的一种保护，通过设定一个合理的阈值，避免初学者分页查询时由于单页数据过而导致OOM。很多人会告诉你如果想要增加单页数据的上限，放开这个参数就行，但是如果你不知道这个参数的意义，那么就会导致频分的内存溢出而且很难找到原因，设置一个合理的大小是需要通过你的各项参数来确定的，比如你用户量、数据量、物理内存的大小等等，通过监控数据和分析各项指标从而确定一个最佳值，并非越大约好。JVM不能超过物理内存的一半。



**

python elasticsearch包

**添加数据**

```python
from elasticsearch import Elasticsearch

# 默认host为localhost,port为9200.但也可以指定host与port
es = Elasticsearch()

# 添加或更新数据,index，doc_type名称可以自定义，id可以根据需求赋值,body为内容
es.index(index="my_index",doc_type="test_type",id=1,body={"name":"python","addr":"深圳"})

# 或者:ignore=409忽略文档已存在异常
es.create(index="my_index",doc_type="test_type",id=1,ignore=409,body={"name":"python","addr":"深圳"})
```

**查询数据**

```python
from elasticsearch import Elasticsearch

es = Elasticsearch()

# 获取索引为my_index,文档类型为test_type的所有数据,result为一个字典类型
result = es.search(index="my_index",doc_type="test_type")

# 或者这样写:搜索id=1的文档
result = es.get(index="my_index",doc_type="test_type",id=1)

es.count(index="my_index",doc_type="test_type")
# 打印所有数据
for item in result["hits"]["hits"]:
    print(item["_source"])
```

**删除数据**

```python
form elasticsearch import Elasticsearch

es = Elasticsearch()

# 删除id=1的数据
result = es.delete(index="my_index",doc_type="test_type",id=1)
```

```
def read_csv(file):
    csv_rows = {}
 
    def format(source):
        try:
            source = literal_eval(source)
        except:
            pass
        finally:
            return source
 
    with open(file) as csvfile:
        reader = csv.DictReader(csvfile)
        title = reader.fieldnames
        for row in reader:
            csv_rows[format(row[title[0]])] = {title[i]:format(row[title[i]]) for i in range(len(title)) if not title[i].startswith('_')}
 
    return csv_rows
    
    
with open(json_file, "w") as f:
        if format:
            f.write(json.dumps(data, sort_keys=False, indent=4, separators=(',', ': '),encoding="utf-8",ensure_ascii=False))
        else:
            f.write(json.dumps(data))


```

