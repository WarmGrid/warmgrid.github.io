---
layout: post
title: Python Cookbook II 学习笔记 - 常用技巧
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第04章：常用技巧。介绍了一些有用的贴士，如何简明快速的处理 dict，转换二维的 list 等。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>


Python Cookbook II 的学习笔记，第04章：常用技巧。介绍了一些有用的贴士，如何简明快速的处理 dict，转换二维的 list 等。



# 浅拷贝和深拷贝

需要拷贝某对象。不过对一个对象赋值，将其作为参数传递，或者作为结果返回时，Python 通常会使用指向原对象的引用，并不是真正的拷贝。

标准库的 copy 模块提供了两个函数来创建拷贝。第一个常用的函数叫做 copy，会返回一个具有同样的内容和属性的对象：

    import copy
    new_list = copy.copy(existing_list)

如需要对象中的属性和内容被分别地和递归地拷贝，
应该使用 deepcopy：

    import copy
    new_list_of_dicts = copy.deepcopy(existing_list_of_dicts)

对于普通的浅拷贝，另一些方法可以实现同样的功能，前提是你知道想拷贝的对象的类型：

- 对于列表 l，调用 list(l)
- 对于字典 d，调用 dict(d)
- 对于集合 s，调用 set(s)

这样也可以拷贝这些对象。





# 检查对象相同或相等

is 检查对象是否相同，== 操作符则检查两个对象是否相等。

对于不可改变的对象来说，检査是否相同几乎没有什么用处。对于可改变的对象，检査相同性有时却是至关重要的。

举个例子，假设你不确定 a 和 b 是分别指向不同的对象还是引用同一个对象，可以用 a is b 这样简单快速的检査语句来找到答案。

    d = dict(enumerate(l))

用此法获得的字典 d 是等价于列表 l 的，因为对于任意一个有效的非负索引 d[i] is l[i] 都成立。





# 展平嵌套的元素

递归版

    def list_or_tuple(x):
      return isinstance(x, (list, tuple))
    def flatten(sequence, to_expand=list_or_tuple):
      for item in sequence:
        if to_expand(item):
          for subitem in flatten(item, to_expand):
            yield subitem
        else:
          yield item
    
    for x in flatten([1, 2, [3, [], 4, [5, 6], 7, [8, ], ], 9]):
      print(x, end=' ')
    # => 1 2 3 4 5 6 7 8 9

非递归版

    def list_or_tuple(x):
      return isinstance(x, (list, tuple))
    def flatten(sequence, to_expand=list_or_tuple):
      iterators = [iter(sequence)]
      while iterators:
        # 循环当前的最深嵌套（最后）的迭代器
        for item in iterators[-1]:
          # print(item)
          if to_expand(item):
            # 找到了子序列，循环子序列的迭代器
            iterators.append(iter(item))
            break
          else:
            yield item
        else:
          # 最深嵌套的迭代器耗尽，回过头来循环它的父迭代器
          iterators.pop()
    
    for x in flatten([1, 2, [3, [], 4, [5, 6], 7, [8, ], ], 9]):
      print(x)





# 在行列表中对列删除和排序

一般用列表的列表来代表二维的阵列。这些列表可以看做以二维阵列的"行"为元素的列表。处理这种二维阵列的列，一般是重新排序，有时还会忽略列中的部分内容。

尽管列表推导在别处大显神通，但粗略一看，它在这里似乎用处不大。列表推导会产生一个新的列表，而不是修改现有的列表。当需要修改一个现有的列表时，最好的办法是将现有列表的内容赋值为一个列表推导。

举个例子，假设要修改 listOfRows 对于本节的任务，可以写成：

    listOfRows[:] = [[row[0], row[3], row[2]] for row in listOfRows]

还可以考虑使用一个辅助的序列，其中包含的列索引按需要的顺序排列。这样在外层对 listOfRows 进行循环操作的列表推导的内部，加了一个嵌套的对辅助序列进行循环操作的内层列表推导。

    newList = [[row[ci] for ci in (0, 3, 2)] for row in listofRows]

采用辅助序列的方式会获得更多的通用性，因为可以给辅助序列一个名字，并使用这个名字来对一些行列表进行重新排序，或者将其作为参数传递给一个函数：

    def pick_and_reorder_columns(listofRows, column_indexes):
      return [[row[ci] for ci in column_indexes] for row in listofRows]
      
    columns = 0, 3, 2
    newListOfPandas = pick_and_reorder_columns(oldListOfPandas, columns)
    newListOfCats = pick_and_reorder_columns(oldListOfCats, columns)





# 矩阵转置

变换一个列表的列表，将行换成列，列换成行。

需要一个列表，其中的每一项都是同样长度的列表，像这样：

    arr = [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]

列表推导提供了简单方便的方法以完成二维阵列的转换：

    print([[r[col] for r in arr] for col in range(len(arr[0]))])
    [[1, 4, 7, 10]，[2, 5, 8, 11], [3, 6, 9, 12]]

