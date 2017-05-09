---
layout: post
title: Python Cookbook II 学习笔记 - 迭代器
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第19章：迭代器。介绍了几个迭代的高级方法，并行循环，切片，预览迭代器之后的条目等。解决手段都适用于无限迭代器。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>


Python Cookbook II 的学习笔记，第19章：迭代器 Iterator。介绍了几个迭代的高级方法，并行循环，切片，预览迭代器之后的条目等。解决手段都适用于无限迭代器。

# 用生成器写的 Fibonacci 序列

需要一个无界的生成器，它能每次一项生成 Fibonacci 数的无限序列。
由于本身是“惰性求值”的，生成器特别适合这种实现无限序列的任务：

> 用 yield 关键字返回的就是生成器，yield 类似 return，只是这个函数返回的是个生成器。所谓生成器的意思是，yield 返回一个可迭代的对象，并没有真正的执行函数。生成器拥有 next 方法且行为与迭代器完全相同，这意味着生成器也可以用于 for 循环中。

    def fib():
      ''' Fibonacci 数的无界生成器 '''
      x, y = 0, 1
      while True:
        yield x
        x, y = y, x+y

Probe: `x, y = y, x+y` 这种并行赋值，实际是将 `y, x+y` 建立为一个暂时的元组。

现在可以从无限的迭代器中返回前 n 项：

    import itertools
    print(list(itertools.islice(fib(), 10)))
    # => [0, 1, 1, 2, 3, 5, 8, 13, 21, 34)





# 将可迭代对象切成 n 片

有一个可迭代对象 p，想得到 n 个不重叠的步长为 n 的扩展切片，换句话说，如果这个可迭代对象是一个支持扩展切片的序列，那么得到的应该是
`p[0::n]`, `p[1::n]`, 以此类推，直到 `p[n-1::n]` 为止。

    strider('1234567',3)
    # => [['1', '4', '7'], ['2', '5'], ['3', '6']]

    import itertools
    def strider(p, n):
      result = [[] for x in itertools.repeat(0, n)]
      resiter = itertools.cycle(result)
      for item, sublist in zip(p, resiter):
        sublist.append(item)
      return result

    print(strider('1234567', 3))
    # => [['1', '4', '7'], ['2', '5'], ['3', '6']]

Probe: `[[] for x in itertools.repeat(0, n)]` 建立了三个空的 list，之后在 `resiter = itertools.cycle(result)` 中生成了无限循环的空 list，但第 4 个与第 1 个是相同的实例。于是后面添加元素的时候，向 第 1 4 7 个位置添加，实际到了同一个 list 中。

这个 strider 版本使用了 itertools 中的三个函数，它们都返回一个可迭代对象。

函数 repeat 根据指定的次数，重复地生成一个对象，在这里，我们使用此函数而不是内建函数 range 来控制为 result 创建初始值的列表推导。

函数 cycle 接收一个可迭代对象并返回一个迭代器，此迭代器将反复和循环地遍历 cycle 接收的可迭代对象。换句话说，cycle 的执行产生了我们需要的循环效果。

函数 zip 返回的是一个迭代器，因此它避免了将整个结果列表放入内存。





# 通过重叠窗口循环序列

有个可迭代对象 s，现在需要另一个可迭代对象且其中的项是子列表（即滑动窗口），每个列表的长度相同，列表项都是 s 的项，而且相邻的两个窗口存在一定程度重疊。

    import itertools
    def windows(iterable, length=2, overlap=0):
      it = iter(iterable)
      results = list(itertools.islice(it, length))
      while len(results) == length:
        yield results
        results = results[length-overlap:]
        results.extend(itertools.islice(it, length-overlap))
      if results:
        yield results
    
    seq = 'foobarbazer'
    for length in (3, 4):
      for overlap in (0, 1):
        ret = 'len:%d overlap:%d - %s' % (length, overlap, list(map(''.join, windows(seq, length, overlap))))
        print(ret)
    # => len:3 overlap:0 - ['foo', 'bar', 'baz', 'er']
    # => len:3 overlap:1 - ['foo', 'oba', 'arb', 'baz', 'zer', 'r']
    # => len:4 overlap:0 - ['foob', 'arba', 'zer']
    # => len:4 overlap:1 - ['foob', 'barb', 'baze', 'er']

如果确定不需要任何重叠，那么还有一个更快更简洁的可选方法：

    def chop(iterable, length=2):
      return zip(*(iter(iterable),)*length)
      
    print(list(chop('1234567', 2)))
    # => [('1', '2'), ('3', '4'), ('5', '6')]

Probe: `zip(*(iter(iterable),)*length)` 的顺序是先算相乘。



# 并行循环多个可迭代对象

