---
layout: post
title: Python Cookbook II 学习笔记 - 类和对象
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第06章：类和对象。介绍了面向对象编程中的高级技术，自动托管，对特殊方法的托管，简单的设计模式如单例模式，Null 模式等，以及怎样更方便的初始化实例参数。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>



Python Cookbook II 的学习笔记，第06章：类和对象。介绍了面向对象编程中的高级技术，自动托管，对特殊方法的托管，简单的设计模式如单例模式，Null 模式等，以及怎样更方便的初始化实例参数。

# 托管

子类的方法常常会覆盖从超类获得的方法，但有时它也需要在自己的操作中调用超类的方法。只需将这个方法作为类的属性来获取，并传入实例作为它的第一个参数即可：

    class OneMore(Behave):
      def repeat(self, N):
        Behave.repeat(self, N+1)

    zealant = OneMore("Worker Bee")
    zealant.repeat(3)

OneMore 类实现了它自己的 repeat 方法，方法名和超类 Behave 的方法完全一样，但行为略有不同。这种方式叫托管。托管意味着让原有的代码片段完成大部分工作，从而实现某些功能，但常常也带有一些细微的修改。

执行超类托管的首选方式是使用内建的 super:

    class OneMore(Behave):
      def repeat(self, N):
        super(OneMore, self).repeat(N+1)

super 结构等价于使用 `Behave.repeat` 但它也使得 OneMore 类可以被平滑地用于多继承。即使一开始对多继承不感兴趣，也应该养成习惯用 super 而不是显式地通过名字委托给基类。



# 实现一个转换温标的类

在开氏温度 Kelvin、摄氏度 Celsius、华氏温度 Fahrenheit 以及兰金 Rankine 温度之间做转换。

    class Temperature(object):
      coefficients = { 'c': (1.0, 0.0, -273.15), 
                       'f': (1.8, -273.15, 32.0), 
                       'r': (1.8, 0.0, 0.0)
                      }
      def __init__(self, **kwargs):
        # 默认是绝对（开氏）温度 0 但可接受一个命名的参数
        # 名字可以是 k、c、f 或 r 分别对应不同的溫标
        try:
          name, value = kwargs.popitem()
        except KeyError:
          # 无参数默认k=0
          name, value = 'k', 0
        # 若参数过多或参数不能识别，报错
        if kwargs or name not in 'kcfr':
          kwargs[name] = value # 将其置回，做诊断用
          raise TypeError('invalid arguments %r' % kwargs)
        setattr(self, name, float(value))

      def __getattr__(self, name):
        # 将 c、f、r 的获取映射到k的计算
        try:
          eq = self.coefficients[name]
        except KeyError:
          # 未知名字，提示错误
          raise AttributeError(name)
        return (self.k + eq[1]) * eq[0] + eq[2]

      def __setattr__(self, name, value):
        # 将 c, f, r 的设置映射到对 k 的设置；并禁止其他的选项
        if name in self.coefficients:
          # 名字是 c、f 或 r 计算并设置 k
          eq = self.coefficients[name]
          self.k = (value - eq[2]) / eq[0]- eq[1]
        elif name == 'k':
          # 名字是 k,设置之
          object.__setattr__(self, name, value)
        else:
          # 未知名，给出错误信息
          raise AttributeError(name)

      def __str__(self):
        # 以易读和简洁的格式打印
        return "%s K" % self.k
      def __repr__(self):
        # 以详细和准确的格式打印
        return "Temperature (k=%r)" % self.k

    # from te import Temperature
    t = Temperature(f=70)
    print(t.c) # => 21.111111111111086
    t.c = 23
    print(t.f) # => 73.4

在本例中使用 `__getattr__` 比使用命名的属性 named properties 更好，因为针对每种属性的计算模式都差不多（除了作为参考的"k"），我们只需要使用在类中指定的字典 `self.coefficients` 并采用不同的折算系数即可。

