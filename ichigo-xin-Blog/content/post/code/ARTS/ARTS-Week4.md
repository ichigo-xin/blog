---
title: "ARTS Week4"
date: 2023-09-14T19:04:17+08:00
categories: [ARTS]
---

## Algorithm

[125. 验证回文串](https://leetcode.cn/problems/valid-palindrome/)

用双指针解决，首末两个指针同时向中间逼近

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        s = ''.join(filter(str.isalnum, s.lower()))
        if len(s) <= 1:
            return True
        left = 0
        right = len(s) - 1
        while left < right:
            if s[left] != s[right]:
                return False
            left += 1
            right -= 1
        return True
```

看他人的题解中有两行的写法

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        s = ''.join(filter(str.isalnum, s.lower()))
        return s == s[::-1]
```

## Review

[Understanding Multiprocessing and Multithreading in Python | HackerNoon](https://hackernoon.com/understanding-multiprocessing-and-multithreading-in-python)

这篇文章介绍了python里面的多线程和多进程。

python里的多线程也是有共享内存的，但是和Java不同的是由于GIL的存在，其实python里的多线程是加了全局锁的，只会在一个cpu上运行，其实不是真正的并行执行。

python里的多进程，就是操作系统多个进程同时执行。

## Tips

记录一下最近用到的git操作

1.rebase的时候解决冲突，把自己的代码给删除了。在idea的控制台上看提交历史也没有需要的代码。

解决方法：

- git reflog.找到rebase之前的提交的hash值，确定丢失的代码在哪个提交里

- 使用git cherry-pick （hash值）,引入提交

- 解决冲突

## Share

[第164期 | 你工作开心吗？ (geekbang.org)](https://time.geekbang.org/column/article/179021)
这是以前看到的一篇文章，讲的职业倦怠。

总结来说，职业倦怠是不可避免，但是我们采取措施措施，做出改变。之前也看过一篇文章说，职业倦怠也是周期性的，就像脉冲一下，过一阵子又有了。

在我看来做到工作和生活平衡，然后给自己充电，有时候做些稍微有些挑战性的工作，就可以减少个人倦怠的情绪。
