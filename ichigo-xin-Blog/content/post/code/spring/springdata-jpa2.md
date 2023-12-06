---
title: "Spring Data Jpa中的OSIV"
date: 2023-12-05T17:41:55+08:00
draft: false
tags: [Spring Data JPA]
categories: [code, spring]
image: /imgs/OIP.jpg
---

# 说明

本文只对OSIV做总结，以及笔者在实际工作过程中遇到的问题和提出解决办法，还有思考等等。具体的懒加载和OSIV的说明代码示例参见我读过的几篇文章。

[A Guide to Spring's Open Session In View | Baeldung](https://www.baeldung.com/spring-open-session-in-view)

[Working with Lazy Element Collections in JPA | Baeldung](https://www.baeldung.com/java-jpa-lazy-collections)

# 总结

在实际工作过程中有个表A关联了将近二十张表，导致有单个接口查询了将近2分钟。

我的处理办法是因为不好改数据库表结构了，这个接口的内部逻辑就直接写原生的sql来解决。

对于简单的系统而言，也就是crud，直接开启OSIV，加快开发进程，同时需要注意不要将单个表做过多的关联，有些可以自己通过定义数据库的字段来解决关联关系，没必要通过关联操作来解决。

对于复杂的系统来说，可以使用mybatisplus等其他技术。

主要是在项目初期做好评估和技术选型。