很重要的一点是，每当我们对属性进行设置时 `__setattr__` 就会被调用，因此对那些需要被记录在实例（本例中的 `__setattr__` 的实现将属性 k 的工作委托出去了）中的属性，必须把 `__setattr__` 委托给 object 来完成属性设置，而且无法设置的属性也必须抛出 AttributeError 异常。

另一方面，只有在获取某个不能以其他"普通"方式(比如此例中访问属性 k 时 `__getattr__` 并未被调用，属性 k 被记录在实例中，因此可以以普通的方式获取)获取的属性时， `__getattr__` 才会被调用。当遇到无法访问的属性时， `__getattr__` 也必须抛出 AttributeError 异常。



# 定义常量

需要定义一些模块级别的变量，而且客户代码无法将其重新绑定。将下列代码存为一个模块 const.py 并放在 sys.path 指定的目录中：

    class _const(object):
      class ConstError(TypeError):
        pass
      def __setattr__(self, name, value):
        if name in self.__dict__:
          raise self.ConstError("Can't rebind const (%s)" % name)
        self.__dict__[name] = value
      def __delattr__(self, name):
        if name in self.__dict__:
          raise self.ConstError("Can't unbind const (%s)" % name)
        raise NameError(name)
    import sys
    sys.modules[__name__]= _const()
    # file save to const.py

现在任何客户代码都可以导入 const 并将 const 模块的属性绑定一次，但仅能绑定一次：

    # user file
    import const
    const.magic = 23
    # 一旦某属性已经被绑定，程序无法将其重新绑定或者解除绑定：
    const.magic = 88  # 抛出 const.ConstError
    del const.magic   # 拋出 const.ConstError



# 链式字典查询

想在一些映射(通常是dict)中以一种链式的方式进行査询：首先在第一个映射中査找，如果键不在其中，尝试第二个，以此类推。准确地说，需要一个单一的映射，这个映射跨越了许多"虚拟融合"的映射对象，而且可以在其中以指定的优先级顺序进行查询，这样的结构可以在使用上带来很多方便。

映射是字典的概括和抽象的说法，映射提供的接口非常类似字典，但其实现却可能完全不同。所有的字典都是映射，但反过来不成立。在这个需求中，需要实现的是一个映射，在内部这个映射可以将任务按顺序委托给其他映射。用一个类来封装这个功能是很好的选择：

    class Chainmap(object):
      def __init__ (self, *mappings):
        # 记录映射的序列
        self._mappings = mappings
      def __getitem__(self, key):
        # 在序列的字典中查询
        for mapping in self._mappings:
          try:
            return mapping[key]
          except KeyError:
            pass
          # 没有在任何字典中找到 key，所以拋出 KeyError 异常
        raise KeyError(key)
      def get(self, key, default=None):
        # 若 self[key] 存在则返回之，否则返回default
        try:
          return self[key]
        except KeyError:
          return default
      def __contains__(self, key):
        # 若 key 在 self 中返回 True, 否则返回 False
        try:
          self[key]
          return True
        except KeyError:
          return False

    c = Chainmap( {1:1, 2:2}, {4: 4})
    print(c[1])
    print(c[4])
    print(c[5])   # => KeyError: 5
    print(4 in c) # => True
    print(5 in c) # => False

除了很明显的只读限制，Chainmap 类还有其他的限制，它不是一个"完全映射"。可以通过继承标准库 UserDict 模块的 DictMixin 类并提供少量的关键方法将一个部分映射改变成"完全映射"。

    import UserDict
    class FullChainmap(Chainmap, UserDict.DictMixin):
      def copy(self):
        return self.__class__(self._mappings)
      def __iter__(self):
        seen = set()
        for mapping in self._mappings:
          for key in mapping:
            if key not in seen:
              yield key
              seen.add(key)
      iterkeys = __iter__
      def keys(self):
        return list(self)

除了 Chainmap 对映射的要求，FullChainmap 类又对封装的映射增加了一项要求：映射必须是可迭代的。

同时我们也注意到，如果从 DictMixin 继承，Chainmap 中的 get 和 `__contains__` 的实现就变得多余了(虽然是无害的)，因为 DictMixin 已经实现了这两个比较底层的方法（当然还有其他方法)。



