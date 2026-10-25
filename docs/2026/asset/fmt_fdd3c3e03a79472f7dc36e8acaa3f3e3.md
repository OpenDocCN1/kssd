
正如你所看到的，`F5(Hex-Rays插件)`在这种情况下没有实质性的帮助。由于`IDA`和`Hex-Rays`是高度可编程的，因此其实可以自己改进`Hex-Rays`反编译器的输出并教导它简化这些表达式（当然这就算是另一个话题了）。除非我们将这个函数作为黑盒算法来处理，否则我们通过`F5`并真正理解它的操作。我们将使用`Z3`库并查看它是否可以将所有这些指令简化为可读的有意义的内容。

## Z3库的快速上手

`Dennis`和`Eric`对`Z3`的介绍已经做得很好，因此我会尽量缩短关于`Z3`的基础部分的章节。假设我们遇到这样一段汇编代码：

```
.text:0040EF04     mov     eax, [ebp+c1]

.text:0040EF07     mov     edx, [ebp+c2]

.text:0040EF0A     mov     ecx, [ebp+c3]

.text:0040EF0D     mov     ebx, [ebp+c4]

.text:0040EF10     inc     eax

.text:0040EF11     inc     ebx

.text:0040EF12     inc     edx

.text:0040EF13     inc     ecx

.text:0040EF14     add     ebx, edx

.text:0040EF16     add     ecx, 123h

.text:0040EF1C     add     eax, ebx

.text:0040EF1E     add     ebx, 456h

.text:0040EF24     add     edx, eax

.text:0040EF26     add     ecx, eax

.text:0040EF28     add     edx, ebx

.text:0040EF2A     sub     eax, 12312312h

.text:0040EF2F     add     ecx, eax

.text:0040EF31     add     eax, ebx

.text:0040EF33     add     eax, ecx

.text:0040EF35     add     eax, edx

.text:0040EF37     mov     [ebp+result], eax
```

从`0x040EF04`到`0x040EF0D`，我们看到`eax == c1`，`edx == c2`，`ecx == c3`，`ebx == c4`（4个输入参数）。从`0x040EF10`到`0x040EF35`，我们看到一些操作发生，并且操作的结果被复制到`0x040EF37`处的`eax`。我们可以将上面的列表翻译成一系列表达式：

```
.text:0040EF10     eax = eax + 1

.text:0040EF11     ebx = ebx + 1

.text:0040EF12     edx = edx + 1

.text:0040EF13     ecx = ecx + 1

.text:0040EF14     ebx = ebx + edx

.text:0040EF16     ecx = ecx + 0x123

.text:0040EF1C     eax = eax + ebx

.text:0040EF1E     ebx = ebx + 0x456

.text:0040EF24     edx = edx + eax

.text:0040EF26     ecx = ecx + eax

.text:0040EF28     edx = edx + ebx

.text:0040EF2A     eax = eax - 0x12312312

.text:0040EF2F     ecx = ecx + eax

.text:0040EF31     eax = eax + ebx

.text:0040EF33     eax = eax + ecx

.text:0040EF35     eax = eax + edx
```

现在让我们把这些表达式通过`Z3`库来表示一下（注意`Z3`重载算术运算符）：

```
import z3



c1, c2, c3, c4 = z3.BitVecs('c1 c2 c3 c4', 32)

eax, edx, ecx, ebx = c1, c2, c3, c4



eax = eax + 1

ebx = ebx + 1

edx = edx + 1

ecx = ecx + 1

ebx = ebx + edx

ecx = ecx + 0x123

eax = eax + ebx

ebx = ebx + 0x456

edx = edx + eax

ecx = ecx + eax

edx = edx + ebx

eax = eax - 0x12312312

ecx = ecx + eax

eax = eax + ebx

eax = eax + ecx

eax = eax + edx



print(eax)
```

最后的结果是：

```
c1 + 1 + c4 + 1 + c2 + 1 - 305210130 +c4 + 1 + c2 + 1 + 1110 + c3 +1 + 291 + c1 + 1 + c4 + 1 + c2 +1 + c1 + 1 + c4 + 1 + c2 + 1 - 305210130 +c2 + 1 + c1 + 1 + c4 + 1 + c2 + 1 + c4 + 1 +c2 + 1 + 1110
```

