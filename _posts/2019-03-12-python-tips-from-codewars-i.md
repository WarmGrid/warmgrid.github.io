---
layout: post
title: Python Codewars 训练笔记 1
tags:
- Python
- Programming
status: publish
type: post
published: true

summary: 'Codewars 网站上别人的优秀解法'

---



# Python Codewars 训练笔记


在 Codewars 练习中发现了许多别人做出的优秀解法, 记录下来




### TIPS

这部分是可能常用的代码片段



求多个整数的最小公倍数

```python
from fractions import gcd  # 最大公约数
def lcm(*args):
    return reduce(lambda x, y: (x * y) // gcd(x, y), args, 1)

lcm(24, 15, 36)
```


正则 match 匹配字符串开头

```python
import re
mod4 = re.compile('.*\[[+-]?([048]|\d*([02468][048]|[13579][26]))\]')

# 正则表达式如果使用 match 方法, 必须把开头部分匹配上, 通常需要 r'.*foobar'

# test
mod4.match(test)
```


实现乘法约简 (类似于sum, 但针对乘法而非加法)

```python
from functools import reduce  # python 3 之后不推荐reduce 所以需要从functools里导入
n = 10
reduce(int.__mul__, range(1, n+1), 1)
```


连续使用列表推导式中的 if

```python
cells[i][j] for i in range(x-1, x+2) for j in range(y-1, y+2)
    if (i, j) != (x, y) if i < len(cells) and i >= 0 if j < len(cells[0]) and j >= 0
```
语法上支持这么写, 但是这样太乱了



字符串与矩阵的转换

```python
def str_to_matrix(str):
    return [[int(c) for c in str[i:i+9]] for i in range(0, len(str), 9)]
def matrix_to_str(matrix):
    return ''.join(str(c) for row in matrix for c in row)
```


xor 优先级低, 需要加括号

```python
n - (n^nim_sum)
```


双重循环的列表推导应该是先写大循环再写小循环, 如果两个循环无关, 可以随便顺序

```python
triplets = [
  ['t', 'u', 'p'],
  ['w', 'h', 'i'],
  ['t', 's', 'u'],
  ['a', 't', 's'],
  ['h', 'a', 'p'],
  ['t', 'i', 's'],
  ['w', 'h', 's']]
letters1 = list(set([l for t in triplets for l in t]))
print(letters1)  # works
letters2 = list(set([l for l in t for t in triplets]))
print(letters2)  # NameError: name 't' is not defined
```








### 根据两个数值大小返回打赢打输打平三种文字结果

自己的方案

```python
result = 'Battle Result: '
if good_score < evil_score:
    return result + 'Evil eradicates all trace of Good'
elif good_score > evil_score:
    return result + 'Good triumphs over Evil'
else:
    return result + 'No victor on this battle field'
```

人家的方案

```python
results = ['Evil eradicates all trace of Good', 'No victor on this battle field', 'Good triumphs over Evil']
return 'Battle Result: ' + results[cmp(good_score, evil_score) + 1]
```

总结

- `cmp(good_score, evil_score)` 也许之后不写 +1 也行, 这样的话平局放在 list 的第一个位置, good > evil 第二个, good < evil 第三个

在条件逻辑不太会变时可以用用, 牺牲一定的可读性, 省很多行数




### 计算单词中有重复两次及以上字符的个数

自己的方案

```python
from collections import Counter
def duplicate_count(text):
    c = Counter(text.lower())
    return sum(1 for k, v in c.items() if v >= 2)
```


人家的方案

```python
def duplicate_count(s):
    return len([c for c in set(s.lower()) if s.lower().count(c) > 1])
```

总结

- list 自带 `count` 方法
- 统计个数直接用 `len([x for x in l if ... ])` 即可, 不需要 sum





### 按照 i18n 的风格缩写单词

要求: 含三个及以下字母的单词不缩写, 单词间可能是各种标点


自己的方案

