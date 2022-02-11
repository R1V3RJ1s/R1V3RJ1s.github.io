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


input_data = pd.read_csv('./foo.csv', sep='|', dtype=str)

'''
In[1]: input_data.columns
Out[1]: Index(['phenotype', 'category_number', 'patient_number'], dtype='object')
'''
{% endcodeblock %}

但接下来遇到一个问题，由于重叠的点数量过多，使用`sns.swarmplot`指令无法完整显示所有的数据点，而这个绘图命令无法控制点自动换行，如何处理？  
最后想到的办法是将所有的点进行一次限制上下限的正态变换。根据正态分布的概率密度分布函数可知，这个变换可以保证有较多的数据点落在原值上，且小部分的点会等概率地分布在原值上下不远的位置。同时这个更接近原值的值和远离原值的值的比例可以通过控制标准差的大小来控制。其实最理想的情况是当重叠的数据点少于一定数量的时候可以直接不进行变换，但因为懒就没写。

##### 将对应的病人数目进行正态变换

{% codeblock lang:python %}
# np.clip函数可以同时更改输入的最大值和最小值，这里我们选取原值的上下0.4作为上下限，这样画出来的效果比较好
input_data['scaled_ind'] = input_data['patient_number'].apply(lambda x: np.clip(np.random.normal(x, 0.01, 1), x-0.4, x+0.4)[0])
input_data = input_data.drop(columns=['patient_number'])
{% endcodeblock %}

##### 绘制曼哈顿图的主体并上色

本来绘图函数是可以自己选择上色的，但为了增加对比效果我这里选择了四种更容易分辨的颜色。因此需要对这10类数据点用这四种颜色进行循环上色。这里需要用到`sns.set_palette`和`sns.color_palette`两个函数。首先先创建一个包含所有想要绘制的颜色（以16进制RGB颜色编码表示）的列表，列表内颜色的顺序是单次循环上色的顺序。然后将其通过复制延长至要上色的数据类别的数量，并通过`sns.color_palette`将其转换成`seaborn`可以识别的色号，最后再通过`sns.set_palette`将接下来要绘图的色盘更改至输入的色号集合即可。

{% codeblock lang:python %}
color_item = ['#D55E00','#009E73','#0072B2', '#F0E442']
df_group_num = len(set(input_data.category_number))
# 通过将颜色列表内元素多次重复后再截取为要上色的类数的长度
# 最小的重复次数应为要上色的总类数除以单次上色循环所包含的颜色数所得的商加1
colors = list(color_item * (df_group_num // len(color_item) + 1))[0: df_group_num]
sns.set_palette(sns.color_palette(colors))

# 绘图
fig, ax = plt.subplots(figsize=(20, 14))
ax = sns.swarmplot(x="category_number", y="scaled_ind", data=input_data, s=7.5)
ax.set_xlabel('Phenotype Category ID', fontsize=20)
ax.set_ylabel('Phenotype occurrence number', fontsize=20)
ax.set_title('Phenotype occurrence number among all patients', fontsize=28)

plt.xticks(fontsize=16)
plt.yticks(fontsize=16)
{% endcodeblock %}

##### 绘制注释

接下来要给想要重点观察的疾病制作注释。基本的函数是`ax.annotate`，需要输入待注释的文字以及注释的位置，对于二维散点图自然就是一个形如`(pos_x, pos_y)`的数组。但是我们发现由于这里横坐标中的类别也是以数字形式表示的，因此传入类别编号无法让作图函数识别成正确的对应位置，因此此处需要传入注释点的绝对位置才行。另外如果待注释的散点较多且分布较密的话也很容易重叠，所以这里使用了一个叫做`adjustText`的包，它可以让注释位置分布更合理。

{% codeblock lang:python %}
from adjustText import adjust_text

# 创建一个字典tick_dict让每个编号都对应到它的X轴上的绝对位置
tick_key = input_data.category_number[~input_data.category_number.duplicated()].to_list()
tick_dict = dict(zip(tick_key, range(len(tick_key))))

# 创建一个用来注释的表，3.5是阈值，本质上阈值是3，此处阈值非整数是因为前面病人数经过了转换，3的范围会落在[2.6, 3.4]之间
anno_df = pd.concat((input_data.sort_values('scaled_ind', ascending=False)[0:len(input_data[input_data.scaled_ind > 3.5])],
                     input_data[input_data['phenotype'].str.contains('renal|Renal|Kidney|kidney')]))
anno_df = anno_df[~anno_df.duplicated()].reset_index(drop=True)

# 利用Series.apply(dict.get)方法可以根据所输入字典的键值对应关系填出一列新值
anno_df['category_pos'] = anno_df['category_number'].apply(tick_dict.get)

texts = []
for i in range(len(anno_df)):
    texts.append(ax.text(anno_df.loc[i]['category_pos'], anno_df.loc[i]['scaled_ind'], anno_df.loc[i]['phenotype'],
                             color="#4d4d4d", fontsize=18, fontname="Helvetica"))

adjust_text(
    texts,
    expand_points=(1, 1),
    arrowprops=dict(
        arrowstyle="->",
        color="#7F7F7F",
        lw=2
    ),
    ax=ax
)

plt.show()
{% endcodeblock %}

成品如下：
![Axial symmetrical barplot](/images/barplot.png)

#### 小结

本博文解决了如下问题：

1. Fus roh dah
2. 一个字典以表中某列的值为键，如何通过所给字典的键值对应关系，为该表填出一列新值？
3. 如何为散点图按类别循环上色？
4. 如何为散点图绘制注释

#### 参考链接

[Manhattan plot - Wikipedia](https://en.wikipedia.org/wiki/Manhattan_plot)