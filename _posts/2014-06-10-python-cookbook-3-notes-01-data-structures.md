---
layout: post
title: Python Cookbook III 学习笔记 - 数据结构和算法
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true

summary: '自己整理的 Python Cookbook III 的学习笔记, 没有严格照原文翻译但保留了大部分内容, 第01章, 数据结构和算法. 介绍了序列的索引, 切片, 优先级队列, 字典的布尔计算等实用技巧.'


---


<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>

自己整理的 Python Cookbook III 的学习笔记, 没有严格照原文翻译但保留了大部分内容, 第01章, 数据结构和算法. 介绍了序列的索引, 切片, 优先级队列, 字典的布尔计算等实用技巧.

# 目录

- 使用 deque 实现缓存环
- 使用 heapq 查找最大或最小的 N 个元素
- 使用 heapq 实现优先级序列
- 使用 defaultdict 实现字典的一键对多值
- 能记录键的顺序的字典
- 对字典的值进行运算
- 计算两个字典的交集差集
- 在序列中去除重复且保持原有顺序
- 为切片对象命名
- 用 Counter 列举频次最高的元素
- 以指定的键或值排序一组字典
- 排序不具备原生的比较方法的对象
- 对一些记录按照指定的字段分组
- 过滤序列中的元素
- 使用字典推导提取字典的子集
- 使用 namedtuple 实现命名元组
- 对一组数据同时完成映射 map 和化简 reduce
- 使用 ChainMap 合并多个映射



# 使用 deque 实现缓存环

### 问题

需要记录有限个数的历史, 当历史记录不断增加时挤掉最早的.

### 方案

`collections.deque` 非常适合来完成这类任务. 下面的代码可以匹配一段文本中出现了指定关键词的行, 然后返回找到的最近的 N 行.

    from collections import deque

    def search(lines, pattern, history=5):
        previous_lines = deque(maxlen=history)
        for line in lines:
            if pattern in line:
                yield line, previous_lines
            previous_lines.append(line)

    # Example use on a file
    if __name__ == '__main__':
        with open('somefile.txt') as f:
            for line, prevlines in search(f, 'python', 5):
                for pline in prevlines:
                    print(pline, end='')
                print(line, end='')
                print('-'*20)

### 讨论

从一些项中搜索符合条件的元素的代码通常会用 `yield`, 本例中就是这么做的. 这样可以把"搜索"和"对搜索结果的使用"分开. `deque(maxlen=N)` 建立了一个定长的队列, 队列满后再添加新元素时, 会把最老的元素挤掉. 像这样:

    >>> q = deque(maxlen=3)
    >>> q.append(1)
    >>> q.append(2)
    >>> q.append(3)
    >>> q
    deque([1, 2, 3], maxlen=3)
    >>> q.append(4)
    >>> q
    deque([2, 3, 4], maxlen=3)
    >>> q.append(5)
    >>> q
    deque([3, 4, 5], maxlen=3)

用 list 当然也可以做到这些, 但是使用队列就更加快速优雅.

此外如果建立 deque 时不指定长度, 可以得到一个允许从两端插入元素的队列.

    >>> q = deque()
    >>> q.append(1)
    >>> q.append(2)
    >>> q.append(3)
    >>> q
    deque([1, 2, 3])
    >>> q.appendleft(4)
    >>> q
    deque([4, 1, 2, 3])
    >>> q.pop()
    3
    >>> q
    deque([4, 1, 2])
    >>> q.popleft()
    4

这种允许两端插入弹出元素的队列, 复杂度是 O(1). 与之相比, list 如果从起首位置插入弹出元素, 复杂度是 O(N).



# 使用 heapq 查找最大或最小的 N 个元素

### 问题

需要建立集合中最大或最小的 N 个元素的列表.

### 方案

heapq 模块的两个函数 `nlargest()` 和 `nsmallest()` 就是做这个的:

    import heapq

    nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
    print(heapq.nlargest(3, nums))  # Prints [42, 37, 23]
    print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]

这两个函数都可以接受可选的参数 `key`, 用来指定复杂的数据应如何排序:

    portfolio = [
       {'name': 'IBM', 'shares': 100, 'price': 91.1},
       {'name': 'AAPL', 'shares': 50, 'price': 543.22},
       {'name': 'FB', 'shares': 200, 'price': 21.09},
       {'name': 'HPQ', 'shares': 35, 'price': 31.75},
       {'name': 'YHOO', 'shares': 45, 'price': 16.35},
       {'name': 'ACME', 'shares': 75, 'price': 115.65}
    ]

    cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
    expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])

### 讨论

如果从巨大的集合中查找少数一些最大最小的元素, 这个方法的性能非常优秀. 实现原理是先将数据转为按照 heap 排序的列表:

    >>> nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
    >>> import heapq
    >>> heap = list(nums)
    >>> heapq.heapify(heap)
    >>> heap
    [-4, 2, 1, 23, 7, 2, 18, 23, 42, 37, 8]

heap 的最重要特点是 `heap[0]` 永远是最小的那个元素, 此外, `heapq.heappop()` 则用于查找随后的元素, 这个方法将弹出当前的最小元素, 然后以 O(log N) 复杂度查找下一个最小元素. 比如想要找到前三个最小的元素, 可以这样做:

    >>> heapq.heappop(heap)
    -4
    >>> heapq.heappop(heap)
    1
    >>> heapq.heappop(heap)
    2