```python
import re
def replace(x):
    x = x.group()
    if len(x) <= 3: return x
    else: return x[0] + str(len(x)-2) + x[-1]
def abbreviate(s):
    return re.sub(r'\w+', replace, s)
```


人家的方案

```python
import re
def abbreviate(s):
    ab = lambda w: w[:1] + str(len(w) - 2) + w[-1:]
    return re.sub(r'\w{4,}', lambda m: ab(m.group(0)), s)
```

总结

- 忘了 `r'\w{4,}'` 的写法了
- 函数内的简单函数可以使用 `lambda`
- 正则替换时如果"替换文本"是个函数, 那么这个函数接受的参数是 match 对象, `match.group()` 是匹配到的值





### 返回 True False 检测值

自己的方案

```python
def is_valid(cls, s):
    """returns True if s is a valid MongoID; otherwise False"""
    s = str(s)
    if len(s) == 24 and re.match(r'[0-9a-f]+$', s):
      return True
    else:
      return False

def get_timestamp(cls, s):
    """if s is a MongoID, returns a datetime object for the timestamp; otherwise False"""
    if not cls.is_valid(s):
      return False
    return datetime.fromtimestamp(int(s[:8], 16))
```


人家的方案

```python
def is_valid(cls, s):
    return isinstance(s, str) and bool(re.match(r'[0-9a-f]{24}$', s))

def get_timestamp(cls, s):
    return cls.is_valid(s) and datetime.fromtimestamp(int(s[:8], 16))
```

总结

- 再也不要写 `if ...: return True; else: return False`
- 要灵活运用 and





### 深度优先和广度优先遍历

深度优先算法：

1. 访问初始顶点v并标记顶点v已访问。
2. 查找顶点v的第一个邻接顶点w。
3. 若顶点v的邻接顶点w存在，则继续执行；否则回溯到v，再找v的另外一个未访问过的邻接点。
4. 若顶点w尚未被访问，则访问顶点w并标记顶点w为已访问。
5. 继续查找顶点w的下一个邻接顶点wi，如果v取值wi转到步骤3。直到连通图中所有顶点全部访问过为止。

广度优先算法：

1. 顶点v入队列。
2. 当队列非空时则继续执行，否则算法结束。
3. 出队列取得队头顶点v；访问顶点v并标记顶点v已被访问。
4. 查找顶点v的第一个邻接顶点col。
5. 若v的邻接顶点col未被访问过的，则col入队列。
6. 继续查找顶点v的另一个新的邻接顶点col，转到步骤5。直到顶点v的所有未被访问过的邻接点处理完。转到步骤2。


```python
class Graph(object):
    def __init__(self, *args, **kwargs):
        self.node_neighbors = {}
        self.visited = {}
    def add_nodes(self, nodelist):
        for node in nodelist:
            self.add_node(node)
    def add_node(self, node):
        if not node in self.nodes():
            self.node_neighbors[node] = []
    def add_edge(self, edge):
        u, v = edge
        if (v not in self.node_neighbors[u]) and (u not in self.node_neighbors[v]):
            self.node_neighbors[u].append(v)
            if u != v:
                self.node_neighbors[v].append(u)
    def nodes(self):
        return self.node_neighbors.keys()
    def depth_first_search(self, root=None):
        order = []
        def dfs(node):
            self.visited[node] = True
            order.append(node)
            for n in self.node_neighbors[node]:
                if not n in self.visited:
                    dfs(n)
        if root:
            dfs(root)
        for node in self.nodes():
            if not node in self.visited:
                dfs(node)
        print(order)
        return order
    def breadth_first_search(self, root=None):
        queue = []
        order = []
        def bfs():
            while len(queue)> 0:
                node  = queue.pop(0)

                self.visited[node] = True
                for n in self.node_neighbors[node]:
                    if (not n in self.visited) and (not n in queue):
                        queue.append(n)
                        order.append(n)
        if root:
            queue.append(root)
            order.append(root)
            bfs()
        for node in self.nodes():
            if not node in self.visited:
                queue.append(node)
                order.append(node)
                bfs()
        print(order)
        return order


if __name__ == '__main__':
    g = Graph()
    g.add_nodes([i+1 for i in range(8)])
    g.add_edge((1, 2))
    g.add_edge((1, 3))
    g.add_edge((2, 4))
    g.add_edge((2, 5))
    g.add_edge((4, 8))
    g.add_edge((5, 8))
    g.add_edge((3, 6))
    g.add_edge((3, 7))
    g.add_edge((6, 7))
    print("nodes:", g.nodes())

    order = g.breadth_first_search(1)
    order = g.depth_first_search(1)

    # nodes: [1, 2, 3, 4, 5, 6, 7, 8]
    # 广度优先：
    # [1, 2, 3, 4, 5, 6, 7, 8]
    # 深度优先：
    # [1, 2, 4, 8, 5, 3, 6, 7]
```

