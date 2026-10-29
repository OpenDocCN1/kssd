
# [原创]shellcode xor编码/解码

shellcode的初级变形的一点点学习，大部分搜集、翻译，少量原创，姑且算原创吧，![](https://bbs.kanxue.com/view/img/face/1.gif)

01 病毒上重定位问题
02 shellcode解码
03 一个unix例子
04 源码解读
05 参考

shellcode xor编码/解码器是一种较为初级的shellcode变形的方法，当然它从中采用的方法跟病毒上的方法有些类似

## 01 病毒上重定位问题

病毒上的解决重定位的方法，可以方便的用到shellcode的设计中。

下面部分摘自hume的《病毒编程技术》

在病毒技术上至少有两种方法可以解决重定位的问题：

A)第一种方法就是利用上述PE文件重定位表项的特殊作用构造相应的重定位表项。

在感染目标PE文件时，将引用自身数据的需要被重定位的地址全部写入目标PE文件的重定位表中，如果目标PE无任何重定位表项（如用MS linker的`/fixed`）则创建重定位表节并插入新的重定位项；

若已经存在重定位表项，则在修改已存在的重定位表节，在其中插入包含了这些地址的新表项。重定位的工作就完全由系统加载器在加载PE文件的时候自动进行了。重定位表项由PE文件头的`DataDirectory`数据中的第6个成员`IMAGE_DIRECTORY_ENTRY_BASERELOC`指向。

该方法需要的代码稍多，实现起来也相对比较复杂，另外如果目标文件无重定位表项（为了减小代码体积，这种情况也不少见），处理起来就比较麻烦，只有用高级语言编写病毒才常用该种方法，在一般的PE病毒中很少使用。

B)利用Intel X86体系结构的特殊指令，`call`或`fnstenv`等指令动态获取当前指令的运行时地址，利用自定位计算该地址与编译时预定义地址的差值（被称为delta offset），再将该差值加到原编译时预定的地址上，得到的就是运行时数据的正确地址。对于intel x86指令集而言，在书写代码时，通过将delta offset放在某个寄存器中，然后通过变址寻址引用数据就可以解决引用数据重定位的难题。还以上例说明，假如上述指令块被操作系统映射在0x500000处那么代码及其在内存中的地址将变为：

```
501000: 

mov eax,dword ptr [402035] 

......

502035:

db "hello world!",0
```

显然，`mov`指令引用的操作数地址是不正确的，如果我们知道了`mov`指令运行时地址是0x501000，那么计算该地址和编译时该指令预设地址的差值：`0x501000-0x401000 = 0x100000`。很显然指令引用的实际数据地址应该为`0x402035+0x100000 = 0x502035`。从上例可以看出，只要能够在运行时确定某条指令动态运行时的地址，而其编译时地址已知，我们就能够通过将delta offset加到相应的地址上正确重定位任何代码或数据的运行时地址。

通常只要在病毒代码的开始计算出delta offset，通过变址寻址的方式书写引用数据的汇编代码，即可保证病毒代码在运行时被正确重定位。假设`ebp`包含了delta offset，使用如下变址寻址指令则可保证在运行时引用的数据地址是正确的：

```
;ebp包含了delta offset值

401000: 

mov eax,dword ptr [ebp+0x402035]

......

402035:

db "hello world!",0
```

在书写源程序时可以采用符号来代替硬编码的地址值，上述的例子中给出的不过是编译器对符号进行地址替换后的结果。现在的问题就转换成如何获取delta offset的值了，显然：

```
call delta

delta:

pop ebp

sub ebp,offset delta
```

在运行时就动态计算出了delta offset值，因为`call`要将其后的第一条指令的地址压入堆栈，因此`pop ebp`执行完毕后`ebp`中就是`delta`的运行时地址，减去`delta`的编译时地址“`offset delta`”就得到了delta offset的值。除了用明显的`call`指令外，还可以使用不那么明显的`fstenv`、`fsave`、`fxsave`、`fnstenv`等浮点环境保存指令进行，这些指令也都可以获取某条指令的运行时地址。以`fnstenv`为例，该指令将最后执行的一条FPU指令相关的协处理器的信息保存在指定的内存中。

该结构偏移12字节处就是最后执行的浮点指令的运行时地址，因此我们也可以用如下一段指令获取

```
delta offset：



fpu_addr:

fnop

call GetPhAddr

sub ebp,fpu_addr



GetPhAddr:

sub esp,16

fnstenv [esp-12]

pop ebp

add esp,12

ret
```

delta offset也不一定非要放在`ebp`中，只不过是`ebp`作为栈帧指针一般过程都不将该寄存器用于其它用途，因此大部分病毒作者都习惯于将delta offset保存在`ebp`中，其实用其他寄存器也完全可以。

## 02 shellcode解码

病毒中采用的技术，在shellcode解码中加以利用

1.FNSTENV XOR decoder

```
[BITS 32]



global _start



_start:



fabs ;fabs指令

fnstenv [esp] ;保存环境，该结构偏移12字节处就是最后执行的浮点指令的运行时地址

pop edx

pop edx

pop edx

pop edx ;此处将fabs指令的运行时地址传给edx

sub dl, -25 ; offset from fabs -> xor buffer    edx = edx + 25,25的大小指的是从shllcode到fabs的偏移



short_xor_beg:

xor ecx,ecx ;清零ecx

sub cx, -0x15F ;size of xor'd payload 设置ecx大小为0x15F



short_xor_xor:

xor byte [edx], 0x99 ; the byte to xor with ;开始循环解码

inc edx

loop short_xor_xor



shellcode:

db ...........................
```

