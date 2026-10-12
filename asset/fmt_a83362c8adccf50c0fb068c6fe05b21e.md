
# Delphi DLL 入口点分析

```text
debug032:01071160 E8 B3 24 FC FF          call    _Halt0  
debug032:01071160                    EP_protector_dll endp  
```

---

Delphi DLL 中的 `DllProc` 类似于 VC++ 中的 `DllMain`。下面是 `DllProc` 的入口点，可以看见它往栈中存放了若干函数地址，执行的顺序与函数压栈的顺序相反。

---

```text
debug032:010701FC                    DllProc proc near  
debug032:010701FC 55                push    ebp  
debug032:010701FD 8B EC             mov     ebp, esp  
debug032:010701FF 53                push    ebx  
debug032:01070200 56                push    esi  
debug032:01070201 57                push    edi  
debug032:01070202 A1 E0 2B 07 01    mov     eax, ds:off_1072BE0  

…...略去几行  

debug032:01070230 68 60 ED 03 01    push    offset Fun7  
debug032:01070235 68 98 02 07 01    push    offset Fun6  
debug032:0107023A 68 BC F9 06 01    push    offset Fun5  
debug032:0107023F 68 F4 F9 06 01    push    offset Fun4  
debug032:01070244 68 04 F4 06 01    push    offset Fun3  
debug032:01070249 68 78 EF 06 01    push    offset Fun2  
debug032:0107024E 68 3C FF 06 01    push    offset Fun1  
debug032:01070253 C3               retn  
debug032:01070253                  DllProc endp  
```

---

为内存中某一特定地址处赋值以表示某一段代码的开始，好象生产线上的流水号，执行到某一步，打一个标记。（但也可能是 `Flags`、`Sign`、`Mask`、whatever…）

### Fun1() 开头

```text
debug032:0106FF5D A1 8C 2B 07 01    mov     eax, ds:off_1072B8C  
debug032:0106FF62 C6 00 C9          mov     byte ptr [eax], 0C9h  
```

### Fun2() 开头

```text
debug032:0106EF7D A1 8C 2B 07 01    mov     eax, ds:off_1072B8C  
debug032:0106EF82 C6 00 CD          mov     byte ptr [eax], 0CDh  
```

程序中有一张大表 `TTableX`，其中的每一个数据项 `Item` 又是一个表，存储运行所需的一种数据信息，如 `protector.dll` 自需的输入表、`protectee.exe` 需要的输入表、`protectee.exe` 中 `AIP` 表、…大致是以 `TItemX` 的结构存储，当然每一个具体的 `TItemX` 的内部结构又不相同。利用函数 `TObjX_1_Fun1_Search()` 来查找。

程序执行到某一阶段时，总有例程会解密这张表的某个表项，读取其内容。记住这一点，在适当地方下断点编写脚本读取内容，可以节省大量的人工。

```pascal
TTableX = record
  Hdr: HdrX
  Body: array[0..9999] of TItemX;
  End;
```

---

```pascal
THdrX = record
  sign: char;
  size: integer;
  unknown: integer;
  end;
```

---

```pascal
TItemX = record
  Hdr: THdrX;
  Bdy: array[0..9999] of char;
  end;
```

### 调用查找例程

```text
debug032:0106F66F A1 0C 2B 07 01    mov     eax, ds:ppObj_1_Fun1  
debug032:0106F674 8B 00             mov     eax, [eax]  
debug032:0106F676 B2 0F             mov     dl, 0Fh  
debug032:0106F678 E8 5B 65 FF FF    call    TObjX_1_Fun1_Search  
```

### Fun1() 工作任务

`Fun1()` 的工作任务：创建了若干全局对象，组建了 `protector.dll` 自需的输入表，…

所有后续调用的函数都是这其中某一对象的方法。

`; 全局对象列表如下，除了 TH\TH2\TH3 是程序中已经注明的外，其它都是我编的名字。其中的各个域只是跟踪时的笔记，读者无需详究。`

---

```text
TObjX_1_Fun1, 0x14, class(TObject)
+04 pMem in loader1.exe
+08 VirMem for pMem in loader1.exe
+1C Num
TObjX_1_1_Fun1, 0x28
TAsEmulator, 0x58, class(TObject)
+04  pTItemX
+08
+28  Checksum(Mem in protector.dll)
+2C
+30  pTObjX_2_1_Fun1, 0x24, class(TObject)
+10, +11, +12 init to 110
+34－+3C sign
+40  pLoadLibrary
+44  pLoadLibrary XOR First4 bytes in LoadLibrary
+48  pGetProcAddress
+4C  pGetProcAddress XOR First4 bytes in GetProcAddress
+50  ImageBase 400000
TObjX_3_Fun1, 0x30, class(TObject)
TObjX_4_Fun1, 0x98, class(TObject)
TObjX_5_Fun1, 0x1B8, class(TObject)
+08
+14  400000
+18  Number of AIP
+24  401000
+20  bDoSth
+28  Size
+54  TIATx
  010B18B  VirMem_Size     分配 20 个 200 字节的堆
TH, 0x01, class(TObject)
TH2, 0x70, class(TH)
TH3, 0x70, class(TH), 10E07D8
+04 sizeofseed
+08 - +10 from Seed in 5924CC
+40 big sizeofseed
VirMethod 00        Init
VirMethod 02        Checksum
TH4, 0x70, class(TH)
```

