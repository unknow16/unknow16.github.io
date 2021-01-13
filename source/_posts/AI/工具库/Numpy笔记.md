---
title: Numpy笔记
toc: true
date: 2021-01-12 17:24:28
tags:
categories:
---

官网：https://numpy.org/

axis取值从0开始
创建array时，若不指定，整数默认int64，小数默认float64

属性名字|	属性解释
-- | --
ndarray.shape|	数组维度的元组
ndarray.ndim|	数组维数
ndarray.size|	数组中的元素数量
ndarray.itemsize|	一个数组元素的长度（字节）
ndarray.dtype|	数组元素的类型

```
>>> a = np.array([[1, 2, 3],[4, 5, 6]], dtype=np.float32)
>>> a.dtype
dtype('float32') 
```



## 参考资料
> - []()
> - []()
