
```
栈段：
Access byte 10010010b
Pr = 1 ; 标记为有效的段
Privl = 0 ; ring3权限
S = 1 ; 代码段或数据段
Ex = 0 ; 不可执行，即数据段
DC = 0 ; 标记该段只能在Privl权限（即ring3权限）下执行
RW = 1 ; 可写
Ac = 0 ; 该段的内容是否被修改，默认为0，由CPU置位
Flags 1100b
Gr = 0 ; 粒度为1B
Sz = 1 ; 32位模式
```

接着我们定义6字节的GDT描述符：

```asm
; GDT descriptior
gdt_descriptor:
dw gdt_end - gdt_start - 1 ; Size of our GDT, always less one of the true size
dd gdt_start ; Start address of our GDT
```

接着调用`lgdt`指令将GDT描述符加载到`gdtr`寄存器，整个GDT的初始化就算是完成了：

```asm
lgdt [gdt_descriptor]
```

### 三、将`CR0`最低位置1

`CR0`是系统内的32位控制寄存器之一，可以控制CPU的一些重要特性。其中最低位是保护允许位（Protected Mode Enable, `PE`），`PE`位置1后CPU进入保护模式（注意此时还是16位保护模式，不是32位保护模式），置0时则为实模式。现在我们要进入保护模式，即将`CR0`的最低位置1，汇编代码如下：

```asm
; 把 cr0 的最低位置为 1，开启保护模式
mov eax, cr0
or eax, 0x1
mov cr0, eax
```

有关`CR0`其他位的功能以及其他控制寄存器可以查看<a href="elink@be6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6o6L8$3&6@1M7X3!0D9i4K6g2X3M7X3g2Y4K9i4y4@1k6i4t1`.">Control register</a>，这里不再赘述。

### 四、执行远跳转

将`cr0`最低位置1后，CPU就进入了保护模式，此时需要马上执行一条远跳转指令：

```asm
jmp 08h:PModeMain
```

这条指令有两个作用，第一个作用是将`cs`段寄存器的值修改为08h，切换到保护模式后，CPU寻址的方式就从实模式中的`段地址 * 16 + 偏移地址`改为了通过gdt寻址，所以这里的08h是段选择子而不是段地址，并且远跳转指令会自动将`cs`的值修改为对应的段选择子，这里是08h。

之前提到过，段描述符中的`Sz`位确定操作数的默认大小，将代码段寄存器切换到08h后，代码段描述符的`Sz`位为1，即32位模式，此时才真正进入了32位保护模式。

远跳转的另一个作用是清空CPU的流水线，流水线的作用在计组中有提到过，为了加速指令的执行，CPU在执行当前指令时会同时加载并解析接下来的一些指令，在进入保护模式之前，已经有许多指令进入了流水线，这些指令都是按16位模式处理的，而进入保护模式后的指令都是32位，所以这里通过一个远跳转来让CPU清空流水线。

切换到32位模式后，就应该执行32位的指令了，所以从`PModeMain`开始的指令都采用32位模式编译，通过`[bits 32]`这个标记实现：

```asm
[bits 32]
PModeMain:
```

### 五、初始化段寄存器和栈指针

上一步中我们将代码段寄存器`cs`初始化成了0x08，现在我们还需要初始化其他的段寄存器如数据段寄存器`ds`，拓展段寄存器`es`，栈段`ss`以及`fs`，`gs`两个由操作系统使用的段。

另外我们还需要初始化栈指针`ebp`和`esp`，代码如下：

```asm
[bits 32]
PModeMain:
mov ax, 0x10 ; 将数据段寄存器ds和附加段寄存器es置为0x10
mov ds, ax
mov es, ax
mov fs, ax ; fs和gs寄存器由操作系统使用，这里统一设成0x10
mov gs, ax
mov ax, 0x18 ; 将栈段寄存器ss置为0x18
mov ss, ax
mov ebp, 0x7c00 ; 现在栈顶指向 0x7c00
mov esp, ebp
```

此时我们就可以开始执行32位保护模式下的代码了。

## 0x03. 测试：32位保护模式下的打印

接下来我们要实现的是32位保护模式下的打印功能，测试我们是否已经成功进入32位保护模式了。

在32位保护模式中我们无法使用实模式下的BIOS中断指令进行打印，但是天无绝人之路，我们仍然可以通过直接修改显存来进行打印。

显卡包括显存本身是外围设备，但CPU很友好的把显存映射到了0x8B000这个物理地址，因此我们可以直接通过`mov`指令来访问显存。在文本模式下，被打印的字符在显存中总共占2个字节，第一个字节是字符的ASCII码，第二个字节是字符的颜色属性，这里与BIOS中`int 10`调用的颜色属性是一样的：

| BIT | 属性 |
| ---- | --------------------------------------------------- |
| 0~2 | 字体颜色( 0:黑, 1:蓝, 2:青, 4:红, 5:紫, 6:棕, 7:白) |
| 3 | 字体亮度( 0:正常亮度, 1:高亮度) |
| 4~6 | 背景颜色( 0:黑, 1:蓝, 2:青, 4:红, 5:紫, 6:棕, 7:白) |
| 7 | 字体闪烁( 0:不闪烁, 1:闪烁) |