`nlargest()` 和 `nsmallest()` 是用来查找 N 个最大最小值的, 如果仅需要一个值, 最快的方法是使用 `min()` 和 `max()`. 类似的, 如果 N 与集合本身差不多大了, 就应该先排序再取切片(`sorted(items)[:N]` or `sorted(items)[-N:]`).



# 使用 heapq 实现优先级序列


### 问题

需要实现一个能按照指定优先级排序的队列, 并且每次 `pop` 时总是返回最高优先级的元素.

### 方案

下面用 heapq 模块实现了一个带优先级的队列:

    import heapq

    class PriorityQueue:
        def __init__(self):
            self._queue = []
            self._index = 0

        def push(self, item, priority):
            heapq.heappush(self._queue, (-priority, self._index, item))
            self._index += 1

        def pop(self):
            return heapq.heappop(self._queue)[-1]

之后可以像这样使用:

    >>> class Item:
    ...     def __init__(self, name):
    ...         self.name = name
    ...     def __repr__(self):
    ...         return 'Item({!r})'.format(self.name)
    ...
    >>> q = PriorityQueue()
    >>> q.push(Item('foo'), 1)
    >>> q.push(Item('bar'), 5)
    >>> q.push(Item('spam'), 4)
    >>> q.push(Item('grok'), 1)

    >>> q.pop()
    Item('bar')
    >>> q.pop()
    Item('spam')
    >>> q.pop()
    Item('foo')
    >>> q.pop()
    Item('grok')

注意第一次 `pop()` 时返回的是最高优先级的元素, 另外也要留意相同优先级的元素是按照添加的顺序依次返回的.

### 讨论

核心概念是 `heapq` 模块的用法, 函数 `heapq.heappush()` 负责推入元素, 且最小优先级的元素总是在序列的起首(在"使用 heapq 查找最大或最小的 N 个元素"中有讨论). `heapq.heappop()` 总是弹出"最小的"元素. 此外, 这两个操作的复杂度是 O(log N), 在大量数据时性能也不错.

队列中每个元素都存储为 tuple (-priority, index, item), 优先级用了负数, 这样最高优先级就排在了最前端. 第二个位置要存放 index, 因为须用客户指定的优先级 prioriry, 以及插入元素的顺序 self._index 做联合主键, 否则两个元素具有相同优先级就没法判断顺序.

具体的说, 仅有 Item 的实例没法排序:

    >>> a = Item('foo')
    >>> b = Item('bar')
    >>> a < b
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unorderable types: Item() < Item()

如果改为使用 (priority, item) 的 tuple, 那么优先级不同时就可以排序了, 优先级相同仍然不行:

    >>> a = (1, Item('foo'))
    >>> b = (5, Item('bar'))
    >>> a < b
    True
    >>> c = (1, Item('grok'))
    >>> a < c
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unorderable types: Item() < Item()

最后, 代码中使用的 (priority, index, item) 方式, 因为每个元素的 index 都不同, 就可以完全回避这个问题.

    >>> a = (1, 0, Item('foo'))
    >>> b = (5, 1, Item('bar'))
    >>> c = (1, 2, Item('grok'))
    >>> a < b
    True
    >>> a < c
    True

