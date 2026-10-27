
`case` 后支持任何对象做匹配。我们先来一个错误的示例：

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

def class_pattern(obj):
    match obj:
        case Point(x, y):
            print(f'Point({x=},{y=})')

class_pattern(Point(1, 2))
```

运行结果：

```text
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Input In [], in <cell line: 1>()
----> 1 class_pattern(Point(1, 2))

Input In [], in class_pattern(obj)
      1 def class_pattern(obj):
      2     match obj:
----> 3         case Point(x, y):
      4             print(f'Point({x=},{y=})')

TypeError: Point() accepts 0 positional sub-patterns (2 given)
```

这是因为对于匹配来说，位置需要确定，所以需要使用位置参数来标识：

```python
def class_pattern(obj):
    match obj:
        case Point(x=1, y=2):
            print(f'match')

class_pattern(Point(1, 2))
```

输出：

```text
match
```

另外一个解决这种自定义类不用位置参数的匹配方案，使用 `__match_args__` 返回一个位置参数的数组，就像这样：

```python
class Point:
    __match_args__ = ('x', 'y')

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

还有就是用 `dataclass`。这里的 `Point2` 使用了标准库的 `dataclasses.dataclass` 装饰器，它会提供 `__match_args__` 属性，所以可以直接用：

```python
from dataclasses import dataclass

@dataclass
class Point2:
    x: int
    y: int

def class_pattern(obj):
    match obj:
        case Point(x, y):
            print(f'Point({x=},{y=})')
        case Point2(x, y):
            print(f'Point2({x=},{y=})')

class_pattern(Point(1, 2))
class_pattern(Point2(1, 2))
```

输出：

```text
Point(x=1,y=2)
Point2(x=1,y=2)
```

```python
def eval_expr(expr):
    """Evaluate an expression and return the result."""
    match expr:
        case BinaryOp('+', left, right):
            return eval_expr(left) + eval_expr(right)
        case BinaryOp('-', left, right):
            return eval_expr(left) - eval_expr(right)
        case BinaryOp('*', left, right):
            return eval_expr(left) * eval_expr(right)
        case BinaryOp('/', left, right):
            return eval_expr(left) / eval_expr(right)
        case UnaryOp('+', arg):
            return eval_expr(arg)
        case UnaryOp('-', arg):
            return -eval_expr(arg)
        case VarExpr(name):
            raise ValueError(f"Unknown value of: {name}")
        case float() | int():
            return expr
        case _:
            raise ValueError(f"Invalid expression value: {repr(expr)}")
```

另外一个例子：

```python
match media_object:
    case Image(type="jpg"):
        return media_object
    case Image(type="png") | Image(type="gif"):
        return render_as(media_object, "jpg")
    case Video():
        raise ValueError("Can't extract frames from video yet")
    case other_type:
        raise Exception(f"Media type {media_object} can't be handled yet")
```

`namedtuple` 例子，也属于是 class pattern：

```python
from collections import namedtuple
Mov = namedtuple('mov', ['dst', 'src', 'sz', 'ridx'])
switch op:
case Mov(dst, src, 8, ridx):
    pass
```

### Type Unions, Aliases, and Guards

`numbers` 的类型指定为 `List`，元素类型可以是 `float` 或 `int`：

```python
def mean(numbers: list[float | int]) -> float:
    return sum(numbers) / len(numbers)
```

可以定义类型别名，类型检查器和程序员都可以识别到这种模式：

```python
from typing import TypeAlias

Card: TypeAlias = tuple[str, str]  # ('', '')
Deck: TypeAlias = list[Card]  # [('', '')]
```

**Type guards** 用于缩小 type union 的范围。

# new disassembler of 2020GKCTF-EzMachine

一般这种 disassembler 都是逐渐去优化的，优化到最后能使用 [c20K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2M7s2N6F1N6r3!0G2L8s2y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8X3q4K6L8g2)9J5k6h3S2@1L8h3I4Q4x3U0y4H3N6$3&6D9K9h3u0Q4x3X3g2S2M7$3#2Q4x3X3g2E0j5h3E0W2i4K6g2X3k6h3I4X3i4K6g2X3k6Y4u0G2L8g2)9#2k6X3q4K6M7$3g2E0j5X3I4&6](<https://bbs.kanxue.com/elink@557K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2M7s2N6F1N6r3!0G2L8s2y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8X3q4K6L8g2)9J5k6h3S2@1L8h3I4Q4x3U0y4H3N6$3&6D9K9h3u0Q4x3X3g2S2M7$3#2Q4x3X3g2E0j5h3E0W2i4K6g2X3k6h3I4X3i4K6g2X3k6Y4u0G2L8g2)9#2k6X3q4K6M7$3g2E0j5X3I4&6>)

![图片描述](https://bbs.kanxue.com/upload/attach/202311/921830_ZYSZKM6UA5GY9R5.webp)

直接装配成一个 elf

## 1：建立指令类型，写出 `parse`

| Opcode | Instruction | Notes |
| --- | --- | --- |
| `0` | `nop` | `pc += 1` |
| `1` | `mov dst, imm` | `imm` is an 1-byte immediate value |
| `2` | `push imm` | `imm` is an 1-byte immediate value |
| `3` | `push reg` | put (1-byte) `reg` in stack |
| `4` | `pop reg` | pop (1-byte) from stack to `reg` |
| `5` | `PrintStr` | print `str` by `edx`: `0:'right'`, `1:'wrong'`, `3:'plz input:'`, `4:'hacker’` |
| `6` | `add reg1, reg2` | `reg1 += reg2` |
| `7` | `sub reg1, reg2` | `reg1 -= reg2` |
| `8` | `mul reg1, reg2` | `reg1 *= reg2` |
| `9` | `div reg1, reg2` | `reg1 /= reg2` (Put the quotient into `eax` and the remainder into `ebx`) |
| `10` | `xor reg1, reg2` | `reg1 ^= reg2` |
| `11` | `jmp addr` | directly jump to address |
| `12` | `cmp reg1, reg2` | `edx = reg1 - reg2` |
| `13` | `jz addr` | jump if `edx == 0`（`reg1 == reg2`） |
| `14` | `jnz addr` | jump if `edx != 0`（`reg1 != reg2`） |
| `15` | `jg addr` | jump if `edx > 0`（`reg1 > reg2`） |
| `16` | `jl addr` | jump if `edx < 0`（`reg1 < reg2`） |
| `17` | `InputStr` | `gets(mem); eax = strlen(mem);` |
| `18` | `InitMem` | `memset(mem_addr, 0, sz)` |
| `19` | `MovRegStack` | `mov reg, [ebp-src]` |
| `20` | `MovRegMem` | `mov reg, mem[src]` |
| `0xff` | `Exit` | `exit(0)` |

- `Ezmachine-disassembler-parsefunc.py`
