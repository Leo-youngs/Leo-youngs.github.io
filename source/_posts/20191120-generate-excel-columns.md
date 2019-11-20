---
title: generate_excel_columns
date: 2019-11-20 10:00:03
categories: python
---

## 问题描述

在工作中需要有序生成excel列类似 A B C ... Z AA AB;
即传入一个数字生成对应的序列表示 例如: 26 -> Z, 27 -> AA;

## 解题思路

1. 题目类似进制的转换 27进制的数据

2. 不同的地方是不会 各个位数的取值范围可能不相同

3. 通过获取商与余数转换对应的值

## 代码展示

```python

def ColIdxToXlName(idx):
    if idx < 1:
        raise ValueError("Index is too small")
    result = ""
    while True:

        if idx > 26:
            idx, r = divmod(idx - 1, 26)
            print(idx, r)
            result = chr(r + ord('A')) + result
        else:
            return chr(idx + ord('A') - 1) + result
        print(result)


print(ColIdxToXlName(677))

```

## 思考

1. 开始写代码的时候天真的以为只是简单的进制转换，直到出问题才细致观察

2. 一定要多测试多观察找出规律

3. 开始实现是字典存储数字与字母的映射关系，巧用 ascii 码可能代码更加简单