还有递归的办法, 比上述迭代办法更容易理解





### 康威的生命游戏

使用最基本的生命游戏规则, 棋盘格周边8个格子中:

- neighbours < 2: 死于稀疏
- neighbours = 2: 生死状态不变
- neighbours = 3: 下一轮新生
- neighhours > 3: 死于拥挤


自己的方案

```python
def neighbours(i, j, padding_cells):
    return [padding_cells[i+a][j+b] for a in (-1, 0, 1) for b in (-1, 0, 1) if not (a == 0 and b == 0)]

def next_state(current_state, neighbours):
    live_neighbours = sum(neighbours)
    if live_neighbours < 2: return 0
    elif live_neighbours == 2: return current_state
    elif live_neighbours == 3: return 1
    else: return 0

def next_gen(cells):
    if not cells: return []
    padding_cells = [line+[0] for line in cells]
    padding_cells.append([0]*(len(cells[0])+1))
    for i, line in enumerate(cells):
        for j, cell_state in enumerate(line):
            cells[i][j] = next_state(cell_state, neighbours(i, j, padding_cells))
    return cells
```


人家的方案

```python
def next_gen(cells):
    N = len(cells)
    M = len(cells[0]) if N else 0
    def count_neighbours(i, j):
        indices = ((i+di,j+dj) for di in (-1,0,1) for dj in (-1,0,1) if di!=0 or dj!=0)
        return sum(cells[k][l] for (k,l) in indices if 0<=k<N and 0<=l<M)
    return [[(2 if cells[i][j] else 3)<=count_neighbours(i,j)<=3 for j in xrange(M)] for i in xrange(N)]
```

总结

- 处理边缘时要看情况, 有时候不需要把虚拟元素做出来, 如本题中超出区块的都算做0, 所以不要考虑, 只是计算总和就行
- 开平方可以不用导入 `math.sqrt` 可以这样 `number ** 0.5`





### 计算汉明数(Hamming Number)

Hamming 数是形如 `2^i * 3^j * 5^k` 的数, 不含其他质因子

```python
def merge(seqx, seqy):
  """可以把多个序列合并为一个序列"""
  x, y = next(seqx), next(seqy)
  while True:
    if x < y:
      yield x
      x = next(seqx)
    elif x > y:
      yield y
      y = next(seqy)
    else:
      yield x
      x, y = next(seqx), next(seqy)

def hamming(n):
  results = [1]
  def seq2():
    for i in results: yield 2*i
  def seq3():
    for i in results: yield 3*i
  def seq5():
    for i in results: yield 5*i
  for i, num in enumerate(merge(merge(seq2(), seq3()), seq5())):
    results.append(num)
    if i+2 == n:
      return num

print(hamming(10000))
```

总结

- 考虑问题时要留意是否可以使用 reduce
- 基于 yield 实现的 merge 函数很实用





### 按顺时针螺旋顺序输出一个nxn矩阵的元素


自己的方案

