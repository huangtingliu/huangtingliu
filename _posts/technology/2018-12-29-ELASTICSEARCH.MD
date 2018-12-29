---
layout: default
title: ELASTICSEARCH搜索引擎（下）
tags: [大数据,搜索引擎]
imgPath: 2018-09-14-ELASTICSEARCH
---

在上一篇文章[ELASTICSEARCH搜索引擎（上）]({{site.url}}/2018/09/14/ELASTICSEARCH/)中介绍了在windows下elasticsearch的基本安装，本文继续介绍elasticsearch的基本用法

# 一、elasticsearch的基础概念
### 1. 集群
多个elasticsearch节点的集合
### 2. 节点
elasticsearch主节点或elasticsearch从节点
### 3. 索引
含有相同属性的文档集合（名字来识别），相当于一个database数据库，比如说（图书数据库）
### 4. 类型
索引可以定义一个或多个类型、文档必须属于一个类型，相当于一个table表（比如说科学类图书、历史类图书...）
### 5. 文档
文档是可以被索引的基本数据单位，相当于一行row记录（比如说三国演义）
### 6. 分片
每个索引都有多个分片，每个分片是一个Lucene索引（假设一个索引数据量很大，硬盘压力大，搜索速度出现瓶颈），提高搜索以及其他操作性能效率
> es默认创建5个分片，一个备份,分片只能创建索引的时候指定

### 7. 备份
拷贝一份分片就完成了分片的备份（当某分片失败或出现问题备份的分片就可以代替工作），提高可用性，还可以拿来执行搜索操作，分担搜索的压力
> 可以动态修改

# 二、基本用法
### 1. 索引创建（restful api风格）
请求API基本格式：http://ip:port/索引/类型/文档id,参数为json格式  
请求方法为常用HTTP动词 GET/PUT/POST/DELETE

### 2.在head插件浏览器页面上创建索引（非结构化索引）
a.选择索引-新建索引，默认分片5个，备份1个，输入索引名称，点击OK，会弹出创建成功的提示信息  
b.成功后在集群概览上回出现该索引  
c.点击索引信息，其中"mappings": { }表示为非结构化索引  
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/6head-plugin-create-index.png)

### 3.创建结构化索引
a.点击符合查询[+]  
b.在查询框的第二个输入框输入：索引名/<类型/_mappings，如book/novel/_mappings   
c.在textarea框输入json串    
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/7head-plugin-create-index.png)

d.也可以在postman（htpp模拟器）中创建索引，本质上就是一个http请求，参数为json格式，
请求地址可以填：http://ip:port/索引，如127.0.0.1:9200/people，HTTP提交方式为PUT
```
{
	"settings":{
		"number_of_shards":3,
		"number_of_replicas":1
	},
	"mappings":{
		"man":{
			"properties":{
				"name":{
					"type":"text"
				},
				"country":{
					"type":"keyword"
				},
				"age":{
					"type":"integer"
				},
				"date":{
					"type":"date",
					"format":"yyyy-MM-dd||yyyy-MM-dd HH:mm:ss||epoch_millis"
				}
			}
		}
	}
}
```
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/8postman-create-index.png)
> settings里的number_of_shards为分片数  
 number_of_replicas为备份数  
 注意这里的参数mappings里面的type在5.x版本是可以有多个的，比如这里有一个man,还可以有一个woman，但在6.x版本只允许一个type，7.x将移除type

### 4.插入文档
- 指定文档id插入

使用PUT提交,请求格式http://ip:port/索引/类型/文档id，如127.0.0.1:9200/people/man/1  
参数json格式：
```
{
	"name":"huangtl",
	"country":"China",
	"age":26,
	"date":"1992-01-01"
}
```
- 自动产生文档id插入  

使用POST提交，请求格式http://ip:port/索引/类型/，如127.0.0.1:9200/people/man/  
参数json格式：
```
{
	"name":"xiaoming",
	"country":"China",
	"age":20,
	"date":"1998-01-01"
}

```

### 5.修改文档
- 覆盖修改

使用POST方法，请求格式http://ip:port/索引/类型/文档id/_upate，如127.0.0.1:9200/people/man/1/_update  
参数为：
```
{
	"doc":{
		"name":"修改后的name"
	}
}
```
> 可以看到运行后id为1的记录name值已经变成"修改后的name"了

![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/12edit-by-doc-id-after.png)
- 通过脚本修改文档

使用POST方法，请求格式http://ip:port/索引/类型/文档id/_upate，如127.0.0.1:9200/people/man/1/_update  
参数为：
```
{
	"script":{
		"lang":"painless",
		"inline":"ctx._source.age += 10"
	}
}
```
> 其中lang指定脚本语言，这里使用es内置的painless，inline重的ctx为上下文,_source为上下文属性，age为之前定义的字段, +=10表示将年龄加10后赋值给age

![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/14edit-by-script.png)

也可以将值放在外部,修改参数为：
```
{
	"script":{
		"lang":"painless",
		"inline":"ctx._source.age = params.age",
		"params":{
			"age":100
		}
	}
}
```
### 6.删除文档和索引
- 删除文档：使用DELETE提交,请求格式http://ip:port/索引/类型/文档id，如127.0.0.1:9200/people/man/1
- 删除索引：使用DELETE提交,请求格式http://ip:port/索引，如127.0.0.1:9200/people