# 继承的替代方案：自动托管

你需要从某个类或者类型继承，但也要对继承做一些调整。比如要选择性地隐藏某些基类的方法，而继承并不能做到这一点。

继承是很方便的，但它无法隐藏基类的方法或者属性。而自动托管技术则提供了一种很好的选择。假设需要把一些对象封起来变成只读对象，从而避免意外修改的情况，那么除了禁止属性设置的功能，还需要隐藏修改属性的方法。下面给出一个办法：

    class ROError(AttributeError):
      pass
    class Readonly:
      # Readonly 没有继承 object，
      # 因为这种基于 __getattr__ 的方式也可用于特殊方法，
      # 但仅对旧风格类的实例有效。
      # 在新对象模型中 Python 操作直接通过类的特殊方法来进行，而不是实例的。
      # 本方案让 Readonly 类成为旧风格类，从而避开这个问题，
      mutators = {
        list: set('''__delitem__ __delslice__ __iadd__ __imul__
                  __setitem__ __setslice__ append extend insert
                  pop remove sort'''.split()),
        dict: set('''__delitem__ __setitem___ clear pop popitem
                  setdefault update'''.split()),
      }
      def __init__(self, o):
        object.__setattr__(self, '_o', o)
        object.__setattr__(self, '_no', self.mutators.get(type(o), ()))
      def __setattr__(self, n, v):
        raise ROError("Can't set attr %r on RO object" % n)
      def __delattr__(self, n):
        raise ROError("Can't del attr %r from RO object" % n)
      def __getattr__(self, n):
        if n in self._no:
          raise ROError("Can't get attr %r from RO object" % n)
        return getattr(self._o, n)

通过修改 mutators 即 `Readonly.mutators[sometype] = the_mutators` 还可以轻松地增加其他需要处理的类型。

自动托管是强大而通用的，能得到和类继承几乎完全一样的效果，还能隐藏一些名字。这个例子中我们用模拟的子类将对象封装起来，使之变成只读对象。它的性能不如真正的继承，但我们获得了更好的灵活度和更精细的粒度控制。基本的想法是，类的每个实例都含有我们想要封装的类型的实例。每当客户代码试图从类的实例中获取属性时，除非该属性已经在类中被定义了(比如定义在 mutators 字典中)，否则 `__getattr__` 在完成检査之后，会透明地将这个请求转交给被封装的实例。

托管比继承更加灵活，而有时这种灵活是无价的。除了选择性地托管从而高效地实现了某些属性的"隐藏"，对象还可以在不同的时间托管给不同的子对象，或者一次托管给多个子对象，继承无法提供任何能够与之相比的特性。

下面是托管给多个特定子对象的例子，假设有个类，提供各种转发方法：

    class Pricing(object):
      def __init__(self, location, event):
        self.location = location
        self.event = event
      def setlocation(self, location):
        self.location = location
      def getprice(self):
        return self.location.getprice()
      def getquantity(self):
        return self.location.getquantity()
      def getdiscount(self):
        return self.event.getdiscount()
      # and many more such methods

继承很明显不适用，因为 Pricing 实例需要托管给特定的 location 和 event 实例，这些实例在初始化阶段传入而且可能会被修改。自动托管的补救方法是：

    class AutoDelegator(object):
      delegates = ()
      do_not_delegate = ()
      def __getattr__(self, key):
        if key not in self.do_not_delegate:
          for d in self.delegates:
            try:
              return getattr(d, key)
            except AttributeError:
              pass
        raise AttributeError(key)

    class Pricing(AutoDelegator):
      def __init__(self, location, event):
        self.delegates = [location, event]
      def setlocation(self, location):
        self.delegates[0] = location

在此例中没有托管属性的删除和设置，只是托管了属性获取(还有一些非特殊方法)。当然这只有在想要托管的各个对象的方法、属性不会干扰时才会充分有效。如 location 最好不要有个 getdistance 方法，否则它会抢先进行方法的托管，而此方法原本应该由 event 对象来执行的。

