---
layout: post
title: Python Cookbook II 学习笔记 - 测试技巧
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第08章：测试。如何从调试中得到更详尽的信息，如何做一个自动化的单元测试等。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>



# 在调试模式中跟踪表达式和注释

你正在编写的程序无法使用可交互的、单步执行的调试器。因此，你需要详细记录状态和控制分支的信息来进行高效的调试。

traceback 模块的 extract_stack 函数是解决问题的关键，此函数使得你的调试代码能够很容易地执行运行时内省(runtime introspection)从而找出调用它的代码：

    import sys, traceback
    
    #调用 watch(secretOfUniverse) 打印出如下的信息：
    # => File "trace.py", line 57, in __testTrace
    # =>   secretOfUniverse <int> = 42
    
    def watch(variableName, watchOutput=sys.stdout):
      watch_format = ('File "%(fileName)s", line %(lineNumber)d, in'
                      ' %(methodName)s\n  %(varName)s <%(varType)s>'
                      ' = %(value)s\n\n')
      if __debug__:
        stack = traceback.extract_stack()[-2:][0]
        actualCall = stack[3]
        if actualCall is None:
          actualCall = "watch([unknown])"
        left = actualCall.find('(')
        right = actualCall.rfind(')')
        paramDict = dict( varName=actualCall[left+1:right].strip(),
                          varType=str(type(variableName))[8:-2],
                          value=repr(variableName),
                          methodName=stack[2],
                          lineNumber=stack[1],
                          fileName=stack[0])
        watchOutput.write(watch_format % paramDict)
    
    # 调用 trace("this line was executed") 打印出如下信息：
    # => File "trace.py", lina 64, in ?
    # =>   this line was executed
    
    def trace(text, traceOutput = sys.stdout):
      trace_format = ('File "%(fileName)s", line %(lineNumber)d, in'
                      ' %(methodName)s\n %(text)s\n\n')
      if __debug__:
        stack = traceback.extract_stack()[-2:][0]
        paramDict = dict( text=text,
                          methodName=stack[2],
                          lineNumber=stack[1],
                          fileName=stack[0])
        traceOutput.write(trace_format % paramDict)
    
    

对于很多各种类型的程序来说，要想使用传统的、交互式的单步调试器是很不容易的。可以在程序中加上一堆 print 语句来应对这种无法交互调试的困境，但这种方式很不系统，而且在问题修复之后还要做清理工作。

本节展示的是一种架构良好的可行方式，即提供一些函数使得可以打印出表达式、变量或函数调用的值，同时还带有范围信息、跟踪信息以及通用的注释。

`traceback` 模块的关键就是 `extract_stack` 函数。`traceback.extract_stack` 返回的是4个子项的元组的列表，4个子项分别是文件名、行号、函数名以及调用声明的代码，每个元组都代表着栈中的一次调用记录，列表的`Item[-2]`(倒数第二个子项)是直接调用者的元组，它也正是我们将要输出到类文件对象(绑定到了 traceOutput 和 watchOutput 变量)的信息。

当 `__debug__` 是 False 的时候(以 -O 或 -OO 开关运行解释器的时候)，所有与调试相关的代码都会被自动清除。这种方式不会使得执行码变大，因为编译器知道 `__debug__` 变量的存在，当打开了优化选项，它会清除被 `if __debug__` 保护的代码。

下面是一个使用示例，它把所有的信息打印到了标准输出设备，釆用的测试方式也是常用的自测方式，即将测试用例直接放到模块的末尾：

    def __testTrace():
      secretOfUniverse = 42
      watch(secretOfUniverse)
    
    if __name__ == "__main__":
      a = "something else"
      watch(a)
      __testTrace()
      trace("This line was executed!")
      raw("Just some raw text")
    
    # => File "E:\GitHub\Darter\test.py", line 69, in <module>
    # =>   a <str> = 'something else'
    # =>
    # => File "E:\GitHub\Darter\test.py", line 65, in __testTrace
    # =>   secretOfUniverse <int> = 42
    # =>
    # => File "E:\GitHub\Darter\test.py", line 71, in <module>
    # =>  This line was executed!
    


# 从 traceback 中获得更多信息

当一个未被捕获的异常发生时，你希望能够打印出所有变量的信息。

一个 traceback 对象基本上是一个互相关联的节点的列表，每个节点都指向一个帧对象。而帧对象则反过来根据 traceback 的关联节点列表以相反的顺序建立它们自己的链表，所以，我们可以根据需要向前或向后移动。

本节方案利用了这个结构和帧对象持有的丰富信息，具体地说，我们利用了帧的对应函数的局部变量的字典：

    import sys, traceback
    def print_exc_plus():
      """打印通常的回溯信息，且附有每帧中的局部变量的列表"""
      tb = sys.exc_info()[2]
      while tb.tb_next:
        tb = tb.tb_next
      stack = []
      f = tb.tb_frame
    
      while f:
        stack.append(f)
        f = f.f_back
    
      stack.reverse()
      traceback.print_exc()
      print("Locals by frame, innermost last")
      for frame in stack:
        print()
        print("Frame %s in %s at line %s" % ( frame.f_code.co_name,
                                              frame.f_code.co_filename,
                                              frame.f_lineno))
      for key, value in frame.f_locals.items():
        print("\t%20s = " % key, end='')
        # 我们必须*绝对*避免异常的扩散，
        # 而 str(value) *能够*引发任何异常, 所以*必须*捕获所有异常
        try:
          print(value)
        except:
          print("<ERROR WHILE PRINTING VALUE>")
    
    try:
      "1" + 1
    except Exception as e:
      print_exc_plus()



