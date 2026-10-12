
# MoleBox 脱壳的一些总结

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-499.htm`](https://www.pediy.com/kssd/pediy09/pediy09-499.htm)

> 【文章标题】: `MoleBox` 脱壳的一些总结  
> 【文章作者】: `CCDebuger`  
> 【软件名称】: `Ultra Video Converter`  
> 【下载地址】: `http://www.zxmedia.com/ultra_videoconverter.exe`  
> 【作者声明】: 无他，仅是自己调试时的一些记录，方便查阅而已  

> ---

> 【详细过程】

> 论坛上写 `MoleBox` 脱壳教程的已经有不少了，大家应该知道 `MoleBox` 会加密输入表。我看了几个 `MoleBox` 加的壳和 `MoleBox` 的主程序，发现这个壳还是挺有意思的。只要你找好关键点，可以 `dump` 出来一个完整的未加壳前的程序，`MoleBox` 的那个加密输入表的功能根本没用。这篇文章只是我自己的一个调试记录，只是为了方便自己查阅。大家可以参考一下。OK，现在进入正题。

### 一、主程序脱壳

在 `OD` 中忽略所有异常，载入程序：

```
00470B53 >  E8 00000000      CALL Ultra_Vi.00470B58                     ; OD 载入后停在这
00470B58    60               PUSHAD
00470B59    E8 4F000000      CALL Ultra_Vi.00470BAD                     ; F8 到这里的时候在命令行中输入 hr esp
```

完成上面的操作后，直接按 `F9`，会断在这里：

```
00470731    58               POP EAX                                    ; 断在这里
00470732    58               POP EAX
00470733    FFD0             CALL EAX                                   ; 这里 F7 进去就是 OEP
```

我们到 `00470733` 地址处时按 `F7` 进去，就到 `OEP` 了：

```
0042304E    55               PUSH EBP                                   ; OEP
0042304F    8BEC             MOV EBP,ESP
00423051    6A FF            PUSH -1
00423053    68 90A74200      PUSH Ultra_Vi.0042A790
00423058    68 B8314200      PUSH Ultra_Vi.004231B8                     ; JMP 到 msvcrt._except_handler3
0042305D    64:A1 00000000   MOV EAX,DWORD PTR FS:[0]
00423063    50               PUSH EAX
00423064    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
0042306B    83EC 68          SUB ESP,68
0042306E    53               PUSH EBX
0042306F    56               PUSH ESI
00423070    57               PUSH EDI
00423071    8965 E8          MOV DWORD PTR SS:[EBP-18],ESP
00423074    33DB             XOR EBX,EBX
00423076    895D FC          MOV DWORD PTR SS:[EBP-4],EBX
00423079    6A 02            PUSH 2
0042307B    FF15 38764200    CALL DWORD PTR DS:[427638]                  ; msvcrt.__set_app_type
```

看 `OEP` 开始的代码就知道这是个典型的 `VC` 程序。记住 `OEP` 的 `RVA` = `0042304E` - `00400000` = `2304E`。随便找个 `API` 调用，如 `0042307B` 处的这句。左键选择这句，按鼠标右键，选择弹出菜单 数据窗口中跟随->内存地址。现在在数据窗口中按 `CTRL+G` 转到 `427638`（这样做主要是为了便于 `OD` 记住这个地址），重新启动程序，在数据窗口中转到我们上次记住的地址，设内存写入断点。在数据窗口中按右键切换一下视图，选择长型->地址，这样便于观察。经过几次中断后，数据窗口中会看到这样的内容：

```
00427624  73DC067B  MFC42.#952_??_afxSOCK@@3UAfxSOck@@A@@YAXXZ
00427628  73DCC3C8  MFC42.#3716_?GetRuntimeClass@CSliderCtrl@@UBEPAUCRuntimeClass@@XZ
0042762C  73D8ACCE  MFC42.#541_??0CStringArray@@QAE@XZ
00427630  00000000
00427634  77C1EE2F  msvcrt._controlfp
00427638  0002E8E4
0042763C  0002E8D6
00427640  0002E8C6
00427644  0002E8B6
00427648  0002E8A2
0042764C  0002E896
00427650  0002F52C
00427654  0002E87C
00427658  0002E874
0042765C  0002E866
```

可见 `00427638` 下面的都没被填充，而上面已经填入了 `API` 函数的实际地址了。重来，在上面填充过地址的地方重设内存断点，直到输入表部分都显示为这样的：

```
00427638  0002E8E4
0042763C  0002E8D6
00427640  0002E8C6
00427644  0002E8B6
00427648  0002E8A2
```

现在就可以 `dump` 了。用 16 进制工具打开 `dump` 后的文件，推荐用 `010Editor`。先定位到输入表所在区段，查找 `ASCII` 字串 `.dll`，看看找到的地方，比如看到了 `KERNEL32.dll`。记下 `KERNEL32.dll` 的开始地址，逆序转换一下（可以用 `Hpmbcalc Hex Calculator` 来转换），再在输入表所在段中搜索 `KERNEL32.dll` 开始地址的逆序字串，一般只会找到一个地方。根据 `IAT` 的结构，名称在第四个字段，往前再数 12 个字节就是这个 `DLL` 调用信息开始的地方。再根据 `IAT` 最后以 20 个 0 结束，就可以确定输入表的开始地址和大小了。把我们找到的 `OEP` 和 `IAT` 信息用 `PE` 编辑工具填入 `dump` 后的文件就可以了。

> ---

上面说的是一种跟踪加壳程序处理输入表比较通用的方法。对于 `MoleBox`，为了得到完好的输入表，可以 `BP VirtualProtect` 设断点。这也分两种情况：

### 1、`MoleBoxPro_2.6.1.2387` 主程序脱壳例子

