---
title: "检测博客中的个人信息脚本和环境配置"
date: 2023-07-01T14:38:30+08:00
tags: [tool]
categories: [tool]
---

这篇文章介绍一个小小的检测脚本，用来检测写的博客有没有个人信息，以及如何在电脑上快速配置命令方便使用。

需要电脑上安装了python3，并且配置了环境变量

```python
# 文件名 check_personnal_info.py
import sys

def detect_personal_info(file_path):
    target_strings = ['root', '729']  # 替换为你要检测的个人信息字符串列表

    with open(file_path, 'r', encoding='utf-8') as file:
        lines = file.readlines()

        for i, line in enumerate(lines):
            for target_string in target_strings:
                if target_string.lower() in line.lower():
                    print(f"个人信息字符串 '{target_string}' 出现在第 {i+1} 行：\n{line}")

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print("请提供文件名作为参数")
    else:
        file_path = sys.argv[1]
        detect_personal_info(file_path)
```

然后在e盘新建一个文件夹 E:\PythonScript，里面用于存放bat文件和py脚本，将E:\PythonScript添加到环境变量里面（在系统高级设置里面，设置环境变量，将E:\PythonScript添加到path里面）
![](/imgs/check_path.png)

在PythonScript文件家里面新建check_personnal_info.py文件，将代码copy进去
在该文件夹里面新建check.bat文件,文件内容如下

```
@python.exe  E:\PythonScript\check_personnal_info.py %*
```

![](/imgs/check_test.png)