打印字符串的代码我们可以写成一个函数，具体的内容大家看下面的代码和注释：

```asm
; 打印字符串
; @param 被打印的字符串
print:
push ebp
mov ebp, esp
mov ebx, [ebp+8] ; [ebp+8]即传入的字符串参数
xor ecx, ecx
mov ah, 0x0f ; ah为打印的颜色属性，0x0f为白字黑底
mov edx, 0xb8000 ; 显存的地址
loop1_begin:
mov al, [ebx] ; al为被打印的字符
cmp al, 0 ; 若al为0，结束打印
je loop1_end
mov [edx], ax ; 向显存中写入字符及其颜色属性（2字节）
inc ebx
add edx, 2
jmp loop1_begin
loop1_end:
mov esp, ebp
pop ebp
ret
```

函数调用：

```asm
PModeTest:
push TEST_STRING ; 被打印字符的地址
call print
PModePause:
hlt
jmp PModePause
TEST_STRING db 'We are in protected mode!', 0
```

效果如下：

![图片描述](upload/attach/202109/910514_H8NCCXGBGW42VW6.png)

## 0x04. 完整代码

完整的代码，需要在引导程序中加载这段代码到正确的位置，这里是09000h，或者直接把`org 09000h`改成`org 07c00h`，把这段代码写到主引导扇区：

```asm
org 09000h
cli ; 屏蔽中断
lgdt [gdt_descriptor] ; 初始化GDT
; 把 cr0 的最低位置为 1，开启保护模式
mov eax, cr0
or eax, 0x1
mov cr0, eax
jmp 08h:PModeMain
[bits 32]
PModeMain:
mov ax, 0x10 ; 将数据段寄存器ds和附加段寄存器es置为0x10
mov ds, ax
mov es, ax
mov fs, ax ; fs和gs寄存器由操作系统使用，这里统一设成0x10
mov gs, ax
mov ax, 0x18 ; 将栈段寄存器ss置为0x18
mov ss, ax
mov ebp, 0x7c00 ; 现在栈顶指向 0x7c00
mov esp, ebp
jmp PModeTest
PModeTest:
push TEST_STRING ; 被打印字符的地址
call print
PModePause:
hlt
jmp PModePause
; 打印字符串
; @param 被打印的字符串
print:
push ebp
mov ebp, esp
mov ebx, [ebp+8] ; [ebp+8]即传入的字符串参数
xor ecx, ecx
mov ah, 0x0f ; ah为打印的颜色属性，0x0f为白字黑底
mov edx, 0xb8000 ; 显存的地址
loop1_begin:
mov al, [ebx] ; al为被打印的字符
cmp al, 0 ; 若al为0，结束打印
je loop1_end
mov [edx], ax ; 向显存中写入字符及其颜色属性（2字节）
inc ebx
add edx, 2
jmp loop1_begin
loop1_end:
mov esp, ebp
pop ebp
ret
TEST_STRING db 'We are in protected mode!', 0
gdt_start:
; 第一个描述符必须是空描述符
gdt_null:
dd 0
dd 0
; 代码段描述符
gdt_code:
dw 0xffff ; Limit (bits 0-15)
dw 0x0 ; Base (bits 0-15)
db 0x0 ; Base (bits 16-23)
db 10011010b ; Access Byte
db 11001111b ; Flags , Limit (bits 16-19)
db 0x0 ; Base (bits 24-31)
; 数据段描述符
gdt_data:
dw 0xffff ; Limit (bits 0-15)
dw 0x0 ; Base (bits 0-15)
db 0x0 ; Base (bits 16-23)
db 10010010b ; Access Byte
db 11001111b ; Flags , Limit (bits 16-19)
db 0x0 ; Base (bits 24-31)
; 栈段描述符
gdt_stack:
dw 0x7c00 ; Limit (bits 0-15)
dw 0x0 ; Base (bits 0-15)
db 0x0 ; Base (bits 16-23)
db 10010010b ; Access Byte
db 01000000b ; Flags , Limit (bits 16-19)
db 0x0 ; Base (bits 24-31)
gdt_end:
; GDT descriptior
gdt_descriptor:
dw gdt_end - gdt_start - 1 ; Size of our GDT, always less one of the true size
dd gdt_start ; Start address of our GDT
```

## 0x05. A20地址线

A20的“A”是“Address”的首字符，A20也就是第21根地址线的意思。这里涉及到一个历史遗留问题，但如果我们是在Bochs这样的模拟器上进行调试的话没有影响，所以我把这个问题放到了最后来讲。

在8086处理器上只有20根地址线，最多支持20位的地址，因此每当物理地址达到最高端0xFFFFF时，再加上1又会绕回最低端0x00000。为了兼容地址回绕这个特性，后来生产的CPU都会默认关闭第21根地址线，需要操作系统在进入保护模式时手动开启。

