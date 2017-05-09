---
layout: post
title: Python Cookbook II 学习笔记 - 描述符，装饰器，元类
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第20章：描述符，装饰器，元类。介绍了在面向对象编程时管理实例属性的一些高级方式，如缓存属性值，对属性起别名，在首次调用时自动初始化属性等。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>




# 描述符的基本概念

描述符的概念非常简单。每当一个属性被査询时，一个动作就会发生。这个动作默认是 get, set 或 delete。

不过，有时可能会有一些需求，需要你设计一些更复杂的动作。每当某属性被访问时，也许想创建一个日志记录，也可以将査询方法重定向到其他方法。最好的解决方案就是编写一个执行符合需要的动作的函数，然后指定它在属性被访问时运行。一个具有这种功能的对象被称为描述符。

虽然描述符的概念很简单淸楚，但它的应用范围似乎并没有什么限制。描述符是方法、被绑定方法、super, property, classmethod 和 staticmethod 的实现基础。学会描述符各种不同的应用是掌握这门语言的关键。



# 装饰器的基本概念

装饰器比描述符更简单。`myfunc = wrapper(myfunc)` 的写法是很常见的修改或者记录其他函数的方法，这种写法总是发生在 `myfunc` 被定义之后的某处。


它们的应用没有什么限制。常见的例子包括 @staticmethod 和 @classmethod。这种思路在很多方面得到了进一步发展，比如:

- 用于字节码优化的 @make_constants
- 可注册一个在退出之前运行的函数的 @atexit
- 自动给函数或方法增加互斥锁的 @synchronized
- 以及每当函数被调用时创建一个日志记录的 @log



# 元类的基本概念

元类的概念听上去很陌生，但是它是我们很熟悉的东西。无论何时当你编写一个类定义时，某种机制就会使用名字、基类以及类字典来创建一个类对象。对于旧风格的类，这个机制是 `types.ClassType`。对于新风格类，这个机制是 `type`。

前一种机制实现了人们熟悉的经典类的行为方式，包括属性査询以及当 `repr` 被调用时显示类的名字的行为。后一种机制则增加了一些装饰，包括了对 `__slots__` 和 `__getattribute__` 的支持。

如果这个机制本身是可以编程定制的，你就能够在 Python 中为所欲为。事实上就是这样，而且它还有个吓人的名字：元类。

本章的内容说明编写元类可以是很简单的事。很多元类子类化 `type` 并简单地扩展或重写需要的行为。一些元类甚至更简单，它们只是修改了类字典，然后把参数转发给 `type` 就算完成了工作。

比如，假设你想自动地为槽(slot)中的所有的私有变量生成 `getter` 方法，你只需定义一个元类 M 查询映射中的 `__slots__` 搜寻名字以下划线开头的所有变量，然后为每个变量创建一个访问方法，最后将新方法加入到类字典：

    class M(type):
      def __new__(cls, name, bases, classdict):
        for attr in classdict.get('__slots__', ()):
          if attr.startswith('_'):
            def getter(self, attr=attr):
              return getattr(self, attr)
            getter.__name__ = 'get' + attr[1:]
            classdict['get' + attr[1:]] = getter
        return type.__new__(cls, name, bases, classdict)
    
    class Point1(object, metaclass=M):
      __slots__ = ['_x', '_y']

可以对每一个你想自动生成访问函数的类应用这个新元类：
现在 `print(dir(Point))` 会看到两个访问方法，就好像你曾经写过这些方法一样：

    class Point2(object):
      __slots = ['_x', '_y']
      def getx(self):
        return self._x
      def gety(self):
        return self._y