如果需要大量托管的类涉及这种问题，它可以简单地定义一些对应的方法，这是因为只有在用别的方式无法找到属性和方法时 `__getattr__` 才会介入。而通过 `do_not_delegate` 属性还可以隐藏托管对象的一些属性和方法，而且它也可以被子类改写。

比如类 Pricing 想要隐藏叫做 setdiscount 的方法，此方法由 event 提供，做一点点修改就可以了：

    class Pricing(AutoDelegator):
      do_not_delegate = ('setdiscount',)



# 在代理中托管特殊方法

在新风格对象模型中 Python 操作其实是在类中査找特殊方法的。现在需要将一些新风格的实例包装到代理类中，此代理可以选择将一些特殊方法委托给内部的被包装对象。

    class Proxy(object):
      """所有代理的基类"""
      def __init__(self, obj):
        super(Proxy, self).__init__()
        self._obj = obj
      def __getattr__(self, attrib):
        return getattr(self._obj, attrib)

    def make_binder(unbound_method):
      def f(self, *a, **k):
        return unbound_method(self._obj, *a, **k)
      f.__name__ = unbound_method.__name__
      return f
    known_proxy_classes = { }

    def proxy(obj, *specials):
      '''能够委托特殊方法的代理的工厂函数'''
      # 我们手上有合适的自定义的类吗？
      obj_cls = obj.__class__
      key = obj_cls, specials
      cls = known_proxy_classes.get(key)
      if cls is None:
        # 我们手上没有合适的类，那就现做一个
        cls = type("%sProxy" % obj_cls.__name__, (Proxy,), { })
        for name in specials:
          name = '__%s__' % name
          unbound_method = getattr(obj_cls, name)
          setattr(cls, name, make_binder(unbound_method))
        # 缓存之以供进一步使用
        known_proxy_classes[key] = cls
      # 实例化并返回需要的代理
      return cls(obj)

在旧风格(经典)对象模型中 `__getattr__` 同样适用于特殊方法，这些方法常常被认为是 Python 操作的一部分。在使用中也需要当心错误地提供一个我们不想提供的特殊方法。

从 object 或任何内建类型派生子类得到的是新风格的类。在新风格对象模型中 Python 操作并不会在运行时査找特殊方法：它们依赖于类对象的"槽(slot)"，这些槽会在类对象被创建或者修改时更新。因此对于代理对象，如果它要把特殊方法托管给被封装的对象，它本身必须属于某个量身定做的类。本例中每个代理都是由工厂函数 proxy 创建的，该函数接受一个封装的对象以及要托管的特殊方法的名字(除去前后的下划线符号)。

把代码存为文件 proxy.py 并放入 sys.path 中，就可以用下面的方法使用它：

    import proxy
    # 只托管 __len__ __iter__, __repr__ 未被托管
    a = proxy.proxy([], 'len', 'iter')
    print(a)           # => <proxy.listProxy object at 0x0113C370>
    print(a.__class__) # => <class 'proxy.liatProxy'>
    print(a._obj)      # => []
    # 所有的非特殊方法都被托管了
    print(a.append)    # => <built-in mathod append of list object at 0x010FlAl0>
    # 由于 __len__ 被托管了，于是 len(a) 像预期那样工作：
    print(len(a)) # => 0
    a.append(23)
    print(len(a)) # => 1
    # 由于 __iter__ 被托管了，for 循环也如同预期那样工作，
    for x in a: print(x) # => 23
    print(list(a))       # => [23]
    print(sum(a))        # => 23
    print(max(a))        # => 23
    # 不过由于 __getitem__ 没有被托管，a 无法进行索引或切片操作：
    print(a.__getitem__) # => <method-wrappar objoct at 0x010FUF0>
    # a[1]
    # => Traceback (most recent call last):
    # =>   File "D:\Darter\test.py", line 81, in <module>
    # =>     a[1]
    # => TypeError: 'listProxy' object does not support indexing

