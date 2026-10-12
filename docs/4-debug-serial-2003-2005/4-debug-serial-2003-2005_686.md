
# 菜鸟破 ACS Capture 2.11 的思路

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-797.htm`](https://www.pediy.com/kssd/pediy07/pediy7-797.htm)

## ACS Capture 2.11

说明：

`kyc` 已经有详细的破文：`http://bbs.pediy.com/showthread.php?s=&threadid=19504`

我只是加了些菜鸟（3 个月）破解思路，给需要的人参考。

详细的地方可以参考 `kyc` 的文章。

### 1. 试运行：

- Name：`cyto`
- Key：`87654321`

确认后无提示，停在注册画面。

未注册限制：14 天。

### 2. 查壳：

`PEID`: `Microsoft Visual C++ 6.0 [Overlay]`

### 3. 点击注册没反应，只好上硬件断点：

`OD` 载入程序，`F9` 运行，跳出注册框，输入注册码，搜索内存：

```
001432F8  38 37 36 35 34 33 32 31   87654321
```

点注册确认后搜索内存：

```
00C95D58  38 37 36 35 34 33 32 31   87654321
```

#### 3.1 好，再次加载运行，下 `hr 00C95D58`，点注册确认，断下，返回：

```
004102C6      call <jmp.&MFC42.#540_CString::CString>          取注册码
004102CB      push dword ptr ss:[ebp+C]
004102CE      xor ebx,ebx
004102D0      lea ecx,dword ptr ss:[ebp-18]
004102D3      mov dword ptr ss:[ebp-4],ebx
004102D6      call <jmp.&MFC42.#537_CString::CString>          取用户名
004102DB      push dword ptr ss:[ebp+8]            ; /s = "cyto"；返回在此
004102DE      mov byte ptr ss:[ebp-4],1            ; |
004102E2      call <jmp.&MSVCRT.strlen>            ; \\strlen
004102E7      push eax
004102E8      lea eax,dword ptr ss:[ebp-14]
004102EB      push dword ptr ss:[ebp+8]
004102EE      push eax
004102EF      call capture.00427833                         用户名计算得到字符串
004102F4      add esp,10
004102F7      push eax
004102F8      lea ecx,dword ptr ss:[ebp-10]
004102FB      mov byte ptr ss:[ebp-4],2
004102FF      call <jmp.&MFC42.#858_CString::operator=>
00410304      lea ecx,dword ptr ss:[ebp-14]
00410307      mov byte ptr ss:[ebp-4],1
0041030B      call <jmp.&MFC42.#800_CString::~CString>       取出上面计算的字符串
00410310      mov esi,dword ptr ss:[ebp-10]
00410313      lea eax,dword ptr ss:[ebp-130]
00410319      push 104                                     ; /BufSize = 104 (260.)
0041031E      push eax                                     ; |PathBuffer
0041031F      push ebx                                     ; |hModule => NULL
00410320      mov dword ptr ss:[ebp-28],401                 ; |
00410327      mov dword ptr ss:[ebp-24],8D4                 ; |
0041032E      mov dword ptr ss:[ebp-20],0A83                ; |
00410335      call dword ptr ds:[<&KERNEL32.GetModuleFileNameA>]  ; \\GetModuleFileNameA
```

**小结：**

以上根据用户名得到字符串：`369d1922e56a67d428fedd5c31ee6887`

初始化赋值：`0012E82C  01 04 00 00 D4 08 00 00 18 0A 00 00`

#### 3.2 对字符串(ASCII "369d1922e56a67d428fedd5c31ee6887")的计算：

```
00410392       /cmp ecx,3
00410395       |jge short capture.004103A2
00410397       |movzx edx,byte ptr ds:[ecx+esi]       顺取字符串
0041039B       |imul edx,dword ptr ds:[eax]            注意 ds:[eax]，总共 C 位
0041039E       |mov dword ptr ds:[eax],edx            计算值又存回原地址
004103A0       |jmp short capture.004103B7
004103A2       |mov eax,ecx
004103A4       |push 3
004103A6       |cdq
004103A7       |pop edi
004103A8       |idiv edi
004103AA       |lea eax,dword ptr ss:[ebp+edx*4-28]  其实也就是在 ds:[eax]处总共 C 位轮转
004103AE       |movzx edx,byte ptr ds:[ecx+esi]       顺取字符串
004103B2       |add dword ptr ds:[eax],edx           计算值又存回原地址
004103B4       |mov eax,dword ptr ss:[ebp+8]
004103B7       |inc ecx
004103B8       |add eax,4
004103BB       |cmp ecx,dword ptr ss:[ebp-2C]         循环指针＝20
004103BE       |mov dword ptr ss:[ebp+8],eax
004103C1       \\jl short capture.00410392
```

