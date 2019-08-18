---
title: "Python编程中的小技巧以及踩过的坑"
copyright: true
date: 2019-08-17 22:11:37
tags: 
  - Python
category:
  - Coding
published: true
mathjax : true
---

本博文专门用来记录在学习和使用Python编程时遇到的一些与问题，解决方案和总结的技巧。

<!-- more -->

#### 创建一个默认值为随机数矩阵的defaultdict
{% codeblock lang:python %}
import collections
import numpy as np


# 创建一个默认值为m * n的在[j, k)上均匀分布的随机浮点数矩阵的defaultdict
random_matrix_dictionary = collections.defaultdict(lambda: np.random.uniform(low=j, high=k, size = (m, n)))
# 如果只想创建一个默认值为m * n的在[0, k)上均匀分布的随机浮点数矩阵的defaultdict, 则可以只用np.random.rand函数:
random_matrix_dictionary = collections.defaultdict(lambda: np.random.rand(m, n) * k)
# 如果要创建一个默认值为m * n的在[j, k)上均匀分布的随机整数矩阵的defaultdict
random_integer_matrix_dictionary = collections.defaultdict(lambda: np.random.randint(low=j, high=k, size = (m, n)))
{% endcodeblock %}

#### 创建一个默认值为与键相关的函数的defaultdict
{% codeblock lang:python %}
import collections


class KeyDefaultdict(collections.defaultdict):
    def __init__(self, key_func):
        super().__init__(None)
        self.key_func = key_func

    def __missing__(self, key):
        val = self.key_func(key)
        self[key] = val
        return val
{% endcodeblock %}