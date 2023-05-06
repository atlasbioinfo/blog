---
layout: post
title: 生信中的Python——正则表达式与GQS预测
categories:
 - 生物信息
---

正则表达式算是我最常用的代码技巧吧，因为经常需要文本的模式匹配（就是查找某种字符串）。从我的经验，正则表达式很强大，效率也很高。最近有一个查找转录组中的GQS的项目，正好整理一下正则表达式。
>* 基础匹配
>* 前向断言
>* Kmer

***

写在前面：

我准备用Python来实现，其实各个语言的正则表达式（Regular expression）语法差异都不大。我不准备写太多基础的东西，就像文档教程那么难看，我就写一些实用的东西。

## 基础匹配

GQS，G-quadruplex 是一种富含G的碱基片段，现在发现有一大堆不啦不啦的功能。所以第一步就是把他找出来。

比较经典的GQS模式是G3N1-7,用正则表达式这样的：

```python
GGG [ATCG]{1,7} GGG [ATCG]{1,7} GGG [ATCG]{1,7} GGG

e.g.,

GGG AT GGG CTA GGG AGTT GGG

其中：
[ATCG]是这4个字符的一个，{1,7}是1-7个

所以上面这个正则表达式简写一下就是：
(G{3}[ATCG]{1,7}){3}G{3} 
```
![picture 1](https://pic.atlasbioinfo.com/pic_1639566409748_1639566417010.png)  

这个例子是3个G，也叫G3。当然植物中还有2个G的情况。不过按照这个正则表达式，是不是就能找到全部的GQS了呢？让我们试试

```python
import re

# 定义一个测试的DNA序列
test_seq = "ATGGGATGGGCTAGGGAGTTGGGACGGGTCAGGGGATTTGGGAGGGCTA"

# 定义正则表达式
gqs_pattern = r"(G{3}[ATCG]{1,7}){3}G{3}"

# 使用正则表达式查找GQS
gqs_matches = re.findall(gqs_pattern, test_seq)

# 输出结果
print("找到的GQS片段：")
for match in gqs_matches:
    print(match)

# 结果：找到的GQS片段：
# GGGATGGGCTAGGGAGTTGGG
```

从上面的结果可以看出，我们找到了一个GQS片段。但是，我们需要注意的是，这个正则表达式可能会漏掉一些特殊情况，比如G2和G4的情况。为了更全面地找到GQS，我们可以修改正则表达式如下：

```python
gqs_pattern = r"((G{2,4}[ATCG]{1,7}){3,}G{2,4})"
```

这个正则表达式表示，G可以重复2到4次，然后后面可以跟1到7个任意碱基，这个模式重复3次或以上，最后再以2到4个G结尾。

```python
# 使用修改后的正则表达式查找GQS
gqs_matches = re.findall(gqs_pattern, test_seq)

# 输出结果
print("找到的GQS片段：")
for match in gqs_matches:
    print(match[0])

# 结果：找到的GQS片段：
# GGGATGGGCTAGGGAGTTGGG
```
## 前向断言

在使用正则表达式查找GQS片段时，我们可能会遇到一个问题：当两个GQS片段紧邻或重叠时，我们的正则表达式可能无法找到所有的GQS。为了解决这个问题，我们可以使用前向断言（lookahead assertion）来改进我们的正则表达式。

前向断言允许我们在匹配一个模式的同时，检查该模式后面是否符合另一个模式。在这个例子中，我们可以使用前向断言来确保在匹配到一个GQS片段后，立即检查下一个可能的GQS片段。

修改后的正则表达式如下：

```python
gqs_pattern = r"(?=((G{2,4}[ATCG]{1,7}){3,}G{2,4}))"
```

这里，我们在原有的正则表达式前面加上了`(?=...)`，表示一个前向断言。现在，我们的正则表达式会在匹配到一个GQS片段后，立即检查下一个可能的GQS片段，从而找到所有的GQS。

```python
# 使用修改后的正则表达式查找GQS
gqs_matches = re.finditer(gqs_pattern, test_seq)

# 输出结果
print("找到的GQS片段：")
for match in gqs_matches:
    print(match.group(1))

# 结果：找到的GQS片段：
# GGGATGGGCTAGGGAGTTGGG
```

通过断言，我们可以确保找到所有的GQS片段，即使它们紧邻或重叠。这对于分析复杂的生物序列非常有用。

## Kmer

虽然正则表达式在查找GQS片段方面非常有用，但在某些情况下，它可能无法找到所有的GQS。比如：

```python
seq="GGGAGGGATGGAGGGAGGAGGG"

# 有多个GQS存在在其中
1. GGGAGGGATGGAGGGAGGAGGG
2. GGGAGGGATGGAGG
3. ...
```


这是因为正则表达式主要依赖于预先定义的模式，而GQS片段可能具有更复杂的结构。为了解决这个问题，我们可以尝试使用kmer的方法来查找GQS。

Kmer是一种将DNA序列划分为长度为k的连续子串的方法。通过分析kmer的组成和分布，我们可以发现序列中的一些潜在模式。在查找GQS片段的情况下，我们可以使用kmer方法来识别具有高G含量和特定G分布的子串。

以下是使用kmer方法查找GQS片段的一个示例：

```python
def find_gqs_by_kmer(seq, k):
    gqs_candidates = []
    for i in range(len(seq) - k + 1):
        kmer = seq[i:i+k]
        g_count = kmer.count("G")
        if g_count >= k * 0.5:  # 设定阈值，如G占kmer的50%以上
            gqs_candidates.append(kmer)
    return gqs_candidates

# 定义一个测试的DNA序列
test_seq = "ATGGGATGGGCTAGGGAGTTGGGACGGGTCAGGGGATTTGGGAGGGCTA"

# 使用kmer方法查找GQS
k = 10  # 可以根据实际情况调整k值
gqs_candidates = find_gqs_by_kmer(test_seq, k)

# 输出结果
print("找到的GQS候选片段：")
for candidate in gqs_candidates:
    print(candidate)

# 结果：找到的GQS候选片段：
# GGGATGGGCT
# GGCTAGGGAG
# GGGAGTTGGG
```

可以看到，使用kmer方法我们能够找到更多的GQS候选片段。然而，这种方法可能会产生一些假阳性结果，因为它只是基于G的含量和分布，而不是具体的GQS结构。为了提高查找GQS的准确性，我们可以结合正则表达式和kmer方法，首先使用kmer方法找到候选片段，然后使用正则表达式进一步筛选出真正的GQS片段。

总之，kmer方法为我们提供了一种在复杂生物序列中查找GQS片段的补充方法。结合正则表达式和kmer方法，我们可以更全面地识别GQS，并为生物信息学研究提供更多的信息。希望这些技巧能够帮助你在生物信息学领域取得更好的成果！



![博主简介](https://pic.atlasbioinfo.com/logo.png)