
# [原创]python310新特性 -> Structural Pattern Matching 在 VM 虚拟机逆向中的妙用

## 前言

这种写法最初是在 2022-GoogleCTF-eldar 中，国外的 DiceGang 的 hgarrereyn 师傅的 wp：<mark class="encrypted">952K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3g2Z5j5i4u0J5K9i4y4G2L8X3N6J5k6h3g2F1i4K6u0W2L8h3g2Q4x3V1j5J5x3o6t1J5i4K6u0r3k6$3!0G2k6$3I4W2j5%4c8X3i4K6u0r3k6h3I4V1j5i4u0Q4x3V1k6Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0W2Q4b7e0k6Q4z5e0k6Q4c8e0k6Q4b7f1y4Q4b7e0q4Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0c8Q4b7U0W2Q4z5f1k6Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0c8Q4b7V1q4Q4z5p5g2Q4c8e0S2Q4b7e0N6Q4b7e0y4Q4c8e0k6Q4z5f1g2Q4z5e0m8Q4c8e0S2Q4z5e0W2Q4z5f1q4Q4c8e0k6Q4z5p5u0Q4z5f1k6Q4c8e0k6Q4z5f1y4Q4b7V1q4Q4x3V1x3</mark>，不过是用于解析的 ELF metadata-driven turing weird machine。

然后后来国内的 2022-强网杯中的 deeprev 又将这个 ELF metadata-driven turing weird machine 出了一次，我也用其来写过这个重定位机的解析，确实效果非常好，可以说毫不夸张像魔法一样。

当时就在 Todolist 中写道，用 Structural Pattern Matching 这种新特性去写下正常虚拟机的解析，肯定属于是轻轻松松。后续工作上的事情就放下了没去完成，在我的 Todolist 中吃灰了接近一年，这一年都在被工作推着走，每天就像机器人一样去执行自己头天写的指令，记忆好像也变差了，经常忘事情，年末项目交付了一些了才有时间弄些自己的，创业之路真的很难。

言归正传，后续在 dicectf-2022-breach 这道题的 wp：<mark class="encrypted">bd5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6h3c8#2j5%4c8G2M7W2)9J5c8X3c8A6j5$3g2Q4x3X3c8U0N6r3k6Q4x3X3b7J5x3o6t1J5i4K6u0V1j5Y4u0W2j5h3y4Z5i4K6u0V1N6%4u0A6N6r3g2#2M7q4!0q4y4q4!0n7z5q4!0m8c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4!0m8x3W2!0m8b7W2!0q4y4W2!0m8c8q4!0m8x3#2!0q4y4g2!0n7b7#2)9^5c8W2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4q4!0n7b7g2)9^5c8g2!0q4y4q4!0n7b7g2)9^5y4W2!0q4z5q4!0m8y4#2!0m8x3#2!0q4y4W2)9&6c8g2)9&6x3q4!0q4y4g2!0n7z5q4!0n7z5q4!0q4z5q4!0m8y4#2)9^5y4q4!0q4z5q4)9&6z5g2)9&6b7g2!0q4y4W2)9^5b7W2)9&6c8W2!0q4y4W2)9&6b7#2!0n7b7g2!0q4x3#2)9^5x3q4)9^5x3R3`.`</mark>。

直至放到了今日，才回来写个这个。

其实虚拟机解析之前我在之前已经发过不少：

