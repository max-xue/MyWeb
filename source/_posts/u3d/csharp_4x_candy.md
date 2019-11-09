---
title: .Net 4.x Equivalent语法糖
date: 2019-11-09 10:18:11
categories:
  - U3D
tags:
---

Unity2017的Scripting Runtime Version支持.net 4.6。
运行时的提升随之带来了新的语法糖。下边介绍主要常见的几种。

1 Null条件运算符
    
    // 原写法
    var str = "string";
    if (config != null)
    {
        str = config.Value;
    }
    
    // 新写法
    var str = config?.Value;
    if (str == null)
    {
        str = "string";
    }
    
    // 结合??写法
    var str = config?.Value ?? "string";

2 索引初始化
    
    // 原写法
    var dic = new Dictionary<string, int>()
    {
        {"a", 1},
        {"b", 1},
        {"c", 1},
    };
    
    // 新写法
    var dic = new Dictionary<string, int>()
    {
        ["a"] = 1,
        ["b"] = 2,
    };
    
    // 还可用于List。虽然大部分情况下都没有意义
    var list = new List<int>()
    {
        10,
        20,
        30,
        40
    };
    
    var list = new List<int>()
    {
        [0] = 10,
        [1] = 10,
        [2] = 10,
        [3] = 10,
    };
    
3 nameof表达式
    
    public class ClassName 
    { 
        public string StringValue;
    }
    
    var className = nameof(ClassName); // 返回ClassName
    var fieldName = nameof(ClassName.StringValue); // 返回StringValue

4 字符串格式化
    
    // 原写法
    var str = string.Format("{0} : {1}", value1, value2);
    
    // 新写法
    var str = $"{value1} : {value2}";
    
 
 [转载自:Unity2017新语法糖](https://blog.csdn.net/oyji1992/article/details/74188660)  