---

比如会利用 `TH3` 虚拟函数 `02` 号来校验代码段。

`Fun1` 在最后调用函数进入虚拟机中执行检测，其实很简单，就是调用 `IsDebuggerPresent()` 来确定调试器是否存在，如果存在，提示并退出 `MessageBox()`, `ExitProcess()`。

#### 提示信息

```text
Debugger detected  - please close it down and restart!(\r\n\r\n)Windows NT users: Please note that having the..WinIce/SoftIce service installed means that you are(\r\n)running a debugger!
```

### Fun1() 中调用 Wired() 检测调试器的存在

```text
debug032:0107019E E8 1D FC FF FF    call    Wired  
debug032:010701A3 E8 94 ED FF FF    call    nullsub_25  
debug032:010701A8 33 C0             xor     eax, eax  
debug032:010701AA 5A                pop     edx  
```

### Wired() 原型

```text
debug032:0106FF0B A1 F8 29 07 01    mov     eax, ds:dword_10729F8  
debug032:0106FF10 E8 D3 EC FF FF    call    DetectDebugger  
```

函数 `Fun2()` 完成对原程序的解压缩，函数 `Fun3()` 复原近调用 (`E8 X`) 和 `Jmp` 近跳转 (`E9 X`)。

这是解压缩（解密）原程序需要的节信息

```text
[起址，解压缩后大小，解压缩前大小]
```

```text
.data:00590A13 00 10 00 00 00 80 0F 00 +Section_Hdr_Target dd 1000h, 0F8000h, 5DC2Ah; 0
.data:00590A13 2A DC 05 00 00 90 0F 00 +dd 0F9000h, 0C600h, 81D7h; 3
.data:00590A13 00 C6 00 00 D7 81 00 00 +dd 10A000h, 2E00h, 2902h; 6
.data:00590A13 00 A0 10 00 00 2E 00 00 +dd 1202DCh, 42524h, 1F88Ah; 9
.data:00590A13 02 29 00 00 DC 02 12 00 +dd 163000h, 2CE00h, 1663Ah; 12
```

### Fun4()

`Fun4()` 检测程序是否被更改。

### Fun5()

在这个版本中，`Fun5()` 负责相当相当重要的工作，包括重建 `IAT` 和 `AIP` (Advanced Import Protection)，重建 `obfuscatee` 例程，…

在 `Fun1()` 中，在对象 `TAsEmulator` 的数据域中保存了 `GetProcAddress` 和 `LoadLibrary` 的首地址和前四个字节的 `XOR` 值，在 `Fun5()` 中检查器检查这几个值是否一样，将结果参与计算。

#### 流程

```text
HandleIAT()
GetIAT()
AIP()
```

#### 流程

```text
GetIAT()
GetpFirstThunk()
GetRandInt()      用随机数覆盖 pFirstThunk
GetStrLen()       获取字符串 DLL 名的长度，如 kernel32.dll 的长度是 12
GetProcAddr()     不是 kernel32.dll 中的那个函数
```

### Fun5() 伪代码

```text
debug032:0106F9BC                    Fun5 proc near  
debug032:0106F9BC 68 DC 2A B0 33     push    33B02ADCh  
debug032:0106F9C1 68 FC 54 00 00     push    54FCh      ; 特殊点 2 106B8B0  
debug032:0106F9C6 68 B4 63 03 00     push    363B4h     ; 特殊点 2 363B4+54FC = 3B8B0  
debug032:0106F9CB 68 98 03 00 00     push    398h       ; 特殊点 1
debug032:0106F9D0 68 20 F6 03 00     push    3F620h     ; 特殊点 1 3F620+398 = 3F9B8  
debug032:0106F9D5 68 00 50 05 00     push    55000h     
debug032:0106F9DA FF 35 D4 34 07 01  push    ds:HInstance   ; ImageBase_protector.dll
debug032:0106F9E0 E8 C7 C4 FC FF     call    sub_103BEAC  
debug032:0106F9E5 31 04 24           xor     [esp], eax  
debug032:0106F9E8 8B 05 D4 34 07 01  mov     eax, ds:HInstance  
debug032:0106F9EE 01 04 24           add     [esp], eax  
debug032:0106F9F1 C3                retn               ; 跳转到 HandleIAT()继续执行
debug032:0106F9F1                    Fun5 endp  
```

### HandleIAT() 中调用 GetIAT() 重建输入表

```text
debug032:0106F752 A1 F4 2A 07 01     mov     eax, ds:ppObj_2_Fun1  
debug032:0106F757 8B 00              mov     eax, [eax]  
debug032:0106F759 E8 02 5E FF FF     call    near ptr GetIAT()  
```

#### 在获取到 FirstThunk 值后马上用随机数覆盖

```text
debug032:010655A2 6A FF              push    0FFFFFFFFh  
debug032:010655A4 E8 87 17 FE FF     call    GetRandInt  
debug032:010655A9 40                 inc     eax  
debug032:010655AA 89 03              mov     [ebx], eax  
```

#### 检查器计算 XOR 值，将结果参与计算

```text
debug032:010655C1 8B 46 48           mov     eax, [esi+48h]  
```
