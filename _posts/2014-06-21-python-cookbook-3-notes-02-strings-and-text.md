---
layout: post
title: Python Cookbook III 学习笔记 - 文本处理
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true

summary: '自己整理的 Python Cookbook III 的学习笔记, 没有严格照原文翻译但保留了大部分内容, 第02章, 文本处理. 介绍了文本搜索替换, 文本清理, 格式化输出等.'

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>

自己整理的 Python Cookbook III 的学习笔记, 没有严格照原文翻译但保留了大部分内容, 第02章, 文本处理. 介绍了文本搜索替换, 文本清理, 格式化输出等.

# 目录

- 以多种分隔符拆分文本
- 匹配文本的开头或结尾
- 用 Shell 通配符匹配文本
- 按照模式匹配文本
- 查找替换文本
- 查找替换时忽略大小写
- 在匹配时跨越多行文字
- 把 Unicode 字符转换为标准表现形式
- 清理文本中无用的部分
- 更高级的清理和净化文本操作
- 对齐文本
- 在字符串中插入变量
- 将文本的每行调整到指定字符数
- 处理 HTML 字符实体




# 以多种分隔符拆分文本

### 问题

需要分割字符串, 但是分隔符(有时也包括周围的空白字符)是不确定的.

### 方案

字符串的 `split()` 方法在简单的场合很好用, 但是不能处理有多种分割字符, 以及分割符周围有空白字符的情况. 这时需要的是 `re.split()`:

    >>> line = 'asdf fjdk; afed, fjek,asdf,      foo'
    >>> import re
    >>> re.split(r'[;,\s]\s*', line)
    ['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']

### 讨论

`re.split()` 可以指定多种分割字符, 在这个例子里, 分隔符是逗号, 分号或者空白字符, 其后可以跟着任意多的空白字符. 一旦匹配到这个模式, 整个匹配都将成为分割字符. 这个办法像 `str.split()` 一样, 也是返回分段的列表.

用 `re.split()` 时应该小心正则表达式中有括号分组的情况. 如果用到了捕获分组, 则捕获到的内容也会出现在结果列表中.

    >>> fields = re.split(r'(;|,|\s)\s*', line)
    >>> fields
    ['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']