函数 proxy 使用的是以前创建的类的"缓存"，即全局字典 `known_proxy_classes` 它以被封装的对象的类和被托管的特殊方法的名字为键。如果要生成新类 proxy 就调用内建 type 将新类的名字作为参数传入，并在被包装的对象的类名后加了个"Proxy"。

类 Proxy 作为唯一基类，是一个空的类字典（还没加入任何属性)。基类 Proxy 进行初始化处理和对普通属性的查询的托管。然后工厂函数 proxy 循环处理被托管的特殊方法名：对每个名字，它都从被封装对象的类获得未绑定方法，并将其作为属性赋给闭包 make_binder 中的新类。当遇到对这些未绑定方法的调用时 `make_binder` 会提供适合的参数(如被封装对象本身 self._obj)。

一旦完成了新类的准备，proxy 将其存入 `known_proxy_classes` 并以适当的键标记之。最后，无论类是被创建或从 `known_proxy_classes` 中获取的，proxy 都会使用被封装对象作为唯一参数，将其实例化，然后返回最后的代理实例。



# 缓存环

想定义一个固定尺寸的缓存，当它被填满时，新加入的元素会覆盖第一个(最老的)元素。这种数据结构在存储日志和历史信息时非常有用。

当缓存填满时，本解决方案修改了缓存对象，使其从未填满的缓存类变成了填满的缓存类：

    class RingBuffer(object):
      """这是未填满的缓存类"""
      def __init__(self, size_max):
        self.max = size_max
        self.data = []
      class __Full(object):
        """这是填满了的缓存类"""
        def append(self, x):
          """加入新的元素覆盖最旧的元素"""
          self.data[self.cur] = x
          self.cur = (self.cur+1) % self.max
        def tolist(self):
          """以正确的順序返回元素列表"""
          return self.data[self.cur:] + self.data[:self.cur]
      def append(self, x):
        """在缓存末尾增加一个元素"""
        self.data.append(x)
        if len(self.data) == self.max:
          self.cur = 0
          # 永久性地将 self 的类从非满改成满
          self.__class__ = self.__Full
      def tolist(self):
        """返回一个从最旧的到最新的元素的列表"""
        return self.data

    # 用法示例
    x = RingBuffer(5)
    x.append(1); x.append(2); x.append(3); x.append(4);
    print(x.__class__, x.tolist())
    # => <class '__main__.RingBuffer'> [1, 2, 3, 4]
    x.append(5)
    print(x.__class__, x.tolist())
    # => <class '__main__.RingBuffer.__Full'> [1, 2, 3, 4, 5]
    x.append(6)
    print(x.data, x.tolist())
    # => [6, 2, 3, 4, 5] [2, 3, 4, 5, 6]
    x.append(7); x.append(8); x.append(9); x.append(10);
    print(x.data, x.tolist())
    # => [6, 7, 8, 9, 10] [6, 7, 8, 9, 10]

值得注意的是，这些对象在它们的生命周期中会经历某种不可逆转的状态转变：从未填满的缓存变成填满的缓存，此时开始它的行为方式也发生了变化，我们通过修改 `self.__class__` 来完成这个转变。和其他语言不同，虽然 __Full 类的实现位于 RingBuffer 类的内部，但两者并没有什么特别的联系。

相比于其他随意的、无法逆转的、零散的修改方式，修改类实例是一种很好的选择。而且这样的修改对于所有的类都是可行的。

另一种选择是切换实例的两个方法而非整个类，来使它变成填满的状态：

    class RingBuffer(object):
      def __init__(self, size_max):
        self.max = size_max
        self.data = []
      def _full_append(self, x):
        self.data[self.cur] = x
        self.cur = (self.cur+1) % self.max
      def _full_get(self):
        return self.data[self.cur:] + self.data[:self.cur]
      def append(self, x):
        self.data.append(x)
        if len(self.data) == self.max:
          self.cur = 0
          self.append = self._full_append
          self.tolist = self._full_get
      def tolist(self):
        return self.data