需要并行地循环多个可迭代对象，首先得到所有可迭代对象的第一个项形成的元组，然后是第二项构成的元组，以此类推。

可以编写一个通用的函数，应用于任意数目的序列：

    import itertools
    def par_loop(padding_item, *sequences):
      iterators = list(map(iter, sequences))
      num_remaining = len(iterators)
      result = [padding_item] * num_remaining
      while num_remaining:
        for i, it in enumerate(iterators):
          try:
            result[i] = it.__next__()
          except StopIteration:
            iterators[i] = itertools.repeat(padding_item)
            num_remaining -= 1
            result[i] = padding_item
        if num_remaining:
          yield tuple(result)
    
    print(list(map(''.join, par_loop('_', 'foo', 'zapper', 'ui'))))
    # => ['fzu', 'oai', 'op_', '_p_', '_e_', '_r_']

part_loop 一开始对所有参数都调用内建函数 iter 然后使用生成的迭代器。
这很重要，因为这两个函数都依赖这些迭代器维护的状态。

其关键是保存未耗尽的迭代器的数目，并将每个耗尽的迭代器替换成一个不断生成 padding_item 的无穷迭代器。

而 num_remaining 则被用来记录未耗尽的迭代器的数目，yield 语句和 while 的继续都依赖于仍有某些迭代器未被耗尽这个前提。

如果事先知道哪个可迭代对象是最长的，可将其他各个可迭代对象 x 封装为
`itertools.chain(iter(x), itertools.repeat(padding_item))`
然后调用 zip。但不能对所有的可迭代对象做这种封装，因为生成的迭代器都是无穷的。对全部是无穷的迭代器调用 zip, 那 zip 本身也会无法终止。

举个例子，下面这个版本只在最长的可迭代对象是第一个对象时才能按照预期工作：

    import itertools
    def par_longest_first(padding_item, *sequences):
      iterators = list(map(iter, sequences))
      for i, it in enumerate(iterators):
        if not i:
          continue
        iterators[i] = itertools.chain(it, itertools.repeat(padding_item))
      return zip(*iterators)
      # for t in zip(*iterators):
      #   yield tuple(t)
    
    
    par_iter = par_longest_first('_', 'longest', 'foo', 'zapper', 'ui')
    print(list(map(''.join, par_iter)))
    # => ['lfzu', 'ooai', 'nop_', 'g_p_', 'e_e_', 's_r_', 't___']





# 逐段读取文本文件

    import itertools
    
    def paragraphs(lines, is_separator=str.isspace, joiner=''.join):
      print(repr(lines))
      for sep_group, lineiter in itertools.groupby(lines, key=is_separator):
        if not sep_group:
          yield joiner(lineiter)
    
    s = '''
    aaaa
    bbb
    
    ccc
    ddddd
    eee
    
    '''
    for p in paragraphs(s.splitlines(), lambda x: x=='', '-'.join):
      print(repr(p))
    # => 'aaaa-bbb'
    # => 'ccc-ddddd-eee'


Probe: `itertools.groupby(lines, key)` 返回的是两个值，第一个表示分组依据，相同的值会分到同一组中，后一个值是组内元素的迭代器。

> The operation of groupby() is similar to the uniq filter in Unix. It generates a break or new group every time the value of the key function changes (which is why it is usually necessary to have sorted the data using the same key function).



# 生成排列、组合以及选择

需要对一个序列的排列(permutation)组合(combination)选择(selection)进行迭代搡作。

即使初始的序列长度并不长，组合计算的规则却显示生成的序列可能非常庞
大，比如一个长度为13的序列有超过60亿种可能的排列。所以不能在开始迭代前计算并生成序列中的所有项。

生成器允许在迭代的时候一次一个地计算需要的对象。如果有很多这种对象，而且你也必须逐个地检査它们，那么程序无可避免地会用很长时间才能完成循环。但至少不用浪费很多内存来保存所有项。

    def _combinators(_handle, items, n):
      '''抽取下列组合的通用结构'''
      if n==0:
        yield []
        return
      for i, item in enumerate(items):
        this_one = [item]
        for cc in _combinators(_handle, _handle(items, i), n-1):
          yield this_one + cc
    def combinations(items, n):
      '''取得n个不同的项，顺序是有意义的'''
      def skipIthItem(items, i):
        return items[:i] + items[i+1:]
      return _combinators(skipIthItem, items, n)
    def uniqueCombinations(items, n):
      '''取得n个不同的项，顺序无关'''
      def afterIthItem(items, i):
        return items[i+1:]
      return _combinators(afterIthItem, items, n)
    def selections(items, n):
      '''取得n项（不一定要不同），顺序是有意义的'''
      def keepAllItems(items, i):
        return items
      return _combinators(keepAllItems, items, n)
    def permutations(items):
      '''取得所有项，顺序是有意义的'''
      return combinations(items, len(items))
    
    print("Permutations of 'bar'")
    print(list(map(''.join, permutations('bar'))))
    # => ['bar', 'bra', 'abr', 'arb', 'rba', 'rab']
    print("Combinations of 2 letters from 'bar'")
    print(list(map(''.join, combinations('bar', 2))))
    # => ['ba', 'br', 'ab', 'ar', 'rb', 'ra']
    print("Unique Combinations of 2 letters from 'bar'")
    print(list(map(''.join, uniqueCombinations('bar', 2))))
    # => ['ba', 'br', 'ar']
    print("Selections of 2 letters from 'bar'")
    print(list(map(''.join, selections('bar', 2))))
    # => ['bb', 'ba', 'br', 'ab', 'aa', 'ar', 'rb', 'ra', 'rr']





