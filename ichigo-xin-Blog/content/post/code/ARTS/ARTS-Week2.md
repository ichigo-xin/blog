---
title: "ARTS Week2"
date: 2023-08-23T17:36:11+08:00
categories: [ARTS]
---

## Algorithm

这周继续回溯算法，题目是[n皇后](https://leetcode.cn/problems/n-queens/submissions/)

解析:直接用一个数组来表示放置位置，例如[1,3,0,2]表示第一行放1的位置，第二行放3的职位...，path表示路径，进入前先判断是否合法，dfs之前将当前节点加入path，dfs之后做反操作，状态重置。然后就是判断是否合法，同一列就是值在集合里面，斜着的话打个比方,目前path是[1.3],现在第三行加入2，|2-3|=|2的下标-3的下标|

```python
class Solution:
    def solveNQueens(self, n: int) -> list[list[str]]:
        path = []
        result = []
        self.dfs(0, n, path, result)
        return result

    def dfs(self, index, n, path, result):
        if index == n:
            result.append(self.trans(path, n))
        for i in range(n):
            if not self.is_valid(i, path):
                continue
            path.append(i)
            self.dfs(index + 1, n, path, result)
            path.pop()

    def is_valid(self, num, path):
        for i in range(len(path)):
            if num in path:
                return False
            if abs(path[i] - num) == abs(len(path) - i):
                return False
        return True

    def trans(self, path, n):
        """转换为输出结果"""
        temp = "." * n
        result = []
        for item in path:
            result.append(temp[:item] + "Q" + temp[item+1:])
        return result
```

## Review

[Spring Data JPA中@Joincolumn和@mappedBy的区别](https://www.baeldung.com/jpa-joincolumn-vs-mappedby)
结论：@Joincolumn在关系拥有端定义映射，然后这个关系的另一端使用mappedBy进行定义

发现这个网站是Java生态相关的，主要是spring和restful，后面可以多来逛逛。

## Tips
项目开发过程中可以开启feign的日志功能方便接口开发测试，配置文件中开启full级别

## Share

今天看了极客时间的学习进度，《左耳听风》专栏上的进度是67%。这个专栏我是断断续续学习的，应该是从前年开始看的，前年看了一些章节，去年也看了一些章节。感觉入这行时间不长的原因，好多章节看得懵懵懂懂，像编程范式这些章节就是这样。让我留下深刻印象的主要还是耗子叔的一些观点吧。

- 查看官网资料。我学习技术也是这样做的，好的博客，官网，极客时间专栏这些我都看，我后面也会去看官方文档。我自己也写博客，主要是记录和分享，但是我还是推荐大家去官网查阅资料，所以我在博客里面都会加上官网的链接。
- 学习几门语言。目前主要Java开发,工作中写过半年c++（菜鸡一个），python和go都会一点吧。
- 扎实基本功。算法方面在leetcode做了100多道题，每道题做几遍，隔几天做一次，通过提交次数将近500次。计算机网络方面还没开始，打算后续找本书看，极客时间上也有相关专栏，最近在看spring相关的资料，等这个模块结束后就开始学习计算机网络。
