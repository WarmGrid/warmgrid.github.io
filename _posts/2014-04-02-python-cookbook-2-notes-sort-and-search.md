---
layout: post
title: Python Cookbook II 学习笔记 - 排序和搜索
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第05章：排序和搜索。介绍了排序的 DSU 模式的简单运用。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>



# 对字典排序

> 排序的 DSU 模式：decorate-sort-undecorate，先建立辅助列表，把用于排序的键放在首位，排序这个列表，再把第二位的值取回来。

这意味着需要先根据字典的键排序，然后再让对应值也处于同样的顺序。最简单的方法可以通过这样的描述来概括：先将键排序，然后由此选出对应值：

    def sortedDictValues(adict):
      keys = adict.keys()
      keys.sort()
      return [adict[key] for key in keys]

DSU 的另一个示例：

对一个字符串列表排序，并忽略掉大小写信息。举个例子，要小写的 a 排在大写的 B 前面。

默认的情况下，字符串比较是大小写敏感的，所有的大写字符排在小写字符之前。

    def case_insensitive_sort(string_list):
      auxiliary_list = [(x.lower(), x) for x in string_list]
      auxiliary_list.sort()
      return [x[1] for x in auxiliary_list]

Python 2.4 以后提供了对 DSU 的原生支持，因此可以用更简短更快的方式：

    def case_insensitive_sort(string_list):
      return sorted(string_list, key=str.lower)

如果希望 case_insensitive_sort 函数也能够直接作用于原列表，只需要将return语句修改为对列表本体的赋值：

    string_list[:] = [x[1] for x in auxiliary_list]




# 根据对应值将键或索引排序

需要统计不同元素出现的次数，并且根据它们的出现次数安排它们的顺序，比如制作一个柱状图。

如果不考虑柱状图在图形图像上的含义，它实际上就是基于各种不同元素出现的次数，根据对应值将键或索引排序。下面是 dict 的一个子类，为了这种应用加入了两个方法：
    
    class hist(dict):
      def add(self, item, increment=1):
        '''为item的条目增加计数'''
        self[item] = increment + self.get(item, 0)
      def counts(self, reverse=False):
        '''返回根据对应值排序的鍵的列表'''
        aux = [(self[k], k) for k in self]
        aux.sort()
        if reverse:
          aux.reverse()
        return [k for v, k in aux]
    



# 以随机顺序处理列表的元素

如果允许修改输入列表中的元素顺序，那么下面的函数就是最简单和最快的：

    def process_all_in_random_order(data, process):
      random.shuffle(data)
      for elem in data:
        process(elem)



# 根据姓的首字母将人名排序和分组

将一组人名写入地址簿，同时还希望地址簿能够根据姓的首字母进行分组，且按照字母顺序表排序。

    import itertools
    def groupnames(name_iterable):
      sorted_names = sorted(name_iterable, key=_sortkeyfunc)
      name_dict = { }
      for key, group in itertools.groupby(sorted_names, _groupkeyfunc):
        name_dict[key] = tuple(group)
      return name_dict
    pieces_order = { 2: (-1, 0), 3: (-1, 0, 1) }
    def _sortkeyfunc(name):
      '''name 是带有名和姓以及可选的中名或首字母的字符串，
      这些部分之间用空格隔开；返回的字符串的顺序是
      姓-名-中名，以满足排序的需要'''
      name_parts = name.split()
      return ' '.join([name_parts[n] for n in pieces_order[len(name_parts)]])
    def _groupkeyfunc(name):
      '''返回的鍵（即姓的首字母）被用于分组'''
      return name.split()[-1][0]
    
    names = ['Python PyQt', 'Python Pyside', 'Protoss Pylon', 'Ruby Rails']
    print(groupnames(names))
    # => {'R': ('Ruby Rails',), 'P': ('Python PyQt', 'Protoss Pylon', 'Python Pyside')}
    




