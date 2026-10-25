
```python
def execute(self, *args, **kwargs):
"""
This function is a symbolic execution helper in the simple style
supported by triton and manticore. It designed to be run after
setting up hooks (see Project.hook), in which the symbolic state
can be checked.
该函数是一个由triton和manticore支持的简单风格的符号执行助手。被设计为在设置hook之后运行，使得可以在hook之后检查符号状态
This function can be run in three different ways: 该函数可以用三种不同方式运行
- When run with no parameters, this function begins symbolic execution
from the entrypoint.
无参数，该函数从入口点开始符号执行
- It can also be run with a "state" parameter specifying a SimState to
begin symbolic execution from.
带state参数，从指定的state开始符号执行
- Finally, it can accept any arbitrary keyword arguments, which are all
passed to project.factory.full_init_state.
任意参数，这些参数将被传递到project.factory.full_init_state
If symbolic execution finishes, this function returns the resulting
simulation manager. 如果符号执行结束，将把执行结果返回给simulation manager
"""
if args: # 如果有参数，第一个参数给state
state = args[0]
else: #
state = self.factory.full_init_state(**kwargs)
pg = self.factory.simulation_manager(state) # 设置simulation manager
self._executing = True # _executing设置为True，该变量标志开始或终止
return pg.run(until=lambda lpg: not self._executing) # run函数待分析
```

### 3.2 terminate_execution（结束符号执行）

```python
def terminate_execution(self):
"""
Terminates a symbolic execution that was started with Project.execute().
终止一个由execute开始的符号执行
"""
self._executing = False
```

## 4 load_shellcode

```python
def load_shellcode(shellcode, arch, start_offset=0, load_address=0, thumb=False, **kwargs):
"""
Load a new project based on a snippet of assembly or bytecode.
基于一串汇编代码或字节码加载一个新的Project
:param shellcode: The data to load, as either a bytestring of instructions or a string of assembly text 要加载的数据，可以是指令的字节串或者是汇编文本的字符串
:param arch: The name of the arch to use, or an archinfo class 要用的arch的名字，或者是一个archinfo类
:param start_offset: The offset into the data to start analysis (default 0) 开始分析的数据的偏移，默认是0
:param load_address: The address to place the data in memory (default 0) 在内存中放置数据的地址，默认是0
:param thumb: Whether this is ARM Thumb shellcode 是否是ARM架构Thumb模式的shellcode
"""
if not isinstance(arch, archinfo.Arch): # 如果传入的arch不是archinfo.Arch类，就把它转换成它
arch = archinfo.arch_from_id(arch)
if type(shellcode) is str: # 如果shellcode是字符串，就对其进行汇编，得到对应的机器码
shellcode = arch.asm(shellcode, load_address, thumb=thumb)
if thumb: # 如果是thumb模式
start_offset |= 1 # 数据偏移的最低位变成1
return Project( # 返回一个新的 Project
BytesIO(shellcode), # 流
main_opts={
'backend': 'blob', #后端使用的加载器（“elf”，“pe”，“mach-o”，“ida”，“blob”）
'arch': arch, # 架构
'entry_point': start_offset, # 入口
'base_addr': load_address, # 基址
},
**kwargs
)
```

## 5 总结

还有很多细节没有搞懂，比如`cle.Loader`，很多地方用到了loader里面的东西，但目前只是通过在解释器里运行代码看一看这些结构长什么样。还有`SimProcedures`等，另外还有`Pickling`序列化那些函数也没看出个所以然，后面再分析吧。

- `Project`涉及
- `cle`：二进制加载
- `engine`：模拟执行引擎
- `analyses`：CFG分析模块
- `factory`：工厂
- `simprocedure`：hook
- `simos`：io交互

## 参考文献

- <a href="elink@e41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1K6x3e0b7$3x3l9`.`."><mark class="encrypted">e08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1K6x3e0b7$3x3l9`.`.</mark></a>
- <a href="elink@d91K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6%4u0Q4x3X3g2A6L8#2)9J5c8X3q4H3K9g2)9J5k6r3c8G2j5#2)9J5c8X3q4F1k6%4u0Q4x3X3g2Z5N6r3#2D9"><mark class="encrypted">d8aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6%4u0Q4x3X3g2A6L8#2)9J5c8X3q4H3K9g2)9J5k6r3c8G2j5#2)9J5c8X3q4F1k6%4u0Q4x3X3g2Z5N6r3#2D9</mark></a>
- <a href="elink@253K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3!0@1K9r3g2J5x3e0l9J5y4q4)9J5c8X3q4F1k6%4u0Q4x3X3c8S2L8X3q4D9P5i4y4A6M7H3`.`."><mark class="encrypted">cdeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3!0@1K9r3g2J5x3e0l9J5y4q4)9J5c8X3q4F1k6%4u0Q4x3X3c8S2L8X3q4D9P5i4y4A6M7H3`.`.</mark></a>
- <a href="elink@834K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1I4y4o6t1^5z5q4)9J5x3$3R3J5i4K6u0V1y4H3`.`."><mark class="encrypted">608K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1I4y4o6t1^5z5q4)9J5x3$3R3J5i4K6u0V1y4H3`.`.</mark></a>
- [<mark class="encrypted">473K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4j5h3&6Y4j5h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6H3i4K6u0r3y4$3k6&6k6$3k6Y4j5X3y4W2k6X3t1K6z5r3y4X3j5#2)9#2c8q4)9J5z5l9`.`.</mark>

> - 源码版本：9.0.10281

一般`angr`的利用脚本首先都是创建一个`Project`对象，如下所示。`Project`类是`angr`模块的主类，它包含了一组二进制文件及它们之间的关系，并对它们执行分析。

|  |  |
| --- | --- |
| 1 | `>>> project` `=` `angr.Project`(`"/home/lzx/tools/angr_ctf/solutions/00_angr_find/00_angr_find"`) |

## 1 __init__ （构造函数）

`Project`类的`__init__`函数调用图。

![image.png](https://bbs.kanxue.com/upload/attach/202110/830671_U4RZ55VHA2KZUYT.jpg)

### 1.1 参数

|  |  |
| --- | --- |
