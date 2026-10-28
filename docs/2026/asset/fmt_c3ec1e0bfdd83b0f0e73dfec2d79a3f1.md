
# [翻译] Unicorn引擎教程

在该篇教程中，你将通过实际操作来学习 Unicorn 引擎。接下来将会有 `4` 个练习，我会解决第一个。对于其他的我将提供提示和解决方案。（译注：原网页中的提示和解决方案内容通过按钮可以显示或者隐藏，翻译后的文章不具备此功能，所有内容将直接呈现出来）

## FAST FAQ

- **Unicorn 引擎是什么？**

  简单的来讲，一款模拟器。尽管不太常见，你不能用来模拟整个程序或者系统，同时它也不支持`syscall`。你只能通过手动的方式来映射内存以及数据写入，然后就可以从某个指定的地址开始执行模拟了。

- **模拟器在什么时候是有用的？**

  - 你可以执行一些恶意软件中你感兴趣的函数而不必创建整个进程
  - CTF 比赛中也很常用
  - Fuzzing
  - GDB 插件扩充，例如支持长跳转
  - 模拟混淆后的代码

- **在这篇教程开始之前有什么需要准备的？**

  - 安装带有 Python 支持的 Unicorn 引擎
  - 一个反汇编器

## 目录

- 任务1
- 一些笔记
- 任务2
- 任务3
- 任务4
- 备忘录
- 参考

## 任务1

该任务是 hxp CTF 2017 上的一个叫做 `Fibonacci`（译注：斐波那契）的例子。二进制文件可以从这里下载。

当我们运行这个程序的时候，可以注意到这个程序计算和输出 Flag 非常的慢。Flag 的下一个字节计算的越来越慢。

`The Flag is:hxp {F`

这就意味着有必要优化程序来获取 Flag（在合理的时间内）。

在 IDA Pro 的帮助下，我们得到了像 C 语言一样的伪代码。虽然反编译代码不一定正确，但是仍然可以知道大致发生了一些事。

```c
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
void *v3; // rbp@1
int v4; // ebx@1
signed __int64 v5; // r8@2
char v6; // r9@3
__int64 v7; // r8@3
char v8; // cl@3
__int64 v9; // r9@5
int a2a; // [sp+Ch] [bp-1Ch]@3
v3 = &encrypted_flag;
v4 = 0;
setbuf(stdout, 0LL);
printf("The flag is: ", 0LL);
while ( 1 )
{
LODWORD(v5) = 0;
do
{
a2a = 0;
fibonacci(v4 + v5, &a2a);
v8 = v7;
v5 = v7 + 1;
}
while ( v5 != 8 );
v4 += 8;
if ( (unsigned __int8)(a2a << v8) == v6 )
break;
v3 = (char *)v3 + 1;
_IO_putc((char)(v6 ^ ((_BYTE)a2a << v8)), stdout);
v9 = *((char *)v3 - 1);
}
_IO_putc(10, stdout);
return 0LL;
}
```

```c
unsigned int __fastcall fibonacci(int i, _DWORD *a2)
{
_DWORD *v2; // rbp@1
unsigned int v3; // er12@3
unsigned int result; // eax@3
unsigned int v5; // edx@3
unsigned int v6; // esi@3
unsigned int v7; // edx@4
v2 = a2;
if ( i )
{
if ( i == 1 )
{
result = fibonacci(0, a2);
v5 = result - ((result >> 1) & 0x55555555);
v6 = ((result - ((result >> 1) & 0x55555555)) >> 2) & 0x33333333;
}
else
{
v3 = fibonacci(i - 2, a2);
result = v3 + fibonacci(i - 1, a2);
v5 = result - ((result >> 1) & 0x55555555);
v6 = ((result - ((result >> 1) & 0x55555555)) >> 2) & 0x33333333;
}
v7 = v6 + (v5 & 0x33333333) + ((v6 + (v5 & 0x33333333)) >> 4);
*v2 ^= ((BYTE1(v7) & 0xF) + (v7 & 0xF) + (unsigned __int8)((((v7 >> 8) & 0xF0F0F) + (v7 & 0xF0F0F0F)) >> 16)) & 1;
}
else
{
*a2 ^= 1u;
result = 1;
}
return result;
}
```

