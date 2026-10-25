
`movq     mm1, mm6        ; XXXX000000000000`

`pmullw   mm0, mm7        ; 8765705800870070`

`pand     mm1, mm0        ; 8765000000000000`

`psrld    mm0, 28         ; 0000000000004321`

`psrlw    mm1, 8          ; 0000000087650000`

`por      mm0, mm1        ; 0000000087654321`

`packuswb mm0, mm0        ; [?B?A?B?A]`

`packuswb mm0, mm0        ; [BABABABA]`

`movd     eax, mm0`

`mov      [edi],ax`

`add      esi, 8`

`add      edi, 2`

`dec      ecx`

`jnz      @quads`

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

因为这些个bits都是孤立的，一序列的shift和或操作可以用一系列的shift和add操作代替。但一序列的shift和add又可以合并进单单一个的乘。(/:^] )透过一序列的虚拟检测，可以发现这个乘对我们的性能而言是一个好交易，它在现在的X86 CPU上还是一个完全适应CPU流水线的质量，这个循环里的很多质量，都应该可以并行的同步在CPU中执行(指令间没有数据依存)。这个方案处理64个源字节只用了可以接受的几个时钟-它的性能看上去就不像任何c写的公式能接近的。

第二个，这是给平均分布而且有规律的字节流设计(averaging streams of bytes together (and rounding down.))。Gordy找的是一个给per-K6-2 MMX/3DNow!的解决方案(这个CPU的指令集里没有pavgusb和pavb)，也就是一个很古怪的，要用int实现的方案。这是我通过的int方案:

!!!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!

```
; Integer Solution by Paul Hsieh

LTop:

mov     eax, [esi]

mov     ebx, [edx]

mov     ebp, eax

and     eax, ebx

xor     ebx, ebp

shr     ebx, 1

and     ebx, 7F7F7F7Fh

add     eax, ebx

mov     [edi], eax

dec     ecx

jnz     LTop
```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```
; 这是MMX Solution by Paul Hsieh

LTop:

movq    mm0, [esi]

movq    mm1, [edx]

movq    mm2, mm0

pand    mm0, mm1

pxor    mm1, mm2

psrlb   mm1, 1

paddb   mm0, mm1

movq    [edi], mm0

dec     ecx

jnz     LTop
```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

这二个方案用的下面的这个发现

-------quote-----------------------

```
A + B = (A and B) * 2 + (A xor B)

And therefore:

(A + B)/2 = (A and B) + (A xor B)/2
```

------------------------------

对于int而言，原样的A+ B可能会引起数值溢出。但`(A and B) + (A xor B)/2`就不会了。

第三个pixel bashing的函数是一个饱和加(当溢出时，把被塞饱饱的数字返回来，比方32bits的int溢出了0xffffffff)。用 MMX的指令实现的话，这只是一个琐碎的只需要一个paddusb的小程序(Gordy指出这个指令了)。但用int实现的函数呢？我来用下面的函数实现了int的饱和加：

!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```
; Integer solution by Paul Hsieh

mov eax, [esi]      ; src0

mov ebx, [edi]      ; src1

mov ecx, eax

mov edx, ebx

and ecx, ebx        ; first bit carry

xor edx, eax        ; first bit add (mod 2)

and eax, 0xFEFEFEFE

and ebx, 0xFEFEFEFE

add eax, ebx        ; Add top 7 bits (A&0xFE)+(B&0xFE)

sar eax, 1          ; >>= 1 to capture carry bits

and ecx, 0x01010101 ; Is there a carry to the second bit?

add eax, ecx        ; (...)>>1

mov ecx, eax

and edx, 0x01010101 ; first bit

and eax, 0x7F7F7F7F

shr ecx, 7

shl eax, 1          ; (...)&0xFE

and ecx, 0x01010101 ; overflows

or  eax, edx        ; ((...)&0xFE) + (((A&0x01)+(B&0x01))&1)

xor ecx, 0x81818181 ; blockers

sub ecx, 0x01010101 ; 0->80, 1->7F

and ecx, 0x7F7F7F7F ; 0->00, 1->7F

or  eax, ecx

shl ecx, 1

or  eax, ecx
```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

现在，我们要把低位的bits们分开，以确定我们可以用8bits大小的空间提高一个并行同时实现的数个加法。

-------quote-----------------------

```
A = (A&0xFE) + (A&0x01)

B = (B&0xFE) + (B&0x01)

So

A + B        = ((A&0xFE)+(B&0xFE)) + ((A&0x01)+(B&0x01))

= ((A&0xFE)+(B&0xFE)) + ((A&B&0x01)*2 + ((A xor B)&0x01))
```

------------------------------

carry flag溢出到9th bit的标志，被转换成一个0x00或0x7f mask了。用一个or mask进目标的操作，在左移一位，这样就和有carry flag时的adc/sbb有相同的作用了。

然后现在呢，当carry flag要被第一个add扑捉到并且用掉了，c编译器又不能复述出来我们的这段代码了

update 补充:

Oliver Weichhold 在comp.lang.asm.x86问了一个很有趣的问题：

----Quote--------------------------

```
I need to convert a color value (actually a texel fetched from a A4R4G4B4 texture) to a A15R15G15B15 color value held in an MMX register.

我需要把一个颜色值(事实上是从一个A4R4G4B4 texture上牵引出来的一个像素)，到一个A15R15G15B15 颜色里，结果保存到MMX寄存器里
```

------------------------------

