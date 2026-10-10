

# Sense3 保护的绘图辅助软件分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-267.htm`](https://www.pediy.com/kssd/pediy08/pediy8-267.htm)

【文章标题】: Sense3 保护的绘图辅助软件分析  
【文章作者】: Rufus Xu  
【作者邮箱】: my\_home\_page@163.com  
【软件名称】: 绘画输出中心  
【软件大小】: 2.06M  
【下载地址】: 无  
【加壳方式】: 无  
【保护方式】: 加密狗  
【编写语言】: VC++6.0  
【使用工具】: OLLDBG  
【操作平台】: Windows2000  
【软件介绍】: 服装绘图辅助软件（功能：为没有共享功能的绘图仪提供共享，这个软件会设置一个共享目录，并监测目录内的文件有没有发生变化，其他的客户机把绘好的图通过网络放入共享的文件夹，程序会自动检测并将图输出到绘图仪。）  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
      Sense3 的话题比较多，很多前辈都做了比较细致的教程,紫竹 大侠的《深思 3 乱弹》系列给了我很大的启发，特别是系列 6 中的对狗内代码的猜解部分，特别感谢 紫竹 大侠.  
      废话不多说，进入正题：  
      辅助工作就不多说了，用 IDA 分析，导出 MAP，以便在 OD 分析时使用。  
      程序无狗运行，工具栏上的输出按钮是灰色的，不能使用。用 OD 载入，对读狗的地方下断：bpx CreateFileA.  
      F9 运行，断下：  
    
  004279DE >|> \\6A 00         PUSH 0                                   ; /loc\_4279DE; Case 2 of switch 004279B7  
  004279E0  |.  6A 00         PUSH 0                                   ; |Attributes = 0  
  004279E2  |.  6A 03         PUSH 3                                   ; |Mode = OPEN\_EXISTING  
  004279E4  |.  6A 00         PUSH 0                                   ; |pSecurity = NULL  
  004279E6  |.  6A 01         PUSH 1                                   ; |ShareMode = FILE\_SHARE\_READ  
  004279E8  |.  68 00000080   PUSH 80000000                            ; |Access = GENERIC\_READ  
  004279ED  |.  68 30E34500   PUSH PlotSV.0045E330                     ; |\\\\.\\sense3dev  
  004279F2  |.  FF15 B0224500 CALL DWORD PTR DS:\[<&KERNEL32.CreateFile>; \\CreateFileA  
  004279F8  |.  83F8 FF       CMP EAX,-1  
  004279FB  |.  75 18         JNZ SHORT <PlotSV.loc\_427A15>  
  004279FD  |.  8B8424 980000>MOV EAX,DWORD PTR SS:\[ESP+98\]  
  00427A04  |.  66:C740 5A 08>MOV WORD PTR DS:\[EAX+5A\],8  
  00427A0A  |.  66:B8 0100    MOV AX,1  
  00427A0E  |.  81C4 94000000 ADD ESP,94  
  00427A14  |.  C3            RETN  
    
  运行到返回：  
00427C46 >|> \\66:81FF FF01  CMP DI,1FF                               ;  loc\_427C46  
00427C4B  |.^ 76 DF         JBE SHORT <PlotSV.loc\_427C2C>  
00427C4D >|>  66:85C0       TEST AX,AX                               ;  loc\_427C4D  
00427C50  |.  75 23         JNZ SHORT <PlotSV.loc\_427C75>  
00427C52  |.  56            PUSH ESI  
00427C53  |.  E8 28FDFFFF   CALL <PlotSV.sub\_427980>  
00427C58  |.  83C4 04       ADD ESP,4  //返回到这  
00427C5B  |.  66:85C0       TEST AX,AX  
  
上下看看，没有有用的信息，再运行到返回。（主要是找 SENSE3DATA 结构赋值的地方）  
00427924  |. /7E 3E         JLE SHORT <PlotSV.loc\_427964>  
00427926 >|> |B9 18000000   /MOV ECX,18                              ;  loc\_427926  
0042792B  |. |8D7424 18     |LEA ESI,DWORD PTR SS:\[ESP+18\]  
0042792F  |. |8BFB          |MOV EDI,EBX  
00427931  |. |6A 01         |PUSH 1  
00427933  |. |F3:A5         |REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR D>  
00427935  |. |53            |PUSH EBX  
00427936  |. |E8 75020000   |CALL <PlotSV.sub\_427BB0>  
0042793B  |. |66:8B13       |MOV DX,WORD PTR DS:\[EBX\]  //返回到这  
0042793E  |. |83C4 08       |ADD ESP,8  
00427941  |. |85ED          |TEST EBP,EBP  
00427943  |. |66:8915 34124>|MOV WORD PTR DS:\[461234\],DX  
0042794A  |. |75 18         |JNZ SHORT <PlotSV.loc\_427964>  
0042794C  |. |66:813B 8000  |CMP WORD PTR DS:\[EBX\],80  
00427951  |. |72 11         |JB SHORT <PlotSV.loc\_427964>  
00427953  |. |8B4424 10     |MOV EAX,DWORD PTR SS:\[ESP+10\]  
00427957  |. |8B4C24 14     |MOV ECX,DWORD PTR SS:\[ESP+14\]  
0042795B  |. |40            |INC EAX  
0042795C  |. |3BC1          |CMP EAX,ECX  
0042795E  |. |894424 10     |MOV DWORD PTR SS:\[ESP+10\],EAX  
00427962  |.^|7C C2         \\JL SHORT <PlotSV.loc\_427926>  
  
上下看看，没有有用的信息，再运行到返回。（主要是找 SENSE3DATA 结构赋值的地方）  
  
00427870 >/$  83EC 60       SUB ESP,60                               ;  sub\_427870  
00427873  |.  33C0          XOR EAX,EAX  
00427875  |.  66:C74424 02 >MOV WORD PTR SS:\[ESP+2\],0FFFF  
0042787C  |.  66:894424 04  MOV WORD PTR SS:\[ESP+4\],AX  
00427881  |.  66:894424 06  MOV WORD PTR SS:\[ESP+6\],AX  
00427886  |.  66:894424 08  MOV WORD PTR SS:\[ESP+8\],AX  
0042788B  |.  8D4424 00     LEA EAX,DWORD PTR SS:\[ESP\]  
0042788F  |.  50            PUSH EAX                                 ; /Arg1  
00427890  |.  E8 2B000000   CALL <PlotSV.sub\_4278C0>                 ; \\PlotSV.004278C0  
00427895  |.  66:8B4424 00  MOV AX,WORD PTR SS:\[ESP\]   //返回到这  
0042789A  |.  66:3D 8000    CMP AX,80  
0042789E  |.  74 0C         JE SHORT <PlotSV.loc\_4278AC>  
004278A0  |.  66:3D 8B00    CMP AX,8B  
004278A4  |.  74 06         JE SHORT <PlotSV.loc\_4278AC>  
004278A6  |.  33C0          XOR EAX,EAX  
004278A8  |.  83C4 60       ADD ESP,60  
004278AB  |.  C3            RETN  
004278AC >|>  B8 01000000   MOV EAX,1                                ;  loc\_4278AC  
004278B1  |.  83C4 60       ADD ESP,60  
004278B4  \\.  C3            RETN  
  
有点像，但是不是，因为读狗密码不可能都是 0。事实上拿 SDK 中提供的测试工具验证过不是 0。  
继续返回：  
004277CA  |. /75 07         JNZ SHORT <PlotSV.loc\_4277D3>  
004277CC  |. |800D 2C124600>OR BYTE PTR DS:\[46122C\],2  
004277D3 >|> \\E8 98000000   CALL <PlotSV.sub\_427870>                 ;  loc\_4277D3; Case 1 of switch 0042779A  
004277D8  |.  85C0          TEST EAX,EAX          //返回到这                   ;  //返回到这  
  
上下看看，没有有用的信息，再运行到返回。（主要是找 SENSE3DATA 结构赋值的地方）  
004277F0 >    A1 3C124600   MOV EAX,DWORD PTR DS:\[46123C\]            ;  sub\_4277F0  
004277F5      85C0          TEST EAX,EAX  
004277F7      75 0F         JNZ SHORT <PlotSV.loc\_427808>  
004277F9      E8 12FFFFFF   CALL <PlotSV.sub\_427710>                 ;  Sense3.RefreshDeviceList  
004277FE  |.  C705 3C124600>MOV DWORD PTR DS:\[46123C\],1 //返回到这    
  
上下看看，没有有用的信息，再运行到返回。（主要是找 SENSE3DATA 结构赋值的地方）  
0041D6C5  |.  894D FC       MOV DWORD PTR SS:\[EBP-4\],ECX  
0041D6C8  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
0041D6CB  |.  66:C740 34 46>MOV WORD PTR DS:\[EAX+34\],46   //PASS1  
0041D6D1  |.  8B4D FC       MOV ECX,DWORD PTR SS:\[EBP-4\]  
0041D6D4  |.  66:C741 36 04>MOV WORD PTR DS:\[ECX+36\],4    //PASS2  
0041D6DA  |.  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP-4\]  
0041D6DD  |.  66:C742 38 03>MOV WORD PTR DS:\[EDX+38\],3    //PASS3  
0041D6E3  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
0041D6E6  |.  66:C740 32 FF>MOV WORD PTR DS:\[EAX+32\],0FFFF //读狗功能代码  
0041D6EC  |.  8B4D FC       MOV ECX,DWORD PTR SS:\[EBP-4\]  
0041D6EF  |.  83C1 30       ADD ECX,30  
0041D6F2  |.  51            PUSH ECX  
0041D6F3  |.  E8 F8A00000   CALL <PlotSV.sub\_4277F0>                 ;  Sense3  
0041D6F8  |.  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP-4\]    //返回到这          ;  PlotSV.00460FD4  
  
