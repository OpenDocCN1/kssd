
# 逆向分析与去壳指南

## 1. 初步代码分支分析

```
00B846C9     8B0495 F823B900     mov eax,dword ptr ds:[edx*4+B923F8]

00B846D0     EB 07               jmp short 00B846D9

00B846D2     52                  push edx

00B846D3     50                  push eax

00B846D4     E8 2710FFFF         call 00B75700 ; jmp to kernel32.GetProcAddress

00B846D9     5D                  pop ebp

00B846DA     C2 0800             retn 8
```

### 2. 为 VB 程序准备的分支

```
00B84FCE     68 FC4FB800         push 0B84FFC  ; ASCII "DllFunctionCall"

00B84FD3     56                  push esi

00B84FD4     A1 C026B900         mov eax,dword ptr ds:[B926C0]

00B84FD9     8B00                mov eax,dword ptr ds:[eax]

00B84FDB     FFD0                call eax

00B84FDD     8B15 9426B900       mov edx,dword ptr ds:[B92694]

00B84FE3     8902                mov dword ptr ds:[edx],eax

00B84FE5     B8 E046B800         mov eax,0B846E0

00B84FEA     8903                mov dword ptr ds:[ebx],eax

00B84FEC     5E                  pop esi

00B84FED     5B                  pop ebx

00B84FEE     C3                  retn
```

### 3. 0B84F98 结束与 Protection Error

```
00B84FEF     B8 984FB800         mov eax,0B84F98

00B84FF4     8903                mov dword ptr ds:[ebx],eax

00B84FF6     5E                  pop esi

00B84FF7     5B                  pop ebx

00B84FF8     C3                  retn

00B84F98     68 AC4FB800         push 0B84FAC  ; ASCII "87"

00B84F9D     E8 F2EEFFFF         call 00B83E94

00B84FA2     C3                  retn

00B83E94     55                  push ebp

00B83E95     8BEC                mov ebp,esp

00B83E97     6A 00               push 0

00B83E99     8B45 08             mov eax,dword ptr ss:[ebp+8]

00B83E9C     E8 07FBFEFF         call 00B739A8

00B83EA1     33C0                xor eax,eax

00B83EA3     55                  push ebp

00B83EA4     68 F03EB800         push 0B83EF0

00B83EA9     64:FF30             push dword ptr fs:[eax]

00B83EAC     64:8920             mov dword ptr fs:[eax],esp

00B83EAF     8D45 FC             lea eax,dword ptr ss:[ebp-4]

00B83EB2     8B4D 08             mov ecx,dword ptr ss:[ebp+8]

00B83EB5     BA 043FB800         mov edx,0B83F04  ; ASCII "Error: "

00B83EBA     E8 81F9FEFF         call 00B73840

00B83EBF     8B45 FC             mov eax,dword ptr ss:[ebp-4]

00B83EC2     E8 F1FAFEFF         call 00B739B8

00B83EC7     50                  push eax

00B83EC8     68 0C3FB800         push 0B83F0C      ; ASCII "Protection Error"

00B83ECD     E8 4E000000         call 00B83F20

//Game Over

00B83ED2     33C0                xor eax,eax

00B83ED4     5A                  pop edx

00B83ED5     59                  pop ecx

00B83ED6     59                  pop ecx

00B83ED7     64:8910             mov dword ptr fs:[eax],edx

00B83EDA     68 F73EB800         push 0B83EF7

00B83EDF     8D45 FC             lea eax,dword ptr ss:[ebp-4]

00B83EE2     E8 B1F6FEFF         call 00B73598

00B83EE7     8D45 08             lea eax,dword ptr ss:[ebp+8]

00B83EEA     E8 A9F6FEFF         call 00B73598

00B83EEF     C3                  retn
```

---

## 四、Pre-Dip：用自己的名字注册

取消以前的所有断点，现在设置 `Ollydbg` 忽略除了“内存访问异常”之外的所有其他异常选项。

使用 `Shift+F9` 运行，注意看堆栈，当第 2 次在堆栈中看见 `8wh3JAAQjOI=` 硬盘指纹时，可以处理了。

```
00B90B36     893B                mov dword ptr ds:[ebx],edi

//异常，第 2 次看见硬盘指纹

0012FF20   0012FF2C   指针到下一个 SEH 记录
0012FF24   00B90CEF   SE 句柄
0012FF28   40E2AEE6
0012FF2C   0012FF80   指针到下一个 SEH 记录
0012FF30   00B9114C   SE 句柄
0012FF34   0012FF78
0012FF38   00B70000
0012FF3C   00B40000   ASCII "MZ?
0012FF40   00B9051C   ASCII "U 嬱 SVW′&?
0012FF44   00000000
0012FF48   00BAE59C   ASCII "8wh3JAAQjOI="//硬盘指纹
0012FF4C   00B8F0F6   返回到 00B8F0F6 来自 00B81868
```

`Alt+M` 打开内存查看窗口，在 `00401000` 段下 内存访问断点，`Shift+F9` 通过异常，断下。

```
00535838     55                  push ebp

//中断在这里

00535839     8BEC                mov ebp,esp

0053583B     8B45 08             mov eax,dword ptr ss:[ebp+8]

0053583E     A3 0CCE5400         mov dword ptr ds:[54CE0C],eax

//保存注册名地址

00535843     5D                  pop ebp

00535844     C2 0400             retn 4
```

