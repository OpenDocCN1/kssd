
注释：你可以不用寄存器代替`key`寄存器和指针寄存器。而是对`key`改用立即值，对指针用立即寻址方式，直接增加立即值。同样，可以通过直接在代码上应用解密数学运算，跳过代码寄存器。不过，在这篇教学里，我将使用所有的寄存器。

正如我前面所说的，我将试着说明32位多态的方法，我将考虑生成32位代码的多态引擎。因此，我们可以用任一寄存器表示地址码：`[EAX]`, `[ECX]`, 等等。不像在16位里，你只有较少的选择，像`[DI]`, `[BX+DI]`, 等等。

让我们检验选择随机寄存器的方法。

注释：我将调用`brandom32`过程，它期望的值在`EAX`里，并返回一个0至原始`EAX-1`之间的随机值。

这个过程隐含的想法是为寄存器设置代码，首先是真正使用的寄存器和其它的，无用数据（垃圾）寄存器。在那之后，过程应该在它们之间通过几次2轮交换来混淆它们：

```
        used_registers:

        kreg db 0

        preg db 1

        creg db 2

        lreg db 3

        jrg1 db 5

        jrg2 db 6

        jrg3 db 7

        Choose_random_regs proc near  ;

        lea edi, used_registers       ; point to registers

        lea esi, used_registers       ; point to registers

        mov edx, esi                  ; save position

        mov ecx, 50h                  ; scramble 50h times

        mangle:                       ;

            mov eax, 8                ;

            call brandom32             ; choose a random nr. between 0-7

            mov ebx, eax              ; in EBX

            mov eax, 8                ;

            call brandom32            ; choose a random nr. between 0-7

            cmp ebx, eax              ; in EAX

             je mangle                 ; if EAX=EBX choose again

            add edi, eax              ; increment first pointer

            add esi, ebx              ; increment second pointer

            mov al, byte ptr [edi]    ; and exchange the values

             xchg byte ptr [esi], al   ; between them

            mov byte ptr [edi], al    ;

            mov edi, edx              ; restore position

            mov esi, edx              ;

            loop mangle               ; and do it 50h times

            ret                       ;

        Choose_random_regs endp        ;
```

在这之后，我们可以用那些确实是随机选择的寄存器。对于填充适当的 ModR/M 寄存器，考虑到被对齐的代码像这样：`xxxxxNNN`，要填充正确的 ModR/M 寄存器，你需要清理 ModR/M 字节。查看下面的'指令生成器'。

## 2.2 随机值选择器

对多态引擎来说，选择多个随机值并用它们填充真正的指令是非常重要的。

最普通的随机值被用作加密`key`。为了有一个强壮的`key`，你必须按下面的规则行事，如：

* `key`开头不要出现0填充的情况（类似于`00158A45h`）
* 不要使用对称的`key`（类似于`ABCDDCBAh`）

同样，另一种普通的随机值是'递增'`key`。尽管我在这里引用了递增，但你并不一定总是采用递增/递减`key`。你可以选择任一可逆的数学运算。在我的`MOF32`里，每次都是重复使用`XOR`, `ADD`和`SUB`操作中的一种来改变`key`。

最后但并非最不重要的随机值是加密运算。我的32位多态引擎使用3种加密运算：

1. 无`key`运算：对代码进行`ROR`/`ROL`运算
2. 双代码运算：把代码与后续的代码做`XOR`/`ADD`/`SUB(dword)`运算
3. `key`运算：把代码与`key`做`XOR`/`ADD`/`SUB`运算

你进行的数学运算越多，加密就越坚固。然而你不要因此而提升译码器本身的复杂性。其它可以使用的有趣的运算是`NOT`, `NEG`, `XCHG`。

## 2.3 指令生成器

Ok，现在该生成我们的译码器的指令了。这是一个在`mainloop`中被调用的过程，因此，我们需要维护一个计数器。这个计数器指示我们在这个迭代中必须生成哪条指令。让我们看看我打算怎样声明译码器：

```
        decryptor:

        i1:     <instruction 1>

                db 0FEh

        i2      <instruction 2>

                db 0FEh

        ...

        ...

        in      <instruction n>

                db 0FEh
```

因此，我们有译码器声明的每一条指令，当然，还有一组简单的寄存器，但是要小心：在声明译码器时不要使用`EAX`或`AX`，因为那些操作码是不同的。例如：

```
        we define                    |      will actually become
    -------------------------+--------------------------
        mov ebx, [ebx]             |      mov creg, [preg]

        mov ebx, 0                 |      mov kreg, keyvalue

        etc...                     |
```

因此，你应该理解：你定义了指令，编译器为你计算正确的操作码并把它们放在那里。现在，你所要做的就是复制每条指令并用它们填充适当的值或寄存器。

在这里，你面临两个选择：一个是填入声明译码器的地方，然后把它复制到目的地，或者你也可以逐字节复制并直接填入目的地。我认为第一个方法更快一些。但需要你有一个指针。让我们假定`ESI`指向译码器的字节。为了使事情更简单，如果你的译码器里指令较少（`MOF32`用了12条指令译码器），你可以在你的指令生成器程序里选择生成一个“案例”，手动获得每条指令，像下面这样：

```
        cmp ecx, 1

        je ins1

        cmp ecx, 2

        je ins2

        ...

        jmp over

 ins1: ...

       jmp over

 ins2: ...

       jmp over

       ...

 over:

       ret
```

