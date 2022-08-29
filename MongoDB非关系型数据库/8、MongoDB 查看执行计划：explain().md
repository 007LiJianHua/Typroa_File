[toc]

[MongoDB](https://so.csdn.net/so/search?q=MongoDB&spm=1001.2101.3001.7020)中的`explain()`函数可以帮助我们查看查询相关的信息，这有助于我们快速查找到搜索瓶颈进而解决它，本文我们就来看看`explain()`的一些用法及其查询结果的含义。

整体来说，`explain()`的用法和`sort()`、`limit()`用法差不多，不同的是`explain()`必须放在最后面。

## 一、基本用法

先来看一个基本用法：

```java
db.sang_collect.find({x:1}).explain()
```

直接跟在`find()`函数后面，表示查看`find()`函数的执行计划，结果如下：

```java
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "sang.sang_collect",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "x" : {
                "$eq" : 1.0
            }
        },
        "winningPlan" : {
            "stage" : "COLLSCAN",
            "filter" : {
                "x" : {
                    "$eq" : 1.0
                }
            },
            "direction" : "forward"
        },
        "rejectedPlans" : []
    },
    "serverInfo" : {
        "host" : "localhost.localdomain",
        "port" : 27017,
        "version" : "3.4.9",
        "gitVersion" : "876ebee8c7dd0e2d992f36a848ff4dc50ee6603e"
    },
    "ok" : 1.0
}
```

返回结果包含两大块信息，一个是 `queryPlanner`，即查询计划，还有一个是 `serverInfo`，即`MongoDB`服务的一些信息。

那么这里涉及到的参数比较多，我们来一一看一下：

| 参数           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
|                |                                                              |
| plannerVersion | 查询计划版本                                                 |
| namespace      | 要查询的集合                                                 |
| indexFilterSet | 是否使用索引                                                 |
| parsedQuery    | 查询条件，此处为x=1                                          |
| winningPlan    | 最佳执行计划                                                 |
| stage          | 查询方式，常见的有COLLSCAN/全表扫描、IXSCAN/索引扫描、FETCH/根据索引去检索文档、SHARD_MERGE/合并分片结果、IDHACK/针对_id进行查询 |
| filter         | 过滤条件                                                     |
| direction      | 搜索方向                                                     |
| rejectedPlans  | 拒绝的执行计划                                               |
| serverInfo     | MongoDB服务器信息                                            |

## 二、添加不同参数

`explain()` 也接收不同的参数，通过设置不同参数我们可以查看更详细的查询计划。

queryPlanner：是默认参数，添加queryPlanner参数的查询结果就是我们上文看到的查询结果，so，这里不再赘述。

executionStats：会返回最佳执行计划的一些统计信息，如下：

```java
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "sang.sang_collect",
        "indexFilterSet" : false,
        "parsedQuery" : {},
        "winningPlan" : {
            "stage" : "COLLSCAN",
            "direction" : "forward"
        },
        "rejectedPlans" : []
    },
    "executionStats" : {
        "executionSuccess" : true,
        "nReturned" : 10000,
        "executionTimeMillis" : 4,
        "totalKeysExamined" : 0,
        "totalDocsExamined" : 10000,
        "executionStages" : {
            "stage" : "COLLSCAN",
            "nReturned" : 10000,
            "executionTimeMillisEstimate" : 0,
            "works" : 10002,
            "advanced" : 10000,
            "needTime" : 1,
            "needYield" : 0,
            "saveState" : 78,
            "restoreState" : 78,
            "isEOF" : 1,
            "invalidates" : 0,
            "direction" : "forward",
            "docsExamined" : 10000
        }
    },
    "serverInfo" : {
        "host" : "localhost.localdomain",
        "port" : 27017,
        "version" : "3.4.9",
        "gitVersion" : "876ebee8c7dd0e2d992f36a848ff4dc50ee6603e"
    },
    "ok" : 1.0
}
```

这里除了我们上文介绍到的一些参数之外，还多了executionStats参数，含义如下：

| 参数                        | 含义                                     |
| --------------------------- | ---------------------------------------- |
| executionSuccess            | 是否执行成功                             |
| nReturned                   | 返回的结果数                             |
| executionTimeMillis         | 执行耗时                                 |
| totalKeysExamined           | 索引扫描次数                             |
| totalDocsExamined           | 文档扫描次数                             |
| executionStages             | 这个分类下描述执行的状态                 |
| stage                       | 扫描方式，具体可选值与上文的相同         |
| nReturned                   | 查询结果数量                             |
| executionTimeMillisEstimate | 预估耗时                                 |
| works                       | 工作单元数，一个查询会分解成小的工作单元 |
| advanced                    | 优先返回的结果数                         |
| docsExamined                | 文档检查数目，与totalDocsExamined一致    |

allPlansExecution：用来获取所有执行计划，结果参数基本与上文相同，这里就不再细说了。