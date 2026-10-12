
# 代码优化常识

> 原文：[`www.pediy.com/kssd/pediy05/pediy50098.htm`](https://www.pediy.com/kssd/pediy05/pediy50098.htm)

## 32 位代码优化常识

原作者: `Benny/29A`

翻译改写:`hume/冷雨飘心`

[注意:这不是鹦鹉学舌的翻译,我尽量以我的理解传达原文的本意]

关于代码优化的文章实在太多了,遗憾的是大部分我都没有看,尽管他们就摆在我的床边(每当我要看的时候就忍不住打哈欠...嘿嘿).这篇文章较短所以翻了一下.

代码优化的含义:

代码优化的目标当然是体积小和速度快,但是在通常的情况下二者就象鱼和熊掌一样不能得兼,我们通常寻找的是这二者的折中,究竟应该偏向何方,那就得具体看我们的实际需要.

但有些常识是我们应该牢记的,下面就结合我们最常遇到的具体情况来漫谈一下:

## 1.寄存器清 0

我绝对不想再看到下面的写法:

```
1)      mov eax, 00000000h                    ;5 bytes
```

看起来上面的写法很符合逻辑,但你应当意识到还有更加优化的写法:

```
2)      sub eax, eax                          ;2 bytes
```

```
3)      xor eax, eax                          ;2 bytes
```

看看后面的字节数你就应该理解为什么要这么作了,除此之外,在速度上也没有损失,他们一样快,但你喜欢 `xor` 还是 `sub` 呢?我是比较喜欢 `xor`,原因很简单,因为我数学不好....

不过 Microsoft 比较喜欢 `sub`....我们知道 windows 运行的慢....(呵呵,当然是玩笑这并不是真正原因 X-D!)

## 2.测试寄存器是否为 0

我也不希望看到下面的代码:

```
1)      cmp eax, 00000000h                    ;5 bytes
        je _label_                            ;2/6 bytes (short/near)
```

* 注意很多指令针对 `eax` 作了优化,你要尽可能多地实用 `eax`,比如 `CMP EAX, 12345678h` (5 bytes)

如果INSERT你使用其他寄存器,就是 6bytes *

让我们看看,简单的比较指令居然要用 7/11 bytes,No No No,试试下面的写法:

```
2)      or eax, eax                          ;2 bytes
        je _label_                            ;2/6 (short/near)
```

```
3)      test eax, eax                        ;2 bytes
        je _label_                            ;2/6 (short/near)
```

呵呵,只有 4/8 bytes,看看我们可节省多少字节啊 3/4 字节...那么接下来的问题是你喜欢 `OR` 还是 `TEST` 呢,就我个人而言,比较喜欢 `TEST`,因为 `test` 不改变任何寄存器,并不向任何寄存器写入内容,这通常能在 `pentium` 机上取得更快的执行速度.

别高兴的太早,因为还有更值得我们高兴的事情,假如你要判断的是 `eax` 寄存器,那么看看下面的,是不是更有启发?

```
4)      xchg eax, ecx                        ;1 byte
        jecxz _label_                         ;2 bytes
```

在短跳转的情况下我们比 2)和 3)又节省了 1 字节.oh....\_\_\_...

## 3.测试寄存器是否为 0FFFFFFFFh

一些 API 返回-1,因此如何测试这个值呢?看你可能又要这样:

```
1)      cmp eax, 0ffffffffh                  ;5 bytes
        je _label_                            ;2/6 bytes
```

hey,不要这样,写代码的时候想一想,于是有了下面的写法:

```
2)      inc eax                              ;1 byte
        je _label_                            ;2/6 bytes
        dec eax                               ;1 byte
```

可以节省 3 bytes 并且执行速度会更快.

## 4.置寄存器为 0FFFFFFFFh

看看假如你是 Api 的作者,如何返回-1?这样吗?

```
1)      mov eax, 0ffffffffh                  ;5 bytes
```

看了上面的不会再这么 XXX 了吧?看看下面的:

```
2)      xor eax, eax / sub eax, eax          ;2 bytes
        dec eax                               ;1 byte
```

节省一个字!还有写法:

```
3)      stc                                  ;1 byte
        sbb eax, eax                          ;2 bytes
```

这有时还可以优化掉 1 byte:

```
        jnc _label_
        sbb eax, eax                          ;2 bytes only!
```

`_label_`: ...

我们为什么用 `asm` 呢?这就是原因.

## 5.寄存器清 0 并移入低字数值

```
1)      xor eax, eax                          ;2 bytes
        mov ax, word ptr [esi+xx]             ;4 bytes
```

????--->不会吧,这可能是最多初学者的写法了,我当然原来也是,看了 `benny` 的文章之后我决定改写为:

```
2)      movzx eax, word ptr [esi+xx]          ;4 bytes
```

收获 2 bytes!

下面的

```
3)      xor eax, eax                          ;2 bytes
        mov al, byte ptr [esi+xx]             ;3 bytes
```

就相应改为:

```
4)      movzx eax, byte ptr [esi+xx]          ;4 bytes
```

我们应当尽可能利用 `movzx`

```
5)      xor eax, eax                          ;2 bytes
        mov ax, bx                            ;3 bytes
```

因为执行速度不慢并通常能节省字节...

```
6)      movzx eax, bx                        ;3 bytes
```

## 6.关于 push

下面是着重代码体积的优化,因为寄存器操作总要比内存操作要快.

```
1)      mov eax, 50h                          ;5 bytes
```

这样就小了 1 word

```
2)      push 50h                              ;2 bytes
        pop eax                               ;1 byte
```

当操作数只有 1 字节时候,`push` 只有 2 bytes,否则就是 5 bytes,记住!

下一个问题,向堆栈中压入 7 个 0

```
3)      push 0                                ;2 bytes
        push 0                                ;2 bytes
        push 0                                ;2 bytes
        push 0                                ;2 bytes
        push 0                                ;2 bytes
        push 0                                ;2 bytes
        push 0                                ;2 bytes
```

占用 14 字节,显然不能满意,优化一下

```
4)      xor eax, eax                          ;2 bytes
        push eax                              ;1 byte
        push eax                              ;1 byte
        push eax                              ;1 byte
        push eax                              ;1 byte
        push eax                              ;1 byte
        push eax                              ;1 byte
        push eax                              ;1 byte
```

可以更紧凑,但会慢一点的形式如下:

```
5)      push 7                                ;2 bytes
        pop ecx                               ;1 byte
```

`_label_`:  `push 0`                                ;2 bytes
        `loop _label_`                        ;2 bytes

可以节省 7 字节....

有时候你可能会从将一个值从一个内存地址转移到另外内存地址,并且要保存所有寄存器:

```
6)      push eax                              ;1 byte
        mov eax, [ebp + xxxx]                 ;6 bytes
        mov [ebp + xxxx], eax                 ;6 bytes
        pop eax                               ;1 byte
```

试试 `push,pop`

```
7)      push dword ptr [ebp + xxxx]           ;6 bytes
        pop dword ptr [ebp + xxxx]            ;6 bytes
```

## 7.乘法

当 `eax` 已经放入被乘数,要乘 `28h`,如何来写?

```
1)      mov ecx, 28h                          ;5 bytes
        mul ecx                               ;2 bytes
```

好一点的写法如下:

```
2)      push 28h                              ;2 bytes
        pop ecx                               ;1 byte
        mul ecx                               ;2 bytes
```

哇这个更好::

```
3)      imul eax, eax, 28h                    ;3 bytes
```

`intel` 在新 CPU 中提供新的指令并不是摆设,需要你的使用.

## 8.字符串操作

你如何从内存取得一个字节呢?

速度快的方案:

```
1)      mov al/ax/eax, [esi]                  ;2/3/2 bytes
        inc esi                               ;1 byte
```

代码小的方案:

```
2)      lodsb/w/d                             ;1 byte
```

我比较喜欢 `lod` 因为他小,虽然速度慢了点.

如何到达字符串尾呢?

`JQwerty`'s method:

```
9)      lea esi, [ebp + asciiz]               ;6 bytes
```

`s_check`: `lodsb`                               ;1 byte
        `test al, al`                         ;2 bytes
        `jne s_check`                         ;2 bytes

