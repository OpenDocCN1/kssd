
## 八、Patch 代码汇总

保存一下 Patch 代码，以后再做会省点事吧。

```assembly
008A2040      0F85 C780FDFF             jnz 0087A10D
008A2046      C705 5AA18700 0F85ADFF    mov dword ptr ds:[87A15A],FFAD850F
008A2050      66:C705 5EA18700 FFFF     mov word ptr ds:[87A15E],0FFFF
008A2059      C705 D8A18700 8C7E0200    mov dword ptr ds:[87A1D8],27E8C
008A2063      E9 F880FDFF               jmp 0087A160
008A2068      C705 D8A18700 3A000000    mov dword ptr ds:[87A1D8],3A
008A2072      C705 CFA28700 E9B67D02    mov dword ptr ds:[87A2CF],27DB6E9
008A207C      66:C705 D3A28700 00FF     mov word ptr ds:[87A2D3],0FF00
008A2085      E9 4D81FDFF               jmp 0087A1D7
008A208A      0F85 CC81FDFF             jnz 0087A25C
008A2090      C705 CFA28700 0F8587FF    mov dword ptr ds:[87A2CF],FF87850F
008A209A      66:C705 D3A28700 FFFF     mov word ptr ds:[87A2D3],0FFFF
008A20A3      C705 7FA38700 E9377D02    mov dword ptr ds:[87A37F],27D37E9
008A20AD      66:C705 83A38700 00FF     mov word ptr ds:[87A383],0FF00
008A20B6      E9 1A82FDFF               jmp 0087A2D5
008A20BB      0F85 7882FDFF             jnz 0087A339
008A20C1      C705 7FA38700 0F85B4FF    mov dword ptr ds:[87A37F],FFB4850F
008A20CB      66:C705 83A38700 FFFF     mov word ptr ds:[87A383],0FFFF
008A20D4      C705 27A48700 E9C07C02    mov dword ptr ds:[87A427],27CC0E9
008A20DE      66:C705 2BA48700 00FF     mov word ptr ds:[87A42B],0FF00
008A20E7      E9 9982FDFF               jmp 0087A385
008A20EC      0F85 CC82FDFF             jnz 0087A3BE
008A20F2      C705 27A48700 0F8591FF    mov dword ptr ds:[87A427],FF91850F
008A20FC      66:C705 2BA48700 FFFF     mov word ptr ds:[87A42B],0FFFF
008A2105      C705 97A58700 E9817B02    mov dword ptr ds:[87A597],27B81E9
008A210F      66:C705 9BA58700 0000     mov word ptr ds:[87A59B],0
008A2118      E9 1083FDFF               jmp 0087A42D
008A211D      C705 97A58700 8B85CC01    mov dword ptr ds:[87A597],1CC858B
```

### 1、第 1 个点 返回地址处理

#### 原始代码

```assembly
008A2164      C780 C2150300 687E>mov dword ptr ds:[eax+315C2],8A217E68
008A216E      66:C780 C6150300 0>mov word ptr ds:[eax+315C6],0C300
```

//Patch 第 8 个点 012B15C2

```assembly
008A2177      05 ED100300          add eax,310ED
008A217C      FFE0                 jmp eax
```

//返回 012B10ED 处继续执行

### 2、第 8 个点 CreateFileMappingA 处理

#### 原始代码

```assembly
012B15C1      61                   popad
012B15C2      75 08               jnz short 012B15CC
```

//第 8 个点  
//Patch ⑧、

```assembly
012B15C2      68 7E218A00         push 8A217E
012B15C7      C3                  ret
```

#### Patch 代码

```assembly
008A217E      A1 69A88700         mov eax,dword ptr ds:[87A869]
```

//取得壳代码基址

```assembly
008A2183      C780 C2150300 7508>mov dword ptr ds:[eax+315C2],1B80875
008A218D      66:C780 C6150300 0>mov word ptr ds:[eax+315C6],0
```

//还原 012B15C2 处修改的代码

```assembly
008A2196      C680 37860100 08   mov byte ptr ds:[eax+18637],8
```

//修改 01298636 处的 `push 2` 为 `push 8` ★ CreateFileMappingA 参数

```assembly
008A219D      C680 69860100 01   mov byte ptr ds:[eax+18669],1
```

//修改 01298668 处的 `push 4` 为 `push 1` ★ MapViewOfFileEx 参数

```assembly
008A21A4      C780 7A860100 68BE>mov dword ptr ds:[eax+1867A],8A21BE68
008A21AE      66:C780 7E860100 0>mov word ptr ds:[eax+1867E],0C300
```

//Patch 第 9 个点 0129867A

```assembly
008A21B7      05 C2150300        add eax,315C2
008A21BC      FFE0               jmp eax
```

//返回 012B15C2 继续执行

### 3、第 9 个点 自校验处理

#### 原始代码