[[原创]对VM逆向的分析(CTF)(比较经典的一个虚拟机逆向题目)](https://bbs.kanxue.com/thread-267670.htm)

[[原创]处理VM的一种特殊方法和思路](https://bbs.kanxue.com/thread-268208.htm)

[[原创]CTF之自动化VM分析](https://bbs.kanxue.com/thread-269591.htm)

总结来说，这种方法属于是 disassembler 的升级版，远优于之前发的 disassembler。你说它优于 decompiler 吗？我无法给出一个肯定答案，毕竟 decompiler 属于一种抽象为高级语言的思路。

## python310 Structural Pattern Matching

### Learn Structural Pattern Matching

#### Structural Pattern Matching 介绍

<a href="elink@a03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6i4m8K6i4K6u0W2M7s2W2@1K9r3!0F1i4K6u0W2L8%4u0Y4i4K6u0r3M7r3g2H3i4K6u0V1x3o6j5K6y4q4)9J5c8R3`.`.">PEP 634 – Structural Pattern Matching: Specification</a>：介绍 `match` 语法和支持的模式

<a href="elink@090K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6i4m8K6i4K6u0W2M7s2W2@1K9r3!0F1i4K6u0W2L8%4u0Y4i4K6u0r3M7r3g2H3i4K6u0V1x3o6j5K6y4g2)9J5c8R3`.`.">PEP 635 – Structural Pattern Matching: Motivation and Rationale</a>：解释语法这么设计的理由

<a href="elink@7d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6i4m8K6i4K6u0W2M7s2W2@1K9r3!0F1i4K6u0W2L8%4u0Y4i4K6u0r3M7r3g2H3i4K6u0V1x3o6j5K6y4W2)9J5c8R3`.`.">PEP 636 – Structural Pattern Matching: Tutorial</a>：一个教程，介绍概念、语法和语义

`match` patterns：

```
Mapping patterns: match mapping structures like dictionaries.
Sequence patterns: match sequence structures like tuples and lists.
Capture patterns: bind values to names.
AS patterns: bind the value of subpatterns to names.
OR patterns: match one of several different subpatterns.
Wildcard patterns: match anything.
Class patterns: match class structures.
Value patterns: match values stored in attributes.
Literal patterns: match literal values.
```

#### Capture patterns（捕捉模式）

匹配一个模式，并绑定到一个 name

```python
def sum_list(numbers):
    match numbers:
        case []: # 匹配空列表
            return 0
        case [first, *rest]: # sequence pattern, 由两个 capture pattern 组成的 sequence pattern。*rest 匹配剩下的所有元素
            return first + sum_list(rest)
```

```python
def average(*args):
    match args:
        case [x, y]: # captures the two elements of a sequence
            return (x + y) / 2
        case [x]: # captures the only element of a sequence
            return x
        case []:
            return 0
        case a: # captures the entire sequence
            return sum(a) / len(a)
```

#### guards（向模式添加条件）

用来进一步限制匹配模式，如下：

```python
# 从小到大排序
def sort(seq):
    match seq:
        case [] | [_]: # 匹配空序列 [] 或者 非空列表中的任何单个元素 [_]
            return seq
        case [x, y] if x <= y:
            return seq
        case [x, y]:
            return [y, x]
        case [x, y, z] if x <= y <= z:
            return seq
        case [x, y, z] if x >= y >= z:
            return [z, y, x]
        case [p, *rest]:
            a = sort([x for x in rest if x <= p]) # 比 p 小的去排序
            b = sort([x for x in rest if p < x]) # 比 p 大的去排序
            return a + [p] + b
```

#### AS Patterns（as 模式）

给限制条件取别名，使其能够与 bind name 一起工作。

子模式在 `match` 语法里面是可以灵活组合的。

```python
In : def as_pattern(obj):
...:     match obj:
...:         case str() as s:
...:             print(f'Got str: {s=}')
...:         case [0, int() as i]:
...:             print(f'Got int: {i=}')
...:         case [tuple() as tu]:
...:             print(f'Got tuple: {tu=}')
...:         case list() | set() | dict() as iterable:
...:             print(f'Got iterable: {iterable=}')
...:
...:
In : as_pattern('sss')
Got str: s='sss'
In : as_pattern([0, 1])
Got int: i=1
In : as_pattern([(1,)])
Got tuple: tu=(1,)
In : as_pattern([1, 2, 3])
Got iterable: iterable=[1, 2, 3]
In : as_pattern({'a': 1})
Got iterable: iterable={'a': 1}
```

```python
def simplify_expr(tokens):
    match tokens:
        case [('('|'[') as l, *expr, (')'|']') as r] if (l+r) in ('()', '[]'):
            return simplify_expr(expr)
        case [0, ('+'|'-') as op, right]:
            return UnaryOp(op, right)
        case [(int() | float() as left) | Num(left), '+', (int() | float() as right) | Num(right)]:
            return Num(left + right)
        case [(int() | float()) as value]:
            return Num(value)
```

#### OR Patterns（或模式）

第一种写法，用逗号分隔：

```python
case 401, 403, 404:
    print("Some HTTP error")
```

第二种写法与 C 语言类似：

```python
case 401:
case 403:
case 404:
    print("Some HTTP error")
```

第三种写法：

```python
case in 401, 403, 404:
    print("Some HTTP error")
```

第四种写法：

```python
case ("a"|"b"|"c"):
```

第五种写法：

```python
case ("a"|"b"|"c") as letter:
```

#### Literal Patterns（字面量模式）

使用 Python 自带的基本数据结构，如字符串、数字、布尔值和 `None` 等。

```python
match number:
    case 0:
        print('zero')
    case 1:
        print('one')
    case 2:
        print('two')
```

```python
def simplify(expr):
    match expr:
        case ('+', 0, x): # x + 0
            return x
        case ('+' | '-', x, 0): # x +- 0
            return x
        case ('and', True, x): # True and x
            return x
        case ('and', False, x):
            return False
        case ('or', False, x):
            return x
        case ('or', True, x):
            return True
        case ('not', ('not', x)):
            return x
        return expr
```

#### Wildcard Pattern（通配符模式）

Wildcard Pattern 是一种特殊的 capture pattern，它**接收任何值，但是不将该值绑定到任何一个变量**（其实就是忽略不关心的位置）。

```python
def is_closed(sequence):
    match sequence:
        case [_]: # any sequence with a single element
            return True
        case [start, *_, end]: # a sequence with at least two elements
            return start == end
        case _: # anything
            return False
```

#### Value Patterns（值模式）

这种模式主要匹配常量或者 `enum` 模块的枚举值：

```python
In : class Color(Enum):
...:     RED = 1
...:     GREEN = 2
...:     BLUE = 3
...:
In : class NewColor:
...:     YELLOW = 4
...:
In : def constant_value(color):
...:     match color:
...:         case Color.RED:
...:             print('Red')
...:         case NewColor.YELLOW:
...:             print('Yellow')
...:         case new_color:
...:             print(new_color)
...:
In : constant_value(Color.RED)  # 匹配第一个 case
Red
In : constant_value(NewColor.YELLOW)  # 匹配第二个 case
Yellow
In : constant_value(Color.GREEN)  # 匹配第三个 case
Color.GREEN
In : constant_value(4)  # 常量值一样都匹配第二个 case
Yellow
In : constant_value(10)  # 其他常量
10
```

这里注意，因为 `case` 具有绑定的作用，所以不能直接使用 `YELLOW` 这种常量，例如下面这样：

```python
YELLOW = 4
def constant_value(color):
    match color:
        case YELLOW:
            print('Yellow')
# 这样语法是错误的
```

就是在模式中使用其他变量的值，那么使用的其他变量与 capture 模式的绑定名如何区分呢？用 `.` 区分。

目前只能使用带 `.` 的常量。

```python
class Codes:
    SUCCESS = 200
    NOT_FOUND = 404

def handle(retcode):
    match retcode:
        case Codes.SUCCESS:
            print('success')
        case Codes.NOT_FOUND:
            print('not found')
        case _:
            print('unknown')
```

#### Sequence Patterns（序列模式）

可以在 `match` 里使用列表或者元组格式的结果。

不区分 `[a, b, c]`、`(a, b, c)` 和 `a, b, c`，它们是等价的，若要明确判断类型则需要 `list([a, b, c])`。

加星号的模式会匹配任意长度的元素，例如 `(*, 3, *)`，匹配任何含有 3 的列表。

不会迭代整个迭代器，所有的元素以下标和切片的形式访问。