对于这两个例子，在 print 输出中都能看到名字 'getx' 和 'gety'。

    print(dir(Point1))
    # => ['__class__', '__delattr__', '__dir__', 
    # =>  '__doc__', '__eq__', '__format__', '__ge__',
    # =>  '__getattribute__', '__gt__', '__hash__', 
    # =>  '__init__', '__le__', '__lt__', '__module__',
    # =>  '__ne__', '__new__', '__reduce__', 
    # =>  '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__',
    # =>  '__slots__', '__str__', '__subclasshook__', 
    # =>  '_x', '_y', 'getx', 'gety']
    
    print(dir(Point2))
    # => ['_Point2__slots', '__class__', '__delattr__', 
    # =>  '__dict__', '__dir__', '__doc__', '__eq__',
    # =>  '__format__', '__ge__', '__getattribute__', 
    # =>  '__gt__', '__hash__', '__init__', '__le__', '__lt__',
    # =>  '__module__', '__ne__', '__new__', 
    # =>  '__reduce__', '__reduce_ex__', '__repr__', '__setattr__',
    # =>  '__sizeof__', '__str__', '__subclasshook__', 
    # =>  '__weakref__', 'getx', 'gety']




# 在函数调用中获得常新的默认值

Python 中函数的 `def` 语句执行后会为函数的可选参数计算默认值，但只会做一次。然而，对某些函数，你希望每次函数被调用时，默认值都是重新计算的新鲜值。

    import copy
    def freshdefaults(f):
      '''一个封装 f 的装饰器，可使其默认值在调用时保持常新'''
      fdefaults = f.__defaults__
      def refresher(*args, **kwds):
        f.__defaults__ = copy.deepcopy(fdefaults)
        return f(*args, **kwds)
      refresher.__name__ = f.__name__
      return refresher
    
    def packitem_org(item, pkg=[]):
      pkg.append(item)
      return pkg
    
    @freshdefaults
    def packitem(item, pkg=[]):
      pkg.append(item)
      return pkg
    
    p2a = packitem_org('1')
    print(p2a) # => ['1']
    p2b = packitem_org('2')
    print(p2b) # => ['1', '2']
    
    p2a = packitem('1')
    print(p2a) # => ['1']
    p2b = packitem('2')
    print(p2b) # => ['2']




# 用嵌套函数来编写 property 属性

使用 `@property` 会在类的命名空间里加入许多读写属性的方法。可以写个更好的装饰器完成任务，用不被直接调用的访问方法来编写 `property` 这就可以不弄乱类的名称空间。

    def nested_property(c):
      return property(**c())
    
    import math
    class Rectangle(object):
      def __init__(self, x, y):
        self.x = x
        self.y = y
    
      @nested_property
      def area():
        doc = "Area of the rectangle"
        def fget(self):
          return self.x * self.y
        def fset(self, value):
          ratio = math.sqrt((1.0 * value) / self.area)
          self.x *= ratio
          self.y *= ratio
        return locals()

创建 property 属性的标准惯用法是在类的主体中定义访问方法如 `getter` `setter` 以及 `deleter` 这些方法通常都有相同的命名模式。很常见的情况是，这些函数只在 property 属性中使用，很少程序员们会记得 del 它们，以便在创建 property 实例之后清理类名字空间。

本节推荐的惯用法避免了类名字空间的混乱，只需在类的主体中编写一个函数，名字和你想提供的属性一样。在这个函数中，定义一些适当的嵌套函数，它们的名字必须是 `fget` `fset` 和 `fdel` 并给一个名为 `doc` 的 docstring 赋值。同时我们让外层函数返回一个字典，字典的条目正好就是前面提到的那些名字，而没有其他的东西。返回 `locals()` 字典是可行的，只要你的外层函数没有其他局部变量即可。

如果某个特定的属性禁止了相应操作的话，不用定义全部那四个关键名字，你可以并且应该省略其中的一些。尤其需要指出的是，方案中的 area 函数没定义 `fdel` 因为 area 属性不能也不该被删除。




# 给属性值起别名