入循环前的参数：

字符串：(ASCII "369d1922e56a67d428fedd5c31ee6887")，

赋值：`0012E82C  01 04 00 00 D4 08 00 00 18 0A 00 00`；

然后每次顺取字符串－1 跟相应赋值的地址的值相乘，计算后又存回去，循环 3 次，一次用 8 个；

当循环的指针（总共 20 位，为中间值－1 的长度）超过 3 时，乘法变成加法，但方法一样，得到一个最终值存回去；

得到赋值计算值：`0012E82C  02 CF 00 00 F2 DE 01 00 57 42 02 00`。

#### 3.3 注册码计算及比较：

```
004103D7       push 0C
004103D9       push eax                       赋值计算值
004103DA       mov dword ptr ss:[ebp-1C],edi     18（24）
004103DD       call capture.00410467                 计算得到：As8AAPLeAQBXQgIA
004103E2       mov eax,dword ptr ss:[ebp-1C]
004103E5       add esp,18
004103E8       lea ecx,dword ptr ss:[ebp-14]
004103EB       push esi
004103EC       mov byte ptr ds:[eax+esi],bl
004103EF       call <jmp.&MFC42.#537_CString::CString>
004103F4       push dword ptr ss:[ebp+C]                    ; /s2＝"87654321"
004103F7       mov byte ptr ss:[ebp-4],3                    ; |
004103FB       push dword ptr ss:[ebp-14]                   ; |s1
004103FE       call dword ptr ds:[<&MSVCRT._mbsicmp>]                  ; \\_mbsicmp；注册码比较
```

### 4. 根据 3 所示的主线很清晰，对各个细节进行突破：

#### 4.1 根据用户名算出字符串：`004102EF  call capture.00427833`

`OD` 载入，`F9` 运行后出现注册的框，输入完毕，点击注册，然后 `F8` 一步步从 `004102EF` 往下跟，每碰到一个 `call` 就搜索内存一遍，搜索内容为用户名 `cyto` 的计算值：`36 9D 19 22 E5 6A 67 D4 28 FE DD 5C 31 EE 68 87`，皇天不负有心人，经过 `n` 层 `call` 后：

主层：`004102EF      call capture.00427833`

第一层：`00427864      call capture.00428255`

第二层：`004282A3      call capture.00428371`

在进 `004282A3` 的 `call` 后，在内存找到：`0012E6E0  36 9D 19 22 E5 6A 67 D4 28 FE DD 5C 31 EE 68 87`，好记下地址 `0012E6E0`，再次加载，跟踪 `0012E6E0`，呵呵，这下有戏了，在进主层 `call` 后的 `00427847` 看到好东西：

`0012E6E0  01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10`，

太熟悉了，这个就是 `md5` 的初始值，试了下，果然：`md5(cyto)=36 9D 19 22 E5 6A 67 D4 28 FE DD 5C 31 EE 68 87`。

这下对用户名的计算马上明朗：

```
00427833   /$  B8 E8755700   mov eax,capture.005775E8
00427838   |.  E8 E37D0900   call capture.004BF620
0042783D   |.  83EC 60       sub esp,60
00427840   |.  8365 F0 00    and dword ptr ss:[ebp-10],0
00427844   |.  8D4D 94       lea ecx,dword ptr ss:[ebp-6C]
00427847   |.  E8 81090000   call capture.004281CD                   ;  MD5 初始化，注意 0012E6E0
0042784C   |.  FF75 10       push dword ptr ss:[ebp+10]             ; /Arg2 = 00000004
0042784F   |.  8365 FC 00    and dword ptr ss:[ebp-4],0             ; |
00427853   |.  8D4D 94       lea ecx,dword ptr ss:[ebp-6C]          ; |
00427856   |.  FF75 0C       push dword ptr ss:[ebp+C]              ; |Arg1
00427859   |.  E8 130B0000   call capture.00428371                 ; \\capture.00428371
0042785E   |.  FF75 08       push dword ptr ss:[ebp+8]
00427861   |.  8D4D 94       lea ecx,dword ptr ss:[ebp-6C]
00427864   |.  E8 EC090000   call capture.00428255                 ;  对用户名进行 md5 计算
00427869   |.  8B4D F4       mov ecx,dword ptr ss:[ebp-C]
0042786C   |.  8B45 08       mov eax,dword ptr ss:[ebp+8]
0042786F   |.  64:890D 00000>mov dword ptr fs:[0],ecx
00427876   |.  C9            leave
00427877   \\.  C3            retn
```

