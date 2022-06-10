# ES
es学习文档

## es 倒排索引
1. 切词：英文以空格切分每一个单词
2. 规范化：单复数统一、大小写统一、现在分词过去分词统一
3. 去重
4. 字典序


## ES Mapping
1. 概念
ES 中 Mapping 有点类似于 RDB (关系型数据库) 中 表结构 的概念，在 MySQL 中，表结构里包含了字段名称，字段类型还有索引信息等。在 Mapping 里也包含了一些属性，比如字段名称、
类型、字段使用的分词器、是否评分、是否创建索引等属性
2. ES 数据类型
    1. 常见类型
        1. 数字类型：long,integer,byte,double,float
        2. keyword:keyword 不会被分词，用于精准匹配
        3. Dates(时间类型):包含 date和 date nanos(纳秒)
        4. alias：为现有字段定义别名
        5. text:会被分词，一般被用作全文搜索
    2. 对象关系类型
        1. object:用于单个json对象
        2. nested:用于json对象数组
    3. 结构化类型
        1. geo-point:维度/经度积分
        2. geo-shape:用于多边形等复杂形状
        3. point:笛卡尔坐标点
        4. shape:笛卡尔任意几何图形
    4. 特殊类型
        1. IP 地址：IP 用于 IPV4 和 IPV6 地址
3. 两种映射类型
    1. dynamic mapping（动态映射或者字段映射）：导入数据自动创建 mapping ，字段类型自动识别
    
    |类型|默认映射|
    |---|---|
    |整数|long|
    |浮点型|float|
    |true or false|boolean|
    |日期|date|
    |数组|取决于数组中的第一个有效值|
    |对象|object|
    |字符串|如果不是数字和日期类型，会被映射为text和keyword两种类型|
    **除了上述字段类型之外，其他类型都必须手工指定，因为其他类型ES无法自动识别**
    2. expllcit mapping（静态映射或手工映射或显示映射）：导入数据前手动创建索引，所以字段类型自己设置
4. 映射参数
    1. index：是否对当前字段创建倒排索引，默认是true
    2. analyzer: 指定分析器（character filter\tokenizer.Token filters）
    3. doc_values:为了提升排序和聚合效率，默认true
5. 全文检索
    1. match
    2. match_all
    3. multi_match:
    4. match_phrase
    
    
## 6. es 中 term、match、query_string、match_phrase对于 keyword、text 类型字段的查询
### keyword
1. keyword 不会进行分词，故存入一条数据 "washing machine"，数据还是 ["washing machine"] ,不做任何处理
2. term 查询：term 不会进行分词，需完全匹配，包括单词的 循序，大小写等
3. match 查询：match 会进行分词，但由于 keyword 不进行分词，故需要完全匹配
4. query_string 查询：query_string 会进行分词，但由于 keyword 不进行分词，故需要完全匹配
5. match_phrase 查询：match_phrase 会进行分词，但由于 keyword 不进行分词，故需要完全匹配
### text
1. text 会进行分词，故存入一条数据"washing machine"，会被分词为 ["washing","machine"],并且有先后顺序
2. term 查询：term 不会进行分词，但 text 中存储的数据已经被分词，故需要完全匹配分词后的一个数据；若使用 "washing machine" 进行匹配，反而无法匹配，因为底层数据存储已被分词
3. match 查询：match 会进行分词，若使用语句 "washing my hands" 进行匹配，首先会被match分词为 ["washing","my","hands"]；而底层存储text分词为["washing","machine"], 其中 "washing" 匹配上
4. query_string查 询：query_string 会进行分词，若使用语句 "washing my hands" 进行匹配，首先会被 query_string 分词为 ["washing","my","hands"] ；而底层存储 text 分词为 ["washing","machine"] ，其中 "washing" 匹配上
5. match_phrase查询：match_phrase 会进行分词，match_phrase 的分词结果必须在 text 字段分词中都包含，而且顺序必须相同，而且必须都是连续的。
**注意：是否能够匹配，首先查看字段的存储类型，故设计 es 的 mapping 尤为重要**

## 7. es 中 bool 查询的使用
### must
1. 返回的文档必须满足 must 子句的所有条件,并且参与计算评分
2. must 数组中可以跟随 0-n 个过滤语句，在使用过滤条件时优先考虑需要查询的字段类型，根据字段类型判断使用 term|match|query_string|match_phrase
```
GET hy_enterprise/_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "term": {
               "eid": {
                 "value": "534472fd-7d53-4958-8132-d6a6242423d8"
               }
             }
           },
           {
             "match": {
               "name_ik": "小米科技"
             }
           }
         ]
       }
     }
   }
```
### should
1. 返回满足 should 子句的数据，子句之间以 or 关联
2. 结果会返回所有匹配的语句
```
GET hy_enterprise/_search
   {
     "_source": ["eid","name"], 
     "query": {
       "bool": {
         "should": [
           {
             "term": {
               "eid": {
                 "value": "dfad"
               }
             }
           },
           {
             "term": {
               "eid": {
                 "value": "dfadf"
                 
               }
             }
           }
         ]
       }
     }
   }
```
### must not
1. 返回的数据必须不满足 must not 任何一个子句的条件
```
GET hy_enterprise/_search
{
  "_source": ["eid","name"], 
  "query": {
    "bool": {
      "must_not": [
        {
          "term": {
            "eid": {
              "value": "dfad"
            }
          }
        },
        {
          "term": {
            "eid": {
              "value": "dfadf"
              
            }
          }
        }
      ]
    }
  }
}
```
### filter
1. 返回的数据必须满足 filter 的子句条件（目前测试 filter 里面只能包含一个子句），filter 不会进行评分
2. 因为 filter 无需进行评分，故其查询速率更快；并且 filter 会自动被 es 缓存结果，后续查询效率更高。
```
GET hy_enterprise/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "eid": "534472fd-7d53-4958-8132-d6a6242423d8"
        }
      }
    }
  }
}
```





### datax数据量拉取批次以及并行度的设置
## 1. 调优参数
- batch:批次数量--在es的json配置中自行配置，具体配置根据后面公时计算
- parallel：并行度--在es的json配置中自行配置，具体配置根据后面公时计算
- memory:内存--在配置datax任务时进行配置，其配置大小需在任何极端情况下皆充裕满足
- es_handle:es吞吐量--es自身的属性，初始搭建es集群时应测试
- max：单表中，数据最大的一条数据所占内存
    1. 粗略计算一条 es 数据大小 max 的计算
          1. 使用 max(length),取出表中数据长度最长的数据
          2. length 函数统计字符串长度，1 char = 2 B，1KB=1024B,1M=1024KB,1G=1024MB,1TB=1024GB
          3. 通过上面方法计算出 hive 表中数据大小的 max
## 2. 计算公式
- max * batch * parallels_handle = memory * 30% * 80%(堆内存中年轻代内存占比) + es_handle

    