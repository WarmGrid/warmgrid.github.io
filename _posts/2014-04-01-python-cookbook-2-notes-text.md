---
layout: post
title: Python Cookbook II 学习笔记 - 文本操作
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第01章：文本操作。一些常用的文本操作手段，字符串的提取，删等，构建高级的正则替换函数或对象。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>


Python Cookbook II 的学习笔记，第01章：文本操作。一些常用的文本操作手段，字符串的提取，删等，构建高级的正则替换函数或对象。

# 常用的技巧

测试结尾部分是否为提供参数之一

字符串的 `s.startswith()` `s.endswith()` 方法测试其开始和结束是否为特定的序列，扩展这个方法：

    def ends_with(s, *endings):
      return any(map(s.endswith, endings))
    
    for filename in os.listdir('.'):
      if ends_with(filename, '.jpg', '.png', '.gif'):
        print(filename)

以换行符分割多行字符串

    s.splitlines()

以换行符分割多行字符串，且保留行尾的换行符

    s.splitlines(True)

取得第一个空格前的部分，这种简单的任务不需要用 split

    word = word[:word.index(' ')]

转换 ASCII

    print(ord('a')) # => 97
    print(chr(97)) # => a



# 分割字符串时捕获分割组

    import re
    pieces1 = re.compile(r'\d+').split('aaa 123 bbb 456 ccc')
    print(pieces1)   # => ['aaa ', ' bbb ', ' ccc']
    pieces2 = re.compile(r'(\d+)').split('aaa 123 bbb 456 ccc')
    print(pieces2)   # => ['aaa ', '123', ' bbb ', '456', ' ccc']
    testsplit = re.split(r'((1)((2)(3)))', 'aaa1234bbb')
    print(testsplit) # => ['aaa', '123', '1', '23', '2', '3', '4bbb']



# 反转字符串

按照字符反转

    rev = astring[::-1]

按照单词反转

    rev_words = ' '.join(astring.split()[::-1])

单词反转，保留多个空格

    rev_words = re.split(r'(\s+)', astring)
    rev_words.reverse()
    rev_words = ''.join(rev_words)



# 提取、删除、替换指定字符串的类
    
    import re
    class translator(object):
      def __init__(self, pattern, keep=True, to=''):
        super(translator, self).__init__()
        self.pattern = re.compile(pattern)
        self.keep = keep
        self.replace_by = to
        if to:
          self.keep = False
    
      def __call__(self, text):
        if self.keep:
          return ''.join(re.findall(self.pattern, text))
        else:
          return re.subn(self.pattern, self.replace_by, text)[0]
    
    digits_only = translator(r'\d')
    r = digits_only('Chris Perkins : 224-7992f')
    print(r) # => 2247992
    
    no_digits = translator(r'\d', keep=False)
    r = no_digits('Chris Perkins : 224-7992f')
    print(r) # => 'Chris Perkins : -f'
    
    digits_to_hash = translator(r'\d', to='#')
    r = digits_to_hash('Chris Perkins : 224-7992f')
    print(r) # => 'Chris Perkins : ###-####f'



# locals 的用法

locals() 函数将创建一个字典，字典的 key 就是本地变量：

    msg = string.Template('the square of $number is $square')
    for number in range(10):
      square = number * number
      print(msg.substitute(locals()))
      # same as
      print(msg.substitute(number=number, square=sqaure))



# 一次替换多个子字符串

给出一段文本和字典，所有能够在指定字典中找到的子串都被替换为字典中的对应值：

    import re
    def multiple_replace(text, adict):
      rx = re.compile('|'.join(map(re.escape, adict)))
      def one_xlat(match):
        return adict[match.group(0)]
      return rx.sub(one_xlat, text)
    
    s = 'aaa bbb aaa 12312'
    print(multiple_replace(s, {'aaa':'xxx', '12': '0'}))
    # => 'xxx bbb xxx 030'

首先根据想要匹配的key创建一个正则表达式，形式为 `a1|a2|...|aN`, 然后不直接给 re.sub 传递用于替换的字符串，而是传入一个回调函数参数。每当遇到一次匹配 re.sub 就会调用该回调函数，并将 `re.MatchObject` 的实例作为唯一参数传递给该回调函数，并期望着该回调函数返回作为替换物的字符串。

上面的做法有个缺陷，函数 multiple_replace 每次被调用都会重算正则表达式并重定义 one_xlat 辅助函数。但经常只需要使用同一个固定不变的翻译表来完成很多文本的替换，这种情况下会希望只做一次准备工作。出于这种需求，最好使用下面的基于闭包的方式：

    import re
    def make_xlat(*args, **kwds):
      adict = dict(*args, **kwds)
      rx = re.compile('|'.join(map(re.escape, adict)))
      def one_xlat(match):
        return adict[match.group(0)]
      def xlat(text):
        return rx.sub(one_xlat, text)
      return xlat
    
    text = "Larry Wall is the creator of Perl"
    adict = {
      "Larry Wall" : "Guido van Rossum",
      "creator" : "Benevolent Dictator for Life",
      "Perl" : "Python",
    }
    translate = make_xlat(adict)
    print(translate(text))
    # => 'Guido van Rossum is the Benevolent Dictator for Life of Python'

此外，替换任务常常是基于单词的，而不是基于任意一个子字符串。通过特殊的 `r'\b'` 序列，正则表达式可以很好地找出单词的开始和结束位置。可以修改 multiple_replace 和 make_xlat 中创建和分配正则表达式 rx 的部分从而完成一些自定任务。

    rx = re.compile(r'\b%s\b' % r'\b|\b'.join(map(re.escape, adict)))
    # 当同时有 % 和 join 时，计算顺序是先 join ，然后 %

这样对基于单词和不基于单词的替换，都完成了任务，但是用到了两组函数，大部分片段都是重复的。好代码的一个关键规则是"只做一次"，当注意到代码重复的时候，应该能够很快嗅到不妙的气味，并对原来的代码进行重构以提髙复用性。这里为了便于定制，我们更需要一个类，而不是函数或闭包。

    class make_xlat:
      def __init__(self, *args, **kwds):
        self.adict = dict(*args, **kwds)
        self.rx = self.make_rx()
      def make_rx(self):
        return re.compile('|'.join(map(re.escape, self.adict)))
      def one_xlat(self, match):
        return self.adict[match.group(0)]
      def __call__(self, text):
        return self.rx.sub(self.one_xlat, text)

类的优势是可以子类化或重载某些函数，轻易地实现重新定制。为了对单词进行翻译替换，代码可以这样写：

    class make_xlat_by_whole_words(make_xlat):
      def make_rx(self):
        return re.compile(r'\b%s\b' % r'\b|\b'.join(map(re.escape, self.adict)))
    
    text = "creat, creator"
    adict = {
              "creat" : "will not replace creator",
            }
    translate = make_xlat_by_whole_words(adict)
    print(translate(text))
    # => 'will not replace creator, creator'
    