#### 4.2 对上面的字符串的计算：`004103DD      call capture.00410467`

前期循环次数的计算：

```
00410467   /$  55            push ebp
00410468   |.  8BEC          mov ebp,esp
0041046A   |.  83EC 10       sub esp,10
0041046D   |.  837D 08 00    cmp dword ptr ss:[ebp+8],0
00410471   |.  0F84 5E010000 je capture.004105D5
00410477   |.  837D 10 00    cmp dword ptr ss:[ebp+10],0
0041047B   |.  0F84 54010000 je capture.004105D5
00410481   |.  837D 14 00    cmp dword ptr ss:[ebp+14],0
00410485   |.  0F84 4A010000 je capture.004105D5
0041048B   |.  8B45 0C       mov eax,dword ptr ss:[ebp+C]     长度 C
0041048E   |.  53            push ebx
0041048F   |.  56            push esi
00410490   |.  57            push edi
00410491   |.  6A 03         push 3
00410493   |.  33DB         xor ebx,ebx
00410495   |.  99           cdq
00410496   |.  59           pop ecx                    3，上面 push 3 的结果
00410497   |.  215D FC      and dword ptr ss:[ebp-4],ebx
0041049A   |.  F7F9         idiv ecx                   商＝4
0041049C   |.  6A 4C        push 4C
0041049E   |.  895D F8      mov dword ptr ss:[ebp-8],ebx
004104A1   |.  5E           pop esi
004104A2   |.  6A 13        push 13
004104A4   |.  8BC8         mov ecx,ecx
```



### 004104A6 汇编代码解析

```
004104A6   |.  C1E1 02       shl ecx,2                        商×2×2＝10  
004104A9   |.  8BC1         mov eax,ecx  
004104AB   |.  99           cdq  
004104AC   |.  F7FE         idiv esi  
004104AE   |.  5E           pop esi  
004104AF   |.  85C0         test eax,eax  
004104B1   |.  8945 F0      mov dword ptr ss:[ebp-10],eax  
004104B4   |.  0F8C 9200000 jl capture.0041054C  
004104BA   |.  3945 FC      cmp dword ptr ss:[ebp-4],eax  
004104BD   |>  75 12        /jnz short capture.004104D1  
004104BF   |.  8BC1         |mov eax,ecx  
004104C1   |.  6A 4C        |push 4C  
004104C3   |.  99           |cdq  
004104C4   |.  5E           |pop esi  
004104C5   |.  F7FE         |idiv esi                         10/4C，余数＝10  
004104C7   |.  6A 04        |push 4  
004104C9   |.  5E           |pop esi  
004104CA   |.  8BC2         |mov eax,edx  
004104CC   |.  99           |cdq  
004104CD   |.  F7FE         |idiv esi                         10/4，商＝4  
004104CF   |.  8BF0         |mov esi,eax  
004104D1   |>  85F6         |test esi,esi  
004104D3   |.  7E 3D        |jle short capture.00410512  
```

字符串 `369d1922e56a67d428fedd5c31ee6887` 经过计算后值保存在 `0012E82C`，共 C 位（24 个字符）。

`C/3＝4`；`4×2×2＝10`；`10/4C`，余数＝`10`；`10/4＝4`。

最后值 `4` 作为下面循环的指针。

#### 核心计算

```
004104D5   |.  8975 F4      |mov dword ptr ss:[ebp-C],esi  4，作为下面循环的指针  
004104D8   |>  6A 03        |/push 3         
004104DA   |.  33D2         ||xor edx,edx  
004104DC   |.  58          ||pop eax        3  
004104DD   |>  8B7D 08     ||/mov edi,dword ptr ss:[ebp+8]   
004104E0   |.  0FB63F      |||movzx edi,byte ptr ds:[edi]  
004104E3   |.  0BD7        |||or edx,edi  
004104E5   |.  FF45 08     |||inc dword ptr ss:[ebp+8]  
004104E8   |.  C1E2 08     |||shl edx,8  
004104EB   |.  48          |||dec eax  
004104EC   |.^\\75 EF      ||\\jnz short capture.004104DD        ; 该循环取[0012E82C]的数据进行转换  
004104EE   |.  6A 04       ||push 4  
004104F0   |.  5F          ||pop edi  
004104F1   |> /8B5D 10     ||/mov ebx,dword ptr ss:[ebp+10]     ; 分配保存地址  
004104F4   |.  |8BC2        |||mov eax,edx                       ; 取[0012E82C]8 个数据，循环计算 4 次  
004104F6   |.  |C1E8 1A     |||shr eax,1A                         ; 取得的数据进行运算得到 eax  
004104F9   |.  |FF45 10     |||inc dword ptr ss:[ebp+10]          ; 指向下一个保存地址  
004104FC   |.  |8A80 A8655800 |||mov al,byte ptr ds:[eax+5865A8]   ; 根据 eax 从表中取出字符  
00410502   |.  |C1E2 06     |||shl edx,6  
00410505   |.  4F          |||dec edi                          ; 小循环 4 次  
00410506   |.  |8803        |||mov byte ptr ds:[ebx],al         ; 计算的值一个个保存于[ebx],00c993e8 开始  
00410508   |.^\\75 E7      ||\\jnz short capture.004104F1        ; 该循环对取出的一组数据进行计算  
0041050A   |.  FF4D F4     ||dec dword ptr ss:[ebp-C]           ; 中循环 4 次  
0041050D   |.^ 75 C9       |\\jnz short capture.004104D8  
```