2.Jump/call decoder

```
global _start



_start:

jmp short getdata     ; Get data pointer 跳转到getdata

begin:



pop ebx ; Pop the data address ;弹出codestart的地址，call begin时会压入eip地址

xor ecx,ecx     ; Set up loop counter ;清零ecx

sub cx, -0x15F ; size of data payload ;设置ecx为0x15F



decode:

xor byte [ebx], 0x99     ; XOR ;开始循环解码

inc ebx     ; Increment data address

loop decode     ; Loop back to decode if cx > 0

jmp short codestart     ; Jump into decoded code





getdata:

call begin     ; Push the address of data in stack and jump

; to label begin



codestart: ; This is where the XOR'ed shellcode begins

db ..........................
```

## 03 一个unix例子

这是69cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3E0D9j5h3E0W2i4K6u0W2L8%4u0Y4i4K6u0r3i4K6N6q4K9Y4c8Q4x3V1k6W2L8X3y4G2k6r3g2J5i4K6u0r3i4K6t1&6

```
;
; hello.asm -- simple non-optimized Linux/x86 shellcode

; Compile: nasm -f bin -o hello.s hello.asm

;



[BITS 32]



global _start



_start:

jmp short data ; Jump to our data section

begin:

mov eax, 4 ; syscall #4 = write()

mov ebx, 1 ; stdout

pop ecx ; Pop the data address in ecx

mov edx, 15 ; 15 bytes of data

int 0x80

mov eax, 1 ; syscall #1 = exit()

mov ebx, 0 ; status = 0

int 0x80



data:

call begin ; Call will return our data address

db "Hello, hacker!", 0x0a
```

`NASM`生成下列shellcode:

```
$ hexdump hello.s 

0000000 1eeb 04b8 0000 bb00 0001 0000 ba59 000f

0000010 0000 80cd 01b8 0000 bb00 0000 0000 80cd

0000020 dde8 ffff 48ff 6c65 6f6c 202c 6168 6b63

0000030 7265 0a21 

0000034
```

当然，我们应该通过避免使用32位寄存器消除这些代码中的所有NULL。例如，`xor eax,eax;mov al,4`代替`mov eax,4`。但在这个例子中，需要NULLs来测试编码。

```
$ ./loader hello.s 

Loading shellcode from "hello.s"...

Hello, hacker!
```

移除NULL后:

```
$ ./encoder hello.s 0x00 > h.s



Encoder v0.5 - Encode NULLs and other characters out of shellcode

Copyright (c) Jarkko Turkulainen 2004. All rights reserved.

Read the file encoder.c for documentation.



Using FNSTENV XOR decoder

Using register eax for decoder

Removing bytes 0x00

Reading shellcode from "hello.s"

Using 0x02 for XOR

Ready
```

`Encoder`选择字节0x02替换XOR，现在检查下:

```
$ hexdump h.s 

0000000 e1d9 34d9 5824 5858 8058 e7e8 c931 8166

0000010 cce9 80ff 0230 e240 e9fa ba1c 0206 0202

0000020 03b9 0202 5b02 0db8 0202 cf02 ba82 0203

0000030 0202 02b9 0202 cf02 ea82 fddf fdfd 674a

0000040 6e6e 2e6d 6a22 6163 6769 2370 0008 

000004d
```

依然能够成功

```
$ ./loader h.s 

Loading shellcode from "h.s"...

Hello, hacker!
```

## 04 源码解读

下面解读部分`encoder.c`的源代码(`encoder.c`取自96bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3E0D9j5h3E0W2i4K6u0W2L8%4u0Y4i4K6u0r3i4K6N6q4K9Y4c8Q4x3V1k6W2L8X3y4G2k6r3g2J5i4K6u0r3i4K6t1&6

以`decoder_jump_header`为例来说明解码部分

```
unsigned char decoder_jump_header[] = {

0xEB, 0x10, 0x5B, 0x31, 0xC9, 0x66, 0x81, 0xE9, 

0xA1, 0xFE, 0x80, 0x33, 0x99, 0x43, 0xE2, 0xFA, 

0xEB, 0x05, 0xE8, 0xEB, 0xFF, 0xFF, 0xFF

};
```

```
echo -ne "\xEB\x10\x5B\x31\xC9\x66\x81\xE9\xA1\xFE\x80\x33\x99\x43\xE2\xFA\xEB\x05\xE8\xEB\xFF\xFF\xFF" | ndisasm -u -



00000000 EB10 jmp short 0x12

00000002 5B pop ebx

00000003 31C9 xor ecx,ecx

00000005 6681E9A1FE sub cx,0xfea1

0000000A 803399 xor byte [ebx],0x99

0000000D 43 inc ebx

0000000E E2FA loop 0xa

00000010 EB05 jmp short 0x17

00000012 E8EBFFFFFF call dword 0x2
```

这段就是decoder，也就是前文所说明的部分，然后会在这个解码器的后面放上真正的已经做过处理的shellcode

这个`encoder.c`通过xor来处理需要剔除的字节，原理很简单

下面的代码是寻找一个字节，使它与要剔除的字节异或后，产生的新的字节，不会出现在原本的shellcode中

这样做是为了防止原本的shellcode的字节异或后又生成要剔除的字节