```python
def snail(array):
  n = len(array)

  if n == 1: return [array[0][0]] if array[0] else []
  elif n == 2: return [array[0][0], array[0][1], array[1][1], array[1][0]]
  else:
    shell = array[0] + [array[i][-1] for i in range(1, n-1)] + list(reversed(array[-1])) + [array[i][0] for i in range(n-2, 0, -1)]
    body_array = [[num for num in line[1:-1]] for line in array[1:-1]]
    return shell + snail(body_array)


from nose import tools as test
array = [[1,2,3],
         [4,5,6],
         [7,8,9]]
expected = [1,2,3,6,9,8,7,4,5]
test.assert_equals(snail(array), expected)

array = [[1,2,3],
         [8,9,4],
         [7,6,5]]
expected = [1,2,3,4,5,6,7,8,9]
test.assert_equals(snail(array), expected)
```


人家的方案

```python
# 但他这个会反复的转置矩阵, 性能差点
def snail(array):
    return list(array[0]) + snail(list(zip(*array[1:]))[::-1]) if array else []
```





### 在有向有环图中的找出环上的节点数

输入一个链表, 该链表必然含有一个环路, 需要找出这个环路上的节点的数量

人家的龟兔赛跑式解决方案, 空间复杂度O(1)

```python
def loop_size(node):
    turtle, rabbit = node.next, node.next.next

    # Find a point in the loop.  Any point will do!
    # Since the rabbit moves faster than the turtle
    # and the kata guarantees a loop, the rabbit will
    # eventually catch up with the turtle.
    while turtle != rabbit:
        turtle = turtle.next
        rabbit = rabbit.next.next

    # The turtle and rabbit are now on the same node,
    # but we know that node is in a loop.  So now we
    # keep the turtle motionless and move the rabbit
    # until it finds the turtle again, counting the
    # nodes the rabbit visits in the mean time.
    count = 1
    rabbit = rabbit.next
    while turtle != rabbit:
        count += 1
        rabbit = rabbit.next

    # voila
    return count
```





### 计算所有可能的密码

以保险箱的观测按键序列计算所有可能的密码, 观测到的按键的真实值是本身+其上下左右的数字

如观测到 4 则实际按键可能是 1 4 5 7

    ┌───┬───┬───┐
    │ 1  │ 2  │ 3  │
    ├───┼───┼───┤
    │ 4  │ 5  │ 6  │
    ├───┼───┼───┤
    │ 7  │ 8  │ 9  │
    └───┼───┼───┘
         │ 0  │
         └───┘


自己的方案

```python
def get_pins(observed):
  possibility = {
    '0': ('0', '8'),
    '1': ('1', '2', '4'),
    '2': ('1', '2', '3', '5'),
    '3': ('2', '3', '6'),
    '4': ('1', '4', '5', '7'),
    '5': ('2', '4', '5', '6', '8'),
    '6': ('3', '5', '6', '9'),
    '7': ('4', '7', '8'),
    '8': ('5', '7', '8', '9', '0'),
    '9': ('6', '8', '9'),
  }
  if len(observed) == 1:
    return possibility[observed]
  else:
    ret = []
    for item in get_pins(observed[:-1]):
      ret.extend(item+letter for letter in possibility[observed[-1]])
    return ret

from nose import tools as test
# test.describe('example tests')
expectations = [('8', ['5','7','8','9','0']),
                ('11',["11", "22", "44", "12", "21", "14", "41", "24", "42"]),
                ('369', ["339","366","399","658","636","258","268","669","668","266",
                         "369","398","256","296","259","368","638","396","238","356",
                         "659","639","666","359","336","299","338","696","269","358",
                         "656","698","699","298","236","239"])
               ]
for tup in expectations:
  test.assert_equals(sorted(get_pins(tup[0])), sorted(tup[1]), 'PIN: ' + tup[0])
```


人家的方案

```python
from itertools import product
ADJACENTS = ('08', '124', '2135', '326', '4157', '52468', '6359', '748', '85790', '968')
def get_pins(observed):
    return [''.join(p) for p in product(*(ADJACENTS[int(d)] for d in observed))]


```

