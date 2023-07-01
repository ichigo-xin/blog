---
title: "Neo4j和知识图谱：Neo4j安装和使用"
date: 2023-06-28T21:00:04+08:00
tags: [neo4j, KG]
categories: [code]
---

在本篇文章中，我会对Neo4j图数据库、查询语句Cypher进行介绍，这样可以让你更容易掌握这门技术。其中主要包括Neo4j的安装配置，查询语句Cypher使用。

本文是全系列中的第1/4篇

- Neo4j和知识图谱：Neo4j安装和使用

- Neo4j和知识图谱：Neo4j内置算法库的安装和使用

- Neo4j和知识图谱：使用Neo4j构建知识图谱

- Neo4j和知识图谱：项目中使用py2neo的ogm框架构建知识图谱

## Neo4j介绍

Neo4j是一种广泛使用的图形数据库管理系统，专注于存储和处理具有复杂关系的数据。它使用图形模型来表示数据，其中节点表示实体，边表示实体之间的关系。Neo4j提供了一种高效灵活的方式来处理和查询这些图形数据。

Neo4j的使用场景非常丰富，适用于多个领域和行业。以下是一些常见的使用场景：

1. 社交网络分析：Neo4j可以存储和分析社交网络中的用户关系、兴趣和活动。它能够高效地执行复杂的关系查询，例如查找朋友的朋友、查找共同兴趣的人等。
2. 推荐系统：Neo4j可以帮助构建个性化的推荐系统。通过建模用户、产品和其它相关属性之间的关系，Neo4j能够快速查询和发现潜在的兴趣和推荐。
3. 知识图谱和语义网络：Neo4j能够存储和查询知识图谱，这是一种表示实体之间关系和属性的结构化数据模型。知识图谱广泛应用于搜索引擎、智能助手和推荐系统等领域。

这些场景用关系型数据库实现非常困难，这就是图数据库发挥作用的地方。

## Neo4j安装

Neo4j有社区版和企业版。这里我们选择社区版，同时Neo4j使用Java进行开发的，因此和JDK版本有着对应关系，4.x的版本选择jdk11，因为后续会安装算法库，再加上项目开发的需求就选择了neo4j-community-4.4.22-unix的版本。

| Neo4j version    | Neo4j Graph Data Science |
| ---------------- | ------------------------ |
| `5.9`            | `2.4`, `2.3.9` or later  |
| `5.8`            | `2.4`, `2.3.6` or later  |
| `5.7`            | `2.4`, `2.3.3` or later  |
| `5.6`            | `2.4`, `2.3.2` or later  |
| `5.5`            | `2.4`, `2.3.1` or later  |
| `5.4`            | `2.4`, `2.3`             |
| `5.3`            | `2.4`, `2.3`             |
| `5.2`            | `2.4`, `2.3`             |
| `5.1`            | `2.4`, `2.3`             |
| `4.4.9` or later | `2.4`, `2.3`             |

[具体版本要求官网地址](https://neo4j.com/docs/graph-data-science/current/installation/supported-neo4j-versions/)

[下载地址](https://neo4j.com/download-center/#community)

```
安装步骤
安装前需要安装jdk11
1.解压安装包
tar -xf neo4j-community-4.4.22-unix.tar.gz

2.配置neo4j
cd neo4j-community-4.4.22/
vim conf/neo4j.conf
dbms.memory.heap.initial_size=4g #根据服务器的内存进行修改
dbms.memory.heap.max_size=4g  #根据服务器的内存进行修改
dbms.default_listen_address=0.0.0.0 #设置可以远程访问
dbms.connector.http.listen_address=:7474

3.启动
bin/neo4j start
看下服务有没有启动 ps -ef | grep neo

4.打开浏览器，并访问 http://ip:7474/ ，这将打开Neo4j的Web控制台
用户名和密码 neo4j/neo4j
```

![neo4j web控制台](/imgs/neo4j—web控制台.png "web显示")

## Cypher的使用

用cypher查询neo4j就相当于用sql查询mysql

### 创建节点

创建单个节点

```
create (n:Person {name:"孙悟空"})
```

创建多个节点

```
CREATE (n:Person {name: "唐三藏"}), (m:Person {name: "猪八戒", weapon: "九齿钉耙"})
```

如果没有定义model的话，对于一个节点是可以随意配置属性的，上面的猪八戒中就多了个武器的属性

### 创建关系

语法:  ()代表实体，-[]->代表关系

```
create ()-[]->()
```

创建已有节点的关系

```
MATCH (n:Person )
match (n:Person{name:"孙悟空"}), (m:Person{name:"猪八戒"})
create (n)-[r:shidi{name:"师弟"}]->(m)
```

![](/imgs/neo4j-rel.png)

创建节点和关系

![](/imgs/neo4j-rel2.png)

### 删除节点

```
match (n:Person {name:'唐三藏'})
delete n
```

### 删除关系

```
match (n:Person {name:"李靖"})-[r:child]->(m:Person {name:'哪吒'})
delete r
```

### 修改属性值

```
match (n:Person {name:"孙悟空"})
set n.weapon = "如意金箍棒"
```

### 查询

就是match，然后接个return

```
match (n:Person {name:"孙悟空"}) return n

使用where进行过滤
match (n:Person )  where n.name = "猪八戒" return n

使用limit进行限制
match (n:Person )  return n limit 2
```
