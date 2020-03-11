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

#### 创建一个默认值为随机数矩阵的`defaultdict`
{% codeblock lang:python %}
import numpy as np
from collections import defaultdict

# 创建一个默认值为m * n的在[j, k)上均匀分布的随机浮点数矩阵的defaultdict
random_matrix_dictionary = defaultdict(lambda: np.random.uniform(low=j, high=k, size = (m, n)))

# 如果只想创建一个默认值为m * n的在[0, k)上均匀分布的随机浮点数矩阵的defaultdict, 则可以只用np.random.rand函数:
random_matrix_dictionary = defaultdict(lambda: np.random.rand(m, n) * k)

# 如果要创建一个默认值为m * n的在[j, k)上均匀分布的随机整数矩阵的defaultdict
random_integer_matrix_dictionary = defaultdict(lambda: np.random.randint(low=j, high=k, size = (m, n)))
{% endcodeblock %}

此方法一个比较反直觉的地方在于需要创建一个自变量为空的匿名函数，我觉得从两个方面来理解会显得更容易一些。从思路上看，此处等同于创建一个默认值为与键无关的函数的`defualtdict`，并且此时其他所有的变量都已经作为默认参数（named argument）被赋值，自然不需要自变量；从函数本身角度理解，此处匿名函数等同于：
{% codeblock lang:python %}
def random_matrix_dictionary_function_builder(j=j, k=k, m=m, n=n):
    return np.random.uniform(low=j, high=k, size = (m, n))
{% endcodeblock %}
可见自变量确实为空。从此处我们可以推导出Python3内置的`collections`包里的`defaultdict`类可以支持创建一切默认值为与键不相关的函数的`defaultdict`。但如果想要创建一个默认值为与键相关的函数的`defaultdict`呢？那就需要通过类继承来做一些微小的改进了。

#### 创建一个默认值为与键相关的函数的`defaultdict`

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

#### 通过迭代默认参数来创建函数时的问题

