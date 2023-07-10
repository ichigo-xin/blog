---
title: "Neo4j和知识图谱：Neo4j内置算法库的安装和使用"
date: 2023-07-03T20:52:02+08:00
tags: [neo4j, KG]
categories: [code]
image: /imgs/DM_20230703230529_001.png
---

在本篇文章中，我会对Neo4j Graph Data Science进行介绍，并且使用其中的算法做相似分析，这样可以让你更快的了解Neo4j Graph Data Science，并将其应用到实践中。

本文是全系列中的第2/3篇

- [Neo4j和知识图谱：Neo4j安装和使用]({{< relref "/post/code/neo4j/neo4j-study.md" >}})

- Neo4j和知识图谱：Neo4j内置算法库的安装和使用

- [Neo4j和知识图谱：使用Neo4j构建知识图谱]({{< relref "/post/code/neo4j/neo4j-study-kg.md" >}})

## Neo4j Graph Data Science介绍

The Neo4j Graph Data Science (GDS) library provides efficiently implemented, parallel versions of common graph algorithms, exposed as Cypher procedures. Additionally, GDS includes machine learning pipelines to train predictive supervised models to solve graph problems, such as predicting missing relationships.

这是[官网](https://neo4j.com/docs/graph-data-science/current/introduction/)的介绍，接下来我也将称Neo4j Graph Data Science 为GDS。GDS提供了图算法，同时也可以用于机器学习。

根据处理问题类型，算法分为以下几类：

- 中心性

- 社区

- 相似度

- 路径寻找

- 节点嵌入

- 拓扑链路预测

这篇文章后面会介绍相似分析的案例。

## GDS安装

下载GDS包，[下载地址](https://neo4j.com/deployment-center/#gds-tab)

![](/imgs/neo4j2_download.png)

```
将neo4j-graph-data-science-2.4.0.jar放入plugins目录下修改conf文件

vim conf/neo4j.conf

dbms.security.procedures.unrestricted=my.extensions.example,my.procedures.*,gds.*
dbms.security.procedures.allowlist=apoc.coll.*,apoc.load.*,gds.*

重启neo4j，bin/neo4j restart

在web控制台中输入RETURN gds.version();就可以查询是否安装成功
```

## 相似分析

相似分析主要是通过分析两个节点的共有相邻的节点来进行相似分析的。

对于两个集合A和B，Jaccard相似度计算为：

![jacard nodesim](https://neo4j.com/docs/graph-data-science/current/_images/nodesim-formulas/jacard_nodesim.svg)

重叠系数使用下面的公式:

![overlap nodesim](https://neo4j.com/docs/graph-data-science/current/_images/nodesim-formulas/overlap_nodesim.svg)

对于数据库中的两个节点n和m，A就是和n所有相邻的节点的集合，同样的B是m相邻节点的集合

Neo4j中的相似算法提供了许多有用的功能，适用于各种场景。以下是一些常见的使用场景：

1. 推荐系统：通过计算节点之间的相似性，可以构建基于内容或基于协同过滤的推荐系统。相似算法（如相似度计算或最短路径）可以帮助发现相似的用户、产品或兴趣，并向用户提供个性化的推荐。

2. 社交网络分析：相似算法可以帮助发现社交网络中的相似用户、群组或兴趣。例如，可以使用相似度算法（如Jaccard相似系数）来寻找在兴趣或活动上具有相似性的用户。

3. 产品推荐：相似算法可以帮助发现产品之间的相似性，从而实现商品推荐和交叉销售。通过计算产品之间的相似度，可以根据用户的购买历史或兴趣，向其推荐类似的产品。

4. 知识图谱分析：相似算法可以用于分析和比较知识图谱中的实体、关系或概念。通过计算实体之间的相似度，可以发现潜在的关联和模式，并进行语义推理。

## 相似分析实战

### 数据准备

```
CREATE
  (alice:Person {name: 'Alice'}),
  (bob:Person {name: 'Bob'}),
  (carol:Person {name: 'Carol'}),
  (dave:Person {name: 'Dave'}),
  (eve:Person {name: 'Eve'}),
  (guitar:Instrument {name: 'Guitar'}),
  (synth:Instrument {name: 'Synthesizer'}),
  (bongos:Instrument {name: 'Bongos'}),
  (trumpet:Instrument {name: 'Trumpet'}),
  (rice:Food1 {name: 'Rice'}),
  (pizza:Food1 {name: 'Pizza'}),
  (noodle:Food1 {name: 'Noodle'}),
  (fish:Food1 {name: 'Fish'}),
  (pig:Food1 {name: 'Pig'}),

  (alice)-[:LIKES]->(guitar),
  (alice)-[:LIKES]->(synth),
  (alice)-[:LIKES]->(bongos),
  (bob)-[:LIKES]->(guitar),
  (bob)-[:LIKES]->(synth),
  (carol)-[:LIKES]->(bongos),
  (dave)-[:LIKES]->(guitar),
  (dave)-[:LIKES {strength: 1.5}]->(trumpet),
  (dave)-[:LIKES]->(bongos);

MATCH (alice:Person {name: 'Alice'}), (rice:Food1 {name: 'Rice'})
CREATE (alice)-[:EAT]->(rice);

MATCH (alice:Person {name: 'Alice'}), (pizza:Food1 {name: 'Pizza'})
CREATE (alice)-[:EAT]->(pizza);

MATCH (alice:Person {name: 'Alice'}), (noodle:Food1 {name: 'Noodle'})
CREATE (alice)-[:EAT]->(noodle);


MATCH (bob:Person {name: 'Bob'}), (fish:Food1 {name: 'Fish'})
CREATE (bob)-[:EAT]->(fish);

MATCH (bob:Person {name: 'Bob'}), (pizza:Food1 {name: 'Pizza'})
CREATE (bob)-[:EAT]->(pizza);

MATCH (bob:Person {name: 'Bob'}), (noodle:Food1 {name: 'Noodle'})
CREATE (bob)-[:EAT]->(noodle);

MATCH (carol:Person {name: 'Carol'}), (fish:Food1 {name: 'Fish'})
CREATE (carol)-[:EAT]->(fish);

MATCH (dave:Person {name: 'Dave'}), (pizza:Food1 {name: 'Pizza'})
CREATE (dave)-[:EAT]->(pizza);
```

这样我们就建立了一个如下的关系图
![](/imgs/neo4j2_data.png)

### 建立投影

```
CALL gds.graph.project(
    'myGraph',
    ['Person', 'Instrument'],
    {
        LIKES: {
            properties: {
                strength: {
                    property: 'strength',
                    defaultValue: 1.0
                }
            }
        }
    }
);

CALL gds.graph.project(
    'myGraph1',
    ['Person', 'Instrument', 'Food1'],
    {
        EAT: {
            type: 'EAT',
            properties: {
                strength: {
                    property: 'strength',
                    defaultValue: 0.5
                }
            }
        },
        LIKES: {
            type: 'LIKES',
            properties: {
                strength: {
                    property: 'strength',
                    defaultValue: 1.0
                }
            }
        }
    }
);
```

这样我们就建立了两个投影，graph中包含了Person和Instrument节点，graph1中包含了Person、Instrument和Food1节点

### 计算相似度

1.计算所有节点的相似度

```
CALL gds.nodeSimilarity.stream('myGraph')
YIELD node1, node2, similarity
RETURN gds.util.asNode(node1).name AS Person1, gds.util.asNode(node2).name AS Person2, similarity
ORDER BY similarity DESCENDING, Person1, Person2
```

语法解析：

1. CALL gds.nodeSimilarity.stream('myGraph') 在myGraph投影中使用nodeSimilarity相似分析算法
2. YIELD node1, node2, similarity  使用节点node1、node2和相似度similarity
3. RETURN gds.util.asNode(node1).name AS Person1, gds.util.asNode(node2).name AS Person2, similarity 返回结果
4. ORDER BY similarity DESCENDING, Person1, Person2  排序方式

![](/imgs/neo4j2_similarity2.png)

2.计算myGraph投影中与Alice的相似度

```
CALL gds.nodeSimilarity.stream('myGraph')
YIELD node1, node2, similarity
WITH gds.util.asNode(node1) AS person1, gds.util.asNode(node2) AS person2, similarity
WHERE person1.name = 'Alice'
RETURN person1.name AS Person1, person2.name AS Person2, similarity
ORDER BY similarity DESCENDING
```

![](/imgs/neo4j2_similarity1.png)

3.计算myGraph1投影中与Alice的相似度

```
CALL gds.nodeSimilarity.stream('myGraph1')
YIELD node1, node2, similarity
WITH gds.util.asNode(node1) AS person1, gds.util.asNode(node2) AS person2, similarity
WHERE person1.name = 'Alice'
RETURN person1.name AS Person1, person2.name AS Person2, similarity
ORDER BY similarity DESCENDING
```

![](/imgs/neo4j2_similarity3.png)

4.添加关系之后再计算与Alice的相似度
需要删除投影之后再重新建立（投影在建立的时候就已经固定了，更新了关系之后需要更新投影）

```
  MATCH (dave:Person {name: 'Dave'}), (rice:Food1 {name: 'Rice'}), (pizza:Food1 {name: 'Pizza'}) ,(synthesizer:Instrument {name:"Synthesizer"})
  CREATE 
  (dave)-[:LIKES]->(synthesizer)
  (dave)-[:EAT]->(rice),
  (dave)-[:EAT]->(pizza);


CALL gds.graph.drop('myGraph1')
```

执行结果如下：
![](/imgs/neo4j2_similarity4.png)
现在是alice最相似的是bob了
