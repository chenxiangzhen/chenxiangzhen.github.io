## 索引雇员文档
第一个业务需求就是存储雇员数据。 这将会以 雇员文档 的形式存储：一个文档代表一个雇员。存储数据到 Elasticsearch 的行为叫做 索引 ，但在索引一个文档之前，需要确定将文档存储在哪里。

一个 Elasticsearch 集群可以 包含多个 索引 ，相应的每个索引可以包含多个 类型 。 这些不同的类型存储着多个 文档 ，每个文档又有 多个 属性 。

    Index Versus Index Versus Index

    你也许已经注意到 索引 这个词在 Elasticsearch 语境中包含多重意思， 所以有必要做一点儿说明：
    
    索引（名词）：
    
    如前所述，一个 索引 类似于传统关系数据库中的一个 数据库 ，是一个存储关系型文档的地方。 索引 (index) 的复数词为 indices 或 indexes 。
    
    索引（动词）：
    
    索引一个文档 就是存储一个文档到一个 索引 （名词）中以便它可以被检索和查询到。这非常类似于 SQL 语句中的 INSERT 关键词，除了文档已存在时新文档会替换旧文档情况之外。
    
    倒排索引：
    
    关系型数据库通过增加一个 索引 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 倒排索引 的结构来达到相同的目的。
    
    + 默认的，一个文档中的每一个属性都是 被索引 的（有一个倒排索引）和可搜索的。一个没有倒排索引的属性是不能被搜索到的。我们将在 倒排索引 讨论倒排索引的更多细节。

对于雇员目录，我们将做如下操作：

- 每个雇员索引一个文档，包含该雇员的所有信息。
- 每个文档都将是 employee 类型 。
- 该类型位于 索引 megacorp 内。
- 该索引保存在我们的 Elasticsearch 集群中。

实践中这非常简单（尽管看起来有很多步骤），我们可以通过一条命令完成所有这些动作：

```
curl -X PUT "localhost:9200/megacorp/employee/1" -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
'
```

注意，路径 /megacorp/employee/1 包含了三部分的信息：

- megacorp:索引名称
- employee:类型名称
- 1:特定雇员的ID

请求体 —— JSON 文档 —— 包含了这位员工的所有详细信息，他的名字叫 John Smith ，今年 25 岁，喜欢攀岩。

很简单！无需进行执行管理任务，如创建一个索引或指定每个属性的数据类型之类的，可以直接只索引一个文档。Elasticsearch 默认地完成其他一切，因此所有必需的管理任务都在后台使用默认设置完成。

进行下一步前，让我们增加更多的员工信息到目录中：

```
curl -X PUT "localhost:9200/megacorp/employee/2" -H 'Content-Type: application/json' -d'
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
'
```
```
curl -X PUT "localhost:9200/megacorp/employee/3" -H 'Content-Type: application/json' -d'
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
'
```

## 检索文档
目前我们已经在 Elasticsearch 中存储了一些数据， 接下来就能专注于实现应用的业务需求了。第一个需求是可以检索到单个雇员的数据。

这在 Elasticsearch 中很简单。简单地执行 一个 HTTP GET 请求并指定文档的地址——索引库、类型和ID。 使用这三个信息可以返回原始的 JSON 文档：
```
curl -X GET "localhost:9200/megacorp/employee/1"
```

返回结果包含了文档的一些元数据，以及 _source 属性，内容是 John Smith 雇员的原始 JSON 文档：
```
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

将 HTTP 命令由 PUT 改为 GET 可以用来检索文档，同样的，可以使用 DELETE 命令来删除文档，以及使用 HEAD 指令来检查文档是否存在。如果想更新已存在的文档，只需再次 PUT 。


## 轻量搜索
一个 GET 是相当简单的，可以直接得到指定的文档。 现在尝试点儿稍微高级的功能，比如一个简单的搜索！

第一个尝试的几乎是最简单的搜索了。我们使用下列请求来搜索所有雇员：
```
curl -X GET "localhost:9200/megacorp/employee/_search"

```
可以看到，我们仍然使用索引库 megacorp 以及类型 employee`，但与指定一个文档 ID 不同，这次使用 `_search 。返回结果包括了所有三个文档，放在数组 hits 中。一个搜索默认返回十条结果。
```
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
},{"_index":"megacorp","_type":"employee","_id":"1","_score":1.0,"_source":
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
},{"_index":"megacorp","_type":"employee","_id":"3","_score":1.0,"_source":
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}

```

注意：返回结果不仅告知匹配了哪些文档，还包含了整个文档本身：显示搜索结果给最终用户所需的全部信息。

接下来，尝试下搜索姓氏为 ``Smith`` 的雇员。为此，我们将使用一个 高亮 搜索，很容易通过命令行完成。这个方法一般涉及到一个 查询字符串 （_query-string_） 搜索，因为我们通过一个URL参数来传递查询信息给搜索接口：

```
curl -X GET "localhost:9200/megacorp/employee/_search?q=last_name:Smith"