另一个更快也更让人困惑的方法（输出是一样的）是利用内建函数 zip 实现的:

    print(map(list, zip(*arr)))





# 给字典添加条目

给定字典 d，当 k 是字典的键时，则直接使用 d[k]，若 k 不是 d 的键，则给字典增加一个新条目 d[k]。

字典的 setdefault 方法正是为此而设计的。假设我们创建一个由单词到页数的映射，字典将把每个单词映射到这个词出现过的页的页码构成的列表。这个应用中关键的代码段可能是这样的：

    def addword(theIndex, word, pagenumber):
      thelndex.setdefault(word, []).append(pagenumber)

这段代码等价于下面的更加详尽的版本：

    def addword(theIndex, word, pagenumber):
      if word in theIndex:
        theIndex[word].append(pagenumber)
      else:
        theIndex[word] = [pagenumber]

以及：

    def addword(theIndex, word, pagenumber):
      try:
        theIndex[word].append(pagenumber)
      except KeyError:
        theIndex[word] = [pagenumber]

使用 setdefault 方法能在相当大的程度上简化实现。

对于字典 d，`d.setdefault(k, v)` 非常接近于 `d.get(k, v)`，本质的区别是，如果 k 不是字典的键，setdefault 方法会将 d[k] 赋值为 v，get 方法则仅仅返回 v，对 d 不会有任何影响。





# 不用字面引号创建字典

当键是标识符时，可以用 dict 加上命名的参数来避免援引它们：
    data = dict(red=1, green=2, blue=3)
这看上去比直接用字典形式的语法要整洁一些：
    data = {'red': 1, 'green': 2, 'blue': 3}

如果想创建的字典的每个键都对应相同值，只需调用 `dict.fromkeys(key_sequence, value)` 即可。忽略 value 默认使用 None。

下面的例子，用很清爽的方式初始化一个字典，并用它统计不同的小写字母的出现次数：

    import string
    count_by_letter = dict.fromkeys(string.ascii_lowercase, 0)





# 获取字典的子集

有一个巨大的字典，字典中的一些键属于一个特定的集合，而你想创建一个包含这个键集合及其对应值的新字典。

如果你不想改动原字典：

    def sub_dict(somedict, somekeys, default=None):
      return dict([(k, somedict.get(k, default)) for k in somekeys])

如果你从原字典中删除那些符合条件的条目：

    def sub_dict_remove(somedict, somekeys, default=None):
      return dict([(k, somedict.pop(k, default)) for k in somekeys])

    d = {'a': 5, 'b': 6, 'c': 7}
    print(sub_dict(d, 'ab')) # => {'a': 5, 'b': 6}
    print(d)                 # => {'a': 5, 'b': 6, 'c': 7}
    print(sub_dict_remove(d, 'ab')) # => {'a': 5, 'b': 6}
    print(d)                        # => {'c': 7}





# 字典的一对多映射

正常情况下，字典是一对一映射的，但要实现一对多映射也不难，换句话说即一个键对应多个值。有两个可选方案，但具体要看怎么对待键的多个对应值的重复。

下面这种方法，使用 list 作为 dict 的值，允许重复：

    d1 = {}
    d1.setdefault(key, []).append(value)

另一种方案，使用 set 作为 dict 的值，自然而然地消灭了值重复的可能：

    d2 = {}
    d2.setdefault(key, set()).add(value)

除了给键增加对应值之外，还要做更多的事情。对于使用列表并允许重复的第一个情况，下面代码可取得键对应的值列表：

    list_of_values = d1[key]

如果不介意当键的所有值都被移除后，仍留下一个空列表，可以用下面法删除键的对应值：

    d1[key].remove(value)

要想检查一个键是否至少有一个值，使用一个总是返回列表（也可能是空列表）的函数就行了：

    def get_values_if_any(d, key):
      return d.get(key, [])

比如，为了检査 "freep" 是否是字典 d1 的键 "somekey" 的对应值之一，可以这样：

    if 'freep' in get_values_if_any(d1, 'somekey'):
      pass





# 字典的交集和并集

给定两个字典，需要找到两个字典都包含的键（交集)，或者同时属于两个字典的键（并集)。

    a = dict.fromkeys(range(1000))
    b = dict.fromkeys(range(500, 1500))

最快计算出并集的方法是：

    union = dict(a, **b)
    # Probe: 这种情况下 b 的键必须是字符串
    # 更通用的方式 union = dict(list(x.items()) + list(y.items()))

最快且最简洁地获得交集的方法是：

    inter = dict.fromkeys([x for x in a if x in b])





# 用最小化的类替代字典

想搜集一系列的子项，并命名这些子项，而且用字典来实现有点不便。

任意一个类的实例都继承了一个被封装到内部的字典，它用这个字典来记录状态。
可以很容易地利用这个被封装的字典达到目的，只需要写一个内容几乎为空的类：

    class Bunch(object):
      def __init__(self, **kwds):
        self.__dict__.update(kwds)