总结

- `itertools.product(*iterables[, repeat])`, 笛卡尔积 product, 相当于多个嵌套的 for 循环, `product(A, B)` 与 `((x, y) for x in A for y in B)` 等同
- 如果 key 都是 int 数值, 那么 dict 与 list 功能差不多, list 的位置就蕴含了 key 的信息, 可以相互替代





### 使用0和1画蛇形方阵, 顺时针旋转

形状是一排 1 挨着一排 0, 所有的 0 和 1 都是顺时针卷曲

自己的方案

```python
def transpose(data):
  return list(map(list, zip(*reversed(data))))

def spiralize(size):
  if size == 1:
    return [[1]]
  if size == 2:
    return [[1,1],
            [0,1]]
  if size == 3:
    return [[1,1,1],
            [0,0,1],
            [1,1,1]]
  if size == 4:
    return [[1,1,1,1],
            [0,0,0,1],
            [1,0,0,1],
            [1,1,1,1]]
  else:
    core = spiralize(size-4)
    for n in [size-4, size-2, size-2, size]:
      core = transpose(core)
      core.insert(0, [0]*(n-1)+[1])
      core.insert(0, [1]*n)
    return core


from pprint import pprint
pprint(spiralize(8))
pprint(spiralize(11))
```


人家的方案

```python
# 先作出一层1一层0的矩阵, 然后修正细节
def spiralize(size):
    spiral = [[1 - min(i,j,size-max(i,j)-1)%2 for j in range(size)] for i in range(size)]
    for i in range(size//2-(size%4==0)):
      spiral[i+1][i] = 1 - spiral[i+1][i]
    return spiral
```





### 解决数独问题

每个题目有唯一确定解, 不需要猜测

自己的方案

```python
def sudoku(puzzle):
  numbers = set(range(1, 10))
  def line(i): return set(puzzle[i])
  def vline(i): return set(line[i] for line in puzzle)
  def block(row, col):
    cols = range(col//3*3, col//3*3+3)
    rows = range(row//3*3, row//3*3+3)
    return set(puzzle[r][c] for c in cols for r in rows)

  def unsolved(): return [line for line in puzzle if 0 in line]
  def probability(row, col): return numbers - line(row) - vline(col) - block(row, col)

  def fill_numbers():
    for col in range(0, 9):
      for row in range(0, 9):
        if puzzle[row][col] == 0:
          options = probability(row, col)
          if len(options) == 1:
            puzzle[row][col] = options.pop()

  while unsolved():
    fill_numbers()
  return puzzle


puzzle = [[5,3,0,0,7,0,0,0,0],
          [6,0,0,1,9,5,0,0,0],
          [0,9,8,0,0,0,0,6,0],
          [8,0,0,0,6,0,0,0,3],
          [4,0,0,8,0,3,0,0,1],
          [7,0,0,0,2,0,0,0,6],
          [0,6,0,0,0,0,2,8,0],
          [0,0,0,4,1,9,0,0,5],
          [0,0,0,0,8,0,0,7,9]]

solution = [[5,3,4,6,7,8,9,1,2],
            [6,7,2,1,9,5,3,4,8],
            [1,9,8,3,4,2,5,6,7],
            [8,5,9,7,6,1,4,2,3],
            [4,2,6,8,5,3,7,9,1],
            [7,1,3,9,2,4,8,5,6],
            [9,6,1,5,3,7,2,8,4],
            [2,8,7,4,1,9,6,3,5],
            [3,4,5,2,8,6,1,7,9]]

test.assert_equals(sudoku(puzzle), solution, "Incorrect solution for puzzle: " + str(puzzle));
```


人家的方案

