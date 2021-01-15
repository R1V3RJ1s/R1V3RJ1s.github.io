---
title: "利用Matplotlib制作柱中内嵌标签的柱状图"
copyright: true
date: 2019-08-17 22:11:37
tags: 
  - Python
  - Data visualization
category:
  - Data Science
published: true
mathjax : true
---

现在有一组二元的统计数据（如男女闲暇时间喜欢做的事，显著性上下调基因集各自所富集的功能等等），现希望使用轴对称的水平柱状图进行可视化，要求柱子的颜色深浅需与数据相关，柱中内嵌标签沿柱底垂直对齐，水平居中。

<!-- more -->
#### 导入数据
{% codeblock lang:python %}
import math
import pandas as pd
import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns
from matplotlib.colors import ListedColormap

# 导入xlsx文件时如果报错可能需要安装openpyxl包
input_data_1 = pd.read_excel('foo.xlsx', sheet_name=0, engine='openpyxl')
input_data_2 = pd.read_excel('foo.xlsx', sheet_name=1, engine='openpyxl')
{% endcodeblock %}

{% codeblock lang:python %}
fig, ax = plt.subplots(figsize=(15, 10))
# 左图沿y轴翻转
ax.invert_xaxis()

ax = sns.barplot(x='标签下所含数据', y='标签', data=input_data_1,
                    palette=sns.dark_palette('#fbb9b5', n_colors=math.ceil(len(input_file_dn) * 2))[-len(input_data_1):][::-1])
# ax.patches包含图中每一个柱子的位置信息， 其中柱子的坐标(x, y)指示每个柱子左下的顶点坐标（垂直柱状图）或者是左上的顶点坐标（水平柱状图）
for name, p in zip(input_data_1['标签'], ax.patches):
    ax.text(x=0, y=p.xy[1] + (p.get_height() / 2), s=name,
            horizontalalignment='right', verticalalignment='center',
            fontsize=20, fontfamily='Arial')
{% endcodeblock %}

#### 隐藏坐标轴，多余标签及框线，保存
{% codeblock lang:python %}
# 隐藏x,y轴及其标签。如果只希望隐藏x,y轴，不隐藏其标签可采用注释内的命令
ax.get_xaxis().set_visible(False) # ax.set_yticks([])
ax.get_yaxis().set_visible(False) # ax.set_xticks([])
# 隐藏框线
plt.box(False)
# 保存为透明背景的矢量图
plt.savefig(f'1.svg', transparent=True)
{% endcodeblock %}

#### 同理制作右边的图
{% codeblock lang:python %}
fig, ax = plt.subplots(figsize=(15, 10))
ax = sns.barplot(x='Combined Score', y='Term', data=input_data_2,
                    palette=sns.dark_palette('#91fbfe', n_colors=math.ceil(len(input_data_2) * 2))[-len(input_data_2):][::-1])
for name, p in zip(input_data_2.Term, ax.patches):
    ax.text(0, p.xy[1] + (p.get_height() / 2), name, verticalalignment='center', fontsize=20, fontfamily='Arial')
ax.get_xaxis().set_visible(False)
ax.get_yaxis().set_visible(False)
ax.set_yticks([])
ax.set_xticks([])
plt.box(False)
plt.savefig(f'2.svg', transparent=True)
{% endcodeblock %}

#### 使用Adobe Illustrator进行拼接与微调
成品如下：
![柱状图](/images/barlot.jpg)