有时你的类实例有一些属性的默认值必须和其他属性的当前值一样，而且还需要能够被独立地设置和删除。

    class DefaultAlias(object):
      '''除非明确地赋值，此属性将化名为另一个属性'''
      def __init__(self, name):
        self.name = name
      def __get__(self, inst, cls):
        if inst is None:
          # 类属性访问，返回'self'描述符
          return self
        # print(self, inst, cls)
        return getattr(inst, self.name)
    
    class Alias(DefaultAlias):
      '''此属性无条件地化名为另一个属性'''
      def __set__(self, inst, value):
        setattr(inst, self.name, value)
      def __delete__(self, inst):
        delattr(inst, self.name)
    
    class Book(object):
      def __init__(self, title, shortTitle=None):
        self.title = title
        if shortTitle is not None:
          self.shortTitle = shortTitle
      shortTitle = DefaultAlias('title')
    
    b = Book('The Life and Opinions of Tristram Shandy, Gent.')
    print(b.shortTitle)
    # => The Life and Opinions of Tristram Shandy, Gent.
    b.shortTitle = "Tristram Shandy"
    print(b.shortTitle)
    # => Tristram Shandy
    del b.shortTitle
    print(b.shortTitle)
    # => The Life and Opinions of Tristram Shandy, Gent.

DefaultAlias 并不是大家熟知的数据描述符(data descriptor)类，因为它没有 `__set__` 方法。这意味着当给一个实例的属性赋值时(此属性的名字在类中被定义为一个 DefaultAlias)，实例会正常记录属性，而且实例属性会覆盖类属性。这正是上面代码片段中当明确给 `b.shortTitle` 赋值之后发生的事情。而当我们 `del b.shortTitle` 时，删除了属于实例的属性，所以类属性又再次显现出来。

自定义描述符类 Alias 是 DefaultAlias 的简单变体，从 DefaultAlias 继承而获得。Alias 给一个属性提供了另一个别名属性，不只是允许访问属性的值，还允许设置值和删除。我们通过一个"数据描述符"类(即它有 `__set__` 方法)轻松实现了这个功能。所以，对实例的被定义为 Alias 的属性的任何赋值或删除操作，都会被 Alias 的 `__set__` 方法拦截。

对于已经发布的类，如果想继续开发和扩展它，则 Alias 很适合需求。可以使用更合适的名字来命名方法和其他属性，同时保留旧名字的可用性以便向后兼容。比如想要要在旧名字被使用时输出警告信息：

    import warnings
    class OldAlias(Alias):
      def _warn(self):
        warnings.warn('use %r, not %r' % (self.name, self.oldname),
                      UserWarning, stacklevel=3)
      def __init__(self, name, oldname):
        super(OldAlias, self).__init__(name)
        self.oldname = oldname
      def __get__(self, inst, cls):
        self._warn()
        return super(OldAlias, self).__get__(inst, cls)
      def __set__(self, inst, value):
        self._warn()
        return super(OldAlias, self).__set__(inst, value)
      def __delete__(self, inst):
        self._warn()
        return super(OldAlias, self).__delete__(inst)
    
    class NiceClass(object):
      def __init__(self, name):
        self.nice_new_name = name
      bad_old_name = OldAlias('nice_new_name', 'bad_old_name')

使用这个类的旧代码可能会引用实例中的属性 bad_old_name 但向后兼容性仍然得以保留，这种情况发生时警告信息会被打印出来，告诉用户这种做法不再被推荐，并鼓励旧代码的作者更新代码且使用 nice_new_name。

Python 标准库的 warnings 模块的机制能够保证：在默认情况下，这些瞀告只会在程序每次运行且需要警告的情况下打印一次，而且不会重复提示。

    x = NiceClass(23)
    for y in range(4):
      print(x.bad_old_name)
      x.bad_old_name += 100
    # => 23
    # => test.py:92: UserWarning: use 'nice_new_name', not 'bad_old_name'
    # =>   print(x.bad_old_name)
    # => test.py:93: UserWarning: use 'nice_new_name', not 'bad_old_name'
    # =>   x.bad_old_name += 100
    # => 123
    # => 223
    # => 323

