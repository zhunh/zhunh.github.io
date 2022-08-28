---
layout:     post
title:      TensorFlow常用函数
subtitle:   初识笔记
date:       2019-10-17
categories:	
- 机器学习
tags:
    - AI
    - TensorFlow
    - 深度学习
---

## 1.Session对象

- run(fetched, feed_dict=None, options=None, run_metadata=None)

  `fetches`: 表示数据流图中能接收的任意数据流图元素，各类Op/Tensor对象

  `feed_dict`:可选。给数据流图提供运行时数据。`feed_dict`的数据结构为python中的字典，其元素为各种键值对。

  `options`:
## 2. 常用 tf 函数

```python
result = tf.equal(tf.sign(p - 0.5), tf.sign(t - 0.5))
```

1. tf.equal 判断两个参数的值是否相等
2. tf.sign 用于取出正负符号
```python
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```
3. tf.cast 把布尔值转换为1或0。
4. tf.reduce_mean 是对向量(更多的是多维数组)的各个组成元素求平均值的函数。