然而，我们仍然可以通过调用`z3.simplify(eax)`来让`Z3`简化最终的表达式，并获得更为简单的输出：

```
3684549565 + 4*c1 + 6*c4 + 7*c2 + c3
```

现在我们有了最后的简化的表达式，我们可以对这个表达式进行求解：

```
solver = z3.Solver()



result = z3.BitVec('result', 32)



solver.add(c1 == 1, 

           c2 == 2, 

           c3 == 3, 

           c4 == 4, 

           eax == result)

if solver.check() == z3.sat:

    m = solver.model()

    print("result=%08X" % m[result].as_long())
```

最终，我们让解算器在给定 `c1 == 1`，`c2 == 2`，`c3 == 3`和`c4 == 4`的情况下查找表达式的结果（`eax == result`）。输出为`restult= DB9DC3F1`。

## 将汇编转成Z3表达式

上面我们用`z3`库小试牛刀，手动构建表达式并使用`Z3`简化和求解，那我们考虑能否自动生成反汇编列表中的表达式？答案是肯定的，我们将使用类似于上周仿真文章的技术。我们不会计算这些值，而只是在做`Z3`算法：

```
def simplify_func(emu_start, emu_end):

    # Reset registers    

    regs_initial = { 

      REG_EAX: z3.BitVec('c1', 32),

      REG_EDX: z3.BitVec('c2', 32),

      REG_ECX: z3.BitVec('c3', 32),

      REG_EBX: z3.BitVec('c4', 32),

    }



    regs = {}

    for k, v in regs_initial.items():

        regs[k] = v

    

    def get_opr_val(inst, regs):

        if inst.Op2.type == o_imm:

            return (True, z3.BitVecVal(inst.Op2.value, 32))

        elif inst.Op2.type == idaapi.o_reg:

            return (True, regs[inst.Op2.reg])

        else:

            return (False, 0)

            

    ea = emu_start

    while ea <= emu_end:

        ok = True

        inst = idautils.DecodeInstruction(ea)

        ea += inst.size

        if inst.itype == idaapi.NN_dec and inst.Op1.type == idaapi.o_reg:

            regs[inst.Op1.reg] = (regs.get(inst.Op1.reg, 0) - 1)

        elif inst.itype == idaapi.NN_inc and inst.Op1.type == idaapi.o_reg:

            regs[inst.Op1.reg] = (regs.get(inst.Op1.reg, 0) + 1)

        elif inst.itype == idaapi.NN_sub:

            ok, val = get_opr_val(inst, regs)

            regs[inst.Op1.reg] = (regs.get(inst.Op1.reg, 0) - val)

        elif inst.itype == idaapi.NN_add:

            ok, val = get_opr_val(inst, regs)

            regs[inst.Op1.reg] = (regs.get(inst.Op1.reg, 0) + val)

        else:

            ok = False



        if not ok:

            return (False, "Emulation failed at %08X" % ea)



    # Simplify the final expression which is in EAX

    result_expr = z3.simplify(regs[REG_EAX])

        

    def evaluate(c1, c2, c3, c4):

        """Capture the context and return a function that can be used to 

        evaluate the simplified expression given the input arguments"""

        s = z3.Solver()

        r = z3.BitVec('r', 32)



        # Add contraints for input variables

        s.add(regs_initial[REG_EAX] == c1, regs_initial[REG_EDX] == c2,

              regs_initial[REG_ECX] == c3, regs_initial[REG_EBX] == c4)



        # Add the result constraint

        s.add(result_expr == r)



        if s.check() == z3.sat:

            m = s.model()

            return m[r].as_long()

        else:

            return None



    return (True, evaluate)
```