### 小结

#### 运算参数

赋值计算值：`0012E82C 02 CF 00 00 F2 DE 01 00 57 42 02 00`

**内置表如下：**

```
  005865A8  41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50  ABCDEFGHIJKLMNOP
  005865B8  51 52 53 54 55 56 57 58 59 5A 61 62 63 64 65 66  QRSTUVWXYZabcdef
  005865C8  67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73 74 75 76  ghijklmnopqrstuv
  005865D8  77 78 79 7A 30 31 32 33 34 35 36 37 38 39        wxyz0123456789
```

#### 运算过程

先顺取赋值计算值的数据，每次 8 位，进行转换，如在数据窗口中的 `02 CF 00 00` 转换成寄存器的 `02CF0000`。

对转换后的数据进行计算，小循环共计算 4 次，每次都根据所得的值 `eax` 在表中取得相应的字符。

### 5. 算法总结

#### 5.1 用户名计算得到字符串

`md5("cyto")` = (ASCII `"369d1922e56a67d428fedd5c31ee6887"`)

#### 5.2 对字符串(ASCII `"369d1922e56a67d428fedd5c31ee6887"`)的计算

赋值：`0012E82C 01 04 00 00 D4 08 00 00 18 0A 00 00`

然后每次顺取字符串跟相应赋值的地址的值相乘，计算后又存回去，循环 3 次，一次用 8 个。

当循环的指针（总共 20 位，为中间值－1 的长度）超过 3 时，乘法变成加法，但方法一样，得到一个赋值计算值存回去：

`0012E82C 02 CF 00 00 F2 DE 01 00 57 42 02 00`

#### 5.3 对赋值计算值：`0012E82C 02 CF 00 00 F2 DE 01 00 57 42 02 00` 的计算

先顺取 `[0012E828]` 的数据，每次 8 位，进行转换，如在数据窗口中的 `20 00 00 00` 转换成寄存器的 `20000000`。

对转换后的数据进行计算，小循环共计算 4 次，每次都根据所得的值 `eax` 在表中取得相应的字符。

**内置表如下：**

```
  005865A8  41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50  ABCDEFGHIJKLMNOP
  005865B8  51 52 53 54 55 56 57 58 59 5A 61 62 63 64 65 66  QRSTUVWXYZabcdef
  005865C8  67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73 74 75 76  ghijklmnopqrstuv
  005865D8  77 78 79 7A 30 31 32 33 34 35 36 37 38 39        wxyz0123456789
```

#### 5.4 注册码比较

根据 5.3 的查表结果与输入的注册码比较，相等就成功。

### 6. 上 RegMon，输入正确的注册码

`HKCU\software\ACS Soft Solutions\ACS Capture\Registration\Name` SUCCESS `"cyto"`

`HKCU\software\ACS Soft Solutions\ACS Capture\Registration\Key` SUCCESS `"As8AAPLeAQBXQgIA"`

### 7. 思路总结

以上的破解思路是我经常用到的，不晓得大家的思路如何，因为看了很多文章，大部分对思路的描写几乎没有，只能根据破文的提示去猜测大侠们的思路。

注册没有提示或者不能用常规断点断下的，我就用硬件断点，不过有时候它们在内存的地址会变，需要比如启动时注册和进入程序后注册是不一样的地址，这样的话就只能每次重新加载才能断在需要的地方。

还有一个就是可以根据搜索内存（搜索内容为计算后的值，注册码等）来了解每个经过的 `call` 的作用。像 4.1 的用户名计算，耗费了好多精力，一遍遍的找，才发现原来是 `md5` 算法。

最后祝大家元旦快乐！！

`4dd466d0e1e0cc123afbe504da2462ca`
