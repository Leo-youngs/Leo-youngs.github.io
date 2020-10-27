---
title: javascript map reduce 的一些用法
date: 2020-06-10 18:53:57
tags: 函数式编程
categories: 前端 
---

## 关于 es6一些函数的使用

1. reduce 的一些用法

    ```javascript
    // 1. 需要将数组中的对象转变成 {1: "apple", 2: "banana", 3: "orange"}
    const arr = [
        { id: 1,name: "apple" },
        { id: 2, name: "banana"},
        { id: 3, name: "orange"},
        ]

    arr.reduce((obj, item) => ({
        ...obj,
        [item.id]: item.name
    }), {})


    // 2. 把[1, 3, 5, 7, 9]变换成整数13579
    const arr = [1, 3, 5, 7, 9];

    arr.reduce((result, item)=> ( result=result*10 + item ))

    // 3. 一个字符串中每个字母出现的次数

    const str = "aabbcddeffffa"
    str.split("").reduce((result, item) => {result[item] = result[item] ? result[item] + 1 : 1; return result }, {})

    str.split("").reduce((result, item) => ({[item]: result[item]? result[item]++: 1,...result }),{})
    ```

2. map 的一些用法

    ```javascript
    // 1. 实现函数f(x)=x2的功能
    const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];

    arr.map( item => item* item )


    // 2. 把[1, 3, 5, 7, 9]变换成整数13579
    const arr = [1, 3, 5, 7, 9];

    arr.reduce((result, item)=> ( result=result*10 + item ))
    ```
