---
layout: post
title: Python Codewars 训练笔记 2
tags:
- Python
- Programming
status: publish
type: post
published: true

summary: 'Codewars 网站上别人的优秀解法'

---



# Python Codewars 训练笔记 2

在 Codewars 练习中发现了许多别人做出的优秀解法, 记录下来, 这篇里都是比较长的练习题




### Befunge Interpreter

写一个 [Befunge-93 语言](https://en.wikipedia.org/wiki/Befunge) 的解释器, Befunge-93 的代码是一些单个字符, 分布在二维平面上, 游标开始时处在左上角, 默认将向右移动, 游标移动到终点时将循环至开头, 支持的操作符有:

- `0-9` Push this number onto the stack.
- `+` Addition: Pop `a` and `b`, then push `a+b`.
- `-` Subtraction: Pop `a` and `b`, then push `b-a`.
- `*` Multiplication: Pop `a` and `b`, then push `a*b`.
- `/` Integer division: Pop `a` and `b`, then push `b/a`, rounded down. If `a` is zero, push zero.
- `%` Modulo: Pop `a` and `b`, then push the `b%a`. If `a` is zero, push zero.
- `!` Logical NOT: Pop a value. If the value is zero, push `1`; otherwise, push zero.
- `` ` ``  Greater than: Pop `a` and `b`, then push `1` if `b>a`, otherwise push zero.
- `>` Start moving right.
- `<` Start moving left.
- `^` Start moving up.
- `v` Start moving down.
- `?` Start moving in a random cardinal direction.
- `_` Pop a value; move right if `value = 0`, left otherwise.
- `|` Pop a value; move down if `value = 0`, up otherwise.
- `"` Start string mode: push each character's ASCII value all the way up to the next `"`.
- `:` Duplicate value on top of the stack. If there is nothing on top of the stack, push a `0`.
- `\` Swap two values on top of the stack. If there is only one value, pretend there is an extra `0` on bottom of the stack.
- `$` Pop value from the stack and discard it.
- `.` Pop value and output as an integer.
- `,` Pop value and output the ASCII character represented by the integer code that is stored in the value.
- `#` Trampoline: Skip next cell.
- `p` A "put" call (a way to store a value for later use). Pop `y`, `x` and `v`, then change the character at the position `(x,y)` in the program to the character with ASCII value `v`.
- `g` A "get" call (a way to retrieve data in storage). Pop `y` and `x`, then push ASCII value of the character at that position in the program.
- `@` End program.
- ` ` (i.e. a space) No-op. Does nothing.

示例如下:

```
>987v>.v
v456<  :
>321 ^ _@
```

将打印出 `123456789`.

需要编写 interpret 函数, 如:

```
interpret('>987v>.v\nv456<  :\n>321 ^ _@') == '123456789'
```

自己的方案

```python
from random import randint

def interpret(code, debug=False):
  if debug: print(code)

  code = [list(line) for line in code.split('\n')]
  code_rows = len(code)
  code_cols = max(len(line) for line in code)
  output = ""
  stack = []
  cursor = (0, -1)
  current_direction = '>'
  numbers = list(str(i) for i in range(0, 10))
  directions = '^ v < > ?'.split()
  directions_mapping = { '^': (-1, 0), 'v': (1, 0), '<': (0, -1), '>': (0, 1)}
  math_operators = '+ - * / % `'.split()
  logic_operators = ['!']
  stack_operators = '$ . ,'.split()
  conditions = '_ |'.split()
  string_mode = False
  skip_mode = False

  while True:
    if current_direction == '?':
      current_direction = '^v<>'[randint(0,3)]
    cursor_row = cursor[0] + directions_mapping[current_direction][0]
    cursor_col = cursor[1] + directions_mapping[current_direction][1]
    cursor = (cursor_row % code_rows, cursor_col % code_cols)
    command = code[cursor[0]][cursor[1]]

    if string_mode:
      if command == '"': string_mode = False
      else: stack.append(ord(command))

    elif skip_mode:
      skip_mode = False

    elif command in numbers:
      stack.append(int(command))

    elif command in math_operators:
      a, b = stack.pop(), stack.pop()
      if command == '+': stack.append(a+b)
      if command == '-': stack.append(b-a)
      if command == '*': stack.append(a*b)
      if command == '/': stack.append(b//a if a!=0 else 0)
      if command == '%': stack.append(b%a if a!=0 else 0)
      if command == '`': stack.append(int(b>a))

    elif command in logic_operators:
      a = stack.pop()
      if command == '!': stack.append(int(a==0))

    elif command in directions:
      current_direction = command

    elif command in conditions:
      a = stack.pop()
      if command == '_': current_direction = '>' if a==0 else '<'
      if command == '|': current_direction = 'v' if a==0 else '^'

    elif command in stack_operators:
      a = stack.pop()
      if command == '$': pass
      if command == '.': output += str(a)
      if command == ',': output += chr(a)

    elif command == ':':
      if stack: stack.append(stack[-1])
      else: stack.append(0)

    elif command == '\\':
      if len(stack) >= 2: stack.insert(-1, stack.pop())
      else: stack = [0]

    elif command == ' ': pass
    elif command == '#': skip_mode = True
    elif command == '"': string_mode = True

    elif command == 'p':
      y, x, v = stack.pop(), stack.pop(), stack.pop()
      code[y][x] = chr(v)
    elif command == 'g':
      y, x = stack.pop(), stack.pop()
      stack.append(ord(code[y][x]))

    elif command == '@': return output

    if debug: print("{0} '{1}' stack: {3} output: {2}".format(cursor, command, output, stack))



from nose import tools as test
test.assert_equals(interpret('>987v>.v\nv456<  :\n>321 ^ _@'), '123456789')

hello_world_code = '''>25*"!dlroW olleH":v
                v:,_@
                >  ^'''
test.assert_equals(interpret(hello_world_code), 'Hello World!\n')

print_self_code = '01->1# +# :# 0# g# ,# :# 5# 8# *# 4# +# -# _@'
test.assert_equals(interpret(print_self_code), print_self_code)

random_code = '''v@.<
>1^
>?<^
>2^'''
test.assert_equals(interpret(random_code, debug=True), '1')
```

人家的方案

```python
from random import choice

def interpret(code):
    code = [list(l) for l in code.split('\n')]
    x, y = 0, 0
    dx, dy = 1, 0
    output = ''
    stack = []
    string_mode = False

    while True:
        move = 1
        i = code[y][x]

        if string_mode:
            if i == '"':
                string_mode = False
            else:
                stack.append(ord(i))
        else:

            if i.isdigit(): stack.append(int(i))
            elif i == '+': stack[-2:] = [stack[-2] + stack[-1]]
            elif i == '-': stack[-2:] = [stack[-2] - stack[-1]]
            elif i == '*': stack[-2:] = [stack[-2] * stack[-1]]
            elif i == '/': stack[-2:] = [stack[-2] and stack[-2] / stack[-1]]
            elif i == '%': stack[-2:] = [stack[-2] and stack[-2] % stack[-1]]
            elif i == '!': stack[-1] = not stack[-1]
            elif i == '`': stack[-2:] = [stack[-2] > stack[-1]]
            elif i in '><^v?':
                if i == '?':   i = choice('><^v')
                if i == '>':   dx, dy =  1,  0
                elif i == '<': dx, dy = -1,  0
                elif i == '^': dx, dy =  0, -1
                elif i == 'v': dx, dy =  0,  1
            elif i == '_': dx, dy = (-1 if stack.pop() else 1), 0
            elif i == '|': dx, dy = 0, (-1 if stack.pop() else 1)
            elif i == '"': string_mode = True
            elif i == ':': stack.append(stack[-1] if stack else 0)
            elif i == '\\': stack[-2:] = stack[-2:][::-1]
            elif i == '$': stack.pop()
            elif i == '.': output += str(stack.pop())
            elif i == ',': output += chr(stack.pop())
            elif i == '#': move += 1
            elif i == 'p':
                ty, tx, tv = stack.pop(), stack.pop(), stack.pop()
                code[ty][tx] = chr(tv)
            elif i == 'g':
                ty, tx = stack.pop(), stack.pop()
                stack.append(ord(code[ty][tx]))
            elif i == '@':
                return output

        for _ in range(move):
            x = (x + dx) % len(code[y])
            y = (y + dy) % len(code)

```




总结

- 描述位移时, 自己写的是 cursor, 语义虽然明确, 但是使用时还得 cursor[0], cursor[1], 不如人家直接用 dx dy 省心
- 描述方向转换时, 自己使用了 directions_mapping 做映射, 其实这个逻辑不太可能改变, 不如硬编码 (在 `elif i in '><^v?':` 那里), 而 mapping 应该用在有可能变更需求时
- Python 没 switch case, 在处理多条件分支时不容易看清楚结构, 当逻辑非常简单时, 考虑冒号不换行
- 从表中随机选一项, `random.choice(arr)`
- 当果逻辑比较简单时, 通篇写很多 `stack[-2]`, `stack[-1]` 也可以很快读懂, 不必起名一个新的变量, 相同的道理, `len(code[y])` `len(code)` 也不必起名新的变量





### 根据日程表找到大家共同的空闲时间

Person | Meetings
-------|-----------------------------------
A      | 09:00 - 11:30, 13:30 - 16:00, 16:00 - 17:30, 17:45 - 19:00
B      | 09:15 - 12:00, 14:00 - 16:30, 17:00 - 17:30
C      | 11:30 - 12:15, 15:00 - 16:30, 17:45 - 19:00

规则如下

- 输入输出时间都以24小时制的 `"hh:mm"` 表示
- 会议时间是左闭右开区间, 如 `09:00 - 11:00` 表示 `11:00` 是空闲的
- 找到的空闲时间必须位于 `09:00` (含) - `19:00` (不含) 之间
- 没有解就输出 None

自己的方案

```python
def delta(start, end):
  return 60*(int(end[:2]) - int(start[:2])) + (int(end[3:]) - int(start[3:]))

def free_times(schedule, day_start, day_end, duration):
  start = day_start
  for s, e in schedule:
    if delta(start, s) >= duration:
      yield [start, s]
    start = e
  if delta(start, day_end) >= duration:
    yield [start, day_end]

def merge(free_times1, free_times2, duration):
  range1, range2 = next(free_times1), next(free_times2)
  while True:
    common_start = max(range1[0], range2[0])
    common_end = min(range1[1], range2[1])
    if delta(common_start, common_end) >= duration:
      yield [common_start, common_end]
    if range1[1] < range2[1]:
      range1 = next(free_times1)
    elif range1[1] > range2[1]:
      range2 = next(free_times2)
    else:
      range1, range2 = next(free_times1), next(free_times2)

from functools import reduce

def get_start_time(schedules, duration):
  free_times_all = [free_times(s, '09:00', '19:00', duration) for s in schedules]

  for common_range in reduce(lambda x, y: merge(x, y, duration), free_times_all):
    return common_range[0]
  else:
    return None


# test
from nose import tools as test
schedules = [
  [['09:00', '11:30'], ['13:30', '16:00'], ['16:00', '17:30'], ['17:45', '19:00']],
  [['09:15', '12:00'], ['14:00', '16:30'], ['17:00', '17:30']],
  [['11:30', '12:15'], ['15:00', '16:30'], ['17:45', '19:00']]
]
test.assert_equals(delta('09:00', '11:30'), 150)
test.assert_equals(delta('09:15', '12:00'), 165)
test.assert_equals(delta('09:00', '09:00'), 0)

print(list(free_times(schedules[0], '09:00', '19:00', 60)))
print(list(free_times(schedules[1], '09:00', '19:00', 60)))
print(list(free_times(schedules[2], '09:00', '19:00', 60)))

test.assert_equals(get_start_time(schedules, 60), '12:15')
test.assert_equals(get_start_time(schedules, 90), None)


schedules = [
  [['09:00', '19:00']],
  [],
  [],
  []
]
test.assert_equals(get_start_time(schedules, 1), None)
```


人家的方案


```python
def translate_time(time): 
    return int(time.split(":")[0])*60+int(time.split(":")[1])
def translate_range(range_time): 
    return (translate_time(range_time[0]), translate_time(range_time[1]))
def get_start_time(schedules, duration, start=540, end=1140):
    for work in sorted([translate_range(item) for worker in schedules for item in worker]):
        if start + duration > work[0]: start = max(start, work[1])
        elif start + duration >= end: return None
        else: return "%02i:%02i" % (start/60, start%60)
    if start + duration <= end: return "%02i:%02i" % (start/60, start%60)
    return None
```

总结

- 人家这个简单实用, 但要把所有的日程放一起 sorted()
- 放弃一些细节, 把时间抽象为数字, 把多个日程合并为一个人





### 求最长的公共子序列 LCS (longest common subsequence)

接受两个序列作为参数, 返回最长的公共子序列

序列可以是不连续的, 跟子字符串不一样, 如 `Subsequences of "abc"` 包括 "a", "b", "c", "ab", "ac", "bc"

LCS 举例

    lcs( "abcdef" , "abc" ) => returns "abc"
    lcs( "abcdef" , "acf" ) => returns "acf"
    lcs( "132535365" , "123456789" ) => returns "12356"



自己的方案

```python
def lcs(x, y):
  if len(x) == 1:
    return x if x in y else ''
  elif len(y) == 1:
    return y if y in x else ''

  if x[-1] == y[-1]:
    return lcs(x[:-1], y[:-1]) + x[-1]
  else:
    p1 = lcs(x[:-1], y)
    p2 = lcs(x, y[:-1])
    return p1 if len(p1) > len(p2) else p2

from nose import tools as test
test.assert_equals(lcs("a", "b"), "")
test.assert_equals(lcs("abcdef", "abc"), "abc")
test.assert_equals(lcs("abcdefghijhklxyzuu", "xyzuuabcdefghijhkl"), "abcdefghijhkl")
print('me')
```

人家的方案

单词较长时, 这个办法更快

```python
import itertools
def lcs(x, y):
  for length in reversed(range(len(x)+1)):
    for xItem in itertools.combinations(x, length):
      for yItem in itertools.combinations(y,length):
        if xItem == yItem:
          return "".join(xItem)

```


总结

递归计算会超时的, 人家的方案也不是特别快, 还是得靠动态规划, 参考 [算法导论-最长公共子序列LCS（动态规划）](https://blog.csdn.net/so_geili/article/details/53737001)


> **最大子序列：**
> 
> 找出由数组成的一维数组中和最大的连续子序列。比如{5,-3,4,2}的最大子序列就是{5,-3,4,2}，它的和是8达到最大
> 
> 方法很简单，只要前i项的和还没有小于0那么子序列就一直向后扩展，否则丢弃之前的子序列开始新的子序列，同时记下各个子序列的和
> 
> **最长公共子串：**
> 
> 找两个字符串的最长公共子串，这个子串要求在原字符串中是连续的。是一个序贯决策问题，可以用动态规划求解。用二维矩阵记录中间结果。
> 
>     　　 b　　a　　b
>     c　　0　　0　　0
>     a　　0　　1　　0
>     b　　1　　0　　2
>     a　　0　　2　　0
> 
> 矩阵中的最大元素就是最长公共子串的长度。
> 
> **最长公共子序列LCS：**
> 
> 最长公共子序列与最长公共子串的区别在于最长公共子序列不要求在原字符串中是连续的，比如ADE和ABCDE的最长公共子序列是ADE。
> 
> 设C1是S1的最右侧字符，C2是S2的最右侧字符，S1'是从S1中去除C1的部分，S2'是从S2中去除C2的部分。
> 
> 则 `LCS(S1, S2)` 等于以下四个情况的最大值
> 
> 1. LCS(S1, S2')
> 2. LCS(S1', S2)
> 3. 如果C1不等于C2：LCS(S1', S2')
> 4. 如果C1等于C2：LCS(S1', S2')+C1

这个题不是求 lcs 长度, 而是把这个子序列打印出来, 故需要在状态转移 matrix 里记录构造路径, 当找到目标子序列后还原出来

更简单的办法, 直接在 matrix 里记录全路径

```python

def lcs(x, y):
  lenx = len(x)
  leny = len(y)
  if lenx * leny == 0:
    return ""
  
  data = [[None for _ in range(leny+1)] for _ in range(lenx+1)]

  for i in range(lenx+1):
    for j in range(leny+1):
      if i == 0 or j == 0:
        data[i][j] = 0, ''
        continue
      left, seq_left = data[i][j-1]
      top, seq_top = data[i-1][j]
      top_left, seq_top_left = data[i-1][j-1]
      if x[i-1] == y[j-1]:
        current = max(left, top, top_left+1)
        data[i][j] = current, seq_top_left + x[i-1]
      else:
        data[i][j] = max((left, seq_left), (top, seq_top), (top_left, seq_top_left))
  
  max_lcs = max(max(line) for line in data)
  print(max_lcs)
  return max_lcs[1]

from nose import tools as test
test.assert_equals(lcs("a", "b"), "")
test.assert_equals(lcs("abcdef", "abc"), "abc")
test.assert_equals(lcs("abcdefghijhklxyzuu", "xyzuuabcdefghijhkl"), "abcdefghijhkl")
print('dp')


```