接下来是 `main` 函数的反汇编代码。

```assembly
.text:0x4004E0 main proc near ; DATA XREF: start+1Do
.text:0x4004E0
.text:0x4004E0 var_1C = dword ptr -1Ch
.text:0x4004E0
.text:0x4004E0 push rbp
.text:0x4004E1 push rbx
.text:0x4004E2 xor esi, esi ; buf
.text:0x4004E4 mov ebp, offset unk_4007E1
.text:0x4004E9 xor ebx, ebx
.text:0x4004EB sub rsp, 18h
.text:0x4004EF mov rdi, cs:stdout ; stream
.text:0x4004F6 call _setbuf
.text:0x4004FB mov edi, offset format ; "The flag is: "
.text:0x400500 xor eax, eax
.text:0x400502 call _printf
.text:0x400507 mov r9d, 49h
.text:0x40050D nop dword ptr [rax]
.text:0x400510
.text:0x400510 loc_400510: ; CODE XREF: main+8Aj
.text:0x400510 xor r8d, r8d
.text:0x400513 jmp short loc_40051B
.text:0x400513 ; ---------------------------------------------------------------------------
.text:0x400515 align 8
.text:0x400518
.text:0x400518 loc_400518: ; CODE XREF: main+67j
.text:0x400518 mov r9d, edi
.text:0x40051B
.text:0x40051B loc_40051B: ; CODE XREF: main+33j
.text:0x40051B lea edi, [rbx+r8]
.text:0x40051F lea rsi, [rsp+28h+var_1C]
.text:0x400524 mov [rsp+28h+var_1C], 0
.text:0x40052C call fibonacci
.text:0x400531 mov edi, [rsp+28h+var_1C]
.text:0x400535 mov ecx, r8d
.text:0x400538 add r8, 1
.text:0x40053C shl edi, cl
.text:0x40053E mov eax, edi
.text:0x400540 xor edi, r9d
.text:0x400543 cmp r8, 8
.text:0x400547 jnz short loc_400518
.text:0x400549 add ebx, 8
.text:0x40054C cmp al, r9b
.text:0x40054F mov rsi, cs:stdout ; fp
.text:0x400556 jz short loc_400570
.text:0x400558 movsx edi, dil ; c
.text:0x40055C add rbp, 1
.text:0x400560 call __IO_putc
.text:0x400565 movzx r9d, byte ptr [rbp-1]
.text:0x40056A jmp short loc_400510
.text:0x40056A ; ---------------------------------------------------------------------------
.text:0x40056C align 10h
.text:0x400570
.text:0x400570 loc_400570: ; CODE XREF: main+76j
.text:0x400570 mov edi, 0Ah ; c
.text:0x400575 call __IO_putc
.text:0x40057A add rsp, 18h
.text:0x40057E xor eax, eax
.text:0x400580 pop rbx
.text:0x400581 pop rbp
.text:0x400582 retn
.text:0x400582 main endp
```

`fibonacci` 的反汇编代码：