哈哈，看到没有，找 SENSE3DATA 结构赋值的地方了  
004277F0 就是读狗的函数了，相当于 SDK 中的 SENSE3 函数。CALL 前面的压栈就是结构地址了，记下这个地址=00461000  
以后要用到。我们来到地址 004277F0 看看，  
本地调用来自 sub\_41D6C1+32, sub\_41D716+17, sub\_41D716+40, 0041D79E, sub\_41D81D+186, 0041DB0B, sub\_41DC52+22  
共 7 处，在这 7 处下断，整个运行一次程序，每次运行把结构中的数据拷贝出来，发现每处的功能是：  
0041D6F3   CALL PlotSV.004277F0                      OpenDog1  open  
0041D72D   CALL PlotSV.004277F0                      OpenDog2  open  
0041D756   CALL PlotSV.004277F0                      OpenDog3  read  
0041D79E   CALL PlotSV.004277F0                      OpenDog4  
0041D9A3   CALL PlotSV.004277F0                      OpenDog5  read  
0041DB0B   CALL PlotSV.004277F0                      OpenDog6  
0041DC74   CALL PlotSV.004277F0                      OpenDog7  close  
  
开狗的数据我就不写下来了，我们只看看读狗的数据：  
第 5 处读狗的数据为：  
00461004       00 00 0E 00 46 00 04 00  ...F..  
0046100C       03 00 04 00 1E 31 00 00  ..1..    ax0=311E,ax1=0,bx0=0,bx1=0  
00461014       00 00 00 00 04 00 3C 62  .....<b      ax0=623C,ax1=0,bx0=A935,bx1=0  
0046101C       00 00 35 A9 00 00 00 00  ..5?...  
  
