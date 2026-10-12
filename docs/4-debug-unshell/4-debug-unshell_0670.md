
# Execryptor2.0.26 全过程分析

> 原文：[`www.pediy.com/kssd/pediy11/110094.html`](https://www.pediy.com/kssd/pediy11/110094.html)

## 【前言】

by `hkfans`

这个壳的这个版本主要以 `TLS` 为主线展开的... 两个 `TLS` 函数的调用完成了壳大部分过程

第一个 `TLS` 回调函数: 解压各个区段的数据，修改 `E8` `E9` 的偏移量，再获取几个 `API` 函数的地址,如 `GetModuleHandleA`...

第二个 `TLS` 回调函数: 加载部分 `IAT` 的数据。。因为这个壳可以支持动态 `Import`。。所以部分在这个 `TLS` 函数处理。。另外部分动态加载，也就是当调用到的时候在获取(只获取一次)

当两个 `TLS` 回调函数执行完毕后，就进入 `EP` 处代码... 所以 `EP` 处的代码是会被执行的.....

另外,这个壳有很多检测 `INT3` 断点的代码。。会检测 `EP` 处, `VMOEP` 处, `TLS` 回调函数 等函数地址处的 `INT3` 断点..

检测 `API` 函数的全部指令是否被设软件断点....通过 `SEH` 检测硬件断点.

ps: 有个不理解的问题.. 有的地方设置硬件断点不会被检测到。。有的设置硬件断点会被检测到。。`SOD` 照样还是那样设置。。。呵呵

OK.. 下面就按照每个 `TLS` 进行分析

## 【分析过程】

## 【第一个 TLS 回调函数】

代码:



# 动态分析与脱壳过程分析

## 1. 断点设置与初始检测
对第一个 `TLS` 回调函数下断点。断到这里 `===>` 然后就是后面的内容了...

```assembly
00628BEF    E8 53F4FFFF       call    00628047
```

## 2. TLS 第一个回调函数头部检测
检测了 `TLS` 第一个回调函数头部是否设置了 `INT3` 软件断点。。。

// `[00628BEF]` = `E8 CALL` ----> 计算公式: (`[00628BEF]` - `0x99`) * `[00628BEF]` = (`0xe8` - `0x99` ) * `0xe8` = `0x98` (一个字节)

// 如果是 `INT3` 断点的话 ---> (`0xCC` - `0x99`) * `0xCC` = `0xA4`

```assembly
0062804B   0F89 35030000      jns     00628386
006288D7                 push    ebp                       Morph instruction
006285B8   0F89 7CFAFFFF      jns     0062803A
0062803A   8BEC             mov     ebp, esp
0062803F                 push    ecx                       Morph instruction
00628D79   8D05 EF8B6200     lea     eax, dword ptr [628BEF]
0062985C                 push    00629505                  Morph instruction
0062890F                 push    ebp                       Morph instruction
00628912   8BEC             mov     ebp, esp
00628096                 push    ecx                       Morph instruction
00628099   8945 FC          mov     dword ptr [ebp-4], eax
0062809C                 mov     eax, FFFFFFC              eax = FFFFFFC
00629479   03C5             add     eax, ebp
00629481   8B00             mov     eax, dword ptr [eax]
00629298   8A00             mov     al, byte ptr [eax]
0062929A   2C 99            sub     al, 99
0062929C                 mov     edx, D4237042             edx = D4237042
006295FA   F7C2 00040000     test    edx, 400
00629688   0F84 95FBFFFF     je      00629223
006288FB   81F2 53E2FF84     xor     edx, 84FFE253
00628901   03D5             add     edx, ebp
006284AD   81C2 EB6D23AF     add     edx, AF236DEB
006284B3   8B12             mov     edx, dword ptr [edx]
006284B5   F62A             imul    byte ptr [edx]
0062888B   3C A4            cmp     al, 0A4
0062888D   0F85 EA0D0000     jnz     0062967D
00629682                 pop     ecx                         Morph instruction
00629448                 pop     ebp                         Morph instruction
00629449   C3             retn


// codeclean 之后返回到这里 ..  ==> 006293D0 返回到 TLS 处理完毕收尾的地方...

00628B21                 push    006293D0            Morph instruction
```

## 3. 流程区分与控制
这里修改指令为 `0xC3 retn` 是有目的的... `TLS` 回调函数会被调用 2 次.. 所以.. 这里修改为 `0xC3` 的目的就是区别第一次和第二次的流程。。不可能都走一样的路线的。。。

所以，这个 `C3` 很关键...

// 自己指令的 `EIP` ---> 填充为 `0xC3` `retn` 指令....
```assembly
006297C2   8D05 C2976200     lea     eax, dword ptr [6297C2]  // 这里会变为 retn
0062950F   C600 C3          mov     byte ptr [eax], 0C3
```

## 4. 代码段数据解压
`00555108` 内存数据：
```text
00555108  00 10 40 00 1E 04 10 40 00 03 07 42 6F 6F 6C 65  .@.@.Boole
```

```assembly
00629512   B8 08515500        mov     eax, 00555108       // 存放压缩后的数据
00629517   0F83 67FAFFFF      jnb     00628F84      // 这里的很多 jnb js 指令其实是垃圾代码来着。只是不知道怎么确定
0062951D   0F88 42EDFFFF      js      00628265

00628815   8B10             mov     edx, dword ptr [eax]    // edx = 00401000 --> 代码段起始地址
0062919F   09D2             or      edx, edx
006291A1   0F84 9BF9FFFF     je      00628B42


00628549      push    edx    // edx = 00401000
0062854C   57             push    edi    // edi = 00400000


006290FF                 mov     edi, 00000004                 edi = 00000004
00628063   03C7             add     eax, edi  // eax ==> 存放代码段压缩后的数据
00629468   5F             pop     edi


// 返回地址...
006281E0                 push     0062920A                ; Morph instruction
00628630                 push     00627F50                ; Morph instruction
00628633    C3           retn
```

### 4.1 解压被保护程序的代码段数据
比较执行该代码之前，都是 0. 执行后 有被保护程序字符串数据。。。所以这段代码是解压被保护程序的代码。

```assembly
00627F50    56           push    esi
00627F51    57           push    edi
00627F52    53           push    ebx
00627F53    31DB         xor     ebx, ebx
00627F55    89C6         mov     esi, eax
00627F57    89D7         mov     edi, edx
00627F59    0FB606       movzx   eax, byte ptr [esi]
00627F5C    89C2         mov     edx, eax
00627F5E    83E0 1F      and     eax, 1F
00627F61    C1EA 05      shr     edx, 5
00627F64    74 2D        je      short 00627F93
00627F66    4A           dec     edx
00627F67    74 15        je      short 00627F7E
00627F69    8D5C13 02    lea     ebx, dword ptr [ebx+edx+2]
00627F6D    46           inc     esi
00627F6E    C1E0 08      shl     eax, 8
00627F71    89FA         mov     edx, edi
00627F73    0FB60E       movzx   ecx, byte ptr [esi]
00627F76    46           inc     esi
00627F77    29CA         sub     edx, ecx
00627F79    4A           dec     edx
00627F7A    29C2         sub     edx, eax
00627F7C    EB 32        jmp     short 00627FB0
00627F7E    C1E3 05      shl     ebx, 5
00627F81    8D5C03 04    lea     ebx, dword ptr [ebx+eax+4]
00627F85    46           inc     esi
00627F86    89FA         mov     edx, edi
00627F88    0FB70E       movzx   ecx, word ptr [esi]
00627F8B    29CA         sub     edx, ecx
00627F8D    4A           dec     edx
00627F8E    83C6 02      add     esi, 2
00627F91    EB 1D        jmp     short 00627FB0
00627F93    C1E3 04      shl     ebx, 4
00627F96    46           inc     esi
00627F97    89C1         mov     ecx, eax
00627F99    83E1 0F      and     ecx, 0F
00627F9C    01CB         add     ebx, ecx
00627F9E    C1E8 05      shr     eax, 5
00627FA1    73 07        jnb     short 00627FAA
00627FA3    43           inc     ebx
00627FA4    89F2         mov     edx, esi
00627FA6    01DE         add     esi, ebx
00627FA8    EB 06        jmp     short 00627FB0
00627FAA    85DB         test    ebx, ebx
00627FAC    74 0E        je      short 00627FBC
00627FAE  ^ EB A9        jmp     short 00627F59
00627FB0    56           push    esi
00627FB1    89D6         mov     esi, edx
00627FB3    89D9         mov     ecx, ebx
00627FB5    F3:A4        rep     movs byte ptr es:[edi], byte ptr>
00627FB7    31DB         xor     ebx, ebx
00627FB9    5E           pop     esi
00627FBA  ^ EB 9D        jmp     short 00627F59

00627FBC    89F0         mov     eax, esi    // 结构指针向下移动
00627FBE    5B           pop     ebx
00627FBF    5F           pop     edi
00627FC0    5E           pop     esi
00627FC1    C3           retn
```

### 4.2 解压完成后返回与结构布局
解压完代码段数据后返回 `0062920A`。

压缩数据的结构布局:
```c
struct stCompressData
{
   DWORD  dest;      // 释放压缩数据的内存地址  --> eg. 00401000 -->(如果为 0, 所以区段解压处理完毕)
   char   pData[MAX];    // 压缩后的数据

   DWORD  sectionRawDataSize;  // 下面两个参数为了修复 E8 E9 的偏移量 (如果为 0,解压下一个区段数据)
   DWORD  sectionRVA;
};
```

所以解压完一个段的数据后，`eax` 指针自动指向第二个结构的起始地址

```text
00401000
data
```

// `005BC0A5` 内存数据
```text
005BC0A5  00 08 0C 00 00 20 4C 00 1B 32 13 8B C0 02 00 8B  .... L.2.
005BC0B5  C0 00 8D 40 00 E0 03 11 B4 49 A0 03 E0 00 01 1D  ?.???
```

// `eax` = `005BC0A5`
```assembly
0062920A    8B10             mov     edx, dword ptr [eax]  ; eax = 005BC0A5  edx = 000C0800
0062920C    52             push    edx

// 安装 SEH..
0062920D    E8 CCF7FFFF       call    006289DE
006289DE    67:64:FF36 0000   push    dword ptr fs:[0]
006287C9    67:64:8926 0000   mov     dword ptr fs:[0], esp

// 一下修改 EFLAGS --> TF 标志位 --> 造成单步异常 -> 等同于指令 int1 --> 如果单步跟踪下去的话，程序运行..
// 因为正确的流程是设置了 TF 标志后，就相当于 INT1 指令.. INT1 下面的代码不会被指令，会被执行就是错误的。。

006283BE    9C             pushfd
00629746    870424         xchg    dword ptr [esp], eax
00629749    81C8 00010000  or      eax, 100               ; TF = 1
0062974F    870424         xchg    dword ptr [esp], eax

// 这条指令造成单步异常...
00629752    9D             popfd

// 直接 F4 到这里程序运行??? 流程是正确的走向???? [ 这里应该是错误的..刚好往下走是到退 SEH 的地方。。而且 EBP 可能修正。。所以才没出错 ]
00628253    81D5 1D83E636  adc     ebp, 36E6831D
00627FF1    81CD B403DF7F or      ebp, 7FDF03B4    ; ebp = 7FFF7FFF
00627FF7    C1E9 0C        shr     ecx, 0C
00627FFA   /0F8C 34070000 jl      00628734

// 退掉 SEH....
00628000   |67:64:8F06 0000  pop     dword ptr fs:[0]
00628006   |870424         xchg    dword ptr [esp], eax
00628009   |58             pop     eax

0062891F    81E2 410EE038  and     edx, 38E00E41
00628925    81C2 5D2AEDAE  add     edx, AEED2A5D
0062892B    F7C2 40000000  test    edx, 40
0062807F    0F84 4F150000  je      006295D4
006295D4    81F2 9A328DBF  xor     edx, BF8D329A
006295E4    03C2             add     eax, edx        ; edx = 4  eax = 005BC0A5 处的结构数据
006295E6    5A             pop     edx               ; edx = 000C0800

// 中间的 SEH 被 codeclean...
00628F7C    870424         xchg    dword ptr [esp], eax
006295A5    09D2             or      edx, edx
006295A7    0F84 AB010000  je      00629758
00628786    0F85 000C0000  jnz     0062938C

// 中间的 SEH 被 codeclean...
00628027                 mov     esi, 629758                  ; esi = 629758  ==> 返回地址
006290A6    873424         xchg    dword ptr [esp], esi
00628E56                 push    005550E0                    ; Morph instruction
00628E59    C3             retn
```

## 5. 修正 E9 指令偏移量
返回到 `005550E0`.... 修正被保护程序中所有的 `E9` 指令的偏离量...

参数:
`edx` = `000C0800` (区段的 `RawDataSize`) `eax` = `00401000` ( 区段 `RVA` )

`<<<< 注意: 005550E0 的计算方法: --> E8 E9 都满足条件... >>>`

```assembly
005550E0    56           push    esi
005550E1    51           push    ecx
005550E2    89C6         mov     esi, eax
005550E4    89D1         mov     ecx, edx
005550E6    83E9 04      sub     ecx, 4
005550E9    FC           cld

// 循环查找   所有的 E9 --> far jmp 指令
005550EA    AC           lods    byte ptr [esi]

005550EB    D0E8         shr     al, 1
005550ED    80F8 74      cmp     al, 74    // 0xE9 / 2 = 0x74
005550F0    75 0E        jnz     short 00555100

// 如果是 jmp --> 处理跳转偏移量...
005550F2    8B06         mov     eax, dword ptr [esi]  // [esi] --> 四字节偏离量..

// 交换 32 位寄存器内的字节顺序 比如：[EAX]=0123 4567H，执行指令： BSWAP EAX ；使[EAX]=6745 2301H .
005550F4    0FC8         bswap   eax
005550F6    01C8         add     eax, ecx
005550F8    8906         mov     dword ptr [esi], eax  // 填充正确的偏离量数值
005550FA    83C6 04      add     esi, 4
005550FD    83E9 04      sub     ecx, 4
00555100    49           dec     ecx         // ecx = C0800 --> 被保护程序代码段大小

00555101  ^ 7F E7        jg      short 005550EA
00555103    59           pop     ecx
00555104    5E           pop     esi
00555105    C3           retn
```

## 6. 循环解压多个区段
返回到 `00628B3C`

```text
005BC099                     00 08 0C 00      ..
005BC0A9  00 20 4C 00 1B 32 13 8B C0 02 00 8B C0 00 8D 40  . L.2..
```

// 代码段大小 = `000C8000`
// 第一个区段的开始地址= `004C2000`
// `eax` = `005BC0A9`

```assembly
00628B3C                 pop     eax                           ; Morph instruction
00629517    0F83 67FAFFFF  jnb     00628F84
0062951D    0F88 42EDFFFF  js      00628265
00628815    8B10             mov     edx, dword ptr [eax]
0062919F    09D2             or      edx, edx      ; edx = 004C2000
006291A1    0F84 9BF9FFFF  je      00628B42

006291A1    0F84 9BF9FFFF  je      00628B42
00628549                 push    edx                       ; Morph instruction
0062854C    57           push    edi
006290FF                 mov     edi, 00000004             ; edi = 00000004
00628063    03C7             add     eax, edi            ; eax = 005BC0AD
00629468    5F           pop     edi
006281E0                 push    0062920A                ; 这个是返回地址 --> 下面进入一个函数

// 跳去该函数
00628630                 push    00627F50                ; Morph instruction
00628633    C3           retn

// 跳到这里..
00627F50    56           push    esi
00627F51    57           push    edi
00627F52    53           push    ebx
00627F53    31DB         xor     ebx, ebx
```

注意和前面的是同一个函数.... 所以这个函数被调用了 多次。。解压多个区段<具体说是前几个区段>..

// 解压缩数据到 `004C2000` --> 压缩算法和前面的一样。。 (前面解压 `00401000` 处的数据)... (下面的函数和前面调用的一样)....

```assembly
00627F55    89C6         mov     esi, eax      ; esi = eax = 005BC0AD
00627F57    89D7         mov     edi, edx      ; edi = 004C2000

00627F59    0FB606       movzx   eax, byte ptr [esi]
00627F5C    89C2         mov     edx, eax
00627F5E    83E0 1F      and     eax, 1F
00627F61    C1EA 05      shr     edx, 5
00627F64    74 2D        je      short 00627F93
00627F66    4A           dec     edx
00627F67    74 15        je      short 00627F7E
00627F69    8D5C13 02    lea     ebx, dword ptr [ebx+edx+2]
00627F6D    46           inc     esi
00627F6E    C1E0 08      shl     eax, 8
00627F71    89FA         mov     edx, edi
00627F73    0FB60E       movzx   ecx, byte ptr [esi]
00627F76    46           inc     esi
00627F77    29CA         sub     edx, ecx
00627F79    4A           dec     edx
00627F7A    29C2         sub     edx, eax
00627F7C    EB 32        jmp     short 00627FB0
00627F7E    C1E3 05      shl     ebx, 5
00627F81    8D5C03 04    lea     ebx, dword ptr [ebx+eax+4]
00627F85    46           inc     esi
00627F86    89FA         mov     edx, edi
00627F88    0FB70E       movzx   ecx, word ptr [esi]
00627F8B    29CA         sub     edx, ecx
00627F8D    4A           dec     edx
00627F8E    83C6 02      add     esi, 2
00627F91    EB 1D        jmp     short 00627FB0
00627F93    C1E3 04      shl     ebx, 4
00627F96    46           inc     esi
00627F97    89C1         mov     ecx, eax
00627F99    83E1 0F      and     ecx, 0F
00627F9C    01CB         add     ebx, ecx
00627F9E    C1E8 05      shr     eax, 5
00627FA1    73 07        jnb     short 00627FAA
00627FA3    43           inc     ebx
00627FA4    89F2         mov     edx, esi
00627FA6    01DE         add     esi, ebx
00627FA8    EB 06        jmp     short 00627FB0
00627FAA    85DB         test    ebx, ebx
00627FAC    74 0E        je      short 00627FBC
00627FAE  ^ EB A9        jmp     short 00627F59
00627FB0    56           push    esi
00627FB1    89D6         mov     esi, edx
00627FB3    89D9         mov     ecx, ebx
00627FB5    F3:A4        rep     movs byte ptr es:[edi], byte ptr [esi]
00627FB7    31DB         xor     ebx, ebx
00627FB9    5E           pop     esi
00627FBA  ^ EB 9D        jmp     short 00627F59
00627FBC    89F0         mov     eax, esi
00627FBE    5B           pop     ebx
00627FBF    5F           pop     edi
00627FC0    5E           pop     esi
00627FC1    C3           retn
```

## 7. 返回与异常处理
返回到 `0062920A` (上面也有调用这个函数... 有个异常)

```assembly
0062920A    8B10         mov     edx, dword ptr [eax]    ; eax --> 前面解压缩数据接下来的地址  eax = 005D9272
0062920C    52           push    edx
0062920D    E8 CCF7FFFF  call    006289DE       ; 第二类 SEH 安装函数

// SEH 执行完返回到这里
006295E4    03C2         add     eax, edx      ; eax  = 005D9276
006295E6    5A           pop     edx
00628F7C    870424       xchg    dword ptr [esp], eax
006295A5    09D2         or      edx, edx
```

// `<<<< 这里下断点 --> 判断所有区段是否都已解压数据完毕 >>>>`
// `Exectryptor` 把所有的数据全部压缩到 最后一个区段中..... -----------------------> 外壳的代码也是在最后一个区段中 `/////`
// 而且每次解压数据都是 `eax` --> 存放压缩后数据的内存地址 `edx` --> 目的内存地址...
// `0062920A` --> 解压完数据的出口
// `00629758` --> 修复 `E8` `E9` 后的出口...

流程:
- `循环结构获取结构数据` `stCompressData` `--> 获取区段开始地址 ---> 判读是否为 0 --> 不为 0 调用` `00627F50` `解压数据函数 ---> 返回到` `0062920A`
- `获取区段的大小 ---> 判段是否为 0 --> 不为 0 调用` `00629758` `修复` `E8` `E9` `--> 返回到` `00629758` `--> 再循环这个过程...`

```assembly
006295A7    0F84 AB010000  je      00629758   // 这个是判断每个区段是否有数据需要解压缩的... << PEID 查看区段信息 >>>

// eax = 005D9276 跳到这里...
00628F8A    8B10         mov     edx, dword ptr [eax]    // edx = 004E7000
00628E18    09D2         or      edx, edx
00628E1A    0F84 22FDFFFF je      00628B42

00629017    9D           popfd
00629018    871424       xchg    dword ptr [esp], edx    // 返回地址: 006283C4
00628F20    9C           pushfd
00628645    9D           popfd
00628646                 push    00627F50                ; 这个是解压函数....
00628649    C3           retn

// 解压后返回到到 006283C4 --> 之前是 0062920A
006283C4    8B10         mov     edx, dword ptr [eax]    ; eax = 005D9A0D
006296DE    52           push    edx
006296DF                 mov     edx, BCE8D362            ; edx = BCE8D362
006296F7    F7C2 00000008 test    edx, 8000000
00629080    0F85 C2F7FFFF jnz     00628848
00628848    81F2 66D3E8BC xor     edx, BCE8D366
0062959F    03C2         add     eax, edx          ; edx = 4
006295A1    5A           pop     edx               ; edx = 0x3000 <<< 区段大小>>
006295A2    870424       xchg    dword ptr [esp], eax
006295A5    09D2         or      edx, edx
006295A7    0F84 AB010000 je      00629758

<<<< 注意: 005550E0 的计算方法: --> E8 E9 都满足条件... >>>

// 接下来调用 005550E0 --> 修复区段 004E7000 的 所有 jmp 和 call 的偏移量数值...
00628786    0F85 000C0000 jnz     0062938C
006290A6                 push    00629758                ; Morph instruction
00628E56                 push    005550E0                ; Morph instruction
00628E59    C3           retn

006291FE    0F84 3EF9FFFF je      00628B42
00629204    52           push    edx
00628936    51           push    ecx
00628937    E8 84010000  call    00628AC0
```

## 8. 处理外壳输入表
所有区段的数据解压和修复完毕后返回到这里... ==> 处理外壳自己的输入表..s 获取外壳自己需要的一些 API 函数地址

// 此时 `eax` = `00627F0D`
```text
00627F0D  00 00 00 00 00 00 00 00  ........
```

```assembly
006297A6    8B10         mov     edx, dword ptr [eax]    ; eax = 00627F0D
006297A8    56           push    esi
006297A9                 mov     esi, 00000004             ; esi = 00000004
006280FD    03C6         add     eax, esi
006280FF    5E           pop     esi
00628100    870424       xchg    dword ptr [esp], eax
00628103    09D2         or      edx, edx
00628105    0F84 4D160000 je      00629758
00628B3C                 pop     eax                           ; Morph instruction
00629517    0F83 67FAFFFF jnb     00628F84
00628F84    0F88 DBF2FFFF js      00628265
00628F8A    8B10         mov     edx, dword ptr [eax]
00628E18    09D2         or      edx, edx
00628E1A    0F84 22FDFFFF je      00628B42    ; = 0

00628B42    E8 4A0E0000  call    00629991       // 所有区段数据全部解压和修复完毕后返回到这
00629991    C3           retn

// 返回到 00628B47
00628B47                 mov     edx, 00627F40             ; edx = 00627F40
00629714    8B02         mov     eax, dword ptr [edx]
00628671    68 5E7A4FB5 push    B54F7A5E
00628676    5A           pop     edx
00628677    81C2 50E1FFBB add     edx, BBFFE150
006284EC    0F85 61100000 jnz     00629553
00629553    81EA 788AA43C sub     edx, 3CA48A78
00629589    81C2 9E43A3CB add     edx, CBA3439E
0062958F    8902         mov     dword ptr [edx], eax    ; EDX = 004E14D4 EAX = 0
0062949D    E8 21040000  call    006298C3
```

// 这个函数很好读。。哈哈
```c
006298C3    55           push    ebp
006298C4    8BEC         mov     ebp, esp
006298C6    83C4 F4      add     esp, -0C
006298C9    56           push    esi
006298CA    57           push    edi
006298CB    53           push    ebx

// esi = 004A7EEF
006298CC    BE EF7E4A00 mov     esi, 004A7EEF
006298D1    B8 00004000 mov     eax, 00400000        ; ASCII "MZP"
006298D6    8945 FC      mov     dword ptr [ebp-4], eax
006298D9    89C2         mov     edx, eax

//---------------------------------------------------------------
esi = 004A7EEF 数据结构 --> 壳的输入表结构 s

struct stImport
{
   DWORD  IAT;
   DWORD  DLLname;
   DWORD  INT;
};

004A7EFF  83 7F 0A 00 C7 7F 0A 00 00 00 00 00 FF FF FF FF  ?..?......
004A7F0F  A7 7F 0A 00 C3 7F 0A 00 00 00 00 00 00 00 00 00  ?..?..........
004A7F1F  00 00 00 00 00 00 00 00 00 00 00 00 6B 65 72 6E  ........kern
004A7F2F  65 6C 33 32 2E 64 6C 6C 00 00 00 00 00 00 47 65  el32.dll......Ge
004A7F3F  74 4D 6F 64 75 6C 65 48 61 6E 64 6C 65 41 00 00  tModuleHandleA..
004A7F4F  00 00 4C 6F 61 64 4C 69 62 72 61 72 79 41 00 00  ..LoadLibraryA..
004A7F5F  00 00 47 65 74 50 72 6F 63 41 64 64 72 65 73 73  ..GetProcAddress
004A7F6F  00 00 00 00 00 00 45 78 69 74 50 72 6F 63 65 73  ......ExitProces
004A7F7F  73 00 00 00 3B 7F 0A 00 4F 7F 0A 00 5F 7F 0A 00  s...;..O..\_..
004A7F8F  73 7F 0A 00 3B 7F 0A 00 4F 7F 0A 00 5F 7F 0A 00  s..;..O..\_..
004A7F9F  73 7F 0A 00 00 00 00 00 75 73 65 72 33 32 2E 64  s......user32.d
004A7FAF  6C 6C 00 00 00 00 4D 65 73 73 61 67 65 42 6F 78  ll....MessageBox

//------------------------------------------------------------------
// [esi + c] = 000A7F2B
006298DB    8B46 0C      mov     eax, dword ptr [esi+C]
006298DE    09C0         or      eax, eax
006298E0    0F84 99000000 je      0062997F
006298E6    01D0         add     eax, edx  ; EAX = 00400000  EDX = 000A7F2B

006298E8    89C3         mov     ebx, eax
006298EA    50           push    eax
006298EB    FF15 94505500 call    dword ptr [<&kernel32.GetModu>; kernel32.GetModuleHandleA

006298F1    09C0         or      eax, eax
006298F3    0F85 0F000000 jnz     00629908

// LoadLibrayA
006298F9    53           push    ebx
006298FA    FF15 98505500 call    dword ptr [<&kernel32.LoadLib>; kernel32.LoadLibraryA

00629900    09C0         or      eax, eax
00629902    0F84 62000000 je      0062996A

00629908    8945 F8      mov     dword ptr [ebp-8], eax
0062990B    6A 00        push    0
0062990D    8F45 F4      pop     dword ptr [ebp-C]
00629910    8B06         mov     eax, dword ptr [esi]
00629912    09C0         or      eax, eax
00629914    8B55 FC      mov     edx, dword ptr [ebp-4]
00629917    0F85 03000000 jnz     00629920
0062991D    8B46 10      mov     eax, dword ptr [esi+10]
00629920    01D0         add     eax, edx
00629922    0345 F4      add     eax, dword ptr [ebp-C]
00629925    8B18         mov     ebx, dword ptr [eax]
00629927    8B7E 10      mov     edi, dword ptr [esi+10]
0062992A    01D7         add     edi, edx
0062992C    037D F4      add     edi, dword ptr [ebp-C]
0062992F    09DB         or      ebx, ebx
00629931    0F84 4F000000 je      00629986
00629937    F7C3 00000080 test    ebx, 80000000
0062993D    0F85 04000000 jnz     00629947
00629943    8D5C13 02    lea     ebx, dword ptr [ebx+edx+2]
00629947    81E3 FFFFFF0F and     ebx, 0FFFFFFF
0062994D    53           push    ebx
0062994E    FF75 F8      push    dword ptr [ebp-8]
00629951    FF15 9C505500 call    dword ptr [<&kernel32.GetProc>; kernel32.GetProcAddress
00629957    09C0         or      eax, eax
00629959    0F84 0B000000 je      0062996A

// edi = 004A7F83  ===> 存到这里。。
0062995F    8907         mov     dword ptr [edi], eax
00629961    8345 F4 04   add     dword ptr [ebp-C], 4
00629965  ^ E9 A6FFFFFF  jmp     00629910
0062996A    6A 10        push    10
0062996C    6A 00        push    0
0062996E    53           push    ebx
0062996F    6A 00        push    0
00629971    FF15 D4505500 call    dword ptr [<&user32.MessageBo>; user32.MessageBoxA
00629977    6A FF        push    -1
00629979    FF15 A0505500 call    dword ptr [<&kernel32.ExitPro>; kernel32.ExitProcess
0062997F    5B           pop     ebx
00629980    5F           pop     edi
00629981    5E           pop     esi
00629982    8BE5         mov     esp, ebp
00629984    5D           pop     ebp
00629985    C3           retn
}
```

## 9. 修复入口点与收尾
// 返回到 `006294A2` ---> 修改 `EP` 处的指令... `<<< 这下面的几个地址都是关键地址???? >>>`
// 去掉 `OD` 下的 `EP` 软件断点,让 `OD` 在 `EP` 处段不下来..

```assembly
006294A2    C605 59946200 0F  mov     byte ptr [<ModuleEntryPoint>], 0F

// 修改第一个 TLS 回调函数指令.. (去 INT3 断点)
006294A9    C605 EF8B6200 E8  mov     byte ptr [628BEF], 0E8

006294B0    C605 47806200 87  mov     byte ptr [628047], 87
006294B7    C605 4A806200 5E  mov     byte ptr [62804A], 5E
006294BE    C605 4B806200 0F  mov     byte ptr [62804B], 0F
006294C5    C3           retn

// 返回到这里
006293D0    C745 FC 01000000 mov     dword ptr [ebp-4], 1
006293D7    E8 85F5FFFF  call    00628961

006293D0    C745 FC 01000000 mov     dword ptr [ebp-4], 1
00628965                 mov     eax, B2DFBF52            ; eax = B2DFBF52
00629720    D1C0         rol     eax, 1
00628D5C    03C5         add     eax, ebp
00628CEC    8B00         mov     eax, dword ptr [eax]      ; eax= 0012F9C3  [eax] = 1
00628CF3                 pop     ecx                         ; Morph instruction
006297BE    5D           pop     ebp

// 这里返回到 系统调用 TLS 回调函数的代码处..
006297BF    C2 0C00      retn    0C

{
7C92118A  |.  8BE6       mov     esp, esi
7C92118C  |.  5B         pop     ebx
7C92118D  |.  5F         pop     edi
7C92118E  |.  5E         pop     esi
7C92118F  |.  5D         pop     ebp
7C921190  \\.  C2 1000   retn    10
}
```



## 第二个回调函数

代码:



# 逆向分析笔记：IAT 动态加载与 API 解析机制

## 1. 数据结构定义

本文分析基于以下结构体定义，用于描述 IAT（导入地址表）及相关的 DLL 加载信息。

### 表 1：IAT Thunk 结构
```c
struct stIatThunk
{
  DWORD  thunkOffset;    // 当前 IAT 块相对于前一个 IAT 块的偏移 (第一个该结构的该数值就是 IAT 起始地址)
  DWORD  apiCount;       // 该 IAT 块对应的导入函数个数
};
```

### 表 2：辅助索引结构
```c
struct stTable2
{
  BYTE   by1;      // 索引第三张表用?
  BYTE   by2;      // 
  WORD   wd;       // 辅助获取 API 函数地址用?
  DWORD  ApiNameHashVal;   
};
```

### 表 3：DLL 条目结构
```c
struct stDllEntry
{
  DWORD   dllModuleHandle;  // 模块基地址
  BYTE    encryptedDllName[n];  // 动态库名称字符串..加密后的.. 加载时会先解密..加载完动态库后又重新加密回去..
};
```

## 2. 核心代码段：IAT 数据修改

此处展示了用于修改 IAT 数据的关键汇编指令，通常在此处设置断点以拦截 IAT 更新过程。

```asm
0053210A    8902             mov     dword ptr [edx], eax        ; 修改 IAT 的数据.. 说明已经加载了 IAT 项目的数据

00541A2F    8378 04 00      cmp     dword ptr [eax+4], 0        ; 处理每一个 DLL 的数据
00541A33   ^ 0F87 0CA8FEFF  ja      0052C245         ; 如果 = 0 的话, 全部处理完毕?
00541A39   - E9 F4C5FAFF    jmp     004EE032
```

## 3. TLS 函数调用与 IAT 循环处理

第二个 TLS 函数调用入口为 `0052F6DD`。其主要目的是加载那些不需要动态 Import 的 API 函数。

**局部变量映射：**
*   `[ebp-4]` ==> `0051FF48` => `BYTE* pIatThunk` 指针..指向 `stIatThunk` 结构数值
*   `[ebp-8]` ==> `00542940`
*   `[ebp-C]` ==> `00542D14`
*   `[ebp-28]` ==> `curIatThunkFirstAddress` 这个是当前 IAT 块的第一个项的地址.. 后一个 Iat 块的地址就是这个数值 + 后一个 `stIatThunk.thunkOffset`..
*   `[ebp-2C]` ==> 当前 `stIatThunk` 结构的 `apiCount`
*   `[ebp-18]` ==> `curIatEntryAddress` 存储每个 IAT 项的地址，区别于 `[ebp-28]`,因为 `[ebp-28]` 只存储 IAT 块里面的第一个地址..
*   `[ebp-20]` ==> 需要动态加载的 API 的个数..这个变量在这里没用..

**汇编代码分析：**

```asm
0052F6DD    55                 push    ebp
0052F6DE    8BEC               mov     ebp, esp
0052F6E0    0F83 F7FCFEFF      jnb     0051F3DD

0051F3DD    83C4 D0            add     esp, -30

0051F3E0    B8 48FF5100        mov     eax, 0051FF48
00550620    8945 FC            mov     dword ptr [ebp-4], eax
00550623    B8 40295400        mov     eax, 00542940
00550628    8945 F8            mov     dword ptr [ebp-8], eax
0055062B    B8 142D5400        mov     eax, 00542D14
00550630    8945 F4            mov     dword ptr [ebp-C], eax

00550633    8D05 DDF65200      lea     eax, dword ptr [52F6DD]  ; 这里修改 0052F6DD 为 retn 指令..第二次调用 TLS 函数的时候就不会在执行一次
00550639    C600 C3            mov     byte ptr [eax], 0C3

0053BD05    33C0               xor     eax, eax
0053BD07    8945 D8            mov     dword ptr [ebp-28], eax

0053BD10                 mov     eax, -4
00513A52    03C5               add     eax, ebp
00513A5A    8B00               mov     eax, dword ptr [eax]      ; [ebp-4]
00513A5C    8378 04 00         cmp     dword ptr [eax+4], 0      ; 比较当前 stIatThunk.apiCount 是否大于 0  (即是否包含了导出函数)
0052C5F3    0F87 4CFCFFFF      ja      0052C245

; =============================================> 从这里往下循环处理每个 IAT Thunk

0052C245    8B45 FC            mov     eax, dword ptr [ebp-4]
0052C248    8B00               mov     eax, dword ptr [eax]      ; stIatThunk.thunkOffset (相对 curIatThunkAddress 的偏移量)
0052C24A    0145 D8            add     dword ptr [ebp-28], eax   ; curIatThunkFirstAddress

0052C24D    8B45 D8            mov     eax, dword ptr [ebp-28]
0052C250    8945 E8            mov     dword ptr [ebp-18], eax

0054B8A3                 mov     eax, -4
0051C8FA    03C5               add     eax, ebp
0051C902    8B00               mov     eax, dword ptr [eax]
005387AE    8B40 04            mov     eax, dword ptr [eax+4]    ; stIatThunk.apiCount
005387B1    85C0               test    eax, eax
005387B3    0F8E E4D4FBFF      jle     004F5C9D                  ; 判段 stIatThunk.apiCount 是否小于等于 0,如果是的话, pIatThunk += 8;
00527C5C    0F8F E51AFFFF      jg      00519747

00519747    53                 push    ebx
00519748                 mov     ebx, -2C                  ; ebx = -2C
0054B50D    03DD               add     ebx, ebp
0054B515    8903               mov     dword ptr [ebx], eax    ; 保存 当前 stIatThunk.apiCount
0054B517    5B                 pop     ebx

00525B7C    C745 E0 01000000   mov     dword ptr [ebp-20], 1

00541124                 mov     eax, -18                  ; eax = -18
0053AC3B    03C5               add     eax, ebp
0053AC43    8B00               mov     eax, dword ptr [eax]
0054B321    F700 0000FFFF      test    dword ptr [eax], FFFF0000   ; 测试该 IAT 块的每个存放 API 的地址是否是正常的地址,如果是表示需要动态加载..这里不处理
0054B327    0F85 DF6DFEFF      jnz     0053210C                  ; 如果不是的话，由这里加载对应的 API 函数地址

0053210C    8345 E8 04         add     dword ptr [ebp-18], 4   ; 下一个 IAT 项地址
00532110    FF45 E0            inc     dword ptr [ebp-20]
00546A33    FF4D D4            dec     dword ptr [ebp-2C]      ; 初始为.stIatThunk.apiCount ==> while (count < stIatThunk.apiCount )
00546A36    0F85 47F1FDFF      jnz     00525B83                  ; 等于 0.. 处理完当前的 IAT 块..
0052A0F8    0F84 9FBBFCFF      je      004F5C9D
004F5C9D    8345 FC 08         add     dword ptr [ebp-4], 8      ; 处理完该 IAT 块的所有 IAT 项后,处理下一个 stIatThunk 结构

0052C30E                 mov     eax, -4
00541A25    03C5               add     eax, ebp
00541A2D    8B00               mov     eax, dword ptr [eax]
00541A2F    8378 04 00         cmp     dword ptr [eax+4], 0      ; stIatThunk.apiCount 是否大于 0
00541A33    0F87 0CA8FEFF      ja      0052C245                  ; 拦截这个 ja 才能跟踪直到每个 IAT 块都处理完毕.. 前面那个 ja 不可以

00547994  ^\0F86 A248FEFF     jbe     0052C23C        // 所有 IAT 块都处理完毕后返回..退出函数
0052C23C    8BE5               mov     esp, ebp
0052C23E    5D                 pop     ebp
0052C23F    C3                 retn

0052C245    8B45 FC            mov     eax, dword ptr [ebp-4]    ; 否则的话..跳到前面的 0052C245 继续循环..
```

## 4. 非动态加载 IAT 项的处理

当 IAT 项不是动态加载时（即直接通过表索引获取），处理逻辑如下。

**局部变量映射：**
*   `[ebp-24]` ==> 存储指向第二张表内数据项的地址 (通过 IAT 的数值<<8,计算得到)
*   `[ebp-1C]` ==> 结构 `stDllEntry` 的地址.... (由 IAT 项的数值计算索引得到的)
*   `[ebp-14]` ==> 保存获取到的第三张表的数据
*   `[ebp-10]` ==> 存储 `stDllEntry.encryptedDllName` 地址

**数据示例：**
每个 IAT 的数值是一个编号数值，用于索引另外两张表：
```text
004E7238  00000000
004E723C  00000001
004E7240  00000002
...
```

**汇编代码分析：**

```asm
005408EB                 mov     eax, -18                  ; eax = -18
0051A8B0    03C5               add     eax, ebp
0051A8B8    8B00               mov     eax, dword ptr [eax]    ; curIatEntryAddress
0051A8BA    8B00               mov     eax, dword ptr [eax]    ; 取该 IAT 项存储的数值 IatValue..

0051A8BC    C1E0 03            shl     eax, 3              ; Tab2Index = IatValue << 3 (左移三位得到的数值索引第二张表)
0051A8BF    0345 F8            add     eax, dword ptr [ebp-8]  ; Tab2Addr = Tab2Index + Tab2StartAddr;

004A93EC    8945 DC            mov     dword ptr [ebp-24], eax
004A93EF                 mov     eax, -24                  ; eax = -24
004F8260    03C5               add     eax, ebp
004F8268    8B00               mov     eax, dword ptr [eax]

004F826A    0FB700             movzx   eax, word ptr [eax]   ; 取第二张表的 2 个字节( 这个结构估计是每个字节为一个单位的)
0053770C    25 FF7FFFFF        and     eax, FFFF7FFF           ; Tab3Index = [Tab2Index] & 0xFFFF7FFF
00546515    52                 push    edx
00546516                 mov     edx, -C                   ; edx = -C
004F81B7    03D5               add     edx, ebp
0054A072    8B12               mov     edx, dword ptr [edx]    ; 第三张表的地址 [epb-C]
0054A074    03C2               add     eax, edx              ; 第三张表的首地址 +

0054A077    56                 push    esi
0054A078                 mov     eax, B55071C              ; eax = B55071C
0053DB46    03F5               add     esi, ebp
0053DB48    50                 push    eax
0053DB49                 mov     eax, 6D52304A             ; eax = 6D52304A
0052CB2A    03F0               add     esi, eax
0052CB2C    58                 pop     eax
0052CB2D    8906               mov     dword ptr [esi], eax
004F129E                 pop     esi                       ; Morph instruction
004F129F    9C                 pushfd
004F12A0                 mov     eax, -1C                  ; eax = -1C
005517F5    03C5               add     eax, ebp
0051E6DF    8B00               mov     eax, dword ptr [eax]
0051E6E1    9D                 popfd

0051E6E2    8B00               mov     eax, dword ptr [eax]  ; 索引获取到第三表中的数据

0051E6E4    53                 push    ebx
0051E6E5                 mov     ebx, 29CFE5C3             ; ebx = 29CFE5C3
0051BFEC    03DD               add     ebx, ebp
0051BFEE    57                 push    edi
0051BFEF                 mov     edi, D6301A29             ; edi = D6301A29
0053A7C6    03DF               add     ebx, edi
0053A7C8    5F                 pop     edi
0053A7C9    8903               mov     dword ptr [ebx], eax    ; 保存获取到的第三张表的数据 [ebp-14] -->也是 DLL 模块的基地址
0053A7CB    5B                 pop     ebx

004A8082    837D EC 00         cmp     dword ptr [ebp-14], 0   ; 如果模块的基地址为 0 的话..则加载该 DLL
004A8086    0F85 FCF80700      jnz     00527988
00513070    0F84 98FB0300      je      00552C0E

00552C0E    8B45 E4            mov     eax, dword ptr [ebp-1C]
00552C11    83C0 04            add     eax, 4
00552C14    8945 F0            mov     dword ptr [ebp-10], eax  ; 保存 stDllEntry.encryptDllName 地址
00552C17                 mov     eax, -10                  ; eax = -10
0054E0C4    03C5               add     eax, ebp
005524D2    8B00               mov     eax, dword ptr [eax]

0051AAA6                 push    eax                       ; Morph instruction
0051AAB1    873C24             push    eax           ; stDllEntry.encryptDllName 字符串地址作为参数..
```

**调用加解密字符串函数：**

```asm
//////////////////////////////////////////////////////////////////
//
//  下面调用一个加解密字符串的函数.. 函数过程放到后面分析 ==> 名称: enDeCodeString() ==> 0051866C
//  eax: 待加密或者待解密的字符串的内存地址
//
/////////////////////////////////////////////////////////////////

0053599B                 push    004A94AF                ; 返回地址
0051866C                            ; enDeCodeString();  // 调用解密出 Dll 库文件名
```

**结果示例：**
`00542D14` 处数据解密前为 `00 00 00 00 6F 6C 65 61 75 74 33 32 2E 64 6C 6C` (即 `...oleaut32.dll`)。

**后续处理流程：**
1.  尝试调用 `GetModuleHandleA` (`004A94B5`)。
2.  如果返回 0 (未加载)，则调用 `LoadLibraryA` 加载模块 (如 `oleaut32.dll`)。
3.  `LoadLibraryA` 地址获取依赖于先获取 `Kernel32` 模块基地址 (`004F7609`)。

**LoadLibraryA 调用过程分析：**

```asm
004A94B7    09C0               or      eax, eax
004A94B9    0F85 4ACB0400      jnz     004F6009            ; 这里返回 0.. 因为 oleaut32.dll 未被加载
...
005484BE    8B0424             mov     eax, dword ptr [esp]
005484C4                 push    eax                       ; eax ==> "oleaut32.dll"字符串内存地址

; 堆栈状态:
; 0012F968   00542D18  ASCII "oleaut32.dll"
; 0012F96C   00542D18  ASCII "oleaut32.dll"
```

调用 `004F6004` 处的代码，间接调用 `0054E248` 函数。

**函数 `0054E248` 分析 (LoadLibraryA 获取与调用):**

```asm
005475AC    55                 push    ebp
005475AD    8BEC               mov     ebp, esp
005475AF    51                 push    ecx
005475B0    833D C0F35100 00   cmp     dword ptr [51F3C0], 0   ; 判断 LoadLibrayA 函数是否已经获取..如果是的话就直接调用
...
; 如果未获取，则调用 getKernel32Module() (004F7609)
; 然后调用 myGetProcAddress() (00516067) 获取 DebugBreak, ExitProcess 等
; 最终获取 LoadLibraryA 的地址并存储到 [51F3C0]
...
; 关键步骤：调用 checkFunctionBreakpoint() (004F0C2B) 检测 API 函数是否被下断点
```

**checkFunctionBreakpoint 检查后的调用：**

```asm
; 检查完成后，执行 xchg/call 流程将 LoadLibraryA 地址入栈并调用
00529B2F    5E                 pop     esi
005341A2    52                 push    edx
005341A3                 mov     edx, 51F3C0             ; edx = 51F3C0
004EF644    8B12               mov     edx, dword ptr [edx]
004EF646    871424             xchg    dword ptr [esp], edx      ; 入栈 API 函数 LoadLibraryA 地址
0053965A    C3                 retn                  ; retn 返回去执行
```

**加载完成后的处理：**

```asm
004F6009    870424             xchg    dword ptr [esp], eax      ; eax = 770F0000 oleaut32 模块基地址
004F600C    E8 5B260200        call    0051866C                  ; 重新加密字符串函数 enDecodeString() "oleaut32.dll"
...
; 将获取到的 DLL 模块基地址保存回 stDllEntry 结构
00527986    8910               mov     dword ptr [eax], edx      ; 保存
```

**获取 API 函数地址：**

```asm
00550610                 mov     eax, -24
...
00514C01    E8 12650000      call    0051B118                  ; 这个函数通过查 dll 导出表,比较 API 函数名称字符串 HASH 值来获取 API 函数地址
...
0053210A    8902               mov     dword ptr [edx], eax      ; 最关键的一句话.. 填充 IAT 的数据
```

**最终返回：**

```asm
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
; ==> 整个 IAT 的处理过程完毕后返回到这里...  从这个函数 0052F6DD 返回..
0052C23C    8BE5               mov     esp, ebp
0052C23E    5D                 pop     ebp
0052C23F    C3                 retn
; ==> 第二个 TLS 回调函数处理完毕.. 返回到系统 。。
```

## 5. 函数详细分析

### 5.1 函数 `0053965B`: `checkFunctionBreakpoint()`

**功能：** 检测整个函数的所有指令是否被下软件断点（`0xCC`）。
**参数：** `eax` ==> 需要被检测的 API 函数的地址。
**返回：** `0` 或 `1`。
**检测结束条件：**
a. 遇到会改变程序流程的指令如 `jmp`, `retn`, `jmp dword ptr [eax]`, `iretd`。
b. 计算出的本条指令长度为 0。

```asm
00533FEB                 push    ebp                       ; Morph instruction
00533FEE    8BEC               mov     ebp, esp
...
; 循环检测每条指令
; 调用 0053BCE1 获取指令首字节
; 调用 0052DED6 (isInstructionChangeEip) 判断是否改变 EIP
; 调用 005345AC 获取当前指令长度
```

### 5.2 函数 `005345AC`: 获取指令长度

**功能：** 获取当前指令的长度。

```asm
005345AC    55                 push    ebp
...
; 通过查表 [eax*4+516F7C] 获取指令长度
; 如果是双字节指令 (0x0F)，需要调用 005157D6 进一步定位下一条指令
```

**子函数 `005157D6` 分析：**

```asm
005157D6    55                 push    ebp
...
; 分析 ModRM 字节 (第二个字节)
; 通过移位和掩码操作确定操作数大小和下一指令偏移
```

### 5.3 函数 `0052DED6`: `isInstructionChangeEip()`

**功能：** 判断当前指令是不是 `iretd`, `jmp`, `retn`, `jmp dword ptr [reg]` 等指令。如果是返回 `1`，否则返回 `0`。
**参数：** 指令起始地址。
**返回值：** 该条指令的长度（注意：根据上下文注释，这里似乎混合了长度获取和指令类型判断，或者注释有误，但代码逻辑显示它主要检查指令类型以决定是否终止检测）。

```asm
0052DED6    55                 push    ebp
...
; 检查 opcode:
; 0xCF (iretd)
; 0xE9 (jmp long)
; 0xEB (jmp short)
; 0xC2 / 0xC3 (ret)
; 0xFF / 0x20-0x27 (jmp far / indirect jump)
```

### 5.4 函数 `004F7609`: `getKernel32Module()`

**功能：** 获取 `kernel32.dll` 模块基地址。
**逻辑：**
1. 检查 `51778C` 处是否已存储地址。
2. 如果未存储，调用 `enDecodeString()` (`0051866C`) 解密 "kernel32.dll" 字符串。
3. 调用 `GetModuleHandleA`。
4. 将结果存储到 `51778C`。
5. 调用 `enDecodeString()` 重新加密字符串。

```asm
004A8D82    55                 push    ebp
...
0052FD18    B8 F0235200        mov     eax, 005223F0         ; 005223F0 存储"kernel32.dll"加密后的字符串
0052FD1D    E8 4A89FEFF        call    0051866C              ; 解密字符串函数 enDecodeString()
...
0053B91E    FF10               call    dword ptr [eax]       ; 调用 GetModuleHandle()
0054824A    A3 8C775100        mov     dword ptr [51778C], eax ; 保存到 51778C
...
00548254    E8 1304FDFF        call    0051866C              ; 重新加密"kernel32.dll"字符串
```

### 5.5 函数 `0051866C`: `enDecodeString()`

**功能：** 加解密字符串。
**参数：** `eax` ==> 待加密或者待解密的字符串的内存地址。
**返回：** 直接修改内存中的数据。
**算法特点：** 位旋转加密，Key 基于前一个解密字节和固定常量。

```asm
0051866C    55                 push    ebp
...
; 循环直到遇到 0x00
; 核心解密/加密逻辑:
; 1. 获取当前字节
; 2. 计算 Key (基于前一个字节和低 3 位)
; 3. 执行循环左移 (Rotate Left) 8 - (key & 7) 位
; 4. 更新 Key (XOR 当前解密后的字节)
```

### 5.6 函数 `00516067`: `myGetProcAddress()`

**功能：** 通过 Hash 值查找 API 函数地址。
**逻辑：**
1. 访问动态库 PE 文件选项头中的导出表。
2. 循环每个导出函数。
3. 计算函数名的 Hash 值。
4. 比较传入的 Hash 值。
5. 如果匹配，获取 API 函数地址。

```asm
00516067    55                 push    ebp
00516068    8BEC               mov     ebp, esp
0051606A    83C4 E4            add     esp, -1C
0051606D    8955 F8            mov     dword ptr [ebp-8], edx   ; Hash 值
00516070    8945 FC            mov     dword ptr [ebp-4], eax   ; 模块句柄
...
; 后续代码省略，主要涉及 PE 结构遍历和 Hash 计算
```




### 【返回到 EP 处】

代码:

```
00629459    0F89 2EFDFFFF         jns     0062918D                    
00629548                      push    00628134                        ; Morph instruction
006297C2    C3                retn    

//  检测 EP 处是否有 INT3 断点...
                
00628305    8D05 59946200     lea     eax, dword ptr [<[ModuleEntryPoint>]]
006282F7                      push    006295BC                        ; Morph instruction
00629128                      push    ebp                           ; Morph instruction
0062912B    8BEC              mov     ebp, esp                
00629130                      push    ecx                           ; Morph instruction
00629133    8945 FC           mov     dword ptr [ebp-4], eax     
00628117    8B45 FC           mov     eax, dword ptr [ebp-4]    
       
0062811A    8A00              mov     al, byte ptr [eax]              ; EP 处的第一个字节.. 
0062811C    2C 99             sub     al, 99           
          
00629358    8B12              mov     edx, dword ptr [edx]            ; [edx] = 0x00629459
006287F7    F62A              imul    byte ptr [edx]           
006287F9    3C A4             cmp     al, 0A4                ; 等于就出错。。        
006287FB    0F85 09030000     jnz     00628B0A                    
006287FB    0F85 09030000     jnz     00628B0A      

          
00628B0A    59                pop     ecx                            
00628B10                      pop    ebp                          ; Morph instruction
00628B11    C3                retn                                

// 这里两次 PUSH 都是作为 返回地址
// 
// push 4C1460
// push 628A7B
//
//
00628516                      mov     eax, 4C1460                 ; eax = 4C1460 ==> 4C1460 就是 VM-OEP 处了..
00628DB2                      push    eax                         ; 先入栈.. 等下就返回到 VMOEP...

// push 628A7B

00628DB5    57                push    edi                        
00628DB6                      mov     edi, 628A7B                 ; edi = 628A7B
00628B63    873C24            xchg    dword ptr [esp], edi       
 
00628A84                      push    ebp                         ; Morph instruction
00628A87    8BEC              mov     ebp, esp                
00628A89    51                push    ecx     
                      
00628832    56                push    esi                         
006292F1                      mov     esi, -4                     ; esi = -4
00629309    03F5              add     esi, ebp                    [ebp-4]

// eax = 004C1460 
0062882B    8906              mov     dword ptr [esi], eax        ; mov dword ptr [ebp-4], eax
00628D49    5E                pop     esi    
         

00628D4A    8B45 FC           mov     eax, dword ptr [ebp-4]       
00628D4D    8A00              mov     al, byte ptr [eax]      ; [EAX] = [004C1460] =  0xE9     
00628D4F    2C 99             sub     al, 99                   
00628D51                      mov     edx, -4                     ; edx = -4
00628DE2    03D5              add     edx, ebp                 
0062931F    8B12              mov     edx, dword ptr [edx]         
00629321    F62A              imul    byte ptr [edx]           
00629323    3C A4             cmp     al, 0A4              ; 检测 VmOep 处是否有 CC 断点..         
00629325    0F85 56040000     jnz     00629781                  ; 这句下面有个 SEH..



00628441                      pop    ecx                          ; Morph instruction
00628442    5D                pop     ebp                        
00628F0F    C3                retn             
                
00628A7B    C3                retn                            ; 从这里就是跳到 Vm-Oep 了...

```
