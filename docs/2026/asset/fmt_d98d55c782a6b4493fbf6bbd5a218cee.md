
> 56aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7j5h3E0W2M7%4m8J5K9h3&6Y4k6i4u0Q4x3V1k6S2L8X3N6J5i4K6g2X3j5%4c8X3

题目本身都不是很难，甚至大多都是能靠人力完成的工作。但是即便如此，自动化也有自动化的意义对不对。毕竟我们现在需要的不是马上就能用它解决各种难题，而是把简单的问题解决，然后才能开始做复杂问题。

> 附件使用的是 cd8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K9c8g2u0a6i4K6u0V1b7g2)9J5k6p5!0z5c8g2)9J5c8V1q4F1k6%4u0o6g2p5k6Q4y4h3k6r3d9g2c8y4 仓库下编译好的版本。因为原仓库下只有源代码，而且编译还需要另外去配环境，所以这里直接用了这位师傅编译好的附件。

# 实战

一般的基本流程如下：

- 创建项目：`angr.Project("./binary")`
- 创建 state：`project.factory.entry_state()`
- 创建 SM：`project.factory.simgr(state)`
- 探索路径：`sim.explore(find=addr)`
- 给出结果：`sim.found`

## 00_angr_find

当然还是得从最简单的开始，题目本身是一个直接用 IDA 读就能读明白的简单程序，但出于练习目的，还是得手写一下脚本。

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int i;
  // [esp+1Ch] [ebp-1Ch]
  char v5[9]; // [esp+23h] [ebp-15h] BYREF
  unsigned int v6; // [esp+2Ch] [ebp-Ch]

  v6 = __readgsdword(0x14u);
  printf("Enter the password: ");
  __isoc99_scanf("%8s", v5);
  for ( i = 0; i <= 7; ++i )
    v5[i] = complex_function(v5[i], i);
  if ( !strcmp(v5, "JACEJGCS") )
    puts("Good Job.");
  else
    puts("Try again.");
  return 0;
}
```

首先需要创建项目：

```python
import angr
project = angr.Project("./00_angr_find", auto_load_libs=False)
```

创建 state：

```python
state = project.factory.entry_state()
```

创建 SM：

```python
sim = project.factory.simgr(state)
```

搜索路径：

探索路径时需要给出需要查找到的路径地址，这里我们通过 IDA 可以确定程序输出 “Good Job.” 时的地址为 `0x08048675`

```python
sim.explore(find=0x08048675)
```

求解结果：

```python
if sim.found:
    res = sim.found[0]
    res = res.posix.dumps(0)
    print(res)
```

简单说明一下代码。

- `sim.found[0]` 代表了探索路径时得到的一条可解的路径。
- `res.posix.dumps(0)` 表示去获取对应路径中，`stdin` 的内容。

## 01_angr_avoid

程序本身很大，IDA 虽然也有办法反编译，但是速度极慢，但用 `Angr` 设定好参数就很快了。

前几个步骤是一样的：

```python
import angr
project = angr.Project("./01_angr_avoid", auto_load_libs=False)
state = project.factory.entry_state()
sim = project.factory.simgr(state)
```

我们不妨试试，如果按照上一题的做法会如何：

```python
sim.explore(find=0x080485E0)
if sim.found:
    res = sim.found[0]
    res = res.posix.dumps(0)
    print(res)