00461004       00 00 0E 00 46 00 04 00  ...F..  
0046100C       03 00 04 00 42 18 00 00  ..B..    ax0=1842,ax1=0,bx0=0,bx1=0  
00461014       00 00 00 00 04 00 84 30  .....?      ax0=3084,ax1=0,bx0=648C,bx1=0  
0046101C       00 00 8C 64 00 00 00 00  ..宒....  
  
00461004       00 00 0E 00 46 00 04 00  ...F..  
0046100C       03 00 04 00 DC 1B 00 00  ..?..    ax0=1BDC,ax1=0,bx0=0,bx1=0  
00461014       00 00 00 00 04 00 B8 37  .....?      ax0=37B8,ax1=0,bx0=4539,bx1=0  
0046101C       00 00 39 45 00 00 00 00  ..9E....  
  
总共读 10 次，其他的数据就不写了，有这三组数据足够了。  
对照紫竹的教程分析发现，  
623C=311E \* 2 ,623C+311E=935a(bx0=A935)  
3084=1842 \* 2 ,3084+1842=c648(bx0=648C)  
37B8=1BDC \* 2 ,37B8+1BDC=9453(bx0=4539)  
规律出来了，呵呵，狗内的代码基本上也就出来了。  
第 3 次读狗的数据为：  
00461004       00 00 0A 00 46 00 04 00  ....F..  
0046100C       03 00 04 00 25 18 00 00  ..%..  ax0=1825,ax1=0,bx0=0000,bx1=0  
00461014       00 00 00 00 04 00 0A D7  ....?    ax0=D70A,ax1=0,bx0=0000,bx1=0  
0046101C       00 00 00 00 00 00 00 00  .......  
  