### 7.查询
- 简单查询
使用GET方法，请求格式http://ip:port/索引/类型/文档id，如127.0.0.1:9200/people/man/1
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/18simple-search-by-id.png)
- 条件查询
使用POST方法，请求格式http://ip:port/索引/_search，如127.0.0.1:9200/people/_search
使用query、match_all、match关键字
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/19condition-search.png)
> 1.查询所有  
参数为：
```
{
	"query":{
		"match_all":{}
	}
}
```
返回参数为：
```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 1,
    "hits": [
      {
        "_index": "people",
        "_type": "man",
        "_id": "ZZkUnWQB2w-Qdl1vkEMJ",
        "_score": 1,
        "_source": {
          "name": "wangfei",
          "country": "China",
          "age": 38,
          "date": "1988-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "ZpkVnWQB2w-Qdl1vIUPB",
        "_score": 1,
        "_source": {
          "name": "anna",
          "country": "USA",
          "age": 24,
          "date": "1994-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "huangtl",
          "country": "China",
          "age": 26,
          "date": "1992-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "Y5kSnWQB2w-Qdl1v1UO4",
        "_score": 1,
        "_source": {
          "name": "xiaoming",
          "country": "China",
          "age": 20,
          "date": "1998-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "ZJkUnWQB2w-Qdl1vCUNt",
        "_score": 1,
        "_source": {
          "name": "xiaohong",
          "country": "China",
          "age": 28,
          "date": "1999-01-01"
        }
      }
    ]
  }
}
```
> 返回参数解释
```
{
	took：返回毫秒数，
	hits:代表响应的全部结果
	{
		total:返回总条数
		hits:返回的记录，默认为10条
		[
			{
				_index:索引名
				_type:类型	
				_id: 自动id
				_score: 匹配程度
				_source:实体资源信息
			}
		]
	}
}
```
> 2.查询指定范围  
如何指定hits的返回条数?  
修改请求参数为：
```
{
	"query":{
		"match_all":{}
	},
	"from":1,
	"size":1
}
```
from 代表从第几条记录开始（0为第一条），size表示显示记录数，这里from是1代表从第二条开始显示1条，即为第二条数据
返回参数信息
```
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 1,
    "hits": [
      {
        "_index": "people",
        "_type": "man",
        "_id": "ZpkVnWQB2w-Qdl1vIUPB",
        "_score": 1,
        "_source": {
          "name": "anna",
          "country": "USA",
          "age": 24,
          "date": "1994-01-01"
        }
      }
    ]
  }
}
```
> 3.根据关键词查询  
请求参数修改为：
```
{
	"query":{
		"match":{
			"name":"huangtl"
		}
	}
}
```
> 4.自定义排序  
```
{
	"query":{
		"match_all":{}
	},
	"sort": [
		{
			"date": {
				"order": "desc"
			}
		}
	]
}
```
- 聚合查询  
使用POST方法，请求格式http://ip:port/索引/_search，如127.0.0.1:9200/people/_search  
使用aggs关键字  
1. 简单聚合  
只会返回字段值和count数
```
{
	"aggs":{
		"group_by_country":{
			"terms": {
				"field": "country"
			}
		},
		"group_by_age":{
			"terms": {
				"field": "age"
			}
		}
	}
}
```
2. 详细聚合  
返回count、min、max、avg、sum
```
{
	"aggs":{
		
		"grades_age":{
			"stats":{
				"field": "age"
			}
		}
	}
}
```
返回参数：
```
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 1,
    "hits": [
      {
        "_index": "people",
        "_type": "man",
        "_id": "ZZkUnWQB2w-Qdl1vkEMJ",
        "_score": 1,
        "_source": {
          "name": "wangfei",
          "country": "China",
          "age": 38,
          "date": "1988-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "ZpkVnWQB2w-Qdl1vIUPB",
        "_score": 1,
        "_source": {
          "name": "anna",
          "country": "USA",
          "age": 24,
          "date": "1994-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "huangtl",
          "country": "China",
          "age": 26,
          "date": "1992-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "Y5kSnWQB2w-Qdl1v1UO4",
        "_score": 1,
        "_source": {
          "name": "xiaoming",
          "country": "China",
          "age": 20,
          "date": "1998-01-01"
        }
      },
      {
        "_index": "people",
        "_type": "man",
        "_id": "ZJkUnWQB2w-Qdl1vCUNt",
        "_score": 1,
        "_source": {
          "name": "xiaohong",
          "country": "China",
          "age": 28,
          "date": "1999-01-01"
        }
      }
    ]
  },
  "aggregations": {
    "grades_age": {
      "count": 5,
      "min": 20,
      "max": 38,
      "avg": 27.2,
      "sum": 136
    }
  }
}
```

# 三、总结
本文介绍内容分为两部分
1. elasticsearch的基本概念，描述了集群、节点、索引、类型、文档、分片、备份
2. elasticsearch的基本用法，包括
- 创建索引，包括http请求创建和head插件创建索引
- 插入文档，包括指定文档id插入和自动产生文档id插入
- 修改文档，包括覆盖修改和脚本修改
- 删除文档和索引
- 查询，包括简单查询、条件查询和聚合查询，其中，条件查询又包括了查询所有、指定范围查询、关键词查询，聚合查询包括了简单聚合和详细聚合

感谢阅读