`Super`'s method:

```
10)     lea edi, [ebp + asciiz]               ;6 bytes
        xor al, al                            ;2 bytes
```

`s_check`: `scasb`                               ;1 byte
        `jne s_check`                         ;2 byte

选择哪一个?`Super` 的在 386 以下的更快,`JQwerty` 的在 486 以及 `pentium` 上更快,体积一样,选择由你.

## 9.复杂一点的...

假设你有一个 `DWORD` 表,`ebx` 指向表的开始,`ecx` 是指针,你想给每个 `doword` 加 1,看看如何作:

```
1)      pushad                                ;1 byte
        imul ecx, ecx, 4                      ;3 bytes
        add ebx, ecx                          ;2 bytes
```



## 汇编优化技巧

### 1. `inc` 指令优化

```asm
inc dword ptr [ebx]       ;2 bytes
popad                     ;1 byte
```

可以优化一点,但是好像没人用:

```asm
inc dword ptr [ebx+4*ecx] ;3 bytes
```

一条指令就节省 6 字节,而且速度更快,更易读,但好像没有什么人用?...why?

还可以有立即数:

```asm
pushad                    ;1 byte
imul ecx, ecx, 4          ;3 bytes
add ebx, ecx              ;2 bytes
add ebx, 1000h            ;6 bytes
inc dwor ptr [ebx]        ;2 bytes
popad                     ;1 byte
```

优化为:

```asm
inc dword ptr [ebx+4*ecx+1000h] ;7 bytes
```

节省了 8 字节!

### 2. `lea` 指令用法

看一下 `lea` 指令能为我们干点什么呢?

```asm
lea eax, [12345678h]
```

`eax` 的最后结果是什么呢?正确答案是 `12345678h`.

假设 `EBP = 1`

```asm
lea eax, [ebp + 12345678h]
```

结果是 `123456789h`....呵呵比较一下:

```asm
lea eax, [ebp + 12345678h]   ;6 bytes
```

```asm
mov eax, 12345678h           ;5 bytes
add eax, ebp                 ;2 bytes
```

#### 示例 5

```asm
mov eax, 12345678h           ;5 bytes
add eax, ebp                 ;2 bytes
imul ecx, 4                  ;3 bytes
add eax, ecx                 ;2 bytes
```

#### 示例 6 用 `lea` 进行计算

```asm
lea eax, [ebp+ecx*4+12345678h] ;7 bytes
```

速度上一条 `lea` 指令更快!不影响标志位...记住下面的格式,在许多地方善用他们你可以节省时间和空间.

```
OPCODE <SIZE PTR> [BASE + INDEX*SCALE + DISPLACEMENT]
```

### 10. 病毒重定位优化

下面的代码你不应该陌生

```asm
call gdelta
gdelta: pop ebp
sub ebp, offset gdelta
```

在以后的代码中我们这样使用 delta 来避免重定位问题

```asm
lea eax, [ebp + variable]
```

这样的指令在应用内存数据的时候是不可避免的,如果能优化一下,我门将会得到数倍收益,打开你的 sice 或者 trw 或者 ollydbg 等调试器,看看:

```asm
lea eax, [ebp + 401000h]     ;6 bytes
```

假如是下面这样

```asm
lea eax, [ebp + 10h]         ;3 bytes
```

也就是说如果 ebp 后面变量是 1 字节的话,总的指令就只有 3 字节

修改一下最初的格式变为:

```asm
call gdelta
gdelta: pop ebp
```

在某些情况下我们的指令就只有 3 字节了,可以节省 3 字节,嘿嘿,让我们看看:

```asm
lea eax, [ebp + variable - gdelta] ;3 bytes
```

和上面的是等效的,但是我们可以节省 3 字节,看看 CIH...

### 11. 其他技巧

如果 `EAX` 小于 `80000000h`,edx 清 0:

```asm
xor edx, edx                 ;2 bytes, but faster
```

```asm
cdq                          ;1 byte, but slower
```

我一直使用 `cdq`,为什么不呢?体积更小...

下面这种情况一般不要使用 esp 和 ebp,使用其他寄存器.

