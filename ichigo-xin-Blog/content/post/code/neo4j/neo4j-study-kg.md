---
title: "Neo4j和知识图谱：使用Neo4j构建知识图谱"
date: 2023-07-07T23:49:21+08:00
tags: [neo4j, KG]
categories: [code]
image: /imgs/neo4j_kg1.png
---

在本篇文章中，我会对知识图谱进行介绍，并且会演示如何搭建一个知识图谱，这样你也可以跟着这篇博客搭建知识图谱。本文是全系列中的第3/3篇

- [Neo4j和知识图谱：Neo4j安装和使用]({{< relref "/post/code/neo4j/neo4j-study.md" >}})

- [Neo4j和知识图谱：Neo4j内置算法库的安装和使用]({{< relref "/post/code/neo4j/neo4j-study-algorithm.md" >}})

- Neo4j和知识图谱：使用Neo4j构建知识图谱

## 知识图谱介绍

知识图谱是一种语义化的知识表示形式，它可以将实体、属性和实体之间的关系以图谱的形式呈现出来。知识图谱可以用于自然语言理解、信息检索、智能推荐等领域，是人工智能技术的重要组成部分。

在知识图谱中，实体通常指现实世界中的事物，如人、地点、组织等，而属性则是实体的特征或属性，如人的年龄、地点的经纬度等。实体之间的关系可以分为不同类型，如属于、工作于、是朋友等。通过建立知识图谱，可以将这些实体和关系以结构化的方式进行表达，从而更好地理解和利用这些知识。

总之，知识图谱是一种强大的工具，可以帮助我们更好地理解和利用现实世界中的知识，从而实现更智能化的应用和服务。

## 知识图谱的构建流程

构建一个知识图谱需要经过以下步骤：

### 知识抽取

知识抽取是指从非结构化数据中提取出实体、属性和实体间的关系的过程。这一步通常需要使用自然语言处理技术，如实体识别、关系抽取等。

### 知识表示

知识表示是将抽取出的知识以一定的格式进行表示的过程。常用的表示方式有三元组和RDF等。

### 知识存储

知识存储是将知识表示存储到数据库中的过程。常用的数据库有图数据库、关系型数据库和文档数据库等。

### 知识推理

知识推理是指根据已有的知识推导出新的知识的过程。这一步通常需要使用逻辑推理、规则推理等技术。

### 知识应用

知识应用是指将知识图谱应用到具体的应用场景中的过程。如搜索引擎、智能问答、智能客服等。

知识图谱的搭建大致就是这几个流程，在一个知识图谱项目中，不一定每个步骤都有。比如我接下来的例子中，数据已经以结构化的形式存在数据库中了，不需要做知识抽取的过程，同时知识表示其实就是构建模型的过程。

## 搭建一个演员电影知识图谱

项目使用 python + py2neo + neo4j + mysql 进行搭建，项目代码