# 迭代器的前瞻

用迭代器执行某些任务，如解析任务，这种任务要求具有某种“前瞻”(look ahead)的能力，能亊先看到迭代器将要生成的项，同时不扰乱现有的迭代器状态。

最好的办法是将迭代器封装到一个适合的类中：

    import collections
    class peekable(object):
      '''一个支持前瞻操作的迭代器示例用法：
      >>> p = peekable(range(4))
      >>> p.peek()
      0
      >>> p.next(1)
      [0]
      >>> p.peek(3)
      [1, 2, 3]
      >>> p.next(2)
      [1, 2]
      >>> p.peek(2)
      Traceback (most recent call last):
        ...
      StopIteration
      >>> p.peek(1)
      [3]
      >>> p.next(2)
      Traceback (most recent call last):
        ...
      StopIteration
      >>> p.next()
      3
      '''
      def __init__(self, iterable):
        self._iterable = iter(iterable)
        self._cache = collections.deque()
      def __iter__(self):
        return self
      def _fillcache(self, n):
        if n is None:
          n = 1
        while len(self._cache) < n:
          self._cache.append(self._iterable.__next__())
      def next(self, n=None):
        self._fillcache(n)
        if n is None:
          result = self._cache.popleft()
        else:
          result = [self._cache.popleft() for i in range(n)]
        return result
      def peek(self, n=None):
        self._fillcache(n)
        if n is None:
          result = self._cache[0]
        else:
          result = [self._cache[i] for i in range(n)]
        return result

本节展示的 peekable 类有一个有趣的特点，如果从迭代器中请求太多项，会得到一个 StopIteration 异常，迭代器中最后的那些值并不会被丢弃。

举例如果 p 只剩三项，当调用 `p.next(5)` 会得到异常。但之后仍可以继续调用 `p.next(3)` 并获得最后三项的列表。

方法 peek 和 next 的参数 n 默认并不是 1，而是 None，这就可以用两种不同的方式来查看单项：

- 默认方式调用 `p.peek()` 仅仅返回该项，
- 而调用 `p.peek(1)`，得到的是只有一项的列表。

这种方式甚至还支持 `p.next(0)` 返回的是一个空列表 []，同时迭代器的状态也保持不变。

一般来说，可以不用参数调用 p.peek() 探测下一项，不会引起任何问题。





# 计算汇总报告

有一个用某键值组织起来的数据列表，这些数据通常都是从电子表格或其他类似数据源读取来的，现在想生成这些信息的一个汇总，以便用在报告之中。

    from itertools import groupby
    from operator import itemgetter
    def summary(data, key=itemgetter(0), field=itemgetter(1)):
      '''
      汇总指定的数据（一个行序列）
      数据通过指定的键组织起来（默认为每行的第一项）
      获得指定字段的总计（默认为每行的第二项）
      键和字段参数应该是函数，对一条给定的数据记录返回相关值
      '''
      for k, group in groupby(data, key):
        yield k, sum(field(row) for row in group)
    
    sales = [ ('Scotland', 'Edinburgh',  20000),
              ('Scotland', 'Glasgow',    12500),
              ('Wales',    'Cardiff',    29700),
              ('Wales',    'Bangor',     12800),
              ('England',  'London',     90000),
              ('England',  'Manchester', 45600),
              ('England',  ' Liverpool', 29700) ]
    for region, total in summary(sales, field=itemgetter(2)):
      print("%10s: %d" % (region, total))
    # =>  Scotland: 32500
    # =>     Wales: 42500
    # =>   England: 165300


`operator.itemgetter` 高阶函数接受参数 i 然后生成函数 f，f(x) 获取的是 x 的第 i 项，就像索引操作 x[i] 一样。

输入的记录必须是已经根据给定的键排序的，如果不能确定，可以使用 `groupby(sorted(data, key=key), key)`，否则如果排序键出现在不连续的记录里，那么汇总结果也会不连续。
