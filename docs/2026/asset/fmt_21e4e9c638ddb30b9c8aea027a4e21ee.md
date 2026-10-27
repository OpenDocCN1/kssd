
```java
In : def sequence(collection):
...: match collection:
...: case 1, [x, \*others]:
...: print(f"Got 1 and a nested sequence: {x=}, {others=}")
...: case (1, x):
...: print(f"Got 1 and {x}")
...: case [x, y, z]:
...: print(f"{x=}, {y=}, {z=}")
...:
In : sequence([1])
In : sequence([1, 2])
Got 1 and 2
In : sequence([1, 2, 3])
x=1, y=2, z=3
In : sequence([1, [2, 3]])
Got 1 and a nested sequence: x=2, others=[3]
In : sequence([1, [2, 3, 4]])
Got 1 and a nested sequence: x=2, others=[3, 4]
In : sequence([2, 3])
In : sequence((1, 2))
Got 1 and 2
```

### Mapping Patterns（映射模式）

为了效率，`key`必须是常量（`literals`、`value patterns`）

其实就是`case`后支持使用字典做匹配

```python
In : def mapping(config):
...: match config:
...: case {'sub': sub\_config, \*\*rest}:
...: print(f'Sub: {sub\_config}')
...: print(f'OTHERS: {rest}')
...: case {'route': route}:
...: print(f'ROUTE: {route}')
...:
In : mapping({})
In : mapping({'route': '/auth/login'})
ROUTE: /auth/login
# 匹配有sub键的字典，值绑定到sub\_config上，字典其他部分绑定到rest上
In : mapping({'route': '/auth/login', 'sub': {'a': 1}})
Sub: {'a': 1}
OTHERS: {'route': '/auth/login'}
```

```python
def change\_red\_to\_blue(json\_obj):
match json\_obj:
case { 'color': ('red' | '#FF0000') }:
json\_obj['color'] = 'blue'
case { 'children': children }:
for child in children:
change\_red\_to\_blue(child)
```

### Class Patterns（类模式）

Class Patterns主要实现两个目标：检查对象是某个类的实例、从对象的特定属性中提取数据。

```python
# case 后支持任何对象做匹配。我们先来一个错误的示例:
In : class Point:
...: def \_\_init\_\_(self, x, y):
...: self.x = x
...: self.y = y
...:
In : def class\_pattern(obj):
...: match obj:
...: case Point(x, y):
...: print(f'Point({x=},{y=})')
...:
In : class\_pattern(Point(1, 2))
---------------------------------------------------------------------------
TypeError Traceback (most recent call last)
Input In [], in <cell line: 1>()
----> 1 class\_pattern(Point(1, 2))
Input In [], in class\_pattern(obj)
1 def class\_pattern(obj):
2 match obj:
----> 3 case Point(x, y):
4 print(f'Point({x=},{y=})')
TypeError: Point() accepts 0 positional sub-patterns (2 given)
# 这是因为对于匹配来说， 位置需要确定 ，所以需要使用位置参数来标识:
In : def class\_pattern(obj):
...: match obj:
...: case Point(x=1, y=2):
...: print(f'match')
...:
In : class\_pattern(Point(1, 2))
match
# 另外一个解决这种自定义类不用位置参数的匹配方案，使用 \_\_match\_args\_\_ 返回一个位置参数的数组，
# 就像这样:
In : class Point:
...: \_\_match\_args\_\_ = ('x', 'y')
...:
...: def \_\_init\_\_(self, x, y):
...: self.x = x
...: self.y = y
...:
# 还有就是用dataclass, 这里的 Point2 使用了标准库的 dataclasses.dataclass 装饰器
# 它会提供 \_\_match\_args\_\_ 属性，所以可以直接用
In : from dataclasses import dataclass
In : @dataclass
...: class Point2:
...: x: int
...: y: int
...:
In : def class\_pattern(obj):
...: match obj:
...: case Point(x, y):
...: print(f'Point({x=},{y=})')
...: case Point2(x, y):
...: print(f'Point2({x=},{y=})')
...:
In : class\_pattern(Point(1, 2))
Point(x=1,y=2)
In : class\_pattern(Point2(1, 2))
Point2(x=1,y=2)
```