看到这个问题的本能反应一般都是用多个shift位移和mask来实现，但这个真的是完全没必要的。考虑一下，在需要映射它们之间的关系时你不会真的想把16个独立bits的输入随便的填充到输出里面吧。

是时候找出来"查表法"了，然后的确的，你可以看到一个非常精彩简单的代码了:

!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```
movzx     eax, word ptr input_A4R4G4B4

movzx     ebx, al

shr       eax, 8

movd      mm0, dword ptr Table_G4B4_to_G15B15 [ebx*4]

movd      mm1, dword ptr Table_A4R4_to_A15R15 [eax*4]

punpckldq mm0, mm1
```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

这个表用一些简单的c代码就可以初始化好。

### 11.转化字符串成大写

// /;^] 这个就是我搜索这篇文章出来的原因。我也最后总结下学习下的一点点成果，啰嗦下自己的note。

在传统的智慧里，会说转化ASCII字符串到大写会需要一个查一个表，每一个每一种字节都需要表。可是性能的问题来了，X86处理器想要的是 32(或16) bits的寄存器，那这下这些那样的 byte->dword(或者word)的转化就会扭着CPU的性子发生了。这对AMD来说，就像被罚站了一样，对intel的也像罚抄汉字成语一样。

那我们的改进型算法在哪呢？唔，我们可以用SIMD(Single instruction, multiple data一条指令，多条数据，这儿意指我们自创的高级CPU指令)来减轻运算压力得到想要的性能来，那首先我们需要把数字范围[61h, 7Ah]到[41h, 5Ah]的转化方法的流程也转化转化。而我进行转化的关键就在于ASCII的码只定义在[00h, 7Fh] 之间，然后可以通过往后转转(转5到[66-7f])这个范围里的数字得到在一个特殊范围的数字，再masking一下(确定得到的还是[00h, 7Fh] 之间)，得到的数往后再转转(转1a，之前66-7f范围的得到80-99之间的数字)，这样把得到的数字右移2位(0x80>>2== 0x20)又和mask 0x20 and下，这样就只有61-7a之间的数字才能0x20的结果。恰好是大写和小写的差距。 /:^]

!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```
; Integer SIMD uppercase(string) by Paul Hsieh

mov  eax, src[esi]

mov  ebx, 7f7f7f7fh

mov  edx, 7f7f7f7fh

and  ebx, eax        ; 61-7a | 00-60,7b-7f

add  ebx, 05050505h  ; 66-7f | 05-65,80-84

and  ebx, edx        ; 66-7f | 00-65

add  ebx, 1a1a1a1ah  ; 80-99 | 1a-7f

shr  ebx, 2

and  ebx, 20202020h  ;    20 | 00

sub  eax, ebx        ; 41-5a | 00-60,7b-7f
```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

这个函数可以算是现代C编译器所能达到的最好效果了。粗看了一下，这个函数应该很好转换为MMX版本的。

update 补充:

其实我们还需要支持非ASCII码(例如,[80h, FFh]之间的数)。解决掉这个问题的还有一个额外奖励的，可以处理这样范围的upper也可以处理UTF-8 编码的unicode了(894K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4*7K9h3I4D9K9h3!0F1L8h3!0F1K9$3g2&6M7#2)9J5k6h3y4G2L8g2)9J5c8Y4q4W2k6q4)9J5c8Y4g2F1K9h3y4G2k6r3g2Q4x3X3g2Z5N6r3#2D9 魔兽争霸的encode，熟悉吧 /:^] 应该打个广告，喜欢游戏的可以试试横扫千军，我最近在想办法做他的汉化)。但这个就是一个小改变，只是很不幸的，这个改进让主循环占用了更多的一个时钟。

!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```
; Integer SIMD uppercase(string) on UTF-8 data by Paul Hsieh

mov  eax, src[esi]

mov  ebx, 0x7f7f7f7f

mov  edx, 0x7f7f7f7f

and  ebx, eax          ; 61-7a | 00-60,7b-7f

mov  ecx, eax

add  ebx, 0x05050505   ; 66-7f | 05-65,80-84

not  ecx               ; flip top bits

and  ebx, edx          ; 66-7f | 00-65

add  ebx, 0x1a1a1a1a   ; 80-99 | 1a-7f

and  ebx, ecx          ; turn off top bit for non-ASCII

shr  ebx, 2

and  ebx, 0x20202020   ; 20 | 00

sub  eax, ebx
```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

update 补充:

Norbert Juffa 提示我说，Sun开源的Solaris的源代码，他们在一些C字符串函数中使用了这个机器，包括大小写检测，tolower的例子，源代码可以在这儿看到: 04fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4$3M7#2)9J5k6h3!0H3k6h3&6K6L8$3I4S2M7X3W2K6i4K6u0W2L8%4u0Y4i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3P5s2u0W2k6W2)9J5c8Y4g2K6M7W2)9J5c8Y4y4J5j5#2)9J5c8X3I4A6j5W2)9J5c8X3I4A6j5X3y4Q4x3V1k6K6M7r3q4J5j5%4j5&6i4K6u0r3k6$3g2F1i4K6u0r3 (已失效)

update 补充:

我重新设计了一个更先进的函数，可以处理所有的UTF-8范围的Uppercase。而且这个版本的对指令之间的数据依存进行的处理，提高了并行执行的能力，也就应该执行的更快了。

!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```
; Integer SIMD uppercase(string) on UTF-8 data v2 by Paul Hsieh

mov  eax, src[esi]

mov  ebx, 0x80808080

mov  edx, eax          ; src
```