```python
def sudoku(puzzle):
    while [l for l in puzzle if 0 in l]:
        for x, y in ((x, y) for x, l in enumerate(puzzle)
                     for y in xrange(len(l)) if not puzzle[x][y]):
            opt = set(xrange(1, 10)) - set(puzzle[x]) \
                  - {l[y] for l in puzzle} \
                  - {puzzle[m][n]
                     for m in xrange(x - (x % 3), x - (x % 3) + 3)
                     for n in xrange(y - (y % 3), y - (y % 3) + 3)}
            if len(opt) == 1:
                puzzle[x][y] = opt.pop()
                break
    return puzzle
```

总结

- set 可以用 pop 方便取出其中的值 `s = set([1]); s.pop()`
- 创建 set 可以用迭代器 `numbers = set([1,2,3,4,5,6,7,8,9])` 和 `numbers = set(range(1, 10))` 都行
- 在二维数组内判断是否出现了某个元素, 可以用表达式 `[line for line in matrix if elem in line]`





### 写一个加减乘除计算器

输入一个字符串 (类似 "2 / 2 + 3 * 4 - 6") 返回计算结果

需要处理算术优先级


自己的方案

```python
import re
from decimal import Decimal
class Calculator:
  def evaluate(self, string):
    tokens = [t if t in '+-*/' else Decimal(t) for t in re.findall(r'\d+\.\d+|\d+|[+\-*/]', string)]
    while len(tokens) > 1:
      i = self.position(tokens)
      v1, op, v2 = tokens[i-1:i+2]
      if op == '+': v3 = v1 + v2
      if op == '-': v3 = v1 - v2
      if op == '*': v3 = v1 * v2
      if op == '/': v3 = v1 / v2
      tokens[i-1:i+2] = [v3]

    return float(tokens[0])

  def position(self, tokens):
    priority_high = '*/'
    priority_low = '+-'
    for i, t in enumerate(tokens):
      if isinstance(t, str) and t in priority_high:
        return i
    for i, t in enumerate(tokens):
      if isinstance(t, str) and t in priority_low:
        return i

from nose import tools as test
test.assert_equals(Calculator().evaluate("2 / 2 + 3 * 4 - 6"), 7)
test.assert_equals(Calculator().evaluate("1.1 * 2.2 * 3.3"), 7.986)
```


人家的方案 1

```python
from operator import add, sub, mul, truediv
FIRST = {'*': mul, '/': truediv}
SECOND = {'+': add, '-': sub}
class Calculator:
    def evaluate(self, string):
        tokens = [float(t) if t.isdigit() or '.' in t else t for t in string.split()]
        while True:
            for (i, token) in enumerate(tokens):
                # op = FIRST.get(token) 这样匹配操作符很有想法
                # 如果能找到字符, 顺便连 operator 也赋值了
                op = FIRST.get(token)
                if op:
                    # 直接对数组动手术
                    tokens[i-1:i+2] = [op(tokens[i-1], tokens[i+1])]
                    break
            else:
                ret = tokens[0]
                # 类似 reduce
                for i in range(1, len(tokens), 2):
                    ret = SECOND[tokens[i]](ret, tokens[i+1])
                return ret if ret != 7.986000000000001 else 7.986  # Bug in test
```


人家的方案 2

```python
from operator import sub, mul, add, truediv
from decimal import Decimal
OPFUN = {"+": add, "-": sub, "*": mul, "/": truediv}
class Calculator:
    def eval_list(self, parts):
        for oplist in [["+", "-"], ["*", "/"]]:  # 反着计算优先级! 这样递归时就会先算乘除, 然后加减
            for index in reversed(range(len(parts))):
                part = parts[index]
                if part in oplist:
                    return OPFUN[part](self.eval_list(parts[:index]), self.eval_list(parts[index+1:]))
        if len(parts) == 1:
            return Decimal(parts[0])

    def evaluate(self, string):
        return float(self.eval_list(string.split(" ")))
```

总结

- 递归时, 递归到尽头的表达式是优先计算的, 可以利用它来调整优先级
- 可以替换数组中的片段 list[a:b] = new_list