不过某些BIOS，包括模拟器（qemu, Bochs等）会默认帮我们打开A20地址线，所以我们不需要手动开启了，我们可以用一段程序来测试A20地址线是否打开：

```asm
PModeTest:
mov edi, 0x112345
mov esi, 0x012345
mov [esi], esi
mov [edi], edi
mov eax, [esi]
cmp eax, edi
je A20_DISABLE
push aA20_ENABLE
call print
jmp PModePause
A20_DISABLE:
push aA20_DISABLE
call print
PModePause:
hlt
jmp PModePause
aA20_ENABLE db 'A20 Enable', 0
aA20_DISABLE db 'A20 Disable', 0
```

测试结果，可以看到在Bochs中A20地址线确实是默认开启的：

![图片描述](upload/attach/202109/910514_F75TH3UABYQJWSD.png)

从80486处理器开始，0x92端口的第2位用来控制A20地址线，将0x92端口的第2位置1即可开启A20地址线，如果要手动开启A20地址线的话，可以使用以下代码：

```asm
in al, 92h
or al, 00000010b
out 92h, al
```

这篇文章的内容是我在研究自制操作系统时遇到的一个问题，即如何从16位实模式切换到32位保护模式，网上的资料比较少，讲得也不是很详细，在这跟大家分享一下我的解决方案。

## 0x00. 前置知识

- [计算机是如何启动的？](https://bbs.kanxue.com/elink@64aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0#2j5h3&6&6K9h3k6W2L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4x3#2)9J5c8U0l9J5i4K6u0r3j5X3!0G2N6r3W2F1k6#2)9J5k6h3S2@1L8h3H3`.)

- [自制操作系统：引导扇区的实现](https://bbs.kanxue.com/elink@7ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3K9X3W2I4K9h3&6Y4N6%4g2Q4x3V1k6H3i4K6u0r3L8%4y4Q4y4h3k6T1L8$3!0@1i4K6g2X3M7$3g2U0N6r3!0J5i4K6u0W2K9s2c8E0L8l9`.`.)

## 0x01. 实模式和保护模式

请参考：[实模式和保护模式](https://bbs.kanxue.com/elink@2abK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6M7$3y4e0x3I4y4o6c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3J5z5o6p5^5x3e0V1I4)

现代CPU有三种工作模式：实模式、保护模式和虚拟8086模式。CPU在加电后自动进入16位实模式，在主引导记录MBR加载特定的操作系统后，由操作系通经过一些设置切换到32位保护模式。

以WIN7为例，首先由MBR载入启动管理器`bootmgr`，在`bootmgr`中进行实模式到保护模式的切换，随后装载`winload.exe`，然后通过`winload.exe`加载Windows7内核，从而启动整个Windows7系统。

## 0x02. 实模式到保护模式的切换

从实模式切换到保护模式大致可以分为以下几个步骤：

1. 屏蔽中断

2. 初始化全局描述符表（GDT）

3. 将`CR0`寄存器最低位置1

4. 执行远跳转

5. 初始化段寄存器和栈指针

### 一、屏蔽中断

在16位实模式下的中断由BIOS处理，进入保护模式后，中断将交给中断描述符表`IDT`里规定的函数处理，在刚进入保护模式时`IDTR`寄存器的初始值为0，一旦发生中断（例如BIOS的时钟中断）就将导致CPU发生异常，所以需要首先屏蔽中断。

屏蔽中断可以使用`cli`指令：

|  |  |
| --- | --- |
| 1 | `cli` |

### 二、初始化GDT

屏蔽中断后我们就可以正式开始切换到保护模式的过程了。首先我们要完成GDT的初始化，这也是整个切换过程中最复杂的一步。

32位保护模式下的地址计算方式和16位实模式不一样，16位保护模式支持20位的寻址，物理地址等于`段地址 * 16 + 偏移地址`，其中段地址可以通过段寄存器获得，段地址和偏移地址都是16位。

在32位保护模式中，段与段之间是互相隔离的，当访问的地址超出段的界限时处理器就会阻止这种访问，因此每个段都需要有起始地址、范围、访问权限以及其他属性四个部分，这四个部分合在一起叫做段描述符（Segment Descriptor），总共需要8个字节来描述。但Intel为了保持向后兼容，将段寄存器仍然规定为16-bit，显然我们无法用16-bit的段寄存器来直接存储64-bit的段描述符。

解决的办法是将所有64-bit的段描述符放到一个数组中，将16-bit段寄存器的值作为下标来访问这个数组（以字节为单位），获取64-bit的段描述符，这个数组就叫做全局描述符表（Global Descriptor Table, GDT）。

![图片描述](https://bbs.kanxue.com/upload/attach/202109/910514_8RZQEE69CCG79B6.png)

上面这个就是64位段描述符的具体结构，包括段起始地址Base，限长Limit，Access Byte和Flags都是访问权限，结构如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202109/910514_DWYGSURU2NPHHBX.png)

Access Byte和Flags各项的解释，我直接复制OS Dev)了，更详细的解释可以参考《x86汇编语言：从实模式到保护模式》这本书的11.3节:

- `Pr`: Present bit. This must be **1** for all valid selectors.