你明白`ecx`是计数器。因此，如果我们必须生成指令`nr.3`，我们就跳到`ins3`。你填入每条指令并不是很容易，这是因为你必须非常清楚那是那种指令并知道填在哪。让我们设想你的第三条指令看起来如同下面声明的：

```
        mov ebx, 0
```

你必须把它变成：

```
        mov kreg, keyvalue
```

知道你的`kreg`是`EDX`，`key`值是`12345678h`。

```
        The opcode for MOV EBX, 0         is: BBh 00h 00h 00h 00h

        The opcode for MOV EDX, 12345678h is: BAh 78h 56h 34h 12h
```

REG 字段是操作码的最后三位（如果不明白，重读第一部分），因此，我们首先跳过它（`ESI`指向指令开始处）：

```
             and byte ptr [esi], 00000111b
```

在那之后我们填充适当的寄存器：

```
            or byte ptr [esi], kreg
```

现在，我们填充`key`值：

```
            or dword ptr [esi+1], keyvalue
```

DONE! 我们有了自己的多态指令。我们现在所要做的就是用`lodsb`/`stosb`或其它方法把它复制到目的地。

当写一个多态引擎时，我推荐你首先写不包含任何垃圾的。一旦你生成了你自己的译码器，并且工作正常，你可以再增加垃圾引擎。

作为对你的帮助，这里是`MOF32`所用的通用译码器：

```
       decryptor:

       i01:  mov ebx, 0                   ; mov preg, code_start

             db 0feh                      ;

       i02:  mov ebx, 0                   ; mov kreg, key

             db 0feh                      ;

       i03:  mov ebx, 0                   ; mov lreg, code_length/8

             db 0feh                      ;

       i04:  mov ebx, dword ptr [ebx]     ; mov creg, [preg] (mainloop)

             db 0feh                      ;

       i05:  add ebx, ecx                 ; <op1> creg, kreg

             db 0feh                      ;

       i06:  ror ebx, 0                   ; <op2> creg, key2

             db 0feh                      ;

       i07:  add ebx, dword ptr [ebx+4]   ; <op3> creg, [preg+4]

              db 0feh                      ;

       i08:  mov dword ptr [ebx], ebx     ; mov [preg], creg

             db 0feh                      ;

       i09:  add ebx, keyvalue            ; <op4> kreg, keyvalue

             db 0feh                      ;

       i10:  add ebx, 4                   ; add ebx, 4

             db 0feh                      ;

       i11:  sub ebx, 1                   ; sub lreg, 1

             db 0feh                      ;

       i12:  jnz $                        ; jnz mainloop

              db 0feh, 0ffh               ;
```

我希望这是非常清楚的，就象我们正在接近的多态译码器的真正盔甲：垃圾…

## 2.4 垃圾生成器

就像我在多态引擎图里说明的那样，在生成第一个真正的指令之前和在每条真正的指令之后调用垃圾生成器。下面是垃圾应该包括的主要的指令类型：

* `<math op> jreg, <reg>/<mem>/<imm>`
* `<logical op> jreg, <reg>/<mem>/<imm>`
* `<test op> jreg, <reg>/<mem>/<imm>`
* `<shift op>`
* `jmps` and conditional jumps
* `calls`
* unary operations (`inc`, `dec`,...)
* `pushad`/`popad`
* `FPU` instructions
* priviledged instructions (`smsw`, `sidt`, etc...)

注释：在`pushad`和`popad`之间，寄存器可能被破坏…

Ok，我们现在调用垃圾例程。首先，我们准备生成多少条无用的指令？这可是个哲学问题。依我之见，为了做一个容易的可定制可升级的多态引擎，我建议你生成的无用指令尽量与译码器的真正指令差不多。因此，如果你的译码器有10条真正的指令，你至少应该在它们之间生成不少于10条的无用指令。我想当我说无用指令时你能明白我的意思，我的意思是一组指令，例如: `pushad/mov eax, ebx/popad`… 当生成垃圾指令时，你将执行一个循环直到计数器为0。如果你一次生成多条指令（例如`call/jump`组合），你应该把计数器递减2，否则你会得到太多的垃圾指令。

总之，依我看来，垃圾生成器应当看起来如下：

```
     .-----------------.
     | Garbage routine |
     '-----------------'
       |
 .----+------------------------------------------------------------------.
 | .--v------------------------------------------.                      |
 | |  Choose random number of junk instructions  |                      |
 | '---------------------------------------------'                      |
 |    |                                                                  |
 | .--v-----------------------.                                         |
 | | Garbage generator loop   |                                         |
 | '-^------------------------'                                         |
 |   |      |                                                           |
 |   |      .--v--------------------------------. .-------------------. |
 |   |      | Choose random garbage type         |-> Choose addressing | |
 |   |      '--+-+-+-+-+-+----------------------' '----------+-+-+----' |
 |   |          | | | | | |                                 | | |      |
 |   |          | | | | | '--> Type 1 ---->-<------ mem2reg <-' | |     |
 |   |          | | | | '----> Type 2 ---->|<------ reg2reg <---' |     |
 |   |          | | | '------> Type 3 ---->|<------ etc...  <-----'     |
 |   |          | | '--------> Type 4 ---->|                           |
 |   |          | '----------> Type 5 ---->|                           |
```