```assembly
01298630      6A 00              push 0
01298632      6A 00              push 0
01298634      6A 00              push 0
01298636      6A 02              push 2
01298638      6A 00              push 0
0129863A      53                 push ebx
0129863B      A1 E4972A01        mov eax,dword ptr ds:[12A97E4]
01298640      8B40 1C            mov eax,dword ptr ds:[eax+1C]
01298643      FFD0               call eax ; kernel32.CreateFileMappingA
01298645      A3 14B42A01        mov dword ptr ds:[12AB414],eax
0129864A      53                 push ebx
0129864B      A1 E4972A01        mov eax,dword ptr ds:[12A97E4]
01298650      8B40 18            mov eax,dword ptr ds:[eax+18]
01298653      FFD0               call eax
01298655      833D 14B42A01 00   cmp dword ptr ds:[12AB414],0
0129865C      0F84 66040000      je 01298AC8
01298662      6A 00              push 0
01298664      6A 00              push 0
01298666      6A 00              push 0
01298668      6A 04              push 4
0129866A      A1 14B42A01        mov eax,dword ptr ds:[12AB414]
0129866F      50                 push eax
01298670      A1 E4972A01        mov eax,dword ptr ds:[12A97E4]
01298675      8B40 08            mov eax,dword ptr ds:[eax+8]
01298678      FFD0               call eax ; kernel32.MapViewOfFileEx
0129867A      8BD8               mov ebx,eax
```

//第 9 个点  
//Patch ⑨、

```assembly
0129867A      68 BE218A00        push 8A21BE
0129867F      C3                 ret
```

#### Patch 代码

```assembly
008A21BE      8B1D 69A88700      mov ebx,dword ptr ds:[87A869]
```

//取得壳代码基址

```assembly
008A21C4      C683 37860100 02   mov byte ptr ds:[ebx+18637],2
```

//还原 01298636 处修改的代码

```assembly
008A21CB      C683 69860100 04   mov byte ptr ds:[ebx+18669],4
```

//还原 01298668 处修改的代码

```assembly
008A21D2      C783 7A860100 8BD8>mov dword ptr ds:[ebx+1867A],E850D88B
008A21DC      66:C783 7E860100 4>mov word ptr ds:[ebx+1867E],14A
```

//还原 0129867A 处修改的代码

```assembly
008A21E5      C780 50010000 0020>mov dword ptr ds:[eax+150],4A2000
```

//EAX 值是映像文件的开始地址，还原映像文件的 `SizeOfImage`

```assembly
008A21EF      66:C780 90030000 0>mov word ptr ds:[eax+390],1000
```

//还原映像文件最后一个区段的 `VSize` 为 1000

```assembly
008A21F8      66:C780 98030000 0>mov word ptr ds:[eax+398],0
```

//还原映像文件最后一个区段的 `RSize` 为 0000

```assembly
008A2201      C780 5A851900 0F85>mov dword ptr ds:[eax+19855A],FFAD850F
008A220B      66:C780 5E851900 F>mov word ptr ds:[eax+19855E],0FFFF
```

//还原映像文件中第一个接口修改的代码

```assembly
008A2214      C783 B9CA0100 682F>mov dword ptr ds:[ebx+1CAB9],8A222F68
008A221E      66:C783 BDCA0100 0>mov word ptr ds:[ebx+1CABD],0C300
```

//Patch 第 10 个点 0129CAB9

```assembly
008A2227      81C3 7A860100      add ebx,1867A
008A222D      FFE3               jmp ebx
```

//返回 0129867A 继续执行

### 4、第 10 个点 注册名 Pre-Dip 处理

#### 原始代码

```assembly
0129CAB9      50                 push eax
```

//第 10 个点

```assembly
0129CABA      8B47 04            mov eax,dword ptr ds:[edi+4]
0129CABD      FFD0               call eax
```

//注册名 Pre-Dip 处理  
//Patch 10、

```assembly
0129CAB9      68 2F228A00        push 8A222F
0129CABE      C3                 ret
```

#### Patch 代码

```assembly
008A222F      A1 69A88700        mov eax,dword ptr ds:[87A869]
008A2234      C780 B9CA0100 508B>mov dword ptr ds:[eax+1CAB9],4478B50
008A223E      66:C780 BDCA0100 F>mov word ptr ds:[eax+1CABD],0D0FF
```

//还原 0129CAB9 处修改的代码

```assembly
008A2247      C680 84A50000 EB   mov byte ptr ds:[eax+A584],0EB
008A224E      C680 CAE10100 EB   mov byte ptr ds:[eax+1E1CA],0EB
008A2255      C780 67E20100 E989>mov dword ptr ds:[eax+1E267],89E9
008A225F      C605 B4204C00 01   mov byte ptr ds:[4C20B4],1
```

//修改程序中判断是否注册的标志位

```assembly
008A2266      C705 1D466800 60DC>mov dword ptr ds:[68461D],21DC60
```

//Patch 第 11 个点 0068461C

```assembly
008A2270      68 04208A00        push 8A2004 ; ASCII "fly [2005.12.01]"
```

//Push 注册名地址，预先在 8A2004 处写下注册名

```assembly
008A2275      A1 69A88700        mov eax,dword ptr ds:[87A869]
008A227A      05 BACA0100        add eax,1CABA
008A227F      FFE0               jmp eax
```

//返回 0129CABA 继续执行

### 5、第 11 个点 注册名显示

#### 原始代码

```assembly
00684616      8D45 E4            lea eax,dword ptr ss:[ebp-1C]
00684619      8B55 F8            mov edx,dword ptr ss:[ebp-8]
0068461C      E8 770DD8FF        call 00405398
```

//第 11 个点  
//Patch 11、

```assembly
0068461C      E8 60DC2100        call 008A2281
```

#### Patch 代码

```assembly
008A2281      BA 04208A00        mov edx,8A2004
```

//Patch 注册名地址，注意 8A2004-4 处应该是字符串长度

```assembly
008A2286      E9 0D31B6FF        jmp 00405398
```

//继续流程