先到 `MoleBox` 壳的 `EP`，设断 `BP VirtualProtect`，中断两次后返回，来到下面的地方：

```
0043AB9D    FF15 AC774400   CALL DWORD PTR DS:[4477AC]                 ; kernel32.VirtualProtect
0043ABA3    8B15 84764400   MOV EDX,DWORD PTR DS:[447684]             ; [00447684]=00CD1F00，地址 00CD1F00 中就是 OEP
0043ABA9    8B45 E8         MOV EAX,DWORD PTR SS:[EBP-18]
0043ABAC    0342 08         ADD EAX,DWORD PTR DS:[EDX+8]              ; [00CD1F08]=00027B00，输入表的 RVA。这里直接 dump 文件
0043ABAF    8945 F4         MOV DWORD PTR SS:[EBP-C],EAX
0043ABB2    C705 14794400 0>MOV DWORD PTR DS:[447914],0
0043ABBC    6A 00           PUSH 0
0043ABBE    68 246C4400     PUSH mbox2w.00446C24                      ; ASCII "EXECUTABLE"
0043ABC3    8B0D 10794400   MOV ECX,DWORD PTR DS:[447910]             ; mbox2w.00400108
0043ABC9    51              PUSH ECX
0043ABCA    8B55 E8         MOV EDX,DWORD PTR SS:[EBP-18]
0043ABCD    52              PUSH EDX
0043ABCE    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
0043ABD1    50              PUSH EAX
0043ABD2    E8 E9FAFFFF     CALL mbox2w.0043A6C0
0043ABD7    83C4 14         ADD ESP,14
0043ABDA    E8 B30F0000     CALL mbox2w.0043BB92
0043ABDF    25 FF000000     AND EAX,0FF
0043ABE4    85C0            TEST EAX,EAX
0043ABE6    74 15           JE SHORT mbox2w.0043ABFD
0043ABE8    E8 B60F0000     CALL mbox2w.0043BBA3
0043ABED    25 FF000000     AND EAX,0FF
0043ABF2    85C0            TEST EAX,EAX
0043ABF4    74 02           JE SHORT mbox2w.0043ABF8
0043ABF6   ^ EB F0          JMP SHORT mbox2w.0043ABE8
0043ABF8    E8 950F0000     CALL mbox2w.0043BB92
0043ABFD    68 846C4400     PUSH mbox2w.00446C84                      ; ASCII "imm32.dll"
0043AC02    FF15 18774400   CALL DWORD PTR DS:[447718]                 ; kernel32.GetModuleHandleA
0043AC08    8945 E4         MOV DWORD PTR SS:[EBP-1C],EAX
0043AC0B    837D E4 00      CMP DWORD PTR SS:[EBP-1C],0
0043AC0F    74 0E           JE SHORT mbox2w.0043AC1F
0043AC11    68 846C4400     PUSH mbox2w.00446C84                      ; ASCII "imm32.dll"
0043AC16    8B4D E4         MOV ECX,DWORD PTR SS:[EBP-1C]
0043AC19    51              PUSH ECX
0043AC1A    E8 31000000     CALL mbox2w.0043AC50
0043AC1F    68 746C4400     PUSH mbox2w.00446C74                      ; ASCII "oleoaut32.dll"
0043AC24    FF15 18774400   CALL DWORD PTR DS:[447718]                 ; kernel32.GetModuleHandleA
0043AC2A    8945 F0         MOV DWORD PTR SS:[EBP-10],EAX
0043AC2D    837D F0 00      CMP DWORD PTR SS:[EBP-10],0
0043AC31    74 0E           JE SHORT mbox2w.0043AC41
0043AC33    68 646C4400     PUSH mbox2w.00446C64                      ; ASCII "oleaout32.dll"
```

### 2、极品时刻表的例子

如果未发现与上面代码类似的内容，可以结合堆栈来看。比如极品时刻表，`Delphi` 程序，里面是 `aspack` 的壳。`BP VirtualProtect` 第一次断下来的时候看堆栈：

```
0012FDB8   004F621C  /CALL 到 VirtualProtect 来自 JPSKB061.004F6216
0012FDBC   004E9F5C  |Address = JPSKB061.004E9F5C
0012FDC0   00000004  |Size = 4
0012FDC4   00000004  |NewProtect = PAGE_READWRITE
0012FDC8   0012FDCC  \\pOldProtect = 0012FDCC
0012FDCC   00000000
0012FDD0   003D20EC
0012FDD4   005021B8  JPSKB061.005021B8
0012FDD8   00000000
0012FDDC  /0012FE30
0012FDE0  |004F5AA6  返回到 JPSKB061.004F5AA6 来自 JPSKB061.004F61D0
0012FDE4  |004E9F5C  JPSKB061.004E9F5C
0012FDE8  |00501B44  ASCII "EXECUTABLE"
0012FDEC  |004E9F6C  ASCII "kernel32.dll"
0012FDF0  |00000001
0012FDF4  |00000000
0012FDF8  |00000000
0012FDFC  |0045AB2C  JPSKB061.0045AB2C
0012FE00  |00501B4E  JPSKB061.00501B4E
0012FE04  |00501B4E  JPSKB061.00501B4E
0012FE08  |C1CC0928
0012FE0C  |39ADD078
0012FE10  |004E9F79  JPSKB061.004E9F79
0012FE14  |004E9F6C  ASCII "kernel32.dll"
0012FE18  |004E9F5C  JPSKB061.004E9F5C
0012FE1C  |004E9F6C  ASCII "kernel32.dll"
0012FE20  |7C800000  kernel32.7C800000
0012FE24  |FA46F001
0012FE28  |004E9F5C  JPSKB061.004E9F5C
```
