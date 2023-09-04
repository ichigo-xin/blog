---
title: "ARTS Week3"
date: 2023-08-29T18:04:17+08:00
categories: [ARTS]
---

## Algorithm

[旋转链表](https://leetcode.cn/problems/rotate-list/)

```python
class Solution:
    def rotateRight(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        if k == 0 or not head or not head.next:
            return head
        n = 1
        cur = head
        while cur.next:
            n += 1
            cur = cur.next

        if (add := n - k % n) == n:
            return head

        cur.next = head
        while add:
            cur = cur.next
            add -= 1

        res = cur.next
        cur.next = None
        return res
```

## Review

[pa中使用lombok可能会导致的问题](https://jpa-buddy.com/blog/lombok-and-jpa-what-may-go-wrong/)

 1.@Data或者是@EqualsAndHashCode注解，一般情况下主键id都是由数据库自动生成的，这时候即使其他属性值相同的对象的hash值也是不相同的

 2.@ToString,在有多对多的关系中，如果没有设置懒加载，直接打印的话，就会出现a引用b，b引用a，这样无线循环，栈溢出，最好使用exclude

 3.就是不要忘记添加无参构造函数

## Tips

1.py2neo中rel = self.repository.graph.match((record.__node__, handler.__node__,), r_type="be_handled").first()方法中，如果这个rel是没有任何属性值的，那结果就是None

2.使用jpa的findall方法报错数组越界，发现是实体类里面有个枚举字段，里面还没有定义任何枚举值。

## Share

最近看到一篇大数据杀熟文章的分析，同一个商品，商家和平台根据用户数据找出不同客户的最高承受价格，这就是大数据杀熟。从用户的角度来说，只有随着个人隐私的数据的保护才能避免被杀熟，现阶段还是无法避免的。
