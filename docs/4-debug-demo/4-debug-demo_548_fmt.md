
# unWC ver 1.92 Build 927？

> 原文：[`www.pediy.com/kssd/pediy06/pediy6637.htm`](https://www.pediy.com/kssd/pediy06/pediy6637.htm)

## 软件简介

“E 书伴侣”`unWC`是目前唯一一款专门用来反编译用软件 `webcompiler` 制作的 `EXE` 电子书源文件的工具软件（E 书反编译工具），可以迅速地解压缩包括在 `EXE` 电子书里面的全部源文件（包括网页、文本、图片、`CHM`、`ZIP`、`EXE` 等），并且完美地恢复源文件的全部目录结构及文件名，以便帮助用户得到源文件进行资料恢复或二次编辑。“E 书伴侣”(`unWC`)支持批量操作，支持光盘直接操作。

请大家注意“完美地恢复源文件”！

只有注册用户才可以真正体会“完美恢复”的含义：

- 因为未注册版本的“E 书伴侣”会随机删除一部分网页、图片的内容，所以可能导致页面排版凌乱
- 比如程序恰好删除了 `.css` 文件里面的内容

注册用户不会遇到这种情况。如果想完美恢复源文件，请注册。

“E 书伴侣”不可以与一些专门用于调试类、监视类的软件同时工作。

## 一点调试的体会

![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)

作者的主程序以简单的 `upx` 加壳，一般 `upx` 工具不能脱，采用加密代码，运行中解密，不调试难以静态分析！大量反复检测“调试器”“反汇编工具”“反编译工具”`OllyDbg`、`Trw2000`、`W32Dasm`、`SoftICE`、`WinHex`、`Filemon`、`Regmon` 等等，一经检测出“违禁品”就立刻“自行退出”或“Kill 正运行的禁品工具”或“死循环”或“恶意修改 `eip`”或“锁死程序功能”等等，反编译工具--特别是 `DeDe`，因为主程序是 `Delphi` 写的，在特定的情况下会善自重启机器！并反脱壳校验！单步调试检测，采用大量的异常进行保护（“隐姓埋名”的 `OD` 在 98 下极难调试），插入花指令阻碍分析等等。

关键的“检测 Call”和“标志位指令”比较统一规格，一经查出容易“满门抄斩”！

![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

以为没人玩，想不到。。。

在 `XP` 平台用改名的 `OD`，忽略所有的异常，跟踪了一下：

```
004ECF8F   >   E8 006F0000   CALL unWC.004F3E94 //里面有异常
004ECF94   .   EB 06         JMP SHORT unWC.004ECF9C
```

往里面好像检测 Jmp 类的 CC 断点，不大清楚：

```
004F329F   >   B8 106E4000   MOV EAX,<JMP.&kernel32.CompareFileTime>
004F32AF   >   B8 506F4000   MOV EAX,<JMP.&kernel32.GetLocalTime>
004F32C1   >   B8 B06F4000   MOV EAX,<JMP.&kernel32.GetSystemTime>
004F32D1   >   B8 C86F4000   MOV EAX,<JMP.&kernel32.GetTimeZoneInform>
004F35F1   >   B8 506F4000   MOV EAX,<JMP.&kernel32.GetLocalTime>
004F32F7   >   64:8B0D 20000>MOV ECX,DWORD PTR FS:\[20\] //将当前 unWC.exe 的线成 ID 放进 ECX 里，不知检测什么好像会修改 eip.
004F32FE   .   EB 04         JMP SHORT unWC.004F3304

004EFBA1   .   E8 0A9CF1FF   CALL unWC.004097B0 //   00A5FD4C  ASCII "unwckey.dat"
004EFBA6   .   84C0          TEST AL,AL
004EFBA8   .   74 05         JE SHORT unWC.004EFBAF
```

注册码长度要不少于：`FAh`=250 字节，

主程序原始大小：`81608h`，用 `SetFilePointer` 获得大小！

```
CALL unWC.004070C8                       ; JMP to kernel32.SetFilePointer

004EF936   .   E8 999AF1FF   CALL UNWC.004093D4
004EF93B   .   3BD8          CMP EBX,EAX
004EF93D   .   7E 0E         JLE SHORT UNWC.004EF94D               ;  启动首次自校验 SIZE 检测的关键跳转

004F71B6   .   E8 1922F1FF   CALL UNWC.004093D4
004F71BB   .   3BD8          CMP EBX,EAX                           ;  读取 Key.dat 前自校验检测的关键跳转
004F71BD   .   /7E 07        JLE SHORT unWC.004F71C6
004F71BF   .   |C605 98005000>MOV BYTE PTR DS:\[500098\],0    //可以先

004F723F   .   3BD8          CMP EBX,EAX
004F7241   .   7E 07         JLE SHORT unWC.004F724A
004F7243   .   C605 D4305000>MOV BYTE PTR DS:\[5030D4\],1    //又玩自校验 SIZE 了

004218E4    3BD8            CMP EBX,EAX       //ebx==eax==00081608
004218E6    74 17           JE SHORT unWC.004218FF

004F9191    E8 3E85F2FF     CALL unWC.004216D4    //又玩自校验了
004F9196    83FA 00         CMP EDX,0
004F9199    75 0B           JNZ SHORT unWC.004F91A6
004F919B    83F8 00         CMP EAX,0      //防把文件大小改成 0 或负数。
004F919E    0F86 58010000   JBE unWC.004F92FC
004F91A4    EB 06           JMP SHORT unWC.004F91AC
004F91A6    0F8E 50010000   JLE unWC.004F92FC
004F91AC    8B45 EC         MOV EAX,DWORD PTR SS:\[EBP\-14\]
004F91AF    E8 2085F2FF     CALL unWC.004216D4    //又玩自校验了
004F91B4    83E8 08         SUB EAX,8
004F91B7    83DA 00         SBB EDX,0     

ASCII "D0806CFC"====》这个是主程序末尾的字符串，估计是用来自校验有关：

004F42F8    E8 E903F1FF     CALL unWC.00404A88
004F42FD    0F95C3          SETNE BL      //应该为 FALSE！补丁：xor bl,bl +nop
004F4300    EB 02          JMP SHORT unWC.004F4304
004F4302    33DB           XOR EBX,EBX
004F4304    33C0           XOR EAX,EAX

004F7298   >   E8 8FC3FFFF   CALL unWC.004F362C      //内有 Seh，好像进行解码或移动代码
004F729D   .   EB 06         JMP SHORT unWC.004F72A5
004F729F      55            DB 55                     ;  CHAR 'U'
004F72A0      44            DB 44                     ;  CHAR 'D'
004F72A1      55            DB 55                     ;  CHAR 'U'
004F72A2      03            DB 03
004F72A3   .   A8 09         TEST AL,9
004F72A5   >   33C0          XOR EAX,EAX
004F72A7   .   E8 2C9DFFFF   CALL unWC.004F0FD8
004F72F4   .   E8 DB20F1FF   CALL unWC.004093D4
004F72F9   .   3BD8          CMP EBX,EAX      //又玩自校验了,eax==000FEB0A，不许超越这个值！
004F72FB   .   7E 0D         JLE SHORT unWC.004F730A

004EFBA1    E8 0A9CF1FF     CALL unWC.004097B0               ; 首次判断 unwckey.dat 是否存在！
004EFBA6    84C0            TEST AL,AL
004EFBA8    74 05          JE SHORT unWC.004EFBAF
```

//判断 `unwckey.dat` 是否存在采用 `FindFirstFileA` 检测：

```
00409763    E8 40D7FFFF     CALL unWC.00406EA8               ; JMP to kernel32.FindFirstFileA
00409768    83F8 FF         CMP EAX,-1                       ;  
0040976B    74 34          JE SHORT unWC.004097A1

004097BA    40              INC EAX                          ;  
004097BB    0F95C0         SETNE AL                          ; TRUE
004097BE    5B              POP EBX

00420924    E8 AB0D0000     CALL unWC.004216D4
00420929    8BF0            MOV ESI,EAX  //eax==171h
0042092B    8BC3            MOV EAX,EBX

004F7378   .   FF51 1C       CALL DWORD PTR DS:\[ECX+1C\]                 ; 使用 kernel32.ReadFile 读取 Key.dat 的内容
004F737B   .   EB 04         JMP SHORT unWC.004F7381
004F7384   .   E8 BBD5F0FF   CALL unWC.00404944                         ;  读取 Key.dat 的内容的长度
004F7389   .   85C0          TEST EAX,EAX
004F738B   .   0F84 02050000 JE unWC.004F7893
004F7391   .   EB 06         JMP SHORT unWC.004F7399
004F739C    E8 A3D5F0FF     CALL unWC.00404944
004F73A1    3D FA000000     CMP EAX,0FA        //比较长度：要求要大于等于 250 个字节!
004F73A6    0F8C E7040000   JL unWC.004F7893
004F73AC    8B55 E8         MOV EDX,DWORD PTR SS:\[EBP\-18\]
004F73AF    8B45 DC         MOV EAX,DWORD PTR SS:\[EBP\-24\]
004F73B2    E8 C9D8F0FF     CALL unWC.00404C80
004F73B7    85C0            TEST EAX,EAX
004F73B9    0F9F05 94005000 SETG BYTE PTR DS:\[500094\]
004F7602   .   3BD8          CMP EBX,EAX
004F7604   .   0F8F 89020000 JG unWC.004F7893      //Key 文件不符合，跳往失败处！

004F405F    66:837D F4 00   CMP WORD PTR SS:\[EBP\-C\],0
004F4064    74 0A          JE SHORT unWC.004F4070
004F4066    EB 18          JMP SHORT unWC.004F4080
004F4068    8D45 EC        LEA EAX,DWORD PTR SS:\[EBP\-14\]
004F406B    E8 1C06F1FF    CALL unWC.0040468C
004F4070    8B45 F8        MOV EAX,DWORD PTR SS:\[EBP\-8\]
004F4073    8B55 EC        MOV EDX,DWORD PTR SS:\[EBP\-14\]
004F4076    E8 0D0AF1FF    CALL unWC.00404A88
004F407B    0F95C3          SETNE BL      //补丁：xor bl,bl +nop
004F407E    EB 02          JMP SHORT unWC.004F4082

004F4694    8B45 F8        MOV EAX,DWORD PTR SS:\[EBP\-8\]
004F4697    8B55 EC        MOV EDX,DWORD PTR SS:\[EBP\-14\]
004F469A    E8 E903F1FF    CALL unWC.00404A88
004F469F    0F95C3          SETNE BL      //补丁：xor bl,bl +nop
004F46A2    EB 02          JMP SHORT unWC.004F46A6

004ECFD1    8B15 1C055000   MOV EDX,DWORD PTR DS:\[50051C\]         ; unWC.00500098
004ECFD7    8A12            MOV DL,BYTE PTR DS:\[EDX\]
004ECFD9    80F2 01         XOR DL,1                                 ; //pacth xor dl,dl + nop

004F4D87    E8 244AF1FF     CALL unWC.004097B0           ; //patch check key"unwckey.dat"
004F4D8C    84C0            TEST AL,AL
004F4D8E    75 07          JNZ SHORT unWC.004F4D97
004F4D90    C605 98005000 0>MOV BYTE PTR DS:\[500098\],0

004F5EB6    E8 F538F1FF     CALL unWC.004097B0           ; //同上！
004F5EBB    84C0            TEST AL,AL
004F5EBD    75 07          JNZ SHORT unWC.004F5EC6
004F5EBF    C605 98005000 0>MOV BYTE PTR DS:\[500098\],0

004F6556    E8 5532F1FF     CALL unWC.004097B0           ; //同上！
004F655B    84C0            TEST AL,AL
```



## 01. 汇编代码片段

```
004F655D    75 07           JNZ SHORT unWC.004F6566  

004F655F    C605 98005000 0>MOV BYTE PTR DS:[500098],0  

004F6867    E8 442FF1FF     CALL unWC.004097B0               ; //同上！  

004F686C    84C0            TEST AL,AL  

004F686E    75 07           JNZ SHORT unWC.004F6877  

004F6870    C605 98005000 0>MOV BYTE PTR DS:[500098],0  

004F6D70    E8 3B2AF1FF     CALL unWC.004097B0               ; //同上！  

004F6D75    84C0            TEST AL,AL  

004F6D77    75 07           JNZ SHORT unWC.004F6D80  

004F6D79    C605 98005000 0>MOV BYTE PTR DS:[500098],0  

004F6D80    EB 06           JMP SHORT unWC.004F6D88  

004F8319    E8 9214F1FF     CALL unWC.004097B0               ; //同上！  

004F831E    84C0            TEST AL,AL  

004F8320    75 07           JNZ SHORT unWC.004F8329  

004F8322    C605 98005000 0>MOV BYTE PTR DS:[500098],0  
```

## 02. 分析要点

1. `SetFilePointer` 进行自校验文件大小和某部分数据！

   ASCII `"D0806CFC"`====》这个是主程序末尾的字符串，估计是用来自校验位置，不确定！

2. `CreateFileA`、`FindWindow` 等检测“违禁品”！--改名换姓就 OK 了。

3. `FindFirstFileA` 查找 `Lang` 和 `Key` 文件，`ReadFile` 读取它们！

`500098`

`KeyFile` 应该是那长长的一大串数字，而且对 `KeyFile` 的检测应该有很多处地方，并将其嵌套入异常里面，没有心情跟跟踪。

脱壳修整很麻烦的，最好还是采用 `SMC` 让其补身一下，先下手为强，后下要遭殃----就可以仿真真正的注册版功能！

## 03. SMC 补丁说明

综合上述的地方，偶就来个 `SMC` 补丁（来个有杀错，无放过！-- 希望作者不要介意，据知已经升级到 `v1.95` 版本了，下不了）：

- 未知何解，直接在 `UPX` 壳的 `POPAD` 和 `JMP` 命令下面和 `00` 的地方补丁，会使初始错误，只好加个区段来 `SMC` 了，手工搞个区段贴在尾巴部分，填上代码就行了！

改得不是很标准（注意：并没有去除 Anti 工具的功能!）：

```
0055E55A    61               POPAD  

0055E55B  - E9 A82A0000     JMP unWC.00561008    //MGo to pacth your some code area!  

0055E560  ^ 78 E5           JS SHORT unWC.0055E547  

00561008    B8 32D2908B     MOV EAX,8B90D232  //此处开始自己补丁的代码！  

0056100D    A3 D9CF4E00     MOV DWORD PTR DS:[4ECFD9],EAX  

00561012    B8 32DB90EB     MOV EAX,EB90DB32  

00561017    A3 FD424F00     MOV DWORD PTR DS:[4F42FD],EAX  

0056101C    A3 7B404F00     MOV DWORD PTR DS:[4F407B],EAX  

00561021    A3 9F464F00     MOV DWORD PTR DS:[4F469F],EAX  

00561026    B0 01           MOV AL,1  

00561028    A2 964D4F00     MOV BYTE PTR DS:[4F4D96],AL  

0056102D    A2 C55E4F00     MOV BYTE PTR DS:[4F5EC5],AL  

00561032    A2 65654F00     MOV BYTE PTR DS:[4F6565],AL  

00561037    A2 76684F00     MOV BYTE PTR DS:[4F6876],AL  

0056103C    A2 7F6D4F00     MOV BYTE PTR DS:[4F6D7F],AL  

00561041    A2 28834F00     MOV BYTE PTR DS:[4F8328],AL  

00561046    C705 5CE55500 9>MOV DWORD PTR DS:[55E55C],FFF9DA98 //还原原处代码  

00561050    33C0            XOR EAX,EAX      //原来 EAX 就为 0  

00561052  - E9 04D5FFFF     JMP unWC.0055E55B //跳回原处  
```

简单测试了一个 `webcompiler` 制作的 `EXE` 电子书，没发现 `Unregister` 字样的 `HTM` 文件和空图片。

`4dd466d0e1e0cc123afbe504da2462ca`