上面的代码和我们以前见过的`[345K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0m8^5k6h3u0Q4x3X3g2F1k6i4c8Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6u0Q4x3V1k6%4M7X3W2@1K9h3&6Y4i4K6u0V1j5g2)9J5k6s2y4A6L8i4m8D9k6g2)9J5k6s2R3^5y4W2)9J5k6r3g2E0N6h3I4S2N6r3!0J5i4K6u0V1N6$3W2@1K9q4)9J5k6r3W2V1j5i4m8&6N6r3S2G2L8W2)9J5c8W2)9J5c8U0m8^5k6h3u0Q4x3X3g2F1k6i4c8Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6u0Q4x3V1k6%4M7X3W2@1K9h3&6Y4i4K6u0V1j5g2)9J5k6s2y4A6L8i4m8D9k6g2)9J5k6s2R3^5y4W2)9J5k6r3g2E0N6h3I4S2N6r3!0J5i4K6u0V1N6$3W2@1K9q4)9J5k6r3W2V1j5i4m8&6N6r3S2G2L8W2)9J5c8W2)9J5x3%4y4U0L8%4m8A6L8X3N6Q4y4f1c8Q4c8e0W2Q4z5f1c8Q4z5f1g2Q4c8e0g2Q4b7U0S2Q4b7U0S2Q4c8e0N6Q4z5f1u0Q4b7U0S2Q4c8e0c8Q4b7V1y4Q4b7V1y4Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5o6W2Q4z5o6m8Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0k6Q4z5o6S2Q4z5e0q4Q4c8e0g2Q4z5p5k6Q4b7f1q4Q4c8e0c8Q4b7V1y4Q4z5f1q4Q4c8e0S2Q4b7e0N6Q4b7e0y4Q4c8e0W2Q4z5o6N6Q4z5p5q4K9x3#2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2)9&6b7W2!0n7z5q4!0q4y4g2)9^5y4g2!0n7x3#2!0q4y4q4!0n7b7W2!0m8x3#2!0q4y4#2!0m8x3q4)9^5x3g2!0q4c8W2!0n7b7#2)9&6b7g2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7`有关

- 第3-8行：创建32位`Z3`变量。这些变量对应于初始变量值（和输入值）
- 第10-12行：别名变量。这些别名将根据行更新并包含更复杂的表达式（不仅仅是初始值）
- 第44行：获取最终表达式的简化版本
- 第46-63行：创建捕获当前上下文的嵌套函数。评估函数取4个输入参数并返回简化表达式的评估结果。

我返回一个函数，以便我可以缓存它并调用它来求解特定的函数。为了测试代码，我们可以执行如下操作：

```
Python>ok, challenge_1 = simplify_func(0x401020, 0x40266C)

Python>print('result=%08X' % challenge_1(1, 2, 3, 4))
```

我得到的值为`5E6571B0`。如果代码正常工作，你们也应该具有与运行该程序相同的结果：

```
C:\ida-z3-tests>test 0 1 2 3 4 challenge_1(1, 2, 3, 4) -> 5E6571B0
```

### 参考链接

> `f5bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6i4u0A6j5$3S2W2N6W2)9J5k6h3y4G2L8g2)9J5c8Y4N6J5K9i4c8A6L8X3N6K6i4K6u0r3f1@1q4f1i4K6g2X3f1@1#2f1i4K6g2X3k6s2u0S2k6Y4c8Q4x3X3c8q4e0W2)9J5k6i4m8V1k6R3`.

> `e20K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0m8^5k6h3u0Q4x3X3g2F1k6i4c8Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6u0Q4x3V1k6%4M7X3W2@1K9h3&6Y4i4K6u0V1j5g2)9J5k6s2y4A6L8i4m8D9k6g2)9J5k6s2R3^5y4W2)9J5k6r3g2E0N6h3I4S2N6r3!0J5i4K6u0V1N6$3W2@1K9q4)9J5k6r3W2V1j5i4m8&6N6r3S2G2L8W2)9J5c8W2)9J5y4X3&6T1M7%4m8Q4x3@1t1`.

> `739K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2J5K9h3y4H3L8$3&6&6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6\*7x3%4m8&6i4K6u0V1N6s2g2@1L8%4u0A6j5h3I4Q4x3V1k6Y4N6h3W2V1k6g2)9J5k6r3g2^5j5h3#2H3L8r3g2K6i4K6u0W2K9s2c8E0`

翻译：看雪翻译组-`skeep`

校正：

看雪翻译组-`makefile`