```asm
mov eax, [ebp]               ;3 bytes
```

```asm
mov eax, [esp]               ;3 bytes
```

```asm
mov eax, [ebx]               ;2 bytes
```

交换寄存器中 4 个字节的顺序?用 `bswap`

```asm
mov eax, 12345678h           ;5 bytes
bswap eax                    ;2 bytes
;eax = 78563412h now
```

#### 用 `jmp` 替代 `CALL`

```asm
call _label_                 ;5 bytes
ret                          ;1 byte
```

```asm
jmp _label_                  ;2/5 (SHORT/NEAR)
```

如果仅仅是优化,并且不需要传递参数,请尽量用 `jmp` 代替 `call`

#### 比较 `reg/mem` 时如何节省时间

```asm
cmp reg, [mem]               ;slower
```

```asm
cmp [mem], reg               ;1 cycle faster
```

#### 乘 2 除 2 如何节省时间和空间

```asm
mov eax, 1000h
mov ecx, 4                   ;5 bytes
xor edx, edx                 ;2 bytes
div ecx                      ;2 bytes
```

```asm
shr eax, 4                   ;3 bytes
```

```asm
mov ecx, 4                   ;5 bytes
mul ecx                      ;2 bytes
```

```asm
shl eax, 4                   ;3 bytes
```

#### `loop` 指令

```asm
dec ecx                      ;1 byte
jne _label_                  ;2/6 bytes (SHORT/NEAR)
```

```asm
loop _label_                 ;2 bytes
```

再看:

```asm
je $+5                       ;2 bytes
dec ecx                      ;1 byte
jne _label_                  ;2 bytes
```

```asm
loopXX _label_ (XX = E, NE, Z or NZ) ;2 bytes
```

`loop` 体积小,但 486 以上的 cpu 上执行速度会慢一点...

#### 寄存器交换比较

```asm
push eax                     ;1 byte
push ebx                     ;1 byte
pop eax                      ;1 byte
pop ebx                      ;1 byte
```

```asm
xchg eax, ebx                ;1 byte
```

```asm
xchg ecx, edx                ;2 bytes
```

如果仅仅是想移动数值,用 `mov`,在 pentium 上会有较好的执行速度:

```asm
mov ecx, edx                 ;2 bytes
```

#### 代码合并优化

未优化:

```asm
lbl1:  mov al, 5             ;2 bytes
       stosb                 ;1 byte
       mov eax, [ebx]       ;2 bytes
       stosb                 ;1 byte
       ret                  ;1 byte

lbl2:  mov al, 6             ;2 bytes
       stosb                 ;1 byte
       mov eax, [ebx]       ;2 bytes
       stosb                 ;1 byte
       ret                  ;1 byte
```

--------  
;14 bytes

优化了:

```asm
lbl1:  mov al, 5             ;2 bytes
lbl:    stosb                 ;1 byte
        mov eax, [ebx]       ;2 bytes
        stosb                 ;1 byte
        ret                  ;1 byte

lbl2:  mov al, 6             ;2 bytes
       jmp lbl               ;2 bytes
```

--------  
;11 bytes

#### 读取常数变量

```asm
mov [ebp + variable], eax    ;6 bytes
...
...
variable dd 12345678h        ;4 bytes
```

优化为:

```asm
mov eax, 12345678h           ;5 bytes
variable = dword ptr $ - 4
...
...
```



`mov [ebp + variable], eax`            ;6 bytes

呵呵,好久没看到这么有趣的代码了,前提是编译的时候支持代码段的写入属性要被设置.

最后介绍未公开指令 `SALC`,现在的调试器都支持...什么含义呢:就是 `CF` 位置 1 的话就将 `al` 置为 `0xff`

------------------------------------------------------------------

1)      `jc _lbl1`                             ;2 bytes
        `mov al, 0`                           ;2 bytes
        `jmp _end`                             ;2 bytes
      `_lbl:` `mov al, 0ffh`                   ;2 bytes
      `_end:` ...

2)      `SALC` `db`    `0d6h`                  ;1 byte ;)
\------------------------------------------------------------------>over...

2002---hume  4dd466d0e1e0cc123afbe504da2462ca