注意多线程使用这个队列时需要加锁. 见 [Communicating Between Threads](http://chimera.labs.oreilly.com/books/1230000000393/ch12.html#thread_communication) .


# 使用 defaultdict 实现字典的一键对多值


### 问题

需要一个键能对应多个值的字典, 即 "multidict".

### 方案

字典的键只能对应单个值, 想要对应多个值, 需要把它们放到"容器"里, 比如 list 或者 set. 就像这样:

    d = {
       'a' : [1, 2, 3],
       'b' : [4, 5]
    }

    e = {
       'a' : {1, 2, 3},
       'b' : {4, 5}
    }

采用 list 还是 set, 主要取决于打算怎样处理重复项.

`collections` 模块的 `defaultdict` 可以让事情变简单一些. 这种字典在创建时自动初始化了默认值, 所以我们只管添加元素就行了:

    from collections import defaultdict

    d = defaultdict(list)
    d['a'].append(1)
    d['a'].append(2)
    d['b'].append(4)
    ...

    d = defaultdict(set)
    d['a'].add(1)
    d['a'].add(2)
    d['b'].add(4)
    ...

`defaultdict` 有个需要注意的地方, 创建字典时有些 key 是尚不存在的, 但此后被访问到时, 它也会为这个 key 初始化一个默认值. 如果不需要这种行为, 可以用普通的字典对象上调用 `setdefault()` 的方法来代替:

    d = {}    # A regular dictionary
    d.setdefault('a', []).append(1)
    d.setdefault('a', []).append(2)
    d.setdefault('b', []).append(4)
    ...

### 讨论

创建一个多值的字典很容易, 但要是想要自己手动为它设默认初值, 就有些麻烦. 比如你可能有这么一段代码:

    d = {}
    for key, value in pairs:
        if key not in d:
             d[key] = []
        d[key].append(value)

使用 `defaultdict` 就清晰多了:

    d = defaultdict(list)
    for key, value in pairs:
        d[key].append(value)

这个技巧跟本章后面的 "对一些记录按照指定的字段分组" 一节有很大联系.



# 能记录键的顺序的字典

### 问题

需要一个能控制条目顺序的字典.

### 方案

`collections` 模块的 `OrderedDict` 可以做到这一点. 它在迭代时完全按照条目添加的顺序:

    from collections import OrderedDict

    d = OrderedDict()
    d['foo'] = 1
    d['bar'] = 2
    d['spam'] = 3
    d['grok'] = 4
    for key in d:
        print(key, d[key])
    # => "foo 1", "bar 2", "spam 3", "grok 4"

当建立映射后需要转换到其他格式, 比如转换 JSON 时, `OrderedDict` 尤其有用.

    >>> import json
    >>> json.dumps(d)
    '{"foo": 1, "bar": 2, "spam": 3, "grok": 4}'

### 讨论

`OrderedDict` 内部维持着一个额外的列表, 作用是按照添加的顺序记录字典的键. 当字典增加新的键时会添加到这个表的最末, 当已有键重新赋值时不会改变顺序.

`OrderedDict` 的缺点是有普通字典的两倍大小, 如果数据很多的话, 需要权衡一下再决定是不是要使用.



# 对字典的值进行运算

### 问题

需要对字典的值计算 min(), max(), 或对值排序.

### 方案

考虑如下的记录股票价格的字典:

    prices = {
       'ACME': 45.23,
       'AAPL': 612.78,
       'IBM': 205.55,
       'HPQ': 37.20,
       'FB': 10.75
    }

对字典的值做计算, 可以用 zip 反转键 / 值的顺序, 例如像这样计算最大最小值:

    min_price = min(zip(prices.values(), prices.keys()))
    # min_price is (10.75, 'FB')

    max_price = max(zip(prices.values(), prices.keys()))
    # max_price is (612.78, 'AAPL')

类似的, zip 也能配合 sorted 排序:

    prices_sorted = sorted(zip(prices.values(), prices.keys()))
    # prices_sorted is [(10.75, 'FB'), (37.2, 'HPQ'),
    #                   (45.23, 'ACME'), (205.55, 'IBM'),
    #                   (612.78, 'AAPL')]

注意 zip 生成的迭代器只能用一次, 类似下面的代码将出现问题:

    prices_and_names = zip(prices.values(), prices.keys())
    print(min(prices_and_names))   # OK
    print(max(prices_and_names))   # ValueError: max() arg is an empty sequence

### 讨论

对字典直接用 reduce 方法, 会作用在字典的键上, 例如:

    min(prices)   # Returns 'AAPL'
    max(prices)   # Returns 'IBM'

一般这不是想要的结果, 之后你可能会试着用 values() 来修正:

    min(prices.values())  # Returns 10.75
    max(prices.values())  # Returns 612.78

但不幸的是, 这也不是想要的, 因为这样就只剩下字典的值了, 没法知道对应的键是什么. 可以通过在 `min()` `max()` 的参数中提供排序的 key 来解决:

    min(prices, key=lambda k: prices[k])  # Returns 'FB'
    max(prices, key=lambda k: prices[k])  # Returns 'AAPL'

但是找到了最小值的键后, 为了能够得到值, 还得再查找一次:

    min_value = prices[min(prices, key=lambda k: prices[k])]

使用 zip() 的解决方案反转了键和值的顺序,  变成了 (value, key) 的组对, 比较这样的序列时, value 会首先被比较, 这样就在一行代码中完成了所有该做的事.

需要注意当 value 的最大或最小值有重复时, min() 和 max() 的返回值会依赖于 key. 比如:

    >>> prices = { 'AAA' : 45.23, 'ZZZ': 45.23 }
    >>> min(zip(prices.values(), prices.keys()))
    (45.23, 'AAA')
    >>> max(zip(prices.values(), prices.keys()))
    (45.23, 'ZZZ')



# 计算两个字典的交集差集


### 问题

对两个字典的键或值求出其中相同的部分.

### 方案

考虑下面两个 dict:

    a = {
       'x' : 1,
       'y' : 2,
       'z' : 3 }
    b = {
       'w' : 10,
       'x' : 11,
       'y' : 2 }

要找出两个字典的相同部分, 只需要对其 keys() 或 items() 应用普通的 set 方法, 例如:

    # Find keys in common
    a.keys() & b.keys()   # { 'x', 'y' }
    # Find keys in a that are not in b
    a.keys() - b.keys()   # { 'z' }
    # Find (key,value) pairs in common
    a.items() & b.items() # { ('y', 2) }

这种方法也能用来过滤字典. 比如过滤掉指定 key, 可以像下面这样应用字典推导:

    # Make a new dictionary with certain keys removed
    c = {key:a[key] for key in a.keys() - {'z', 'w'}}
    # c is {'x': 1, 'y': 2}

### 讨论

字典是对键和值的映射, 其 `keys()` 方法返回一个 `keys-view` 对象, 这种对象也支持常规的 set 操作, 合集, 交集, 差集等. 如果要用到这一类型的操作, 不需要显式转换成 set. 同样的, 其 `items()` 方法返回包含 `(key, value)` 的 `items-view` 对象, 也能直接运用集合操作.

字典的 `values()` 返回的值不能直接做交集差集运算, 因为字典的值有可能是重复的.

    print(a.keys().__class__)
    # => <class 'dict_keys'>
    print(a.items().__class__)
    # => <class 'dict_items'>



# 在序列中去除重复且保持原有顺序

### 问题

需要去掉序列中重复元素, 且保持原有的顺序.

### 方案

如果序列中的值都是可哈希的, 那么用 set 和 生成器就能解决问题:

    def dedupe(items):
        seen = set()
        for item in items:
            if item not in seen:
                yield item
                seen.add(item)

    a = [1, 5, 2, 1, 9, 1, 5, 10]
    print(list(dedupe(a)))
    # => [1, 5, 2, 9, 10]

这个方法仅在序列中的元素都可哈希时才能用. 对于不可哈希的元素(如字典), 需要加上一个可选的 key 参数:

    def dedupe(items, key=None):
        seen = set()
        for item in items:
            val = item if key is None else key(item)
            if val not in seen:
                yield item
                seen.add(val)

    a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
    print(list(dedupe(a, key=lambda d: (d['x'],d['y']))))
    [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
    print(list(dedupe(a, key=lambda d: d['x'])))
    [{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]

### 讨论

如果只是要去除重复的话, 对序列直接调用 set 方法就够用了, 但这样就不能保持原有的顺序:

    >>> a
    [1, 5, 2, 1, 9, 1, 5, 10]
    >>> set(a)
    {1, 2, 10, 5, 9}

本节的方法使用了生成器, 目的是保持通用性, 不只是在处理列表时才能用. 比如需要读取文件消除重复的行:

    with open(somefile,'r') as f:
        for line in dedupe(f):
            ...



# 为切片对象命名

### 问题

程序中存在大量硬编码的, 不易阅读的对列表或元组的切片, 需要设法改进.

### 方案

假设有一段代码, 从固定格式的对象(文件或者类似的什么)中取出特定数据:

    ######    0123456789012345678901234567890123456789012345678901234567890'
    record = '....................100          .......513.25     ..........'
    cost = int(record[20:32]) * float(record[40:48])

可以这样处理以避免硬编码这些索引, 保持代码的清晰:

    SHARES = slice(20,32)
    PRICE  = slice(40,48)

    cost = int(record[SHARES]) * float(record[PRICE])

### 讨论

通常来讲, `slice()` 方案更加简单易懂, 即使以后重读代码也能快速弄明白当时在做什么.

内建的 `slice()` 建立一个切片对象, 可以用到所有需要做切片的地方:

    >>> items = [0, 1, 2, 3, 4, 5, 6]
    >>> a = slice(2, 4)
    >>> items[2:4]
    [2, 3]
    >>> items[a]
    [2, 3]
    >>> items[a] = [10,11]
    >>> items
    [0, 1, 10, 11, 4, 5, 6]
    >>> del items[a]
    >>> items
    [0, 1, 4, 5, 6]

切片对象的实例具有 `s.start`, `s.stop`, 和 `s.step` 属性:

    >>> a = slice(10, 50, 2)
    >>> a.start
    10
    >>> a.stop
    50
    >>> a.step
    2

另外, 切片对象实例的 `s.indices(size)` 方法用于把该对象转换为指定的长度, 以后把它用到别的列表时, 可以避免 `IndexError` 异常. 这个方法返回值是 `(start, stop, step)`:

    >>> s = 'HelloWorld'
    >>> a.indices(len(s))
    (5, 10, 2)
    >>> for i in range(*a.indices(len(s))):
    ...     print(s[i])
    ...
    W
    r
    d

> 这例子有问题, sliceobj.indices(length) 应该是按照总长度 length 调整 sliceobj 的上下界, 这样以后把 sliceobj 应用到具有 length 长度的 list 上就不会越界, 所以 slice(10, 50, 2) 按照长度 10 调整完了, 应该是 slice(10, 10, 2) 才对.
>
>     a = slice(10, 50, 2)
>     s = 'HelloWorld'
>     a1 = a.indices(len(s))
>     print(a1)
>     # => (10, 10, 2)
>     b = slice(15, 50, 2)
>     b1 = b.indices(len(s))
>     print(b1)
>     # => (10, 10, 2)



# 用 Counter 列举频次最高的元素

### 问题

需要统计一个序列中出现次数最多的元素.

### 方案

`collections.Counter` 就是用来做这个的, 还提供了一个 `most_common(count)` 方法来输出结果. 比如有一个单词列表, 需要统计出现次数最多的单词:

    words = [
       'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
       'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not',
       'around', 'the', 'eyes', "don't", 'look', 'around',  'the',
       'eyes', 'look', 'into', 'my', 'eyes', "you're", 'under'
    ]

    from collections import Counter
    word_counts = Counter(words)
    top_three = word_counts.most_common(3)
    print(top_three)
    # => [('eyes', 8), ('the', 5), ('look', 4)]

### 讨论

Counter 对象接受任何可哈希的元素序列作为输入, 其内部实现是一个把元素映射到出现频率的字典:

    >>> word_counts['not']
    1
    >>> word_counts['eyes']
    8

可以手动修改 Counter 的统计数据:

    >>> morewords = ['why','are','you','not','looking','in','my','eyes']
    >>> for word in morewords:
    ...     word_counts[word] += 1
    ...
    >>> word_counts['eyes']
    9

也可以用 `update(seq)` 来更新统计:

    >>> word_counts.update(morewords)

很少有人知道, Counter 的实例可以做常规的数学计算:

    >>> a = Counter(words)
    >>> b = Counter(morewords)
    >>> a
    Counter({'eyes': 8, 'the': 5, 'look': 4, 'into': 3, 'my': 3, 'around': 2,
             "you're": 1, "don't": 1, 'under': 1, 'not': 1})
    >>> b
    Counter({'eyes': 1, 'looking': 1, 'are': 1, 'in': 1, 'not': 1, 'you': 1,
             'my': 1, 'why': 1})

    >>> # Combine counts
    >>> c = a + b
    >>> c
    Counter({'eyes': 9, 'the': 5, 'look': 4, 'my': 4, 'into': 3, 'not': 2,
             'around': 2, "you're": 1, "don't": 1, 'in': 1, 'why': 1,
             'looking': 1, 'are': 1, 'under': 1, 'you': 1})

    >>> # Subtract counts
    >>> d = a - b
    >>> d
    Counter({'eyes': 7, 'the': 5, 'look': 4, 'into': 3, 'my': 2, 'around': 2,
             "you're": 1, "don't": 1, 'under': 1})




# 以指定的键或值排序一组字典

### 问题

列表中包含一组字典, 需要按照指定的值排序.

### 方案

`operator` 模块的 `itemgetter` 函数很适合用来取得排序的键, 比如对于数据库中查询的结果:

    rows = [
        {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
        {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
        {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
        {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
    ]

可以非常简单的按照某个字段排序(需要每个字典都有这一字段):

    from operator import itemgetter
    rows_by_fname = sorted(rows, key=itemgetter('fname'))
    rows_by_uid = sorted(rows, key=itemgetter('uid'))
    print(rows_by_fname)
    print(rows_by_uid)

上面的代码输出如下:

    [{'fname': 'Big', 'uid': 1004, 'lname': 'Jones'},
     {'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'},
     {'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
     {'fname': 'John', 'uid': 1001, 'lname': 'Cleese'}]

    [{'fname': 'John', 'uid': 1001, 'lname': 'Cleese'},
     {'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
     {'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'},
     {'fname': 'Big', 'uid': 1004, 'lname': 'Jones'}]

`itemgetter()` 还可以接受多个键作为参数:

    rows_by_lfname = sorted(rows, key=itemgetter('lname', 'fname'))
    print(rows_by_lfname)

将输出这样的结果:

    [{'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
     {'fname': 'John', 'uid': 1001, 'lname': 'Cleese'},
     {'fname': 'Big', 'uid': 1004, 'lname': 'Jones'},
     {'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'}]

### 讨论

本例中一组数据要使用内建的 sorted() 函数处理, 此函数接受一个用于排序的关键字参数(key), 这个参数应该是个可调用对象, 每次接受一个数据作为输入, 返回希望用于排序的值. `itemgetter()` 就是这样一个可调用的对象.

`operator.itemgetter()` 查找对象的索引并取出对应的值, 这个索引可以是字典的键, 列表或元组的下标, 或任何能被对象的 `__getitem__()` 接受的参数. 多个索引时会返回元组, 仍然可以被 sorted() 利用排序. 之前的"先按姓氏再按名字"排序, 就是这样做的.

`itemgetter()` 的功能有时可以用 lambda 表达式代替:

    rows_by_fname = sorted(rows, key=lambda r: r['fname'])
    rows_by_lfname = sorted(rows, key=lambda r: (r['lname'], r['fname']))

通常这样也很好了, 但是 `itemgetter()` 更快一些.

最后, 除 `sorted()` 以外, `min()` `max()` 也可以接受这种参数.

    >>> min(rows, key=itemgetter('uid'))
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001}
    >>> max(rows, key=itemgetter('uid'))
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}



# 排序不具备原生的比较方法的对象

### 问题

需要排序一组对象, 但是它们没有原生的比较大小的方法.

### 方案

与上一节的例子相似, sorted() 函数可以接受排序关键字(key), 这个参数应该是个可调用对象, 每次接受一个数据作为输入, 返回希望用于排序的值, 据此就可以比较对象的大小. 比如有一组 User 实例, 需要按照 user_id 属性排序:

    >>> class User:
    ...     def __init__(self, user_id):
    ...         self.user_id = user_id
    ...     def __repr__(self):
    ...         return 'User({})'.format(self.user_id)
    ...
    >>> users = [User(23), User(3), User(99)]
    >>> users
    [User(23), User(3), User(99)]
    >>> sorted(users, key=lambda u: u.user_id)
    [User(3), User(23), User(99)]

`operator.attrgetter()` 可以代替 lambda:

    >>> from operator import attrgetter
    >>> sorted(users, key=attrgetter('user_id'))
    [User(3), User(23), User(99)]

### 讨论

使用 lambda 还是 attrgetter() 看各人喜好. 通常 attrgetter() 更快一些. 跟上一节 `operator.itemgetter()` 一样, attrgetter() 也能接受多个参数, 返回一个元组:

    by_name = sorted(users, key=attrgetter('last_name', 'first_name'))

同样的, 也可以用在 min() max() 中:

    >>> min(users, key=attrgetter('user_id')
    User(3)
    >>> max(users, key=attrgetter('user_id')
    User(99)



# 对一些记录按照指定的字段分组

### 问题

一个字典或是对象的列表, 需要按照特定的字段分组, 比如日期.

### 方案

`itertools.groupby()` 可以用来完成这类任务. 比如下面这样的由字典组成的序列:

    rows = [
        {'address': '5412 N CLARK', 'date': '07/01/2012'},
        {'address': '5148 N CLARK', 'date': '07/04/2012'},
        {'address': '5800 E 58TH', 'date': '07/02/2012'},
        {'address': '2122 N CLARK', 'date': '07/03/2012'},
        {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'},
        {'address': '1060 W ADDISON', 'date': '07/02/2012'},
        {'address': '4801 N BROADWAY', 'date': '07/01/2012'},
        {'address': '1039 W GRANVILLE', 'date': '07/04/2012'},
    ]

假设现在要按照日期迭代这些元素, 首先要按照这个字段排序, 排序后对其应用 `itertools.groupby()`:

    from operator import itemgetter
    from itertools import groupby

    # Sort by the desired field first
    rows.sort(key=itemgetter('date'))
    # Iterate in groups
    for date, items in groupby(rows, key=itemgetter('date')):
        print(date)
        for i in items:
            print('    ', i)

将输出下面这样的结果:

    07/01/2012
         {'date': '07/01/2012', 'address': '5412 N CLARK'}
         {'date': '07/01/2012', 'address': '4801 N BROADWAY'}
    07/02/2012
         {'date': '07/02/2012', 'address': '5800 E 58TH'}
         {'date': '07/02/2012', 'address': '5645 N RAVENSWOOD'}
         {'date': '07/02/2012', 'address': '1060 W ADDISON'}
    07/03/2012
         {'date': '07/03/2012', 'address': '2122 N CLARK'}
    07/04/2012
         {'date': '07/04/2012', 'address': '5148 N CLARK'}
         {'date': '07/04/2012', 'address': '1039 W GRANVILLE'}

### 讨论

`groupby()` 扫描序列中指定的"分组键", 每次迭代时都返回分组值相同的一组元素. 注意: 需要事先按照这个"分组键"对输入序列进行排序, 这非常重要, 不事先排序的话, 原数据的分组值"相同但不连续"时, groupby() 会保留其原先的位置, 不会把它们拼成一组.

如果只是想根据某个键分组, 不在乎之后的访问顺序, 用 defaultdict 更合适:

    from collections import defaultdict
    rows_by_date = defaultdict(list)
    for row in rows:
        rows_by_date[row['date']].append(row)

然后可以像这样访问元素:

    >>> for r in rows_by_date['07/01/2012']:
    ...     print(r)
    ...
    {'date': '07/01/2012', 'address': '5412 N CLARK'}
    {'date': '07/01/2012', 'address': '4801 N BROADWAY'}

如果采取这个方法, 就不需要事先排序了, 而且运行速度更快一些.



# 过滤序列中的元素

### 问题

需要按照一定的规则提取或合并序列中的元素.

### 方案

最简单的办法是使用列表推导:

    >>> mylist = [1, 4, -5, 10, -7, 2, 3, -1]
    >>> [n for n in mylist if n > 0]
    [1, 4, 10, 2, 3]
    >>> [n for n in mylist if n < 0]
    [-5, -7, -1]

缺点是当输入数据很大时, 使用列表推导也会返回巨大的结果. 这时可以换用生成器表达式, 它返回一个可迭代的对象:

    >>> pos = (n for n in mylist if n > 0)
    >>> pos
    <generator object <genexpr> at 0x1006a0eb0>
    >>> for x in pos:
    ...     print(x)
    ...
    1
    4
    10
    2
    3

有时候过滤规则含有异常处理或者复杂的实现细节, 没法以简单的方式表达, 这时候可以把它写成函数, 然后用内建的 `filter()` 函数调用:

    values = ['1', '2', '-3', '-', '4', 'N/A', '5']

    def is_int(val):
        try:
            x = int(val)
            return True
        except ValueError:
            return False

    ivals = list(filter(is_int, values))
    print(ivals)
    # => ['1', '2', '-3', '4', '5']

`filter()` 返回的是可迭代对象, 如果需要列表的话, 需要再对其应用 `list()`.

### 讨论

列表推导和生成器表达式通常是最简单直观的过滤数据的方式. 过滤的同时还能顺便转换数据:

    >>> mylist = [1, 4, -5, 10, -7, 2, 3, -1]
    >>> import math
    >>> [math.sqrt(n) for n in mylist if n > 0]
    [1.0, 2.0, 3.1622776601683795, 1.4142135623730951, 1.7320508075688772]

列表推导的一种变化是替换不合规则的元素, 而非丢弃它们. 比如之前那个过滤出正数的例子, 现在想要留下负数并替换成 0, 使用条件表达式可以做到这一点:

    >>> clip_neg = [n if n > 0 else 0 for n in mylist]
    >>> clip_neg
    [1, 4, 0, 10, 0, 2, 3, 0]
    >>> clip_pos = [n if n < 0 else 0 for n in mylist]
    >>> clip_pos
    [0, 0, -5, 0, -7, 0, 0, -1]

另一个值得一提的过滤工具是 `itertools.compress()`, 它接受迭代器和一个 bool 值的序列作为参数, 返回对应位置为 True 的部分, 当主序列有个配套的 bool 值序列, 据此来进行过滤时就会用到这个方法:

    addresses = [
        '5412 N CLARK',
        '5148 N CLARK',
        '5800 E 58TH',
        '2122 N CLARK'
        '5645 N RAVENSWOOD',
        '1060 W ADDISON',
        '4801 N BROADWAY',
        '1039 W GRANVILLE',
    ]

    counts = [ 0, 3, 10, 4, 1, 7, 6, 1]

比如现在需要过滤出 addresses 中对应 counts 大于 5 的部分:

    >>> from itertools import compress
    >>> more5 = [n > 5 for n in counts]
    >>> more5
    [False, False, True, False, False, True, True, False]
    >>> list(compress(addresses, more5))
    ['5800 E 58TH', '4801 N BROADWAY', '1039 W GRANVILLE']

这里的关键是先做出对应的仅含有 bool 值的序列, 然后就可以使用 `compress()` 了.

与 `filter()` 类似, `compress()` 也是返回可迭代对象, 需要使用 `list()` 方法转换成列表.



# 使用字典推导提取字典的子集

### 问题

需要得到一个字典对象的子集.

### 方案

最简单的方式是用字典推导:

    prices = {
       'ACME': 45.23,
       'AAPL': 612.78,
       'IBM': 205.55,
       'HPQ': 37.20,
       'FB': 10.75
    }

    # Make a dictionary of all prices over 200
    p1 = { key:value for key, value in prices.items() if value > 200 }

    # Make a dictionary of tech stocks
    tech_names = { 'AAPL', 'IBM', 'HPQ', 'MSFT' }
    p2 = { key:value for key, value in prices.items() if key in tech_names }

### 讨论

也可以先建立键/值的元组, 然后应用 dict() 方法:

    p1 = dict((key, value) for key, value in prices.items() if value > 200)

但是字典推导更简明, 且更快一些.

还有更多的写法, 比如下面这种, 但这个方法是最慢的:

    # Make a dictionary of tech stocks
    tech_names = { 'AAPL', 'IBM', 'HPQ', 'MSFT' }
    p2 = { key:prices[key] for key in prices.keys() & tech_names }




# 使用 namedtuple 实现命名元组

### 问题

使用下标访问列表或者元组中的元素, 但这样的代码令人费解, 很难维护. 希望以名称而非下标来访问元素, 以获得更多的独立性.

### 方案

`collections.namedtuple()` 为 tuple 增加了命名功能. 这是个工厂方法, 接受类型名称和字段名称作为参数, 返回标准元组的子类:

    >>> from collections import namedtuple
    >>> Subscriber = namedtuple('Subscriber', ['addr', 'joined'])
    >>> sub = Subscriber('jonesy@example.com', '2012-10-19')
    >>> sub
    Subscriber(addr='jonesy@example.com', joined='2012-10-19')
    >>> sub.addr
    'jonesy@example.com'
    >>> sub.joined
    '2012-10-19'

虽然 namedtuple 像是普通的类的实例, 但其实它支持所有的元组方法, 如索引, 解包等:

    >>> len(sub)
    2
    >>> addr, joined = sub
    >>> addr
    'jonesy@example.com'
    >>> joined
    '2012-10-19'

命名元组的关键好处是解除对"位置"的依赖. 比如从数据库取出了一大堆元组或者列表对象, 然后通过位置索引访问其中的数据, 那么当数据库有变化时(比如新增了字段)就可能出现问题. 使用 namedtuple 则可以避免这些. 下面是使用普通的元组的例子:

    def compute_cost(records):
        total = 0.0
        for rec in records:
            total += rec[1] * rec[2]
        return total

根据位置来引用元素会降低代码的表达能力, 且更加依赖数据的结构, 现在改为用 namedtuple 的版本:

    from collections import namedtuple
    Stock = namedtuple('Stock', ['name', 'shares', 'price'])
    def compute_cost(records):
        total = 0.0
        for rec in records:
            s = Stock(*rec)
            total += s.shares * s.price
        return total

### 讨论

namedtuple 另一个用法是代替字典, 这样可以减少存储空间. 对于大量的数据, 使用 namedtuple 的效率是很高的. 但需要注意 namedtuple 是不可变对象:

    >>> s = Stock('ACME', 100, 123.45)
    >>> s
    Stock(name='ACME', shares=100, price=123.45)
    >>> s.shares = 75
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: can't set attribute

如果必须改变字段的值, 有个 `_replace()` 方法可以局部更新一些参数, 然后返回新的 namedtuple 对象.

    >>> s = s._replace(shares=75)
    >>> s
    Stock(name='ACME', shares=75, price=123.45)

`_replace()` 一个隐含的用法是先做出带有可选参数或缺失参数的"原型" namedtuple, 之后再提供新的值来更新它:

    from collections import namedtuple

    Stock = namedtuple('Stock', ['name', 'shares', 'price', 'date', 'time'])
    # Create a prototype instance
    stock_prototype = Stock('', 0, 0.0, None, None)
    # Function to convert a dictionary to a Stock
    def dict_to_stock(s):
        return stock_prototype._replace(**s)

有了原型之后, 像这样使用:

    >>> a = {'name': 'ACME', 'shares': 100, 'price': 123.45}
    >>> dict_to_stock(a)
    Stock(name='ACME', shares=100, price=123.45, date=None, time=None)
    >>> b = {'name': 'ACME', 'shares': 100, 'price': 123.45, 'date': '12/17/2012'}
    >>> dict_to_stock(b)
    Stock(name='ACME', shares=100, price=123.45, date='12/17/2012', time=None)

最后要注意, 如果需要修改大量实例属性, namedtuple 并不适合这一用途. 应该用指定了 `__slots__` 的类的实例, 见 [在建立大量实例时节省内存](http://chimera.labs.oreilly.com/books/1230000000393/ch08.html#class_with_slots).



# 对一组数据同时完成映射 map 和化简 reduce

### 问题

需要对数据应用化简函数, 如 sum(), min(), max(), 但之前还需要对其转换, 过滤处理.

### 方案

优雅的方式是使用生成器表达式, 比如要计算平方和:

    nums = [1, 2, 3, 4, 5]
    s = sum(x * x for x in nums)

另一些例子:

    # Determine if any .py files exist in a directory
    import os
    files = os.listdir('dirname')
    if any(name.endswith('.py') for name in files):
        print('There be python!')
    else:
        print('Sorry, no python.')

    # Output a tuple as CSV
    s = ('ACME', 50, 123.45)
    print(','.join(str(x) for x in s))

    # Data reduction across fields of a data structure
    portfolio = [
       {'name':'GOOG', 'shares': 50},
       {'name':'YHOO', 'shares': 75},
       {'name':'AOL', 'shares': 20},
       {'name':'SCOX', 'shares': 65}
    ]
    min_shares = min(s['shares'] for s in portfolio)

### 讨论

这里体现了生成器表达式的一个隐含语法, 即当其是函数的唯一参数时, 可以不写额外的括号, 下面的两个例子都是正确的写法:

    s = sum((x * x for x in nums))    # Pass generator-expr as argument
    s = sum(x * x for x in nums)      # More elegant syntax

用生成器作为参数通常是更有效率和优雅的写法. 如果不使用这种语法, 那就需要建一个临时的列表:

    nums = [1, 2, 3, 4, 5]
    s = sum([x * x for x in nums])

这么写也没问题, 但是建立的这个临时列表其实很没有必要, 用一次就废弃了. 如果数据量很大, 则要消耗大量内存.

此外, 既需要处理或者过滤元素, 又要化简元素的情况下, 返回更全面的信息通常是更好的选择. 像 min() max() 这类化简方法都接受可选的 key 参数用来排序, 此时提供 key=callable 的方式比直接计算 map 元素的值更适用. 比如之前的 portfolio 例子, 第二种写法更方便使用.

    # Original: Returns 20
    min_shares = min(s['shares'] for s in portfolio)

    # Alternative: Returns {'name': 'AOL', 'shares': 20}
    min_shares = min(portfolio, key=lambda s: s['shares'])




# 使用 ChainMap 合并多个映射

### 问题

有许多 dict 或是其他的映射, 需要把它们合并为单一的映射, 并且支持常规的索引方法, 以及检查 key 是否存在.

### 方案

比如这样两个字典:

    a = {'x': 1, 'z': 3 }
    b = {'y': 2, 'z': 4 }

现在需要先在 a 里查找某个键, 不存在的话再去 b 中查找. 使用 `collections.ChainMap` 类可以做到这一点:

    from collections import ChainMap
    c = ChainMap(a,b)
    print(c['x'])      # Outputs 1  (from a)
    print(c['y'])      # Outputs 2  (from b)
    print(c['z'])      # Outputs 3  (from a)

### 讨论

ChainMap 将多个映射体现为单独一个映射的行为, 同时原始的映射并未被合并在一起. ChainMap 把这些映射存储在内部的列表里, 并对外实现了常规的字典方法:

    >>> len(c)
    3
    >>> list(c.keys())
    ['x', 'y', 'z']
    >>> list(c.values())
    [1, 2, 3]

键重复时, 靠前的映射的值会优先使用, 在本例中 `c['z']` 引用的是 a 中的值, 不是 b 中的. 另外, 会改变映射的方法(写入, 删除方法)只影响内部列表中的第一个映射:

    >>> c['z'] = 10
    >>> c['w'] = 40
    >>> del c['x']
    >>> a
    {'w': 40, 'z': 10}
    >>> del c['y']
    Traceback (most recent call last):
    ...
    KeyError: "Key not found in the first mapping: 'y'"

ChainMap 的行为有点像"作用域", 在当前层找不到所需的键, 就去上一层再去找. 在需要模拟类似"作用域"的场合里尤其有用(比如 Python 的 `locals()` `globals()` 等等):

    >>> values = ChainMap()
    >>> values['x'] = 1
    >>> # Add a new mapping
    >>> values = values.new_child()
    >>> values['x'] = 2
    >>> # Add a new mapping
    >>> values = values.new_child()
    >>> values['x'] = 3
    >>> values
    ChainMap({'x': 3}, {'x': 2}, {'x': 1})
    >>> values['x']
    3
    >>> # Discard last mapping
    >>> values = values.parents
    >>> values['x']
    2
    >>> # Discard last mapping
    >>> values = values.parents
    >>> values['x']
    1
    >>> values
    ChainMap({'x': 1})


作为对 ChainMap 的替代, 使用字典的 `update()` 方法组合多个字典也可大致实现这种功能.

    >>> a = {'x': 1, 'z': 3 }
    >>> b = {'y': 2, 'z': 4 }
    >>> merged = dict(b)
    >>> merged.update(a)
    >>> merged['x']
    1
    >>> merged['y']
    2
    >>> merged['z']
    3

但是这种办法有个缺点: 需要生成新字典, 或者破坏原先的字典数据. 另外, 当原先字典更新后, 合并后的对象没法同步其变化:

    >>> a['x'] = 13
    >>> merged['x']
    1

而 ChainMap 内部是直接引用的原先那些映射, 所以不会遇到这种问题:

    >>> a = {'x': 1, 'z': 3 }
    >>> b = {'y': 2, 'z': 4 }
    >>> merged = ChainMap(a, b)
    >>> merged['x']
    1
    >>> a['x'] = 42
    >>> merged['x']   # Notice change to merged dicts
    42






