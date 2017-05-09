---
layout: post
title: Python Cookbook II 学习笔记 - 文件管理
tags:
- Python
- Programming
- Quote
status: publish
type: post
published: true
summary: Python Cookbook II 的学习笔记，第02章：文件管理。介绍了遍历单词，遍历目录中的文件，在 Windows 中修改文件属性的方法等，使用了许多生成器。

---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = ' ';
hljs.initHighlightingOnLoad();
</script>





Python Cookbook II 的学习笔记，第02章：文件管理。介绍了遍历单词，遍历目录中的文件，在 Windows 中修改文件属性的方法等，使用了许多生成器。

# 更快的读取和写入文件

读取文件时如果使用 `readlines` 方法，将一次读完整个文件，并返回一个各行数据的列表：

    for line in input_file.readlines():
      process(line)

`readlines` 方法只有在物理内存足够用的情况下才会很有用。如果文件非常庞大，`readlines` 可能会失败，或者性能急剧降低，虚拟内存不足，操作系统会将物理内存中的数据复制到磁盘上。

这时候可以对这个文件对象执行一个循环，每次取得一行并处理，这样可获得更好的性能和效率：

    for line in input_file:
      process(line)

很多时候想写入的数据不是在一个大字符串中，而是在一个字符串列表（或其他序列）中。为此应该使用 `writelines` 方法，这个方法并不如它的名字仅限于行的写入，其实二进制文件和文本文件都适用：

    file_object.writelines(list_of_text_strings)
    open('a_bin_file', 'wb').writelines(list_of_data_strings)

当然也可以先把子串拼接成大字符串 (比如用 `''.join`) 再调用 `write` 写入，或者在循环中写入，但直接调用 `writelines` 要比上面两种方式快得多。





# 遍历文件中的每个词

最好的办法是使用两重循环，一个用于处理行，另一个则处理每一行中的每个词：
    
    for line in open(thefilepath):
      for word in line.split():
        dosomethingwith(word)

for 语句假定了词是一串非空的字符，并由空白字符隔开。

如果词的定义有变化，还可以使用正则表达式：

    import re
    re_word = re.compile(r"[\w'-]+")
    for line in open(thefilepath):
      for word in re_word.finditer(line):
        dosomethingwith(word.group(0))

在此例中，词被定义为数字字母，连字符或单引号构成的序列。

如果还需要其他的对词的定义，当然也需要不同的正则表达式。外层关于文件行的循环则不用改变。通常把迭代封装成迭代器对象是个好主意，这种封装也很常见和易于使用：

    def words_of_file(thefilepath, line_to_words=str.split):
      the_file = open(thefilepath)
      for line in the_file:
        for word in line_to_words(line):
          yield word
      the_file.close()
    
    for word in words_of_file(thefilepath):
      dosomethingwith(word)

这个方式可以清晰有效地将两部分内容分开：一个是怎么迭代所有的元素（本例中指的是文件中的词），另一个是要对每个元素做什么处理。

一旦将迭代操作的部分封装在一个迭代器对象中，就可以在程序各处重复使用这个迭代器。而且如果需要维护代码的话，只需要修改一处，即迭代器的定义和实现部分，而不用到处寻找需要修改的部分。

通过重构将循环放入一个生成器，我们还获得其他两个小小的增强，文件被显式的确保关闭了，行文本被划分成单词的方式也更通用了。

比如，如果我们需要用正则表达式来取词，可以对 words_of_file 再做一层包装：

    import re
    def words_by_re(thefilepath, repattern=r"[\w'-]+"):
      wre = re.compile(repattern)
      def line_to_words(line):
        for mo in wre.finditer(line):
          return mo.group(0)
      return words_of_file(thefilepath, line_to_words)

这里也给出了一种默认词定义的正则表达式定义，当然如果有必要使用其他不同的词定义，也可以传入不同的表达式。





# 遍历目录树

需要检查一个目录，或者某个包含子目录的目录树，并根据某种模式迭代所有的文件（也可能包含子目录)。

    import os, fnmatch
    def all_files(root, patterns='*', single_level=False, yield_folders=False):
      # 将模式从字符串中取出放入列表中
      patterns = patterns.split(';')
      for path, subdirs, files in os.walk(root):
        if yield_folders:
          files.extend(subdirs)
        files.sort()
        for name in files:
          for pattern in patterns:
            if fnmatch.fnmatch(name, pattern):
              yield os.path.join(path, name)
              break
          if single_level:
            break
    
    thefiles = list(all_files('/tmp', '*.py;*.htm;*.html'))

如果想一次处理一个文件的路径（比如逐行打印它们），不需要先建立一个列表：

    for path in all_files('/tmp', '*.py;*.htm;*.html'):
      print(path)





# 根据指定的搜索路径和模式寻找文件

给定一个搜索路径（一个描述目录信息的字符串），需要在此目录中找出所有符合匹配模式的文件。

实现时需要循环路径中的所有目录。这个循环最好被封装成一个生成器：

    import glob, os
    def all_files(pattern, search_path, pathsep=os.pathsep):
      """给定搜索路径，找出所有满足匹配条件的文件"""
      for path in search_path.split(pathsep):
        for match in glob.glob(os.path.join(path, pattern)):
          yield match

生成器的好处是可以很容易的获取第一个子项，或所有子项，或遍历其中的子项。

    # 打印第一个子项
    print(all_files('*.py', os.environ['PATH']).__next__())
    # 打印所有这种文件，一行一个：
    for match in all_files('*.py', os.environ['PATH']):
      print(match)
    # 以列表形式一次全部打印出来:
    print(list(all_files('*.py', os.environ['PATH'])))
    




# 在 Windows 平台上修改文件属性

需要修改 Windows 文件的属性，比如将某些文件设置为只读、归档等。

    import win32con, win32api, os
    # 创建一个文件，并展示如何操纵它
    thefile = 'test'
    f = open('test', 'w')
    f.close()
    # 设置成隐藏文件...：
    win32api.SetFileAttributes(thefile, win32con.FILE_ATTRIBUTE_HIDDEN)
    # 设置成只读文件
    win32api.SetFileAttributes(thefile, win32con.FILE_ATTRIBUTE_READONLY)
    # 为了刪除它先把它设成普通文件
    win32api.SetFileAttributes(thefile, win32con.FILE_ATTRIBUTE_NORMAL)
    # 最后删掉该文件
    os.remove(thefile)

`win32api.SetFileAttributes` 的一个有趣的用法是用来删除文件。用 `os.remove` 在 Windows 中删除非普通文件会遭遇失败。为了删除文件，必须先通过 `SetFileAttributes` 把该文件的属性设置为普通。