我是在创建默认值为与键相关的函数的`defaultdict`时遇到这个问题的。我希望通过读取文件内不同的参数来创建值生成函数不同的字典，但我发现迭代文件内容后生成的字典值生成函数全部都一样且只和文件中最后的参数有关。而如果在创建某字典后立即调用该字典创建测试键值对进行检测时会发现此时返回的键值对结果是正常的，不过生成完再用其他值创建测试键值对进行检测时会发现此时字典值生成函数又变回一样了。经过搜索之后，我发现这个问题在一个叫[wtfpython](https://github.com/satwikkansal/wtfpython#-the-sticky-output-function)的GitHub repository里讲的比较好，而且举的例子比较简单和清晰，所以我会用该repository里的例子来讲解。另外提一句，这个repository专门讲Python编程中可能会踩到的坑还有各种小彩蛋，发布至今已经获得15k的star了，读完会对Python有更深的了解。

{% codeblock lang:python %}
funcs = []
results = []
for x in range(7):
    def some_func():
        return x
    results.append(some_func())  # 注意这里函数被调用了
    funcs.append(some_func)
    
funcs_results = [func() for func in funcs]
{% endcodeblock %}

输出：
{% codeblock lang:python %}
>>> results
[0, 1, 2, 3, 4, 5, 6] # 生成完立即调用的话，就会看上去一切正常
>>> funcs_results
[6, 6, 6, 6, 6, 6, 6] # 如果之后统一调用就只和变量最终的值有关
{% endcodeblock %}

出现此问题的原因在于我们在全局环境中创建函数的同时也赋予了该函数一个与全局变量相绑定的闭包（closure，可以理解为函数的执行环境）。在Python中，函数闭包绑定的是变量本身而非变量具体的值，所以所有在迭代中创建的函数都会以该全局变量最终的值来作为默认参数。这个问题可以通过两个方法来解决。
1. 我们可以将该变量显式地作为默认参数传递给函数来获得预期的结果。因为这会在函数内再次定义一个局部变量。
{% codeblock lang:python %}
funcs = []
for x in range(7):
    def some_func(x=x):
        return x
    funcs.append(some_func)
{% endcodeblock %}
输出：
{% codeblock lang:python %}
>>> funcs_results = [func() for func in funcs]
>>> funcs_results
[0, 1, 2, 3, 4, 5, 6]
{% endcodeblock %}

2. 我们可以通过创建一个工厂函数（函数嵌套）来来获得预期的结果。这么做可行的原因是每次运行这个工厂函数我们都能给要创建的函数创建一个独立的执行环境来给它绑定。这是因为根据Python的作用域查询规则（LEGB rule, Local scope -> Enclosed scope -> Global scope -> Built-in scope），当我们要以非赋值的方式去引用一个变量时，如果此时局部作用域(Local scope)的任何位置无对某变量的赋值操作（无论此赋值操作是在引用变量之前还是在之后），那么Python会默认此变量为全局变量，从而会去上一级作用域查询，直至查询到该变量或者抛出`NameError`异常。此例中`some_func`函数在工厂函数`some_factory_func`的作用域中即能查询到变量x，所以不再需要向上查询，因此此函数闭包所绑定的变量为此封闭作用域（Enclosed scope）内的变量而非全局变量。
{% codeblock lang:python %}
funcs = []
for x in range(7):
    def some_factory_func(x):
        def some_func():
            return x
        return some_func
    funcs.append(some_factory_func(x))
{% endcodeblock %}
输出：
{% codeblock lang:python %}
>>> funcs_results = [func() for func in funcs]
>>> funcs_results
[0, 1, 2, 3, 4, 5, 6]
{% endcodeblock %}
为了检验我们的函数所绑定的变量是哪个变量，我们通过`global`关键词来让Python直接引用作为全局变量的x，假设之前的理论成立，那么此时的输出列表应该全是最后一次迭代的x值：
{% codeblock lang:python %}
funcs = []
for x in range(7):
    def some_factory_func(x):
        def some_func():
            global x # 此处让Python跳过查询封闭作用域，直接通过全局作用域来引用x
            return x
        return some_func
    funcs.append(some_factory_func(x))
{% endcodeblock %}
输出：
{% codeblock lang:python %}
>>> funcs_results = [func() for func in funcs]
>>> funcs_results
[6, 6, 6, 6, 6, 6] # 假设成立
{% endcodeblock %}
可见之前我们通过工厂函数传递给生成函数的变量x的确绑定的是封闭作用域内的x，且彼此相互独立。

本条其他参考链接：
[A Beginner's Guide to Python's Namespaces, Scope Resolution, and the LEGB Rule](https://sebastianraschka.com/Articles/2014_python_scope_and_namespaces.html)
[Gotcha: Python, scoping, and closures](https://eev.ee/blog/2011/04/24/gotcha-python-scoping-closures/)

#### UnboundLocalError

这条其实在wtfpython里也有，叫做[The out of scope variable](https://github.com/satwikkansal/wtfpython#-the-out-of-scope-variable)。简单描述即是当你定义一个全局的变量，且在函数里对该变量有修改操作，那么如果这个修改操作没有执行（比如在判断语句之内）或者是在对该变量的引用在修改操作之前，那么Python就会抛出`UnboundLocalError`异常。这个问题的隐蔽版本是当你使用形如a += 1的赋值运算符时，如果全局环境中有对a的定义且函数中无对a的定义，那么Python也会抛出`UnboundLocalError`异常。

{% codeblock lang:python %}
a = 1
def first_func():
    return a

def second_func():
    a += 1
    return a

def third_func():
    if False:
        a = 2
    return a

def fourth_func():
    print(a)
    a = 2

def fifth_func():
    a = 3
    def sixth_func():
        a += 1
        return a
    return sixth_func()
{% endcodeblock %}

输出：
{% codeblock lang:python %}
>>> first_func()
1
>>> second_func()
UnboundLocalError: local variable 'a' referenced before assignment
>>> third_func()
UnboundLocalError: local variable 'a' referenced before assignment
>>> fourth_func()
UnboundLocalError: local variable 'a' referenced before assignment
>>> fifth_func()
UnboundLocalError: local variable 'a' referenced before assignment
{% endcodeblock %}

出现该问题的原因在[Python FAQ](https://docs.python.org/3.7/faq/programming.html#what-are-the-rules-for-local-and-global-variables-in-python)中提到：
> 在Python中，如果变量仅仅是被引用而没有被赋值过，那么默认被视作全局变量。如果一个变量在函数中被赋值过，那么就被视作局部变量。

显然，这里的**被赋值过**，指的是在函数体内任何地方被赋值过。即使没有被执行或是变量引用之后再赋值，都依旧会被当做“被赋值过”，从而被Python视作局部变量。而之所以a += 1会报错是因为 a += 1会被视作a = a + 1，此处a由于有赋值操作的存在被Python视作局部变量，而算术运算符的优先级又高于赋值运算符，于是Python会先计算a + 1，但这个时候a在局部作用域中并不能被查询到，于是抛出`UnboundLocalError`异常。解决方案也很简单，使用`global`关键字进行声明即可。如果变量a在工厂函数之内，则应用`nonlocal`关键字进行声明。

{% codeblock lang:python %}
a = 1
def second_func():
    global a
    a += 1
    return a

def fifth_func():
    a = 3
    def sixth_func():
        nonlocal a
        a += 1
        return a
    return sixth_func()
{% endcodeblock %}

输出：
{% codeblock lang:python %}
>>> second_func()
2
>>> fith_func()
4
{% endcodeblock %}

本条其他参考链接：
[理解Python的UnboundLocalError（Python的作用域）](https://www.kawabangga.com/posts/2245)
[全局变量报错：UnboundLocalError: local variable 'l' referenced before assignment](https://blog.csdn.net/my2010Sam/article/details/17735159)

#### 利用conda包更新历史进行版本恢复

有的时候你在利用conda更新完一个python包或者批量更新之后会因为各种原因想要回撤更新操作，比如由于包兼容问题conda把你另一个你需要用的包给升级或者降级了，移除了你需要的功能，又或者是由于你设置错了conda频道的优先级导致你升级的包都升级到了别的版本等等，这时候你就需要在命令行运行`conda list --revisions`来显示包更新历史，找到最近一次的更新历史和`rev_id`（在每次更新历史的第一行的更新时间之后，比如`2019-08-27 22:50:17  (rev 59)`的`59`就是`rev_id`）后运行`conda install --revision 58(rev_id减1)`就可以回到更新之前的conda环境了。此恢复默认在conda输入命令时所在的python环境进行恢复，也可以通过`-n`参数进行指定。

另外此命令的进阶用法就是通过指定`revision_id`恢复到任意指定版本，比如`conda install --revision 0`就可以让你恢复到原始环境（只保留当初创建环境时带的包）。

本条命令整合如下：
{% codeblock lang:sh %}
conda list --revisions # 假设我们现在所在的版本rev_id是59
conda install --revision 58
{% endcodeblock %}

本条其他参考链接：
[Conda revisions: letting you ‘rollback’ to a previous version of your environment](http://blog.rtwilson.com/conda-revisions-letting-you-rollback-to-a-previous-version-of-your-environment/)
[stackoverflow: conda: remove all installed packages from base/root environment](https://stackoverflow.com/questions/52830307/conda-remove-all-installed-packages-from-base-root-environment)

#### 使用pandas将数据框中的两列转换为字典结构

在数据处理过程中有时候需要使用pandas将数据框中的两列转换为字典结构，其中第一列的所有相同值合并作为键，第二列中的对应值合并转换为列表结构储存。命令如下：

{% codeblock lang:python %}
import pandas as pd

# Your data
data = pd.DataFrame({'column1':['key1','key1','key2','key2'],
       'column2':['value1','value2','value3','value3']})

# Grouped dict
data_dict = data.groupby('column1').column2.apply(list).to_dict()  
{% endcodeblock %}

本条其他参考链接：
[如何将两个pandas列转换为字典，但将相同第一列（键）的所有值合并为一个键？](https://cloud.tencent.com/developer/ask/147684)

#### 使用pandas将数据框中的一列列表转成每行单元素输出

有时建立的数据框会存在列中的元素是列表结构的情况。为了让该数据框更加像数据库中的数据，需要将该列表列转成每行单元素，并且保留其他列信息，应该怎么办呢？命令如下：

{% codeblock lang:python %}
import pandas as pd

# Your data
data = pd.DataFrame([['cuisine_1','id_1',['ingredient_1', 'ingredient_2', 'ingredient_3']], 
        ['cuisine_2','id_2',['ingredient_4', 'ingredient_5']]], 
        columns=['cuisine','id','ingredients'])

'''
data
     cuisine    id                                 ingredients
0  cuisine_1  id_1  [ingredient_1, ingredient_2, ingredient_3]
1  cuisine_2  id_2                [ingredient_4, ingredient_5]
'''

# Split a list inside a Dataframe cell into rows
data = data.ingredients.apply(pd.Series) \
        .merge(data, right_index = True, left_index = True) \
        .drop(["ingredients"], axis = 1) \
        .melt(id_vars = ['cuisine', 'id'], value_name = 'ingredient') \
        .drop("variable", axis = 1) \
        .dropna() \
        .sort_values(by=['id']) \
        .reset_index(drop=True)

'''
data
     cuisine    id    ingredient
0  cuisine_1  id_1  ingredient_1
1  cuisine_1  id_1  ingredient_2
2  cuisine_1  id_1  ingredient_3
3  cuisine_2  id_2  ingredient_4
4  cuisine_2  id_2  ingredient_5
'''
{% endcodeblock %}

此处核心命令有两个，list_column.apply(pd.Series)对列表列进行展开，然后使用melt函数对数据框进行反透视操作。

本条其他参考链接：
[How to split a list inside a Dataframe cell into rows in Pandas](https://www.mikulskibartosz.name/how-to-split-a-list-inside-a-dataframe-cell-into-rows-in-pandas/)

