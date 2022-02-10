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

#### 对问题的转化和解决方案的讨论

最直观的思路就是将该病历数据转化为每一种病和其对应被诊断出该病的病人数目的数据然后作散点图，横坐标为特定疾病，纵坐标为的该病的人的数目。但有如下几个问题：
1. 如果选择可视化的是病和其对应被诊断出该病的病人数目的数据，那么如何科学选定阈值？
2. 由于这是一个整数型的离散数据，会有大量的数据点重叠（比如会有大量的病只有一两个人得），而为每一种病都单独留出一个横坐标的位置，又不太现实，如何处理？

在查阅了各种可视化方案中，最后敲定了一种结合蜂群图(beeswarm plot)和曼哈顿散点图(manhattan plot)的方案作为输出结果。

![Sample manhattan barplot](/images/python-manhattan.webp)

曼哈顿图（如上图）本质上是一个散点图，用于显示大量非零大范围波动数值，最早应用于全基因组关联分析(GWAS)研究展示高度相关位点。它得名源于样式与曼哈顿天际线相似（如下图）。一般在GWAS研究中所作的曼哈顿图的X轴为染色体编号，Y轴为相关统计显著性的p值，以10为底进行负对数变换。这样高显著性的基因就会在最上层。

![Manhattan skyline](/images/manhattan-skyline.png)

而蜂窝图（如下图）本质上也是一种散点图，它的优势在于可以无重叠地呈现所有数据信息。

![Sample beeswarm barplot](/images/ggbeeswarm-color.png)

选择这个方案的理由是现有疾病数据都被Phecode统一编码，且使用的编码系统分级比较合理（相对扁平，且每一类所包含的等级数量都相对一致），编码被分成的大类的类数也比较合适（10类）。这样就可以将Phecode的将疾病分成的大类编号作为X轴。但由于没有可以用来对照的数据（比如在整个医院或者社会中该病的发病情况），无法进行假设检验，所以最后还是敲定使用被诊断出该病的病人的绝对数量作为Y轴，并使用病人总数的65%（向下取整）作为作为标注的阈值。

#### 实现细节
{% codeblock lang:python %}
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from adjustText import adjust_text

input_data = pd.read_csv('./foo.csv', sep='|', dtype=str)

'''
In[1]: input_data.columns
Out[1]: Index(['phenotype', 'category_number', 'patient_number'], dtype='object')
'''
{% endcodeblock %}

但接下来遇到一个问题，由于重叠的点数量过多，使用`sns.swarmplot`指令无法完整显示所有的数据点，而这个绘图命令无法控制点自动换行，如何处理？
最后想到的办法是将所有的点进行一次限制上下限的正态变换。根据正态分布的概率密度分布函数可知，这个变换可以保证有较多的数据点落在原值上，且小部分的点会等概率地分布在原值上下不远的位置。同时这个更接近原值的值和远离原值的值的比例可以通过控制标准差的大小来控制。其实最理想的情况是当重叠的数据点少于一定数量的时候可以直接不进行变换，但因为懒就没写。

#### 将对应的病人数目进行正态变换
{% codeblock lang:python %}
# np.clip函数可以同时更改输入的最大值和最小值，这里我们选取原值的上下0.4作为上下限，这样画出来的效果比较好
input_data['scaled_ind'] = input_data['patient_number'].apply(lambda x: np.clip(np.random.normal(x, 0.01, 1), x-0.4, x+0.4)[0])
input_data = input_data.drop(columns=['patient_number'])
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