```python
def eval\_expr(expr):
"""Evaluate an expression and return the result."""
match expr:
case BinaryOp('+', left, right):
return eval\_expr(left) + eval\_expr(right)
case BinaryOp('-', left, right):
return eval\_expr(left) - eval\_expr(right)
case BinaryOp('\*', left, right):
return eval\_expr(left) \* eval\_expr(right)
case BinaryOp('/', left, right):
return eval\_expr(left) / eval\_expr(right)
case UnaryOp('+', arg):
return eval\_expr(arg)
case UnaryOp('-', arg):
return -eval\_expr(arg)
case VarExpr(name):
raise ValueError(f"Unknown value of: {name}")
case float() | int():
return expr
case \_:
raise ValueError(f"Invalid expression value: {repr(expr)}")
```

另外一个例子：

```python
match media\_object:
case Image(type="jpg"):
return media\_object
case Image(type="png") | Image(type="gif"):
return render\_as(media\_object, "jpg")
case Video():
raise ValueError("Can't extract frames from video yet")
case other\_type:
raise Exception(f"Media type {media\_object} can't be handled yet")
```

`namedtuple`例子，也属于是`class pattern`：

```python
from collections import namedtuple
Mov = namedtuple('mov', ['dst', 'src', 'sz', 'ridx'])
switch op:
case Mov(dst, src, 8, ridx):
pass
```

### Type Unions, Aliases, and Guards

`numbers`的类型指定为`List`，元素类型可以是`float`或`int`。

```python
def mean(numbers: list[float | int]) -> float:
return sum(numbers) / len(numbers)
```

可以定义类型别名，类型检查器和程序员都可以识别到这种模式：

```python
from typing import TypeAlias
Card: TypeAlias = tuple[str, str] # ('', '')
Deck: TypeAlias = list[Card] # [('', '')]
```

`Type guards`用于缩小`type union`的范围。

# new disassembler of 2020GKCTF-EzMachine

一般这种`disassembler`都是逐渐去优化的，优化到最后能使用`<mark class="encrypted">93aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2M7s2N6F1N6r3!0G2L8s2y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8X3q4K6L8g2)9J5k6h3S2@1L8h3I4Q4x3U0y4H3N6$3&6D9K9h3u0Q4x3X3g2S2M7$3#2Q4x3X3g2E0j5h3E0W2i4K6g2X3k6h3I4X3i4K6g2X3k6Y4u0G2L8g2)9#2k6X3q4K6M7$3g2E0j5X3I4&6</mark>`

![图片描述](upload/attach/202311/921830\_ZYSZKM6UA5GY9R5.webp)

直接装配成一个`elf`

## 1：建立指令类型，写出parse

| Opcode | Instruction | Notes |
| --- | --- | --- |
| 0 | nop | pc+=1 |
| 1 | mov dst, imm | imm is an 1-byte immediate value |
| 2 | push imm | imm is an 1-byte immediate value |
| 3 | push reg | put (1-byte)reg in stack |
| 4 | pop reg | pop (1-byte) from stack to reg |
| 5 | PrintStr | print str by edx: 0:'right', 1:'wrong', 3:'plz input:', 4:'hacker’ |
| 6 | add reg1, reg2 | reg1 += reg2 |
| 7 | sub reg1, reg2 | reg1 -= reg2 |
| 8 | mul reg1, reg2 | reg1 \*= reg2 |
| 9 | div reg1, reg2 | reg1 /= reg2 (Put the quotient into eax and the remainder into ebx) |
| 10 | xor reg1, reg2 | reg1 ^= reg2 |
| 11 | jmp addr | directly jump to address |
| 12 | cmp reg1, reg2 | edx=reg1-reg2 |
| 13 | jz addr | jump if edx==0（reg1==reg2） |
| 14 | jnz addr | jump if edx!=0（reg1≠reg2） |
| 15 | jg addr | jump if edx>0（reg1>reg2） |
| 16 | jl addr | jump if edx<0 （reg1 < reg2） |
| 17 | InputStr | gets(mem); eax=strlen(mem); |
| 18 | InitMem | memset(mem\_addr, 0, sz) |
| 19 | MovRegStack | mov reg, [ebp-src] |
| 20 | MovRegMem | mov reg, mem[src] |
| 0xff | Exit | exit(0) |

- `Ezmachine-disassembler-parsefunc.py`