```assembly
.text:0x400670 fibonacci proc near ; CODE XREF: main+4Cp
.text:0x400670 ; fibonacci+19p ...
.text:0x400670 test edi, edi
.text:0x400672 push r12
.text:0x400674 push rbp
.text:0x400675 mov rbp, rsi
.text:0x400678 push rbx
.text:0x400679 jz short loc_4006F8
.text:0x40067B cmp edi, 1
.text:0x40067E mov ebx, edi
.text:0x400680 jz loc_400710
.text:0x400686 lea edi, [rdi-2]
.text:0x400689 call fibonacci
.text:0x40068E lea edi, [rbx-1]
.text:0x400691 mov r12d, eax
.text:0x400694 mov rsi, rbp
.text:0x400697 call fibonacci
.text:0x40069C add eax, r12d
.text:0x40069F mov edx, eax
.text:0x4006A1 mov ebx, eax
.text:0x4006A3 shr edx, 1
.text:0x4006A5 and edx, 55555555h
.text:0x4006AB sub ebx, edx
.text:0x4006AD mov ecx, ebx
.text:0x4006AF mov edx, ebx
.text:0x4006B1 shr ecx, 2
.text:0x4006B4 and ecx, 33333333h
.text:0x4006BA mov esi, ecx
.text:0x4006BC
.text:0x4006BC loc_4006BC: ; CODE XREF: fibonacci+C2j
.text:0x4006BC and edx, 33333333h
.text:0x4006C2 lea ecx, [rsi+rdx]
.text:0x4006C5 mov edx, ecx
.text:0x4006C7 shr edx, 4
.text:0x4006CA add edx, ecx
.text:0x4006CC mov esi, edx
.text:0x4006CE and edx, 0F0F0F0Fh
.text:0x4006D4 shr esi, 8
.text:0x4006D7 and esi, 0F0F0Fh
.text:0x4006DD lea ecx, [rsi+rdx]
.text:0x4006E0 mov edx, ecx
.text:0x4006E2 shr edx, 10h
.text:0x4006E5 add edx, ecx
.text:0x4006E7 and edx, 1
.text:0x4006EA xor [rbp+0], edx
.text:0x4006ED pop rbx
.text:0x4006EE pop rbp
.text:0x4006EF pop r12
.text:0x4006F1 retn
.text:0x4006F1 ; ---------------------------------------------------------------------------
.text:0x4006F2 align 8
.text:0x4006F8
.text:0x4006F8 loc_4006F8: ; CODE XREF: fibonacci+9j
.text:0x4006F8 mov edx, 1
.text:0x4006FD xor [rbp+0], edx
.text:0x400700 mov eax, 1
.text:0x400705 pop rbx
.text:0x400706 pop rbp
.text:0x400707 pop r12
.text:0x400709 retn
.text:0x400709 ; ---------------------------------------------------------------------------
.text:0x40070A align 10h
.text:0x400710
.text:0x400710 loc_400710: ; CODE XREF: fibonacci+10j
.text:0x400710 xor edi, edi
.text:0x400712 call fibonacci
.text:0x400717 mov edx, eax
.text:0x400719 mov edi, eax
.text:0x40071B shr edx, 1
.text:0x40071D and edx, 55555555h
.text:0x400723 sub edi, edx
.text:0x400725 mov esi, edi
.text:0x400727 mov edx, edi
.text:0x400729 shr esi, 2
.text:0x40072C and esi, 33333333h
.text:0x400732 jmp short loc_4006BC
.text:0x400732 fibonacci endp
```

有许多种方法来解决该任务。例如，可以通过某种编程语言重新构建代码，然后在该语言中应用优化。重建代码的过程并不容易，可能会引入一些 BUG 和错误。盯着代码找错误并不好笑。通过 Unicorn 引擎来解决这个任务可以跳过重构代码的过程，避免上面提到的问题。当然也可以通过其他的几种方法来避免重写代码——例如 `gdb` 脚本或者使用 Frida。

在开始优化之前，我们先来用 Unicorn 引擎模拟一个正常的程序，不进行优化，成功之后，再开始优化。

### Part1：模拟一个程序

首先，创建一个名为 `fibonacci.py` 的脚本，然后将二进制文件放到同一目录下。

先向脚本中添加如下代码：

```python
from unicorn import *
from unicorn.x86_const import *
```

第一行代码会加载主要的二进制模块和一些 Unicorn 中的一些基本常量。第二行加载了一些特定的 x86 和 x64 的常量。

接下来，加入以下代码：
