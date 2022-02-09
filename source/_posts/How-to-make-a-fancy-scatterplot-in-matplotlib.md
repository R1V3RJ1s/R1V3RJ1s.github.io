---
title: "利用Matplotlib制作曼哈顿散点图"
copyright: true
date: 2022-02-04 00:40:37
tags: 
  - Python
  - Data visualization
category:
  - Data Science
published: true
mathjax : true
---

现在有一组在一段时间内医院某科室中的病人病历数据，现希望对该数据进行可视化，以观察来该科室看病的病人是否更多（或更少）地被确诊出某些特定的疾病。

<!-- more -->

最直观的思路就是将该病历数据转化为每一种病和其对应被诊断出该病的病人数目的数据然后作散点图，横坐标为特定疾病，纵坐标为的该病的人的数目。但有如下几个问题：
1. 如果选择可视化的是病和其对应被诊断出该病的病人数目的数据，那么如何科学选定阈值？
2. 由于这是一个整数型的离散数据，会有大量的数据点重叠（比如会有大量的病只有一两个人得），而为每一种病都单独留出一个横坐标的位置，又不太现实，如何处理？

在查阅了各种可视化方案中，最后敲定了一种结合蜂群图(beeswarm plot)和曼哈顿散点图(manhattan plot)的方案作为输出结果。

![Sample manhattan barplot](/images/python-manhattan.webp)

曼哈顿图（如上图）本质上是一个散点图，用于显示大量非零大范围波动数值，最早应用于全基因组关联分析(GWAS)研究展示高度相关位点。它得名源于样式与曼哈顿天际线相似（如下图）。

![Manhattan skyline](/images/manhattan-skyline.png)

蜂窝图（如下图）本质上也是一种散点图，它的优势在于可以无重叠地呈现所有数据信息。

![Sample beeswarm barplot](/images/ggbeeswarm-color.png)

#### 导入数据
{% codeblock lang:python %}
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from adjustText import adjust_text

# 导入xlsx文件时如果报错可能需要安装openpyxl包
input_data_1 = pd.read_excel('foo.xlsx', sheet_name=0, engine='openpyxl')
input_data_2 = pd.read_excel('foo.xlsx', sheet_name=1, engine='openpyxl')
{% endcodeblock %}

#### 制作沿y轴翻转的水平柱状图
{% codeblock lang:python %}
fig, ax = plt.subplots(figsize=(15, 10))
# 沿y轴翻转
ax.invert_xaxis()
{% endcodeblock %}

首先先通过`sns.barplot`方法把一般的水平柱状图做出来并且上色。`Seaborn` 提供两种比较容易控制的线性色盘，`sns.light_palette`和`sns.dark_palette`。分别表示色盘内颜色由纯白(light_palette)或纯黑(dark_palette)渐变至用户传递的颜色参数（或者可以通过`reverse=True`参数把色盘反过来），颜色的切片数量则由用户传递的`n_colors`参数决定。该色盘还可以以matplotlib接受的colormap数据结构作为输出。此处我将色盘切片数量定为标签数量的两倍是为了平衡渐变的区分度和文字的能见度。

{% codeblock lang:python %}
ax = sns.barplot(x='Number', y='Label', data=input_data_1,
                    palette=sns.dark_palette('#fbb9b5', reverse=True, n_colors=math.ceil(len(input_data_1) * 2))[:len(input_data_1)])
{% endcodeblock %}

接下来给每个柱子内部添加标签。`ax.patches`包含图中每一个柱子的位置信息， 其中柱子的坐标(x, y)指示每个柱子左下的顶点坐标（垂直柱状图）或者是左上的顶点坐标（水平柱状图）。此处由于垂直翻转的缘故所以此时(x, y)指示的是右上的顶点坐标。`ax.text`方法主要控制标签的坐标和内容。`horizontalalignment(ha)`和`verticalalignment(va)`两个参数负责控制我们的标签的哪个位置要对齐我们传递的坐标。比如此处`ha='right', va='center'`说明标签的右端中点应与我们传递的坐标一致。`x=0`说明标签靠y轴（柱底）对齐，`y=p.xy[1] + (p.get_height() / 2)`说明标签中线在与每个水平柱子的上端`(p.xy[1])`对齐后向下移了半个柱子的宽度`(p.get_height() / 2)`，此时标签中线与柱子中线恰好对齐，即水平居中。注意柱子的粗细可以通过`ax.patches.get_height()`方法得到，柱子的长短则可以通过`ax.patches.get_width()`方法得到（如果是垂直柱状图则反之）。另外，此处之所以是加半个柱子的宽度而不是减是因为对于水平柱状图而言，y轴与直角坐标系的y轴是反过来的。

{% codeblock lang:python %}
for name, p in zip(input_data_1['Label'], ax.patches):
    ax.text(x=0, y=p.xy[1] + (p.get_height() / 2), s=name,
            ha='right', va='center',
            fontsize=20, fontfamily='Arial')
{% endcodeblock %}

隐藏坐标轴，多余标签及框线，保存
{% codeblock lang:python %}
# 隐藏x,y轴及其标签。如果只希望隐藏x,y轴，不隐藏其标签，可采用注释内的命令
ax.get_xaxis().set_visible(False) # ax.set_yticks([])
ax.get_yaxis().set_visible(False) # ax.set_xticks([])
# 隐藏框线
plt.box(False)
# 保存为透明背景的矢量图
plt.savefig(f'1.svg', transparent=True)
{% endcodeblock %}

#### 同理制作不翻转的水平柱状图
{% codeblock lang:python %}
fig, ax = plt.subplots(figsize=(15, 10))
ax = sns.barplot(x='Number', y='Label', data=input_data_2,
                    palette=sns.dark_palette('#91fbfe', reverse=True, n_colors=math.ceil(len(input_data_2) * 2))[:len(input_data_2)])
for name, p in zip(input_data_2['Label'], ax.patches):
    ax.text(0, p.xy[1] + (p.get_height() / 2), name, verticalalignment='center', fontsize=20, fontfamily='Arial')
ax.get_xaxis().set_visible(False)
ax.get_yaxis().set_visible(False)
plt.box(False)
plt.savefig(f'2.svg', transparent=True)
{% endcodeblock %}

#### 使用Adobe Illustrator进行拼接与微调
成品如下：
![Axial symmetrical barplot](/images/barplot.png)

#### 参考链接
[Manhattan plot - Wikipedia](https://en.wikipedia.org/wiki/Manhattan_plot)