使用很简单，创建 Bunch 实例：

    point = Bunch(datum=y, square=y*y, coord=x)





# 以指定概率获取元素

需要从一个列表中随机获取元素，就像 `random.choice` 所做的一样，但同时必须根据另一个列表指定的各个元素的概率来获取元素，而不是等同概率。

标准库中的 random 模块提供了生成和使用伪随机数的能力，但并没有提供这样特殊的功能，所以必须得自己写一个函数：

    import random
    def random_pick(some_list, probabilities):
      x = random.uniform(0, 1)
      cumulative_probability = 0.0
      for item, item_probability in zip(some_list, probabilities):
        cumulative_probability += item_probability
        if x < cumulative_probability: break
      return item

这种功能在游戏、模拟和随机测试中是很常见的需求。解决方案使用 random 模块的 uniform 函数获得一个在 0.0 和 1.0 之间分布的伪随机数，之后同时循环元素及其概率，计算不断增加的累积概率，直到这个概率值大于伪随机数。

本节假设（但并未检査）概率序列 probabilities 具有和 some_list 一样的长度，其所有元素都在 0.0 和 1.0 之间，且相加之和为 1.0。如果违反了这个假设，仍能进行一些随机的撷取，但不能完全地遵循（不连贯）函数的参数所规定的行为。

可能想在函数开头加上一些 assert 语句以确保参数的有效性：

    assert len(some_list) == len(probabilities)
    assert 0 <= min(probabilities) and max(probabilities) <= 1
    assert abs(sum(probabilities)-1.0) < 1.0e-5

不过这些检査会消耗一些时间，所以通常都不这么做，在正式的解决方案中也没有把它们纳入。

正如我前面提到的，这个任务要求每一项都有对应的概率，这些概率分布在 0 和 1 之间，且总和相加为 1。类似的任务是根据一个非负整数的序列所定义的权重进行随机撷取：基于机会，而不是概率。对这个问题，最好的解决方案是使用生成器，其内部结构和上文的 random_pick 函数差异很大：

    import random
    def random_picks(sequence, relative_odds):
      table = [z for x, y in zip(sequence, relative_odds) for z in [x]*y]
      while True:
        yield random.choice(table)

生成器首先准备一个 table，它的元素的数目是 `sum(relative_odds)` 个，sequence 中的每个元素都可以在 table 中出现多次，出现的次数等于它在 relative_odds 序列中所对应的非负整数。

一旦 table 被制作完毕，生成器的主体就可以变得又小又快，因为它只需要将随机撷取的工作委托给 `random.choice`。





# 在表达式中处理异常

你想写一个表达式，所以你无法直接用 try / except 语句，但仍需要处理表达式可能抛出的异常。

为了抓住异常，try / except 是必不可少的，但 try / except 是一条语句，在表达式内部使用它的唯一方法是借助一个辅助函数：

    def throws(t, f, *a, **k):
      """
      如果 f(*a, **k) 拋出异常且其类型是 t 的话则返回 True
      或者，如果 t 是一个元组的话，类型是 t 中的某项
      """
      try:
        f(*a, **k)
      except t:
        return True
      else:
        return False

假设有个文本文件，每行有一个数字，但文件也可能有多余的内容如空格行及注释行等。可以生成一个包含文件中的所有数字的列表，只需略去那些不是数字的行即可：

    data = [float(line) for line in open(sorae_file) if not throws(ValueError, float, line)]

throws 函数的问题是实际上做了两次关键操作：一次是看它有没有抛出异常，另一次是获得结果。因此这里有些浪费。最好是能够同时获得结果和被截获异常的提示。

如果能让结果返回一个元组，第一位代表是否发生异常，第二位是返回值，就可以达到目的，不幸这个版本不符合列表推导的要求，没有什么优雅的办法能够同时得到标志和结果。

因此可以选择另一种方法：一个在任何情况下都返回 list 的函数，如果有异常被捕获就返回空列表，否则就返回仅包含结果的列表。这个方法工作得很好，但是为了清晰起见，最好把函数名改一改：
    
    def returns(t, f, *a, **k):
      """正常情况下返回 [f(*a, **k)] 若有异常返回 [] """
      try:
        return [f(*a, **k)]
      except t:
        return []

最后生成的列表推导变得更加优雅，比解决方案中的版本好多了。

    data = [x for line in open(some_file) for x in returns(ValueError, float, line)]





# 确保名字已定义

你需要确保某个名字已经在给定的模块中定义过，比如想确认已经存在内建名字 set，如果该名字未被定义，你想执行一些代码来完成定义。

这是 exec 语句最好的用武之地。exec 使得我们可以执行字符串中的任意 Python 代码，这让我们可以写个很简单的函数来达到目的：

    import __builtin__
    def ensureDefined(name, defining_code, target=__builtin__):
      if not hasattr(target, name):
        d = {}
      exec defining_code in d
      assert name in d, 'Code %r did not set name %r' % (defining_code, name)
      setattr(target, name, d[name])