这种切换方式本质上等价于类切换方式，尽管具体机制不同。当需要成组地切换所有的方法时，类切换的方式可能是最佳的，而方法切换则在需要更细的行为粒度控制的时候更加合适。当需要在新风格类中切换某些特殊方法时，类切换是唯一的办法，因为新风格类固有的特殊方法査询是针对类进行的，而不是针对实例的。



# 单例模式

你想保证某个类从始至终最多只能有一个实例。

    class Singleton(object):
      """一个 Python 风格的单例模式"""
      def __new__(cls, *args, **kwargs):
        if '_inst' not in vars(cls):
          # print(super(Singleton, cls))
          cls._inst = super(Singleton, cls).__new__(cls)
          # cls._inst = super(Singleton, cls).__new__(cls, *args, **kwargs)
        return cls._inst

然后只需从 Singleton 派生子类即可，而且不要重载 `__new__` 。然后所有对此类的调用(通常是创建新实例)都将返回同一个实例。实例只会被创建一次。当第一次调用 Singleton 的子类时，唯一的实例就会被创建出来。

我们可以加上常见的自我测试部分来完成整个模块，并展示其行为方式：

    class SingleSpam(Singleton):
      def __init__(self, s):
        self.s = s
      def __str__(self):
        return self.s

    s1 = SingleSpam('spam')
    print(id(s1), s1) # => 14039856 spam
    s2 = SingleSpam('eggs')
    print(id(s2), s2) # => 14039856 eggs
    print(s1 is s2)   # => True
    print(id(s1), s1) # => 14039856 eggs



# Null 对象设计模式的实现

你想减少代码中的条件声明，尤其是针对特殊情况的检査。

一种常见的代表"这里什么也没有"的占位符是 None，但还可以定义一个类，其行为方式和这种占位符相似，而且效果更好：

    class Null(object):
      """Null 对象总是很可靠地什么也不做"""
      # 可选的优化：确保毎个子类只有一个实例，完全是为节省内存，功能没有任何差异
      def __new__(cls, *args, **kwargs):
        if '_inst' not in vars(cls):
          cls._inst = object.__new__(cls, *args, **kwargs)
        return cls._inst
      def __init__(self, *args, **kwargs): pass
      def __call__(self, *args, **kwargs): return self
      def __repr__(self): return "Null()"
      def __nonzero__(self): return False
      def __getattr__(self, name): return self
      def __setattr__(self, name, value): return self
      def __delattr__(self, name): return self

用 Null 类的实例而不是原生的 None 作为占位符，可以在你的代码中避免很多条件声明，而且能够用极少的特殊值检查来实现算法。本节解决方案是 Null 对象设计模式的一个实现。本节中的 Null 类忽略了所有在构建时和调用实例时传入的参数，也忽略了所有的设置和删除属性的操作。任何调用或者访问操作都返回同一个 Null 实例。

比如，假如有这样的一个计算：

    def compute(x, y):
      try:
        # lots of computation here to return some appropriate object
      except SomeError:
        return None

而你这样用它：

    for x in xs:
      for y in ys:
        obj = compute(x, y)
        if obj is not None:
          obj.somemethod(y, x)

可以将计算修改为：

    def compute(x, y):
      try:
        # lots of computation here to return some appropriate object
      except SomeError:
        return Null()

对它的用法可以简化为：

    for x in xs:
      for y in ys:
        compute(x, y).somemethod(y, x)

其要点是无须检查 compute 究竟是返回了真实的结果还是 Null 的实例：即使是后者，也可以安全无害地调用它的任何方法。

另一个更具体的使用例子：

    log = err = Null()
    if verbose:
      log = open('/tmp/log', 'w')
      err = open('/tmp/err', 'w')
    log.write('blabla')
    err.write('blabla error')

如"if verbose:"这样的防护性代码，总是散布于各处。有了 Null 后，现在可以直接调用：

    log.write('bla')

无须使用以往的表达方式：

    if log is not None: log.write('bla')

在新对象模型中，对于执行某些操作所需要的特殊方法，Python 并不会对实例调用 `__getattr__` (而是检査该实例的类的槽)，需要谨慎地定制 Null 类来满足你的应用对空对象操作的需求，因此需要仔细设计空对象类的特殊方法，不管它们是直接嵌入基类的代码中的，还是以子类化的方式扩展的。