1.数据来源于[从零开始构建影视类知识图谱（一）半结构化数据的获取 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/138031298),在我的[github项目](https://github.com/ichigo-xin/ActorMovieKG)里面也有。

2.数据建模，这里就借用了结构化数据到RDF文件的概念，table对应class，一条记录对应一个实体，记录中的字段对应属性。简化一点，那我们在neo4j数据库中就只有两个类，电影和演员。

### mysql使用orm框架sqlalchemy

下面是mysql实体类

```python
"""实体类文件."""
from sqlalchemy import String, Text, Integer
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column


class Base(DeclarativeBase):
    pass


class Actor(Base):

    __tablename__ = 'actor'
    actor_id: Mapped[int] = mapped_column(primary_key=True)
    actor_bio: Mapped[str] = mapped_column(Text)
    actor_chName: Mapped[str] = mapped_column(String(100))
    actor_foreName: Mapped[str] = mapped_column(String(100))
    actor_nationality: Mapped[str] = mapped_column(String(100))
    actor_constellation: Mapped[str] = mapped_column(String(100))
    actor_birthPlace: Mapped[str] = mapped_column(String(100))
    actor_birthDay: Mapped[str] = mapped_column(String(100))
    actor_repWorks: Mapped[str] = mapped_column(String(100))
    actor_achiem: Mapped[str] = mapped_column(Text)
    actor_brokerage: Mapped[str] = mapped_column(String(100))


class Movie(Base):
    __tablename__ = 'movie'
    movie_id: Mapped[int] = mapped_column(primary_key=True)
    movie_bio: Mapped[str] = mapped_column(Text)
    movie_chName: Mapped[str] = mapped_column(String(100))
    movie_foreName: Mapped[str] = mapped_column(String(100))
    movie_prodTime: Mapped[str] = mapped_column(String(100))
    movie_prodCompany: Mapped[str] = mapped_column(String(100))
    movie_director: Mapped[str] = mapped_column(String(100))
    movie_screenwriter: Mapped[str] = mapped_column(String(100))
    movie_genre: Mapped[str] = mapped_column(String(100))
    movie_star: Mapped[str] = mapped_column(Text)
    movie_length: Mapped[str] = mapped_column(String(100))
    movie_rekeaseTime: Mapped[str] = mapped_column(String(100))
    movie_length: Mapped[str] = mapped_column(String(100))
    movie_achiem: Mapped[str] = mapped_column(Text)


class ActorToMovie(Base):
    __tablename__ = 'actor_movie_id'
    actor_movie_id: Mapped[int] = mapped_column(primary_key=True)
    actor_id: Mapped[int] = mapped_column(Integer)
    movie_id: Mapped[int] = mapped_column(Integer)


class Genre(Base):
    __tablename__ = 'genre'
    genre_id: Mapped[int] = mapped_column(primary_key=True)
    genre_name: Mapped[str] = mapped_column(String(100))


class MovieToGenre(Base):
    __tablename__ = 'movie_genre_id'
    movie_genre_id: Mapped[int] = mapped_column(primary_key=True)
    movie_id: Mapped[int] = mapped_column(primary_key=True)
    genre_id: Mapped[int] = mapped_column(primary_key=True)
```

### neo4j使用ogm框架py2neo

neo4j实体类：

```python
from py2neo.ogm import Model, Property, RelatedFrom, RelatedTo


class Movie(Model):
    __primarylable__ = 'Movie'

    movie_id = Property()
    movie_bio = Property()
    movie_chName = Property()
    movie_foreName = Property()
    movie_prodTime = Property()
    movie_prodCompany = Property()
    movie_director = Property()
    movie_screenwriter = Property()
    movie_genre = Property()
    movie_star = Property()
    movie_length = Property()
    movie_rekeaseTime = Property()
    movie_length = Property()
    movie_achiem = Property()

    actors = RelatedFrom("Actor", "ACTED_IN")


class Actor(Model):
    # 标签
    __primarylable__ = "Actor"

    # 属性
    actor_id = Property()
    actor_bio = Property()
    actor_chName = Property()
    actor_foreName = Property()
    actor_nationality = Property()
    actor_constellation = Property()
    actor_birthPlace = Property()
    actor_birthDay = Property()
    actor_repWorks = Property()
    actor_achiem = Property()
    actor_brokerage = Property()

    acted_in = RelatedTo(Movie)
```

### 图谱构建

 从mysql中查询数据并存到neo4j中：

```python
from py2neo import Relationship
from py2neo.ogm import Repository
from sqlalchemy import create_engine
from sqlalchemy import select
from sqlalchemy.orm import Session

from custom_model import neo4j_model, mysql_model


def covertor_actor(mysql_actor: mysql_model.Actor):
    neo4j_actor = neo4j_model.Actor()
    neo4j_actor.actor_id = mysql_actor.actor_id
    neo4j_actor.actor_bio = mysql_actor.actor_bio
    neo4j_actor.actor_chName = mysql_actor.actor_chName
    neo4j_actor.actor_foreName = mysql_actor.actor_foreName
    neo4j_actor.actor_nationality = mysql_actor.actor_nationality
    neo4j_actor.actor_constellation = mysql_actor.actor_constellation
    neo4j_actor.actor_birthPlace = mysql_actor.actor_birthPlace
    neo4j_actor.actor_birthDay = mysql_actor.actor_birthDay
    neo4j_actor.actor_repWorks = mysql_actor.actor_repWorks
    neo4j_actor.actor_achiem = mysql_actor.actor_achiem
    neo4j_actor.actor_brokerage = mysql_actor.actor_brokerage
    return neo4j_actor


def covertor_movie(actor_movie: mysql_model.Movie):
    neo4j_movie = neo4j_model.Movie()
    neo4j_movie.movie_id = actor_movie.movie_id
    neo4j_movie.movie_bio = actor_movie.movie_bio
    neo4j_movie.movie_chName = actor_movie.movie_chName
    neo4j_movie.movie_foreName = actor_movie.movie_foreName
    neo4j_movie.movie_prodTime = actor_movie.movie_prodTime
    neo4j_movie.movie_prodCompany = actor_movie.movie_prodCompany
    neo4j_movie.movie_director = actor_movie.movie_director
    neo4j_movie.movie_screenwriter = actor_movie.movie_screenwriter
    neo4j_movie.movie_genre = actor_movie.movie_genre
    neo4j_movie.movie_star = actor_movie.movie_star
    neo4j_movie.movie_length = actor_movie.movie_length
    neo4j_movie.movie_rekeaseTime = actor_movie.movie_rekeaseTime
    neo4j_movie.movie_length = actor_movie.movie_length
    neo4j_movie.movie_achiem = actor_movie.movie_achiem
    return neo4j_movie


class ActorMovieKG:

    def __init__(self):
        self.repo = Repository("bolt://neo4j@127.0.0.1:7687", password="123456")
        self.engine = create_engine('mysql+pymysql://root:123456@127.0.0.1:3306/kg')
        self.session = Session(self.engine)

    def build_graph(self):
        self.build_actor()
        self.build_movie()
        self.build_rel()

    def build_movie(self):
        stmt_movie = select(mysql_model.Movie)
        for actor_movie in self.session.scalars(stmt_movie):
            neo4j_movie = covertor_movie(actor_movie)
            self.repo.create(neo4j_movie)

    def build_actor(self):
        stmt_actor = select(mysql_model.Actor)
        for actor_mysql in self.session.scalars(stmt_actor):
            neo4j_actor = covertor_actor(actor_mysql)
            self.repo.create(neo4j_actor)

    def build_rel(self):
        stmt = select(mysql_model.ActorToMovie)
        for element in self.session.scalars(stmt):
            actor = self.repo.match(neo4j_model.Actor).where(id=element.actor_id).first()
            movie = self.repo.match(neo4j_model.Movie).where(id=element.movie_id).first()
            relation_ship = Relationship(actor, "ACTED_IN", movie)
            self.repo.create(relation_ship)


if __name__ == '__main__':
    kg = ActorMovieKG()
    kg.build_graph()
    print("建立知识图谱完成")
```

### 知识图谱使用

```python
from py2neo.ogm import Repository

from custom_model.neo4j_model import Actor


class KGClient:

    def __init__(self):
        self.repo = Repository("bolt://neo4j@127.0.0.1:7687", password="123456")

    def build_graph(self):
        cypher = """CALL gds.graph.project(
            'ActorMovieGraph',
            ['Actor', 'Movie'],
            'ACTED_IN'
        );
        """
        self.repo.graph.run(cypher)

    def query_similarity(self, name: str):
        """找出演员中的紧密程度"""
        cypher = f"""CALL gds.nodeSimilarity.stream('ActorMovieGraph')
        YIELD node1, node2, similarity 
        WITH gds.util.asNode(node1) AS actor1, gds.util.asNode(node2) AS actor2, similarity
        WHERE actor1.actor_chName = '{name}'
        RETURN actor1.actor_chName, actor2.actor_chName, similarity
        ORDER BY similarity DESCENDING
        """
        result = self.repo.graph.run(cypher).data()
        print(result)

    def query_all_movies(self, name: str):
        actor = self.repo.match(Actor).where(
            actor_chName=name
        ).first()
        if actor is None:
            raise ValueError('input error!')
        for movie in actor.acted_in:
            print(movie.movie_chName)


if __name__ == '__main__':
    kg_client = KGClient()
    # kg_client.build_graph()
    # kg_client.query_all_movies("张家辉")
    kg_client.query_similarity("鲍方")
```

### 结果展示
图谱展示：
![](/imgs/neo4j-3-r1.png)
查询和鲍方合作的明星，目前actor_to_movie表中的数据太少了，建立的联系不多，只查询出来2个
![](/imgs/neo4j-3-r2.png)

## 代码地址
[ActorMovieKG](https://github.com/ichigo-xin/ActorMovieKG)

## 参考资料

- [从零开始构建影视类知识图谱（一）半结构化数据的获取 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/138031298)

- [刘焕勇老师的github项目QABasedOnMedicaKnowledgeGraph](https://github.com/liuhuanyong/QASystemOnMedicalKG)