过了 `0053583E` 之后，`[54CE0C]` 处应该是保存的注册用户名，找一段空地，在 `0056AB00` 处写入：`fly [CUG]`，然后把 `[54CE0C]` 处的值改为 `0056AB00`。`OK`，这个 `Pre-Dip` 处理完毕。

---

## 五、以壳解壳——ASProtect V1.3 壳的 Stolen Code 简便解决方案

取消以前的所有断点。继续 `Shift+F9` 运行，直至中断在最后一次典型异常。

```
00B8F3F3     893B                mov dword ptr ds:[ebx],edi

//ASProtect V1.31 壳的最后一次异常

00B8F3F5     3085 5798C864       xor byte ptr ss:[ebp+64C89857],al

00B8F3FB     1B3F                sbb edi,dword ptr ds:[edi]

00B8F3FD     05 7B462835         add eax,3528467B

00B8F402     67:64:8F06 0000     pop dword ptr fs:[0]

00B8F408     26:EB 02            jmp short 00B8F40D
```

提供一个简便的方法快速达到使用“以壳解壳”的时机！

在最后一次异常时，`Ctrl+F` 在“当前位置下”搜索命令：

```
mov esp,dword ptr ss:[ebp-8]
```

`OK`，找到在 `00B8F65E` 处！下断，`Shift+F9` 运行，中断。

```
00B8F64A     E8 8B45FC8B         call 8CB53BDA

00B8F64F     0085 C0752B8B       add byte ptr ss:[ebp+8B2B75C0],al

00B8F655     0D 904BB900         or eax,0B94B90

00B8F65A     85C9                test ecx,ecx

00B8F65C     74 0B               je short 00B8F669

00B8F65E     8B65 F8             mov esp,dword ptr ss:[ebp-8]

//中断后取消所有断点

00B8F661     FF35 904BB900       push dword ptr ds:[B94B90]

00B8F667     C3                  retn

//返回 00DE0303

00DE0303     55                  push ebp

//这里开始处理 Stolen Code ★

//在这儿用 PETools 完全 DUMP 这个进程

00DE0304     EB 02               jmp short 00DE0308

00DE0308     8D6C5E 4A           lea ebp,dword ptr ds:[esi+ebx*2+4A]

00DE0310     8D6C05 B6           lea ebp,dword ptr ss:[ebp+eax-4A]

00DE0318     2BE8                sub ebp,eax

00DE031A     2BEB                sub ebp,ebx
```

---

## 六、Dump、修复输入表

如果仅仅脱壳出上面的壳代码部分是不行的，如下面的访问地址就丢失了：

```
02720167     FFD7                call edi  ;00B8BC98

00409C6C     68 10D6BA00         push 0BAD610

00409C71     C3                  retn

00DE1C99     E8 62E39301         call 02720000

00B8BC53     FF60 24             jmp dword ptr ds:[eax+24]; 02730000
```

所以区域脱壳出以下部分壳代码：

- `00B70000-00B9B000.dmp`
- `00BA8000-00BB0000.dmp`
- `00DE0000-00DE4000.dmp`
- `02720000-02721000.dmp`
- `02730000-02731000.dmp`

“组装”一下 `dumped.exe`，从磁盘载入刚才部分脱壳的五部分。

“组装”一下 `dumped.exe`，从磁盘载入刚才部分脱壳的 3 部分。

减去基址后分别修改其虚拟地址为 `00770000`、`007A8000`、`009E0000`、`02320000`、`02330000`。

只保留 `PETools` 的“验证 PE 文件”选项，重建 `PE`。

运行 `ImportREC`，填入 `RVA=0016A000`、`Size=00000850`。

获取输入表，会发现有 2 个无效指针：

```
0016A078  ?  0000  00B846B8
0016A1E0  ?  0000  00B846B8
```

通过 `AL=3` 的分析，我们知道应该把这 2 个函数修复为：`GetProcAddress`。

修改 `OEP=009E0303`，`FixDump`！

---

## 七、暗桩修复

运行后会发现有异常，其实还是壳造成的问题。

### 1. 004021FA 处异常

```
00402184     55                  push ebp

00402185     8BEC                mov ebp,esp

00402187     83C4 F8             add esp,-8

0040218A     53                  push ebx

0040218B     56                  push esi

0040218C     57                  push edi

0040218D     8BD8                mov ebx,eax

0040218F     803D 30E45400 00    cmp byte ptr ds:[54E430],0

//把 54E430 处的 01 修改为 00，让其重新申请内存

00402196     75 13               jnz short dumped_.004021AB

00402198     E8 13F9FFFF         call dumped_.00401AB0

0040219D     84C0                test al,al

0040219F     75 0A               jnz short dumped_.004021AB

004021A1     33C0                xor eax,eax

004021A3     8945 FC             mov dword ptr ss:[ebp-4],eax

004021A6     E9 54010000         jmp dumped_.004022FF

004021AB     33C9                xor ecx,ecx

004021AD     55                  push ebp

004021AE     68 F8224000         push dumped_.004022F8
```