00461004       00 00 0A 00 46 00 04 00  ....F..  
0046100C       03 00 04 00 25 18 00 00  ..%..    ax0=1825,ax1=0,bx0=0001,bx1=0  
00461014       01 00 00 00 04 00 A3 3C  ....?    ax0=3CA3,ax1=0,bx0=0001,bx1=0  
0046101C       00 00 01 00 00 00 00 00  .......  
  
经过多次测试发现，第 3 次读狗的地方，当 bx0=0000 时，ax0=D70A，当 bx0=0001 时，ax0=3CA3 是固定的数据  
  
数据有了，狗内的代码也都分析出来了，剩下的就是补丁了，还没有试过给 PE 增加区段，所以这次就想练练手。  
用工具给 PE 增加一个区段，这部分论坛上有详细的教程，工具操作也比较简单，就不多说了。  
我们将 004277F0 处的第条句语句改成跳转语句，跳到我们才增加的区段开头。  
0046C000    C605 04104600 0>MOV BYTE PTR DS:\[461004\],0 //returnflag=0,表示有狗  
0046C007    66:833D 0610460>CMP WORD PTR DS:\[461006\],0E//第 5 处读狗的功能代码  
0046C00F    74 0F           JE SHORT PlotSV\_n.0046C020//跳到第 5 处读狗模拟  
0046C011    66:833D 0610460>CMP WORD PTR DS:\[461006\],0A//第 3 处的读狗功能代码  
0046C019    74 36           JE SHORT PlotSV\_n.0046C051//跳到第 3 处读狗模拟  
0046C01B  - E9 47B8FBFF     JMP PlotSV\_n.00427867  //其他的开狗由这里反回  
0046C020    50              PUSH EAX  
0046C021    53              PUSH EBX  
0046C022    52              PUSH EDX  
0046C023    66:A1 10104600  MOV AX,WORD PTR DS:\[461010\]  
0046C029    66:8BD0         MOV DX,AX  
0046C02C    66:D1E0         SHL AX,1  
0046C02F    66:8915 1010460>MOV WORD PTR DS:\[461010\],DX  
0046C036    66:03C2         ADD AX,DX  
0046C039    66:BB 0010      MOV BX,1000  
0046C03D    66:F7E3         MUL BX  
0046C040    66:03C2         ADD AX,DX  
0046C043    66:A3 14104600  MOV WORD PTR DS:\[461014\],AX  
0046C049    5A              POP EDX  
0046C04A    5B              POP EBX  
0046C04B    58              POP EAX  
0046C04C  - E9 16B8FBFF     JMP PlotSV\_n.00427867  //第 5 处读狗功功能模拟返回  
0046C051    66:833D 1410460>CMP WORD PTR DS:\[461014\],1  
0046C059    74 0E           JE SHORT PlotSV\_n.0046C069  
0046C05B    66:C705 1A10460>MOV WORD PTR DS:\[46101A\],0D70A  
0046C064  - E9 FEB7FBFF     JMP PlotSV\_n.00427867  
0046C069    66:C705 1A10460>MOV WORD PTR DS:\[46101A\],3CA3  
0046C072  - E9 F0B7FBFF     JMP PlotSV\_n.00427867 //第 3 处读狗功功能模拟返回  
  
改完了，保存，运行，呵呵，可以正常出图了，完美模拟。  
  
\--------------------------------------------------------------------------------  
【经验总结】  
  1.熟悉了 Sense3 的 SDK 使用.  
  2.复习了 PE 文件结构.  
  3.学会了给 PE 文件增加区段.  
  4.学会了在 PE 文件中增加功能代码实现读狗的模拟.  
    
\--------------------------------------------------------------------------------  
【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 08 月 11 日 10:59:09  
  

 4dd466d0e1e0cc123afbe504da2462ca