---
title: "Spring Data Jpa中懒加载和OSIV"
date: 2023-12-05T17:41:55+08:00
draft: true
---

# 说明

本文只对OSIV和懒加载的特性做说明、总结，以及笔者在实际工作过程中遇到的问题和提出解决办法，还有思考等等。具体的代码示例参见我读过的几篇文章。

[A Guide to Spring's Open Session In View | Baeldung](https://www.baeldung.com/spring-open-session-in-view)

[Working with Lazy Element Collections in JPA | Baeldung](https://www.baeldung.com/java-jpa-lazy-collections)



# 总结

在实际工作过程中有个表A关联了将近二十张表，解雇导致有单个接口查询了将近2分钟。