Getting the split characters might be useful in certain contexts. For example, maybe you need the split characters later on to reform an output string:
有时候捕获分割字符很有必要, 比如之后需要利用分割字符重建文本:

    >>> values = fields[::2]
    >>> delimiters = fields[1::2] + ['']
    >>> values
    ['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
    >>> delimiters
    [' ', ';', ',', ',', ',', '']
    >>> # Reform the line using the same delimiters
    >>> ''.join(v+d for v,d in zip(values, delimiters))
    'asdf fjdk;afed,fjek,asdf,foo'

如果不想让分割字符出现在返回结果中, 但又必须在正则表达式中使用括号分组, 那么可以用"非捕获分组" `?:...`, 举例:

    >>> re.split(r'(?:,|;|\s)\s*', line)
    ['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']





# 匹配文本的开头或结尾

### 问题

需要检查字符串的开头或结尾是否为特定值, 比如在文件类型检测, URL 协议检测等场合.

### 方案

最简单的办法是用 `startswith()` 和 `str.endswith()`:

    >>> filename = 'spam.txt'
    >>> filename.endswith('.txt')
    True
    >>> filename.startswith('file:')
    False
    >>> url = 'http://www.python.org'
    >>> url.startswith('http:')
    True

要检查多个可能的值时, 可以用包含这些值的元组作参数:

    >>> import os
    >>> filenames = os.listdir('.')
    >>> filenames
    [ 'Makefile', 'foo.c', 'bar.py', 'spam.c', 'spam.h' ]
    >>> [name for name in filenames if name.endswith(('.c', '.h')) ]
    ['foo.c', 'spam.c', 'spam.h'
    >>> any(name.endswith('.py') for name in filenames)
    True

另一个例子:

    from urllib.request import urlopen
    def read_data(name):
        if name.startswith(('http:', 'https:', 'ftp:')):
            return urlopen(name).read()
        else:
            with open(name) as f:
                 return f.read()

注意, 这两个方法必须以元组做参数, 这在 Python 中很少见. 如果传入参数的类型是列表或集合, 那需要先转为元组.

    >>> choices = ['http:', 'ftp:']
    >>> url = 'http://www.python.org'
    >>> url.startswith(choices)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: startswith first arg must be str or a tuple of str, not list
    >>> url.startswith(tuple(choices))
    True

### 讨论

要检测前缀后缀用 `startswith()` 和 `endswith()` 是最方便的, 另一个办法是字符串切片, 这就不怎么优雅了:

    >>> filename = 'spam.txt'
    >>> filename[-4:] == '.txt'
    True
    >>> url = 'http://www.python.org'
    >>> url[:5] == 'http:' or url[:6] == 'https:' or url[:4] == 'ftp:'
    True

当然也可以用正则表达式, 但是对这种简单的需求, 用正则表达式有些过力了:

    >>> import re
    >>> url = 'http://www.python.org'
    >>> re.match('http:|https:|ftp:', url)
    <_sre.SRE_Match object at 0x101253098>

最后, `startswith()` 和 `endswith()` 非常适合跟其他操作组合使用, 下面的例子检测目录中是否有指定格式的文件:

    if any(name.endswith(('.c', '.h')) for name in listdir(dirname)):
       ...





# 用 Shell 通配符匹配文本

### 问题

想要以 Unix shell 的风格 (比如 `*.py`, `Dat[0-9]*.csv` 之类) 匹配文本.

### 方案

`fnmatch` 模块有两个函数 `fnmatch()` 和 `fnmatchcase()` 就是做这个的, 用法很简单:

    >>> from fnmatch import fnmatch, fnmatchcase
    >>> fnmatch('foo.txt', '*.txt')
    True
    >>> fnmatch('foo.txt', '?oo.txt')
    True
    >>> fnmatch('Dat45.csv', 'Dat[0-9]*')
    True
    >>> names = ['Dat1.csv', 'Dat2.csv', 'config.ini', 'foo.py']
    >>> [name for name in names if fnmatch(name, 'Dat*.csv')]
    ['Dat1.csv', 'Dat2.csv']

`fnmatch()` 的匹配是否有大小写敏感, 取决于所用的操作系统:

    >>> # On OS X (Mac)
    >>> fnmatch('foo.txt', '*.TXT')
    False

    >>> # On Windows
    >>> fnmatch('foo.txt', '*.TXT')
    True

如果这会造成困扰的话, 可以改用 `fnmatchcase()`, 这个函数会严格检测大小写.

    >>> fnmatchcase('foo.txt', '*.TXT')
    False

这两个函数在数据处理中也大有所为, 比如有如下的地址列表:

    addresses = [
        '5412 N CLARK ST',
        '1060 W ADDISON ST',
        '1039 W GRANVILLE AVE',
        '2122 N CLARK ST',
        '4802 N BROADWAY',
    ]

可以这样写列表推导来提取符合某种规律的地址:

    >>> from fnmatch import fnmatchcase
    >>> [addr for addr in addresses if fnmatchcase(addr, '* ST')]
    ['5412 N CLARK ST', '1060 W ADDISON ST', '2122 N CLARK ST']
    >>> [addr for addr in addresses if fnmatchcase(addr, '54[0-9][0-9] *CLARK*')]
    ['5412 N CLARK ST']

### 讨论

论功能的强大程度, `fnmatch` 的能力大致在普通字符串函数和正则表达式之间, 在许多场合都很适用, 比如要用通配符简单的过滤一些数据时.

另外, 如果要专门解析处理文件名, 应该换用 `glob` 模块, 见第五章的 [Getting a Directory Listing](http://chimera.labs.oreilly.com/books/1230000000393/ch05.html#dirlisting).





# 按照模式匹配文本

### 问题

需要以特定的模式匹配或查找字符串.

### 方案

如果要查找的文本只是简单的字面量, 用 `str.find()`, `str.endswith()`, `str.startswith()` 就够了.

    >>> text = 'yeah, but no, but yeah, but no, but yeah'
    >>> # Exact match
    >>> text == 'yeah'
    False
    >>> # Match at start or end
    >>> text.startswith('yeah')
    True
    >>> text.endswith('no')
    False
    >>> # Search for the location of the first occurrence
    >>> text.find('no')
    10

更复杂的匹配要用正则表达式. 比如匹配像 "11/27/2012" 这样子的日期格式:

    >>> text1 = '11/27/2012'
    >>> text2 = 'Nov 27, 2012'
    >>>
    >>> import re
    >>> # Simple matching: \d+ means match one or more digits
    >>> if re.match(r'\d+/\d+/\d+', text1):
    ...     print('yes')
    ... else:
    ...     print('no')
    ...
    yes
    >>> if re.match(r'\d+/\d+/\d+', text2):
    ...     print('yes')
    ... else:
    ...     print('no')
    ...
    no

如果在大量的匹配检测中都要用同一个模式, 可以把它预编译为正则表达式模式对象:

    >>> datepat = re.compile(r'\d+/\d+/\d+')
    >>> if datepat.match(text1):
    ...     print('yes')
    ... else:
    ...     print('no')
    ...
    yes
    >>> if datepat.match(text2):
    ...     print('yes')
    ... else:
    ...     print('no')
    ...
    no

注意! `match()` 永远是从字符串开头开始检测匹配的. 如果要匹配的模式可以从字符串中间开始, 那需要改用 `findall()`:

    >>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
    >>> datepat.findall(text)
    ['11/27/2012', '3/13/2013']

定义正则表达式对象时, 可以用括号来指定捕获组:

    >>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')

每个捕获组都可以单独取出来, 一般都会用在后续的处理中.

    >>> m = datepat.match('11/27/2012')
    >>> m
    <_sre.SRE_Match object at 0x1005d2750>

    >>> # Extract the contents of each group
    >>> m.group(0)
    '11/27/2012'
    >>> m.group(1)
    '11'
    >>> m.group(2)
    '27'
    >>> m.group(3)
    '2012'
    >>> m.groups()
    ('11', '27', '2012')
    >>> month, day, year = m.groups()

    >>> # Find all matches (notice splitting into tuples)
    >>> text
    'Today is 11/27/2012. PyCon starts 3/13/2013.'
    >>> datepat.findall(text)
    [('11', '27', '2012'), ('3', '13', '2013')]
    >>> for month, day, year in datepat.findall(text):
    ...     print('{}-{}-{}'.format(year, month, day))
    ...
    2012-11-27
    2013-3-13

> 捕获组 `m.group(index)` 的参数为 0 时返回整个匹配的值, 为 1 时返回括号里的第 1 组. 另一个方法 `m.groups()` 可以接受任意参数, 但接受的参数没用, 都是返回全部的括号匹配组的元组.

另外一个细节, `findall()` 方法返回列表, 而 `finditer()` 方法返回可迭代对象, 可以根据需要灵活使用:

    >>> for m in datepat.finditer(text):
    ...     print(m.groups())
    ...
    ('11', '27', '2012')
    ('3', '13', '2013')

### 讨论

定义正则表达式时一般用 `r'...'` 字符串, 该模式不解释反斜杠, 很适合用在有大量特殊字符的情况. 否则之前例子就需要写两个反斜杠 `'(\\d+)/(\\d+)/(\\d+)'`.

再次注意, `match()` 只检查字符串开头是否符合, 不检查结尾.

    >>> m = datepat.match('11/27/2012abcdef')
    >>> m
    <_sre.SRE_Match object at 0x1005d27e8>
    >>> m.group()
    '11/27/2012'

若要确切的匹配整个正则表达式, pattern 最后应该加上 `$` (表示字符串结尾).

    >>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)$')
    >>> datepat.match('11/27/2012abcdef')
    >>> datepat.match('11/27/2012')
    <_sre.SRE_Match object at 0x1005d2750>

大量使用同一个 pattern 时最好预编译来提高速度, 但是对于简单的情形, 可以不必预编译 pattern, 像这样直接用就可以了:

    >>> re.findall(r'(\d+)/(\d+)/(\d+)', text)
    [('11', '27', '2012'), ('3', '13', '2013')]





# 查找替换文本

### 问题

需要替换一段文本中的特定字符串.

### 方案

对于最简单的基于字面值的替换, 用 `str.replace()`:

    >>> text = 'yeah, but no, but yeah, but no, but yeah'
    >>> text.replace('yeah', 'yep')
    'yep, but no, but yep, but no, but yep'

更复杂一些的, 用正则表达式的 `sub()` 方法/函数. 比如把所有的 "11/27/2012" 样式的日期替换为 "2012-11-27" 的样式, 可以这么做:

    >>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
    >>> import re
    >>> re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
    'Today is 2012-11-27. PyCon starts 2013-3-13.'

`sub()` 的三个参数依次是: 需要匹配的模式, 替换文本, 要被替换的输入数据. 反斜杠转义数字如 `\3` 表示引用之前捕获的组.

如果代码中会重复用到某个匹配模式, 应该预编译它来优化性能:

    >>> import re
    >>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
    >>> datepat.sub(r'\3-\1-\2', text)
    'Today is 2012-11-27. PyCon starts 2013-3-13.'

对于更复杂一些的情形, 用回调函数作为 `sub()` 的第二个参数:

    >>> from calendar import month_abbr
    >>> def change_date(m):
    ...     mon_name = month_abbr[int(m.group(1))]
    ...     return '{} {} {}'.format(m.group(2), mon_name, m.group(3))
    ...
    >>> datepat.sub(change_date, text)
    'Today is 27 Nov 2012. PyCon starts 13 Mar 2013.'

这个回调函数接受一个 match 对象, 类似用 `match()` `find()` 返回的那种. 在这个 match 对象上用 `group()` 就可以取得特定的匹配了. 回调函数的返回值应该是替换文本.

如果要统计替换了多少次, 应该改用 `re.subn()`, 它的返回值是两个, 第二个数字就是替换次数:

    >>> newtext, n = datepat.subn(r'\3-\1-\2', text)
    >>> newtext
    'Today is 2012-11-27. PyCon starts 2013-3-13.'
    >>> n
    2

### 讨论

正则表达式的查找和替换没什么可多说的, 其精华在于写出合适的匹配, 这需要大量的练习.





# 查找替换时忽略大小写

### 问题

想在替换文字时忽略大小写格式.

### 方案

应该用 `re.IGNORECASE` 标志.

    >>> text = 'UPPER PYTHON, lower python, Mixed Python'
    >>> re.findall('python', text, flags=re.IGNORECASE)
    ['PYTHON', 'python', 'Python']
    >>> re.sub('python', 'snake', text, flags=re.IGNORECASE)
    'UPPER snake, lower snake, Mixed snake'

如果不仅要查找还要替换, 那就会遇见一个限制: 替换后的文本没法仿照原始的大小写变化. 要实现这个功能, 可以自定义一个回调函数:

    def matchcase(word):
        def replace(m):
            text = m.group()
            if text.isupper():
                return word.upper()
            elif text.islower():
                return word.lower()
            elif text[0].isupper():
                return word.capitalize()
            else:
                return word
        return replace

然后像这样使用:

    >>> re.sub('python', matchcase('snake'), text, flags=re.IGNORECASE)
    'UPPER SNAKE, lower snake, Mixed Snake'

### 讨论

一般的情况用 `re.IGNORECASE` 足以对付了. 但是带 Unicode 字符的大小写转换仍然需要更精细的处理, 见 [Working with Unicode Characters in Regular Expressions](http://chimera.labs.oreilly.com/books/1230000000393/ch02.html#unicodere).





# 在匹配时跨越多行文字

### 问题

对一段文字进行正则表达式匹配时, 需要跨越多行.

### 方案

通常这是在正则表达式中用 `.` 时发生的问题, 用户原意是想匹配任何字符, 但很容易忘记 `.` 不能匹配换行符:

    >>> comment = re.compile(r'/\*(.*?)\*/')
    >>> text1 = '/* this is a comment */'
    >>> text2 = '''/* this is a
    ...               multiline comment */
    ... '''
    >>>
    >>> comment.findall(text1)
    [' this is a comment ']
    >>> comment.findall(text2)
    []

一个解决办法是把换行符加到这个 pattern 中:

    >>> comment = re.compile(r'/\*((?:.|\n)*?)\*/')
    >>> comment.findall(text2)
    [' this is a\n              multiline comment ']

`(?:.|\n)` 指定一个非捕获组, 其行为像普通的组一样, 但在匹配结果中它不会单独被列为一项, 也不会获得索引序号.

### 讨论

`re.compile()` 接受一个可选的 `re.DOTALL` 参数, 能够使 `.` 匹配所有字符, 当然包括换行符.

    >>> comment = re.compile(r'/\*(.*?)\*/', re.DOTALL)
    >>> comment.findall(text2)
    [' this is a\n              multiline comment ']

简单情况下 `re.DOTALL` 很好用, 但对于复杂的正则表达式, 或者联合多个表达式时, 最好还是手动控制, 不要修改 `.` 的默认行为, 确保不用这些标志时你的正则表达式也能正常工作.





# 把 Unicode 字符转换为标准表现形式

### 问题

有一些含 Unicode 字符的文本, 需要确保它们具有一致的内部形式.

### 方案

在 Unicode 中, 一些相同的字符具有多种字符编码, 比如下面的例子:

    >>> s1 = 'Spicy Jalape\u00f1o'
    >>> s2 = 'Spicy Jalapen\u0303o'
    >>> s1
    'Spicy Jalapeño'
    >>> s2
    'Spicy Jalapeño'
    >>> s1 == s2
    False
    >>> len(s1)
    14
    >>> len(s2)
    15

`"Spicy Jalapeño"` 有两种形式, 前者是组合字符 `"ñ"` (U+00F1), 后者是拉丁字母 `"n"` 接一个修饰字符  `"~"` (U+0303).

比较字符串时, 这样就会出现问题, 应该事先用 `unicodedata` 模块提供的函数修正:

    >>> import unicodedata
    >>> t1 = unicodedata.normalize('NFC', s1)
    >>> t2 = unicodedata.normalize('NFC', s2)
    >>> t1 == t2
    True
    >>> print(ascii(t1))
    'Spicy Jalape\xf1o'

    >>> t3 = unicodedata.normalize('NFD', s1)
    >>> t4 = unicodedata.normalize('NFD', s2)
    >>> t3 == t4
    True
    >>> print(ascii(t3))
    'Spicy Jalapen\u0303o'

函数 `normalize()` 的第一个参数指示应如何进行正规化. `NFC` 意思是全都使用组合字符(尽可能使用单独的一个字符). `NFD` 意思是尽可能使用拆分形式, 即普通字符+修饰字符.

`normalize()` 的第一个参数还可以是 `NFKC` 或 `NFKD`, 用来处理特别的细节, 如连写的字符.

    >>> s = '\ufb01'   # A single character
    >>> s
    'ﬁ'
    >>> unicodedata.normalize('NFD', s)
    'ﬁ'
    # Notice how the combined letters are broken apart here
    >>> unicodedata.normalize('NFKD', s)
    'fi'
    >>> unicodedata.normalize('NFKC', s)
    'fi'

### 讨论

对 Unicode 字符进行正规化是非常重要的, 尤其是当数据由用户输入, 且可能使用不同的编码时.

另外, 去除文本中的变音字符也是很常见的需求(比如为了更好地进行搜索匹配时):

    >>> t1 = unicodedata.normalize('NFD', s1)
    >>> ''.join(c for c in t1 if not unicodedata.combining(c))
    'Spicy Jalapeno'

这里, `unicodedata.combining()` 接受一个参数, 检测它是否为组合字符, 该模块中还有一些函数用于检测字符是否属于特定集合等等.





# 清理文本中无用的部分

### 问题

需要清理掉文本中的无效字符, 如开头结尾的空白, 或者把连续的多个空格缩减为一个.

### 方案

`strip()` 用来清理开头和结尾的空白字符, `lstrip()` 和 `rstrip()` 则近仅针对开头或结尾. 这些方法默认是清理空白字符, 但也接受参数来定制要清理的字符:

    >>> # Whitespace stripping
    >>> s = '   hello world  \n'
    >>> s.strip()
    'hello world'
    >>> s.lstrip()
    'hello world  \n'
    >>> s.rstrip()
    '   hello world'

    >>> # Character stripping
    >>> t = '-----hello====='
    >>> t.lstrip('-')
    'hello====='
    >>> t.strip('-=')
    'hello'

### 讨论


`strip()` 常常是配合读取数据时的清理工作, 比如去掉引号等等. 注意 `strip()` 不会去动字符串中间的部分:

    >>> s = '  hello       world   \n'
    >>> s = s.strip()
    >>> s
    'hello       world'

要清掉中间部分的空白字符, 可以用 `replace()` 或者基于正则表达式的替换.

    >>> s.replace(' ', '')
    'helloworld'
    >>> import re
    >>> re.sub('\s+', ' ', s)
    'hello world'

通常这种字符串清理任务是出现在迭代器中的, 比如从某个文件中按行读取数据, 这时候很适合生成器表达式发挥作用:

    with open(filename) as f:
        lines = (line.strip() for line in f)
        for line in lines:
            ...

用 `lines = (line.strip() for line in f)` 的方式做数据转换效率很高, 它不会把整个数据读入临时创建的列表中, 相反它只是建立迭代器, 为每次只提供一行数据, 并对其运用 strip 方法.

更高级的用法要靠下一节中讲的 `translate()` 了.





# 更高级的清理和净化文本操作

### 问题

需要处理用户输入的类似 `"pýtĥöñ"` 这样的文本.

### 方案

文本解析处理是个很宽泛的话题. 从简单的大小写转换到正则表达式替换, 再到使用 `unicodedata.normalize()` 对 Unicode 字符串进行标准化处理都是其中的内容.

还可以把文本净化的操作推的更远, 清理掉某个范围内的所有字符, 或者清理掉字母上的变音符号等等. 这时可以考虑 `str.translate()` 方法, 比如下面这个字符串:

    >>> s = 'pýtĥöñ\fis\tawesome\r\n'
    >>> s
    'pýtĥöñ\x0cis\tawesome\r\n'

第一步是清理掉空白字符, 办法是建立一个字符转换的映射, 然后使用 `translate()`:

    >>> remap = {
    ...     ord('\t') : ' ',
    ...     ord('\f') : ' ',
    ...     ord('\r') : None      # Deleted
    ... }
    >>> a = s.translate(remap)
    >>> a
    'pýtĥöñ is awesome\n'

这样 `\t` `\f` 被替换成了空格, `\r` 也被删除了.

可以扩展这个映射, 使其包含更多的转换规则. 比如移除文本中的所有修饰字符:

    >>> import unicodedata
    >>> import sys
    >>> cmb_chrs = dict.fromkeys(c for c in range(sys.maxunicode)
    ...                          if unicodedata.combining(chr(c)))
    ...
    >>> b = unicodedata.normalize('NFD', a)
    >>> b
    'pýtĥöñ is awesome\n'
    >>> b.translate(cmb_chrs)
    'python is awesome\n'

上面的代码中, `dict.fromkeys()` 建立了一个映射字典, 把每个 Unicode 修饰字符都映射为 None. 输入数据经过 `unicodedata.normalize()` 后变成了分解状态, 然后 `translate` 函数删除了所有重音符号. 类似的也可以使用这个技巧删除控制字符等等.

下面是另一个例子, 把 Unicode 数字字符映射到标准的 ASCII 数字:

    >>> digitmap = { c: ord('0') + unicodedata.digit(chr(c))
    ...             for c in range(sys.maxunicode)
    ...             if unicodedata.category(chr(c)) == 'Nd' }
    ...
    >>> len(digitmap)
    460
    >>> # Arabic digits
    >>> x = '\u0661\u0662\u0663'
    >>> x.translate(digitmap)
    '123'

此外, 结合使用 I/O 编码解码函数也可以用来清理文本. 思路是先做一些预处理, 然后结合使用 ` encode()` 和 `decode()`:

    >>> a
    'pýtĥöñ is awesome\n'
    >>> b = unicodedata.normalize('NFD', a)
    >>> b.encode('ascii', 'ignore').decode('ascii')
    'python is awesome\n'

`unicodedata.normalize('NFD', a)` 分解了原先的组合字符, 随后以 ASCII 编解码抛弃了那些无效字符. 自不必说, 仅当需要结果是 ASCII 字符时, 这个办法才能用.

### 讨论

净化文本时性能是个要考虑的因素. 通常越简单的方法运行的越快, 即便要调用好几次, `str.replace()` 仍然是速度最快的, 所以处理空白字符也可以这么写, 这段代码比 `translate()` 和正则表达式替换都要快:

    def clean_spaces(s):
        s = s.replace('\r', '')
        s = s.replace('\t', ' ')
        s = s.replace('\f', ' ')
        return s

解决性能问题需要大量的练习, 且没有通用的方案, 实践中可以多测试不同的办法. 本节的代码是以字符串为例的, 其实类似的原理也可以用在字节码的查找替换中.





# 对齐文本

### 问题

需要以不同的格式对齐文本.

### 方案

基本方式是使用 `ljust()`, `rjust()`, 和 `center()`:

    >>> text = 'Hello World'
    >>> text.ljust(20)
    'Hello World         '
    >>> text.rjust(20)
    '         Hello World'
    >>> text.center(20)
    '    Hello World     '

这些方法都接受一个填充字符(只能是单个字符)作为参数:

    >>> text.rjust(20,'=')
    '=========Hello World'
    >>> text.center(20,'*')
    '****Hello World*****'

`format()` 函数也能做同样的工作, 格式化参数里写 `<`, `>`, 和 `^` 后跟着要调整的长度:

    >>> format(text, '>20')
    '         Hello World'
    >>> format(text, '<20')
    'Hello World         '
    >>> format(text, '^20')
    '    Hello World     '

默认填充字符是空格, 要自定义的填充字符应该放在尖括号前面:

    >>> format(text, '=>20s')
    '=========Hello World'
    >>> format(text, '*^20s')
    '****Hello World*****'

这些格式符当然也能用在 `format()` 方法中, 这时要写在冒号后面:

    >>> '{:>10s} {:>10s}'.format('Hello', 'World')
    '     Hello      World'

`format()` 有个好处, 它不是只对字符串才能用, 它可以直接格式化数值类型.

    >>> x = 1.2345
    >>> format(x, '>10')
    '    1.2345'
    >>> format(x, '^10.2f')
    '   1.23   '

### 讨论

早期的 Python 代码还能见到用 `%` 做格式化的例子:

    >>> '%-20s' % text
    'Hello World         '
    >>> '%20s' % text
    '         Hello World'

在新的代码中不该用这个方式了, 应该用功能更强大通用的 `format()` 方法. `format()` 有相当多的功能和用法, 可以查看 [文档](http://docs.python.org/3/library/string.html#formatspec).





# 在字符串中插入变量

### 问题

需要在文本中嵌入一些变量名字, 然后以变量的字符串表达形式替换这部分文字.

### 方案

Python 中没有直接支持字符串插值. 但用 `format()` 就可以实现这个:

    >>> s = '{name} has {n} messages.'
    >>> s.format(name='Guido', n=37)
    'Guido has 37 messages.'

如果要替换的值全都基于存在的变量, 那么可以配合 `format_map()` `vars()`, 写成这样:

    >>> name = 'Guido'
    >>> n = 37
    >>> s.format_map(vars())
    'Guido has 37 messages.'

很隐蔽的一点是, `vars()` 也对类的实例有效:

    >>> class Info:
    ...     def __init__(self, name, n):
    ...         self.name = name
    ...         self.n = n
    ...
    >>> a = Info('Guido',37)
    >>> s.format_map(vars(a))
    'Guido has 37 messages.'

`format()` 和 `format_map()` 有个缺点, 没找到变量时就会报异常:

    >>> s.format(name='Guido')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    KeyError: 'n'

解决办法是定义字典的子类, 实现它的 `__missing__()` 方法:

    class safesub(dict):
        def __missing__(self, key):
            return '{' + key + '}'

再用这个类包裹 `vars()` 字典:

    >>> del n     # Make sure n is undefined
    >>> s.format_map(safesub(vars()))
    'Guido has {n} messages.'

如果经常要写这些代码, 还可以用一个技巧叫 "frame hack", 把变量替换的实现细节隐藏起来:

    import sys
    def sub(text):
        return text.format_map(safesub(sys._getframe(1).f_locals))

现在可以这样使用:

    >>> name = 'Guido'
    >>> n = 37
    >>> print(sub('Hello {name}'))
    Hello Guido
    >>> print(sub('You have {n} messages.'))
    You have 37 messages.
    >>> print(sub('Your favorite color is {color}'))
    Your favorite color is {color}

### 讨论

因为 Python 没有真正的变量插值, 多年以来出现了数种替代方式. 比如像这样的格式化字符串的方法:

    >>> name = 'Guido'
    >>> n = 37
    >>> '%(name) has %(n) messages.' % vars()
    'Guido has 37 messages.'

或者这样的模板方法:

    >>> import string
    >>> s = string.Template('$name has $n messages.')
    >>> s.substitute(vars())
    'Guido has 37 messages.'

但是 `format()` 与 `format_map()` 的方法是最现代, 最好用的, 其中一个优势是能利用上 `format()` 的全部格式化功能(如对数值的处理, 对齐, 补白等等).

字典或映射中如果实现 `__missing__()` 方法, 就可以处理缺失的键, 本例的 `safesub` 类中用来返回这个缺失键的占位符, 避免了到时出现 `KeyError` 异常, 在 debug 时会很有用.

`sub()` 函数用了 `sys._getframe(1)` 返回调用者的栈帧, 其 `f_locals` 属性含有其全部局部变量. 不必说, 通常不应该在代码中滥用这个技术, 只在像这样的工具函数里谨慎使用. 另外 `f_locals` 存放的是调用者的局部变量的拷贝, 可以放心修改它, 不会有副作用比如改变调用者的环境.





# 将文本的每行调整到指定字符数

### 问题

需要对一段长文字格式化换行, 使其每行具有用户指定数量的字符.

### 方案

`textwrap` 模块可以重新格式化文字, 使其更适合输出的需要. 比如像这样一段话:

    s = "Look into my eyes, look into my eyes, the eyes, the eyes, \
    the eyes, not around the eyes, don't look around the eyes, \
    look into my eyes, you're under."

可以用 `textwrap` 的很多种方法来调整输出:

    >>> import textwrap
    >>> print(textwrap.fill(s, 70))
    Look into my eyes, look into my eyes, the eyes, the eyes, the eyes,
    not around the eyes, don't look around the eyes, look into my eyes,
    you're under.

    >>> print(textwrap.fill(s, 40))
    Look into my eyes, look into my eyes,
    the eyes, the eyes, the eyes, not around
    the eyes, don't look around the eyes,
    look into my eyes, you're under.

    >>> print(textwrap.fill(s, 40, initial_indent='    '))
        Look into my eyes, look into my
    eyes, the eyes, the eyes, the eyes, not
    around the eyes, don't look around the
    eyes, look into my eyes, you're under.

    >>> print(textwrap.fill(s, 40, subsequent_indent='    '))
    Look into my eyes, look into my eyes,
        the eyes, the eyes, the eyes, not
        around the eyes, don't look around
        the eyes, look into my eyes, you're
        under.

### 讨论

`textwrap` 特别适合格式化文本以便在终端里友好的显示出来. 用 `os.get_terminal_size()` 可以取得终端的列数.

    >>> import os
    >>> os.get_terminal_size().columns
    80

`textwrap.fill()` 方法还有些参数用来控制制表符, 句末标点等等应该怎样格式化. 更详细的信息可以查看 [textwrap.TextWrapper 的文档](http://docs.python.org/3.3/library/textwrap.html#textwrap.TextWrapper).





# 处理 HTML 字符实体

### 问题

需要在 HTML 字符实体 `&entity;` `&#code;` 和代表的字符 `<`, `>`, `&` 之间做转换.

### 方案

`html.escape()` 可以转换 `<` 和 `>` 这种特殊字符:

    >>> s = 'Elements are written as "<tag>text</tag>".'
    >>> import html
    >>> print(s)
    Elements are written as "<tag>text</tag>".
    >>> print(html.escape(s))
    Elements are written as &quot;&lt;tag&gt;text&lt;/tag&gt;&quot;.

    >>> # Disable escaping of quotes
    >>> print(html.escape(s, quote=False))
    Elements are written as "&lt;tag&gt;text&lt;/tag&gt;".

要把带有非 ASCII 字符的文本转为纯 ASCII 文本, 可以用大部分 I/O 函数都支持的 `errors='xmlcharrefreplace'` 参数.

    >>> s = 'Spicy Jalapeño'
    >>> s.encode('ascii', errors='xmlcharrefreplace')
    b'Spicy Jalape&#241;o'

要把带有字符实体的文本转换回来, 找个 HTML 或 XML 的解析器能省不少事. 如果必须自己转换, 可以使用 `html.parser` 等工具:

    >>> s = 'Spicy &quot;Jalape&#241;o&quot.'
    >>> from html.parser import HTMLParser
    >>> p = HTMLParser()
    >>> p.unescape(s)
    'Spicy "Jalapeño".'
    >>>

    >>> t = 'The prompt is &gt;&gt;&gt;'
    >>> from xml.sax.saxutils import unescape
    >>> unescape(t)
    'The prompt is >>>'

### 讨论

生成 HTML 或是 XML 前正确的转义特殊字符是非常必要的, 一般 `html.escape()` 足够管用了. 替代的办法也有很多, 比如 `xml.sax.saxutils.unescape()` 等, 用时应该仔细查阅文档.




























