举个例子，对于 Null 你无法索引实例，也不能取得它的长度，还无法对它迭代。可以根据需要增加特殊方法（在 Null 中直接增加或从 Null 派生并扩展）并提供适当的实现：

    class SeqNull(Null):
      def __len__(self): return 0
      def __iter__(self): return iter(())
      def __getitem__(self, i): return self
      def __delitem__(self, i): return self
      def __setitem__(self, i, v) : return self

Null 对象的关键设计目标是为原始的 None 提供一种智能的替代物。这种"这里什么也没有"的标记或占位符可以用于多种情况，比如其中一种情况是，一组元素除了其一比较特殊，其他都是类似的。这经常导致到处都是条件声明，而条件声明的目的常常只为将普通元素和 None 区分开来，Null 对象则能够帮助你避免过多的条件声明。

Null 的一个很大的缺点是它可能会隐藏一些错误：如果一个函数返回 None 而调用者并不期望这样的返回值，调用者极有可能会接着对 None 调用一个它并不支持的方法或者操作，从而引发提示异常并回溯。如果返回值是并不期望获得的 Null 则问题可能会被掩盖很长一段时间，当最终异常和回溯发生时，重新定位到代码最初的纰漏会更加困难。

这个问题严重到影响实用性了吗？答案是，这仍然是个人选择。如果你的代码在开发中总会有一些适当的单元测试，那么这个问题可能不会发生，而如果你根本就没有单元测试，使用 Null 带来的问题通常也只会是最小的问题。



# 用 init 参数自动初始化实例变量

你想避免编写和维护一种烦人的几乎什么也不做的 `__init__` 方法，这种方法中含有一大堆形如 `self.something = something` 的赋值语句。

可以把那些属性赋值任务抽取出来置入一个辅助函数中：

    def attributesFromDict(d):
      self = d.pop('self')
      for n, v in d.items():
        setattr(self, n, v)

而 `__init__` 方法里的那种千篇一律的赋值语句大概是这个样子的：

    def __init__(self, foo, bar, baz, boom=1, bang=2):
      self.foo = foo
      self.bar = bar
      self.baz = baz
      self.boom = boom
      self.bang = bang

现在可以被缩减为一行：

    def __init__(self, foo, bar, baz, boom=1, bang=2):
      attributesFromDict(locals())

attributesFromDict 并不适用于使用了更多代码尤其是使用了本地变量的 `__init__`，因为对于传递给它的唯一的字典参数，它无法区分是传给 `__init__` 的参数还是 `__init__` 的局部变量。如果在辅助函数中加一点内省的能力，这个限制就可以打破：

    def attributesFromArguments(d):
      self = d.pop('self')
      codeObject = self.__init__.__func__.__code__
      argumentNames = codeObject.co_varnames[1:codeObject.co_argcount]
      for n in argumentNames:
        setattr(self, n, d[n])

通过获取 `__init__` 方法的代码对象，attributesFromArguments 函数能够只处理传递给 `__init__` 的参数名。

attributesFromArguments 最关键的限制是它不支持 `__init__` 的特殊参数，即 *args 和 **kw。

可以通过引入更多的内省来获得处理 **kw 的能力，但那就要求使用更多的黑魔法和引入更多的复杂性，比起获得的这点功能似乎有些不值得。

遇到有 **kw 的情况，在 attributesFromArguments() 之后，可以再手动设置 kw 的属性：

    class AutoInitAttr(object):
      """docstring for AutoInitAttr"""
      def __init__(self, arg1, arg2, arg3, **kw):
        super(AutoInitAttr, self).__init__()
        attributesFromArguments(locals())
        for k, v in kw.items():
          setattr(self, k, v)
    c = AutoInitAttr(1, 2, arg3=3, key1='key1', key2='key2')
    print(c.arg1)
    print(c.arg2)
    print(c.arg3)
    print(c.key1)
    print(c.key2)






