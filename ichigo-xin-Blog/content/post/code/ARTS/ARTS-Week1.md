---
title: "ARTS Week1"
date: 2023-08-16T23:35:48+08:00
categories: [ARTS]
---

## Algorithm

题目：[全排列](https://leetcode.cn/problems/permutations/)

最近工作中使用python搭建一个知识图谱服务，之前都是用python写一些脚本，而且都是参考其他模块的代码，感觉都是照葫芦画瓢，趁着这个项目好好学习下python，也打算在刷算法题也用python写，强行走出舒适区。



题目分析：全排列使用的是回溯算法，和深度优先遍历差不多。就是需要使用递归，在处理当前层进入下一层之前需要添加到状态变量里面，之后返回到当前层进入另一个选择时将刚才加入的部分进行反操作。


回溯算法有三个关键词：状态（路径）、选择列表、结束条件
在当前状态时，for循环选择列表，然后选择一个进入，这时候就需要把选择加入到状态里面，当第一条路走完，回到这个点的时候，就需要进行状态重置，然后选择另一个进入。就这样做完所有的遍历。
[参考文章](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/hui-su-sua-c26da/)

```python
lass Solution:
    def permute(self, nums: list[int]) -> list[list[int]]:
        res = []
        path = []
        if len(nums) == 0:
            return res
        self.dfs(nums, res, path, 0)
        return res

    def dfs(self, nums: list, res: list, path, level):
        if len(path) == len(nums):
            res.append(path.copy())

        for num in nums:
            if num in path:
                continue
            path.append(num)
            self.dfs(nums, res, path, level + 1)
            path.pop()
```

## Review

[python web框架Django和FastAPI的对比](https://hackernoon.com/python-web-frameworks-compared-django-vs-fastapi)
文章从性能、路由、数据库支持、模版、安全、社区支持、部署和开发这几个方面对这两个框架进行了对比


性能：FastAPI使用的是异步框架，性能更快，Django使用的是同步框架，对流程的控制更加容易些
路由：FastAPI是针对api设计的，写路由更加方便

数据库支持：Django内置了ORM映射框架

模版和前端集成：Django内置了一个模版操作系统，更加容易，FastAPI则是凸显了他在API集成方面的优势，与VUE和Angular集成方便

安全：在sql注入攻击和XSS攻击，两个框架都是做了保护的。Django有自己的权限验证系统

社区支持：两个社区都很活跃

部署：都有各自部署领域

开发：FastAPI开发更快

## Tips

py2neo中，如果使用cypher语句先match节点，然后再创建节点关系的方式的话，就会创建重复关系，直接使用Relationship就不会创建重复关系。尽量使用这种方式，如果使用cypher语句的话先要判断关系是否存在。

```python
            event = self.repository.graph.nodes.match(
                "Event", event_id=event_po.event_id
            ).first()
            record = self.repository.graph.nodes.match(
                "Record", record_id=event_po.record_id
            ).first()
            if record is None or event is None:
                continue
            rel = Relationship(record, "has_event", event)
            self.repository.graph.create(rel)
```

## Share

做一件事之前做一定的准备是有必要的。但是一直停留在想的阶段，只会造成内耗，最重要的还是行动起来。在做这件事的过程中自然而然会出现许多问题，一个一个解决就可以了。行动起来最重要！
