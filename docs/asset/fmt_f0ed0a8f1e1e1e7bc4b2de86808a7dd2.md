
# [原创]nsPack2.x 通用脱壳法

> 原文：`www.pediy.com/kssd/pediy07/pediy7-603.htm`

[原创]NsPack2.x 通用脱壳法  
Author: peaceclub[at]PEDIY Goldsun[at]SECURITY  

**脱壳过程描述**：

1. `OD` 加载  
2. `Ctrl+B` 寻找二进制：`61 9d e9`  
3. 修改 `619d` 为 `ebfe`  
4. `F9`  
5. 暂停程序，`Alt+Backspace` 恢复代码，3 次 `F8`，`Ctrl+A`  
6. `Dump+Imprec IAT Fix`  

**案例分析**：

附件中为一个数据包侦听工具，经过 `nspack2.6` 压缩。

下载例子：`packinter_nspack.rar`

我们按照上面的脱壳过程一步步做.  

1. `OD` 加载  

```
0040B4A8 > 9C                 PUSHFD  '停在这里
0040B4A9   60                 PUSHAD
0040B4AA   E8 00000000        CALL PackInte.0040B4AF
0040B4AF   5D                 POP EBP
0040B4B0   B8 07000000        MOV EAX,7
0040B4B5   2BE8               SUB EBP,EAX
0040B4B7   8DB5 E8FDFFFF      LEA ESI,DWORD PTR SS:[EBP-218]
0040B4BD   8A06               MOV AL,BYTE PTR DS:[ESI]
```

2. `Ctrl+B` 寻找二进制：`61 9d e9`  

```
0040B72F   61                 POPAD  '找到了这里
0040B730   9D                 POPFD
0040B731  -E9 A978FFFF        JMP PackInte.00402FDF
0040B736   8BB5 9CFDFFFF      MOV ESI,DWORD PTR SS:[EBP-264]
0040B73C   0BF6               OR ESI,ESI
```

3. 修改 `619d` 为 `ebfe`  

```
0040B72F  -EB FE             JMP SHORT PackInte.0040B72F
0040B731  -E9 A978FFFF       JMP PackInte.00402FDF
```

有的朋友有可能问为什么，`Nspack` 新版本本身对硬软断点有检测，所以断不住。采用此方法直接改代码，让程序自己断 :)  

4. `F9`  
   程序进入死循环  

5. 暂停程序，`Alt+Backspace` 恢复代码，3 次 `F8`，`Ctrl+A`  

```
00402FDF  /. 55             PUSH EBP    '道路的开始 OEP
00402FE0  |. 8BEC           MOV EBP,ESP
00402FE2  |. 6A FF          PUSH -1
00402FE4  |. 68 784A4000    PUSH PackInte.00404A78
00402FE9  |. 68 66314000    PUSH PackInte.00403166           ;   JMP to msvcrt._except_handler3;  SE handler installation
00402FEE  |. 64:A1 00000000 MOV EAX,DWORD PTR FS:[0]
00402FF4  |. 50             PUSH EAX
00402FF5  |. 64:8925 000000>MOV DWORD PTR FS:[0],ESP
00402FFC  |. 83EC 68        SUB ESP,68
00402FFF  |. 53             PUSH EBX
00403000  |. 56             PUSH ESI
00403001  |. 57             PUSH EDI
```

6. `Dump+Imprec IAT Fix`  
   这个就不说了。  

**关于 nsPack2.6 主程序脱壳**

`nspack2.6` 新版本主程序加了两个异常 `seh`（内存访问异常和除零异常），脱法也很容易：

`OD` 中仅保留 `Kernel32` 内存访问异常忽略，`od` 加载后，直接 `F9`，一次 `Shift+F9`，`Ctrl+A`，在从上面**脱壳过程描述**第二步开始即可。  

**任务提高**：

附件中原未压缩文件大小为：`36864` 字节，请尝试把您脱壳的文件整理缩小，看是否能自己实现完美脱壳，并把过程简易描述到此帖。 

`4dd466d0e1e0cc123afbe504da2462ca`