本节的方案依赖于这样一个事实，即每个 `traceback` 对象都通过 `tb_next` 字段引用栈中的下一个 `traceback` 对象，从而建立了一个链表。每个 `traceback` 对象同时也通过 `tb_frame` 字段指向一个对应的帧，每帧又通过 `f_back` 字段指向前一帧（与 `traceback` 对象的链表方向相反)。

为了简化，本节方案首先在一个叫做 stack 的本地列表中存放指向帧对象的所有引用，
然后循环此列表，输出每一帧的信息。对于每一帧，它首先输出基本信息(比如函数名，文件名、行号等)，然后转向之 f_locals 字段指向的该帧的局部变量的字典。就像通过内建的 locals 和 global 函数创建并返回的字典一样，该字典的每一个键都是一个变量的名字，而对应值则是变量的值。

注意，虽然打印名字是安全的(因为它只是宇符串)，打印对应值却可能会失败，因为它可能会调用用户自定义对象的一个随意的、有问题的 `__str__` 方法。因此，在打印值时我们用 try / except 声明加以保护，以避免当我们在处理一个异常时另一个未捕获的异常又开始扩散的情况出现。如果一个 except 子句并未列出感兴趣的异常，那它就干脆捕获所有的异常，通常情况下这么用会是个错误，但本节却展示了它的一个适用场景。

类似的技术最好是用在将所有详细的信息写到日志文件中，以备日后分析。如果在交互式状态下抛出这么多的信息，则可能会让人无从下手，向用户抛出这么多调试信息，即使只是缩减版的回溯信息，也可算是用户接口设计上的失误。而悄悄地把这些信息写入日志文件，对于开发者和用户都是可以接受的。




# 更简单的使用单元测试

你想改进对单元测试的使用以确保那是件非常简单清楚的事，从而让自己养成勤于测试的习惯。

将下列代码存为模块 microtest.py 放入 sys.path 中：
    
    import types
    import traceback
    class TestException(Exception):
      pass
    
    def microtest(modulename, verbose=None, log=sys.stdout):
      '''Execute all functions in named module with _test_ in the name
      that take no arguments
      modulename -- name of the module to be tested.
      verbose    -- If true print sequence of test names as they are executed
      Returns None on success, raises exception on failure.'''
    
      module = __import__(modulename)
      total_tested = 0
      total_failed = 0
      total_pending = 0
      # print(111, file=sys.stdout)
      for name in dir(module):
        # print(name)
        if name.startswith('_test_'):
          obj = getattr(module, name)
          if (isinstance(obj, types.FunctionType) and not obj.__code__.co_argcount):
            if verbose:
              print('        MicroTest: = <%s> on test' % name, file=log)
            try:
              total_tested += 1
              obj()
              print('        MicroTest: . <%s> tested' % name, file=log)
            except Exception as e:
              total_failed += 1
              print('')
              print('---- ↓ MicroTest: %s.%s() FAILED ↓ ----' % (modulename, name), file=sys.stderr)
              traceback.print_exc()
              print('---- ↑ MicroTest: %s.%s() FAILED ↑ ----' % (modulename, name), file=sys.stderr)
              print('')
        elif 'test' in name:
          total_pending += 1
          if verbose:
            print('        MicroTest: ? <%s> detect pending test' % name, file=log)
    
      message = '\n\n        MicroTest: module "%s" failed (%s / %s) unittests. (pending %s unittests)\n\n' % (modulename, total_failed, total_tested, total_pending)
      if total_failed > 0:
        raise TestException(message)
      if verbose:
        print(message, file=log)
    
导入这个模块后将会把每个以 `def _test_` 开头的函数作为测试函数来运行。通常在需要测试的模块里就可以这样使用：

    if __name__ == '__main__':
      microtest(__name__, verbose=False)










# 在单元测试中检查区间

需要检查结果值，不是为了验证它等于或不等于某个指定的值，而是为了确定它在或不在某个特定的区间。

当涉及到大量的浮点计算时，不太可能去测试结果是否精确地等于某个参考值。一般会围绕参考值指定一个正确范围，提供一定程度的容错性。最好的方法是从 `unittest.TestCase` 派生子类并加入额外的检査方法：
    
    import unittest
    class IntervalTestCase(unittest.TestCase):
      def failUnlessInside (self, first, second, error, msg=None):
        """如果 first 不在区间中，失败，区间由 second+-error 构成"""
        if not (second-error) < first < (second+error):
          raise self.failureException(msg or '%r != %r (+-%r)' % (first, second, error))
      def failIfInside(self, first, second, error, msg=None):
        """如果 first 在区间中，失败，区间由 second+-error 构成"""
        if (second-error) < first < (second+error):
          raise self.failureException(msg or '%r == %r (+-%r)' % (first, second, error))
      assertInside = failUnlessInside
      assertNotInside = failIfInside
    
    
    class IntegerArithmenticTestCase(IntervalTestCase):
      def testAdd(self):
        self.assertInside((1 + 2), 3.3, 0.5)
        self.assertInside(0 + 1, 1.1, 0.01)
      def testMultiply(self):
        self.assertNotInside((0 * 10), .1, .05)
        self.assertNotInside((5 * 8), 40.1, .2)
    unittest.main()