```
我们仍然在请求路径中使用 _search 端点，并将查询本身赋值给参数 q= 。返回结果给出了所有的 Smith：
```
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
},{"_index":"megacorp","_type":"employee","_id":"1","_score":0.2876821,"_source":
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

```

## 使用查询表达式搜索
Query-string 搜索通过命令非常方便地进行临时性的即席搜索 ，但它有自身的局限性（参见 轻量 搜索 ）。Elasticsearch 提供一个丰富灵活的查询语言叫做 查询表达式 ， 它支持构建更加复杂和健壮的查询。

领域特定语言 （DSL）， 指定了使用一个 JSON 请求。我们可以像这样重写之前的查询所有 Smith 的搜索 ：

```
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
'

```
返回结果与之前的查询一样，但还是可以看到有一些变化。其中之一是，不再使用 query-string 参数，而是一个请求体替代。这个请求使用 JSON 构造，并使用了一个 match 查询（属于查询类型之一，后续将会了解）。


## 更复杂的搜索
现在尝试下更复杂的搜索。 同样搜索姓氏为 Smith 的雇员，但这次我们只需要年龄大于 30 的。查询需要稍作调整，使用过滤器 filter ，它支持高效地执行一个结构化查询。

```

curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
'


```


- [1]这部分与我们之前使用的 match 查询 一样。
- [2]这部分是一个 range 过滤器 ， 它能找到年龄大于 30 的文档，其中 gt 表示_大于(_great than)。

目前无需太多担心语法问题，后续会更详细地介绍。只需明确我们添加了一个 过滤器 用于执行一个范围查询，并复用之前的 match 查询。现在结果只返回了一个雇员，叫 Jane Smith，32 岁。

```
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
```

## 全文搜索
截止目前的搜索相对都很简单：单个姓名，通过年龄过滤。现在尝试下稍微高级点儿的全文搜索——一项 传统数据库确实很难搞定的任务。

搜索下所有喜欢攀岩（rock climbing）的雇员：

```
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
'

```
显然我们依旧使用之前的 match 查询在about 属性上搜索 “rock climbing” 。得到两个匹配的文档：


```
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, 
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, 
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

 

"_score"相关性得分

Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。第一个最高得分的结果很明显：John Smith 的 about 属性清楚地写着 “rock climbing” 。

但为什么 Jane Smith 也作为结果返回了呢？原因是她的 about 属性里提到了 “rock” 。因为只有 “rock” 而没有 “climbing” ，所以她的相关性得分低于 John 的。

这是一个很好的案例，阐明了Elasticsearch如何在全文属性上搜索并返回相关性最强的结果。Elasticsearch中的相关性概念非常重要，也是完全区别于传统关系型数据库的一个概念，数据库中的一条记录要么匹配要么不匹配。

## 短语搜索
找出一个属性中的独立单词是没有问题的，但有时候想要精确匹配一系列单词或者短语 。 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” 和 “climbing” ，并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。

为此对 match 查询稍作调整，使用一个叫做 match_phrase 的查询：

```
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
'

```
拷贝为 CURL在 SENSE 中查看 
毫无悬念，返回结果仅有 John Smith 的文档。


```
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         }
      ]
   }
}
```

## 高亮搜索
许多应用都倾向于在每个搜索结果中 高亮 部分文本片段，以便让用户知道为何该文档符合查询条件。在 Elasticsearch 中检索出高亮片段也很容易。

再次执行前面的查询，并增加一个新的 highlight 参数：

```
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
'

```
拷贝为 CURL在 SENSE 中查看 
当执行该查询时，返回结果与之前一样，与此同时结果中还多了一个叫做 highlight 的部分。这个部分包含了 about 属性匹配的文本片段，并以 HTML 标签 <em></em> 封装：


```
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" 
               ]
            }
         }
      ]
   }
}
```

[1]原始文本中的高亮片段


## 分析
终于到了最后一个业务需求：支持管理者对雇员目录做分析。 Elasticsearch 有一个功能叫聚合（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 GROUP BY 类似但更强大。

举个例子，挖掘出雇员中最受欢迎的兴趣爱好：

```
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
'


```
拷贝为 CURL在 SENSE 中查看 
暂时忽略掉语法，直接看看结果：


```
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

可以看到，两位员工对音乐感兴趣，一位对林地感兴趣，一位对运动感兴趣。这些聚合并非预先统计，而是从匹配当前查询的文档中即时生成。如果想知道叫 Smith 的雇员中最受欢迎的兴趣爱好，可以直接添加适当的查询来组合查询：

```
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
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
'

```
拷贝为 CURL在 SENSE 中查看 
all_interests 聚合已经变为只包含匹配查询的文档：

  
```
...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
```

聚合还支持分级汇总 。比如，查询特定兴趣爱好员工的平均年龄：

```
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
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
'

```
拷贝为 CURL在 SENSE 中查看 
得到的聚合结果有点儿复杂，但理解起来还是很简单的：

  
```
...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```

输出基本是第一次聚合的加强版。依然有一个兴趣及数量的列表，只不过每个兴趣都有了一个附加的 avg_age 属性，代表有这个兴趣爱好的所有员工的平均年龄。

即使现在不太理解这些语法也没有关系，依然很容易了解到复杂聚合及分组通过 Elasticsearch 特性实现得很完美。可提取的数据类型毫无限制。



## 教程结语
欣喜的是，这是一个关于 Elasticsearch 基础描述的教程，且仅仅是浅尝辄止，更多诸如 suggestions、geolocation、percolation、fuzzy 与 partial matching 等特性均被省略，以便保持教程的简洁。但它确实突显了开始构建高级搜索功能多么容易。不需要配置——只需要添加数据并开始搜索！

很可能语法会让你在某些地方有所困惑，并且对各个方面如何微调也有一些问题。没关系！本书后续内容将针对每个问题详细解释，让你全方位地理解 Elasticsearch 的工作原理。