警告信息被打印了两次，而不是像循环一样重复出现。





# 缓存属性值

想根据要求计算实例或类的属性值，并提供自动化的缓存。

    class CachedAttribute(object):
      '''计算属性，并在实例中缓存之'''
      def __init__(self, method, name=None):
        # 记录未綁定方法和名字
        self.method = method
        self.name = name or method.__name__
      def __get__(self, inst, cls):
        if inst is None:
          # 类属性访问，返回 self
          return self
        # 计算、缓存并返回实例属性值
        result = self.method(inst)
        setattr(inst, self.name, result)
        return result
    
    class CachedClassAttribute(CachedAttribute):
      '''计算属性，并在类中缓存之'''
      def __get__(self, inst, cls):
        # 委托给 CachedAttribute 把 cls 当做实例
        return super(CachedClassAttribute, self).__get__(cls, cls)
    
    class MyObject(object):
      def __init__(self, n):
        self.n = n
      @CachedAttribute
      def square(self):
        return self.n * self.n
    
    myobj = MyObject(23)
    print(vars(myobj))  # => {'n': 23}
    print(myobj.square) # => 529
    print(vars(myobj))  # => {'square': 529, 'n': 23}
    del myobj.square
    print(vars(myobj))  # => {'n': 23}
    myobj.n = 42
    print(vars(myobj))  # => {'n': 42}
    print(myobj.square) # => 1764
    print(vars(myobj))  # => {'square': 1764, 'n': 42}

如果 `myobj.n` 的值被修改了，当 `myobj.square` 被再次访问时，如果希望它的值被重新计算，只需 `del myobj.square` 即可。

自定义描述符 `CachedClassAttribute` 是 `CachedAttribute` 的一个简单变体，它对类而不是对实例调用方法，并将缓存的结果放入类中。如果这个类的所有实例需要看到同样的缓存值，那么这种做法极其高效。`CachedClassAttribute` 主要用于无须清除缓存的场合，因为这时 `__get__` 方法通常会清掉实例描述符：

    class MyClass(object):
      class_attr = 23
      @CachedClassAttribute
      def square(cls):
        print('have not cached')
        return cls.class_attr * cls.class_attr
        
    x = MyClass()
    y = MyClass()
    print(x.square)  # => 529
    print(y.square)  # => 529
    del MyClass.square
    print(x.square)  # => AttributeError: 'MyClass' object has no attribute 'square'





# 封装一个方法来给类增加功能

需要给一个现有的类增加功能，同时还不能修改该类的源代码，而且不能使用继承（因为那相当于创建一个新类，而不是修改现有的类)。具体地说，需要增强此类的某个方法，在现有的方法之上增加额外的功能。

给一个现有的 class 对象增加全新的方法或属性是相当简单的事情，因为内建函数 `setattr` 完成了所有的实质工作。我们需要"装饰"一个现有的方法并增加新功能。

可以创建一个闭包作为新的替换函数。最好的办法是定义通用的封装和解封函数，如下：

    import inspect
    def wrapfunc(obj, name, processor, avoid_doublewrap=True):
      '''给 obj.<name> 打补丁，对它的调用实际上是
      对 processor(original_callable, *args, **kwargs) 的调用
      '''
      # 获得可调用对象 obj.<name>
      call = getattr(obj, name)
      # 可选地避免多次同样的封装
      if avoid_doublewrap and getattr(call, 'processor', None) is processor:
        return
      # 获取最根本的函数（如果有的话），定义封装器闭包
      original_callable = getattr(call, '__func__', call)
      def wrappedfunc(*args, **kwargs):
        return processor(original_callable, *args, **kwargs)
      # 设置属性，为将来的解封做准备，同时也避免了多次封装
      wrappedfunc.original = call
      wrappedfunc.processor = processor
      wrappedfunc.__name__ = getattr(call, '__name__', name)
      # 重新封装 staticmethod 和 classmethod (若 obj 是一个类)
      if inspect.isclass(obj):
        if hasattr(call, '__self__'):
          if call.__self__:
            wrappedfunc = classmethod(wrappedfunc)
        else:
          wrappedfunc = staticmethod(wrappedfunc)
      # 最后，根据要求安装封装器闭包
      setattr(obj, name, wrappedfunc)
    
    def unwrapfunc(obj, name):
      '''撤销 wrapfunc(obj, name, processor) 的效果 '''
      setattr(obj, name, getattr(obj, name).original)