```

结果会发现等了很久也没有算出结果，因为分支实在太多了。

因此要对代码做一点改进：

```python
# 080485E0                 push    offset aGoodJob ; "Good Job."
# .text:080485A8 push ebp
# .text:080485A9 mov ebp, esp
# .text:080485AB mov should_succeed, 0
# .text:080485B2 nop
# .text:080485B3 pop ebp
# .text:080485B4 retn
sim.explore(find=0x080485E0, avoid=0x080485A8)
```

其实只是给 `explore` 增加了一个 `avoid` 的参数。当代码模拟执行遇到了该地址时，将会把这段路径放入到 `avoided` 的一个列表中，用来表示被避开的路径，然后其他照旧，继续执行。

之所以通过添加这样的操作就能够得到答案，其实很简单，是为了避免路径爆炸而必要的。

我们可以用这么一个二插树来表示路径：

![](https://bbs.kanxue.com/upload/attach/202304/924548_4SV4DWJAP78N75D.png)

我们用 `1` 来表示正确的路径，`0` 表示错误的路径。可以看见，在这个树中一共有 8 条不同的路径，而正确的路径只有一个。

假设所有涉及到 `0` 的路径都会进入到某个地址 `x` 处。那么如果没有使用 `avoid` 参数，`Angr` 就会遍历这 8 条路径，然后求解出最左的那条路径所需的输入。

而如果我们添加了 `avoid=x` ，那么当 `Angr` 从根节点进入到右子树时，由于接下来立刻进入到 `x` 地址处，因此停止分析这条路径，将其加入到 `avoided` 中，从而将下面的 4 条路径全都舍弃，将所需的时间直接减少了一半。

同理，当它进入左子树时，仍然存在分叉，而进入右子树的分叉会因为相同的原因被舍弃，从而再次减少一半的时间。

在路径极其庞大的情况下，比如说 2^31 条路径，通过这种方法能够极大程度降低消耗。

## 02_angr_find_condition

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int i;
  // [esp+18h] [ebp-40h]
  int j;
  // [esp+1Ch] [ebp-3Ch]
  char v6[20]; // [esp+24h] [ebp-34h] BYREF
  char v7[20]; // [esp+38h] [ebp-20h] BYREF
  unsigned int v8; // [esp+4Ch] [ebp-Ch]

  v8 = __readgsdword(0x14u);
  for ( i = 0; i <= 19; ++i )
    v7[i] = 0;
  qmemcpy(v7, "VXRRJEUR", 8);
  printf("Enter the password: ");
  __isoc99_scanf("%8s", v6);
  for ( j = 0; j <= 7; ++j )
    v6[j] = complex_function(v6[j], j + 8);
  if ( !strcmp(v6, v7) )
    puts("Good Job.");
  else
    puts("Try again.");
  return 0;
}
```

还是这个模板：

```python
import angr
project = angr.Project("./02_angr_find_condition", auto_load_libs=False)
state = project.factory.entry_state()
sim = project.factory.simgr(state)
```

这题的情况和 `00_angr_find` 有些不太一样。尽管 IDA 将它们反编译后的结果看起来很像，但是在汇编中却有很大差别：

![](https://bbs.kanxue.com/upload/attach/202304/924548_F3PMQU8QZU2N6KJ.png)

可以看见，这行输出在 `main` 函数里到处都是，所以其实很难找到真正的那条路径的地址。

同理的，“Try again.” 也一样，因此需要修改 `find` 参数：

```python
def succ(state):
    res = state.posix.dumps(1)
    if b"Good Job." in res:
        return True
    else:
        return False

sim.explore(find=succ)
```

可以发现，`find` 参数除了能是一个具体的地址外，还可以是一个函数。该函数返回 `True` 时会将路径记录下来，返回 `False` 时则表示路径并非我们想找的。

而区别路径的关键在于 `state.posix.dumps(1)` ，通过该方法，可以将 `stdout` 中的内容 dump 出来进行比较。如果输出包含了 `Good Job.` ，我们就认为是想要的路径。这样就能避开直接使用地址了。

当然了，`avoid` 也可以这么用，读者可以自行试试。

## 03_angr_simbolic_registers

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v3; // ebx
  int v4; // eax
  int v5; // edx
  int v6; // ST1C_4
  unsigned int v7; // ST14_4
  unsigned int v9; // [esp+8h] [ebp-10h]
  unsigned int v10; // [esp+Ch] [ebp-Ch]

  printf("Enter the password: ");
  v4 = get_user_input();
  v6 = v5;
  v7 = complex_function_1(v4);
  v9 = complex_function_2(v3);
  v10 = complex_function_3(v6);
  if ( v7 || v9 || v10 )
    puts("Try again.");
  else
    puts("Good Job.");
  return 0;
}
```

还是老三样：

```python
import angr
project = angr.Project("./03_angr_symbolic_registers", auto_load_libs=False)
state = project.factory.entry_state()
sim = project.factory.simgr(state)
```

有些特殊的地方是，输入使用 `get_user_input` ，而该函数如下：

```c
int get_user_input()
{
  int v1; // [esp+0h] [ebp-18h]
  int v2; // [esp+4h] [ebp-14h]
  int v3; // [esp+8h] [ebp-10h]
  unsigned int v4; // [esp+Ch] [ebp-Ch]

  v4 = __readgsdword(0x14u);
  __isoc99_scanf("%x %x %x", &v1, &v2, &v3);
  return v1;
}
```

前文曾提到过，`Angr` 对 `scanf` 这类使用格式化字符串的函数支持并不是很好，不过或许是最近的版本更新，直接这样写也同样能得到结果了：

```python
sim.explore(find=0x80489E9)
if sim.found:
    res = sim.found[0]
    res = res.posix.dumps(0)
    print(res)
# b'b9ffd04e ccf63fe8 8fd4d959'
else:
    print("No")
```

不过既然是学习，还是照例看看最标准的写法应该是什么吧。

根据汇编可以看到，该函数的实际操作是将值储存在寄存器中：
