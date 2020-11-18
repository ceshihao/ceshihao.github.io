---
title: Python的hash函数每次执行结果不一样的问题
date: 2020-11-18 14:41:50
tags: [python, hash]
---

## 问题描述

近期遇到一个问题Python的 hash() 函数每次得到的哈希值不一样。例如

{% asset_img "hash-inconsistency.png" "Hash Inconsistency" %}

这会造成应用内部的一个根据 uid 哈希值取模的地方有问题，每次得到的结果不一样。为此我稍微深入的研究了一下这个 hash() 函数。

## 调查过程

首先查看了一下Python官方文档。hash() 函数的文档并没有提到为什么每次结果不一样。

https://docs.python.org/3.6/library/functions.html#hash

{% asset_img "python-doc-hash.png" "Python Doc hash" %}

继续看一下内部实现的 __hash__() 函数。其中有一段提到 __hash__() 函数在处理 str，bytes 和datetime类型的对象时，会对其加盐，这个值会是一个不可预测的随机值 (an unpredictable random value)。这个值在同一个进程中是一致的，但在不同的进程之间是随机的。并且这个值可以通过 PYTHONHASHSEED 变量来设定。这个实现是从 Python 3.2.3 开始引入的。

https://docs.python.org/3.6/reference/datamodel.html#object.__hash__

{% asset_img "python-doc-_hash_.png" "Python Doc _hash_" %}

## 验证实验

根据上述的调查结果我们来进行实验验证一下。

### 同一进程下的hash

在同一个Python进程下，对同一个字符串 "123" 每次得到的哈希值是一样的。

{% asset_img "hash_same_process.png" "Hash Same Process" %}

### 不同进程下的hash

和开篇描述的问题类似，不同的进程下对同一个字符串 "123" 每次得到的哈希值是不一样的。

{% asset_img "hash_different_process.png" "Hash Different Process" %}

### 设定PYTHONHASHSEED后，不同进程下的hash

设定 PYTHONHASHSEED 为一个固定值1后，不同的进程下对同一个字符串 "123" 每次得到的哈希值是一样的。

{% asset_img "hash_same_seed.png" "Hash Same PYTHONHASHSEED" %}

## 解决方法

既然Python的 hash() 函数无法保证不同进程间每次计算的哈希值一致，那我们如果想在不同的进程间得到一致的哈希结果要如何做呢？

答案是用 hashlib。hashlib 的哈希结果可以做到可重现可跨进程的一致性。

https://docs.python.org/3.6/library/hashlib.html

针对上面不同进程下的情况，我们用 hashlib 重复做一次实验。从结果我们可以看到，不同进程中每次的哈希结果是一致的 (只不过返回的类型不像 hash() 函数是int)。

{% asset_img "python-hashlib.png" "Python hashlib" %}

## 总结

* 在同一个进程内做简单的哈希比较是可以使用 hash() 函数的，而且哈希的结果是一致的。
* 在不同进程间如果哈希结果只用与散列，而不是结果比较时 hash() 函数也是可以使用的。
* 如果用于不同进程间的哈希值比较，不应该使用 hash() 函数，而应该使用hashlib。