封装方式的设计非常细致，因此它既能工作于普通函数(当 obj 是一个模块时)，也能处理各种方法，如果 obj 是一个实例，就处理被绑定方法，如果是一个类，处理非绑定方法、类方法和静态方法。如果 obj 是内建类型就失效了，因为内建类型是无法被改变的。

对于"大尺度封装"来说，很重要的一点是解除封装，并让被封装的方法回到正常的状态，所以本节还提供了 `unwrapfunc` 函数。

同时应该采取措施避免意外地对同一个方法用同样的封装方式处理两次，这就是为什么 `wrapfunc` 要支持一个可选参数 `avoid_doublewrap`，该参数默认值 True 不允许重复封装。

可惜，`classmethod` 和 `staticmethod` 并不支持基于实例的属性，对它们来说，重复封装和重复解封的情况是无法百分之百杜绝的。

可以用不同的处理器(processor)对同一个方法封装多次。解封也必须遵循后进先出的顺序(last in, first out)。本方案不支持从多层封装构成的封装链的"中部"删除某个封装层。

另外，当另一个不相关的封装同时发生时，"两次封装"无法被检测出来。这种现象叫做"深度两次封装"(deep double wrapping)。

举例，假设需要在一个特定方法被调用时打印关于所发生的事的所有信息。

    def tracing_processor(original_callable, *args, **kwargs):
      r_name = getattr(original_callable, '__name__', '<unknown>')
      r_args = list(map(repr, args))
      r_args.extend(['%s=%r' % x for x in kwargs.items()])
      print("begin call to %s (%s)" % (r_name, ", ".join(r_args)))
      try:
        result = original_callable(*args, **kwargs)
      except:
        print("EXCEPTION in call to %s" % (r_name, ))
        raise
      else:
        print("call to %s result: %r" % (r_name, result))
        return result
    
    def add_tracing_prints_to_method(class_object, method_name):
      wrapfunc(class_object, method_name, tracing_processor)

另外，根据本节的 `wrapfunc` 所要求的签名和语义编写了某个处理器，如 `tracing_processor`，就可以配合装饰器以更直接的方式应用之：

    def processedby(processor):
      """ 用 processor 来封装一个函数的装饰器 """
      def processedfunc(func):
        def wrappedfunc(*args, **kwargs):
          return processor(func, *args, **kwargs)
        return wrappedfunc
      return processedfunc
    
    class SomeClass(object):
      @processedby(tracing_processor)
      def amethod(self, s):
        return 'Hello, ' + s
    
    s = SomeClass()
    s.amethod('SomeClass Inst')




# 允许对方法的链式调用

list 类型的可直接改变原对象的方法(如 append 和 sort)返回 None。为了调用一系列这种方法，需要使用一系列语句。现在希望这些方法能够返回 self，以便于用一个单独的表达式来实现链式调用。

自定义元类能够为这个任务提供一种优雅的解决方式：

    def makeChainable(func):
      '''将一个只返回 None 的方法封装为返回 self'''
      def chainableWrapper(self, *args, **kwds):
        func(self, *args, **kwds)
        return self
      chainableWrapper.__name__ = func.__name__
      return chainableWrapper
    
    class MetaChainable(type):
      def __new__(mcl, cName, cBases, cDict):
        ''' 获得 真正 的基类，并将它的可变方法封入 cDict '''
        for base in cBases:
          if not isinstance(base, MetaChainable):
            for mutator in cDict['__mutators__']:
              if mutator not in cDict:
                cDict[mutator] = makeChainable(getattr(base, mutator))
            break
        # 将其余部分委托给内建类型 type
        return super(MetaChainable, mcl).__new__(mcl, cName, cBases, cDict)
    class Chainable(metaclass=MetaChainable):
      pass
    
示例用法

    class chainablelist(Chainable, list):
      __mutators__ = 'sort reverse append extend insert'.split()
    print(''.join(chainablelist('hello').extend('ciao').sort().reverse()))
    # => oolliheca

另一个办法是委托封装器，这个更简单好用一些，但性能较差。

    class chainable(object):
      def __init__(self, obj):
        self.obj = obj
      def __iter__(self):
        return iter(self.obj)
      def __getattr__(self, name):
        def proxy(*args, **kwds):
          result = getattr(self.obj, name)(*args, **kwds)
          if result is None:
            return self
          else:
            return result
        proxy.__name__ = name
        return proxy
    
示例用法

    print(''.join(chainable(list('hello')).extend('ciao').sort().reverse()))
    # => oolliheca





# 实例属性的自动初始化

想在对象初始化时把某些属性设置为常量值，而且不想强制子类调用你写好的 `__init__` 方法。

对于不可改变类型的常量值，可以在类中设置它们。举个例子，不要使用下面这种自然的方式：

    class counter(object):
      def __init__(self):
        self.count = 0
      def increase(self, addend=1):
        self.count += addend
    
而应当这么做：
    
    class counter(object):
      count = 0
      def increase(self, addend=1):
        self.count += addend

这种方式可行的原因是，当 `self.count` 是不改变类型时，
`self.count` 相当于 `self.count = self.count + added` 对于特定的实例 self 这段代码首次执行时，`self.count` 还没有被初始化为实例的属性，因此在等号的右端，使用的是类属性，但不管怎样实例属性仍是被赋值的那个(等号左边)。一旦实例属性以这种方式完成初始化，对任何进一步的使用获取或设置的都是实例属性。

**对可变类型的值如字典和列表，这种方式不可行。使用的后果是，类的所有实例都共享相同的可变类型对象为属性。**

然而，自定义描述符却可以工作得很好：

    class auto_attr(object):
      def __init__(self, name, factory, *a, **k):
        self.data = name, factory, a, k
      def __get__(self, obj, clas=None):
        name, factory, a, k = self.data
        setattr(obj, name, factory(*a, **k))
        return getattr(obj, name)
    
    class recorder(object):
      count = 0
      events = auto_attr('events', list)
      def record(self, event):
        self.count += 1
        self.events.append((self.count, event))

当首次访问指定实例 obj 的名为 name 的属性时，Python 会在obj 的类中发现描述符(auto_attr 的实例)并用 obj 作参数调用描述符的方法 `__get__`。auto_attr 的 `__get__` 再调用 `factory` 并将其结果根据指定的名字设为实例的属性，因此，以后对实例的同名属性的访问会获得实际的值。

换句话说，在首次访问实例后，描述符会隐藏它自己，从以后对同一实例的同名属性的访问中脱身。基于这个目的，将 auto_attr 设计为非数据(nondata)描述符类是绝对关键的，这也意味着它没有 `__set__` 方法。因此在实例中可以设置同名的属性：有优先权的实例属性覆盖了类属性，也就是非数据描述符类的实例。

可以将本节的方案看做是实例属性的 "just in time" 生成。除了允许不借助 `__init__` 方法完成属性的初始化，在优化方面也有一定意义：假设某个类有很多实例，每个实例都有庞大的属性集合，这将浪费很多时间来初始化，可是实际上对于每个给定的实例来说，它的绝大多数属性从未被访问过。




