
# 读狗程序的逆向对比

> 原文：[`www.pediy.com/kssd/pediy06/pediy7002.htm`](https://www.pediy.com/kssd/pediy06/pediy7002.htm)

是以前写的一个程序,运行时带上狗的驱动库.这里没有带编译好的程序和库,仅供参考吧!

程序只是表明了一种简单的调用机制,以及对于 Delphi 的程序生成 EXE 之后用 OD 分析时具体的分析对比.很简单的一个内容.

\[最近还会公布一个注册机的源程序\]

```pascal
  WW:=1;
  
  AssignFile(F,'DogData.txt');      //指定生成的数据文件
  Getmem(Buffer,101);               //申请缓冲区
  Buffer^:='       ';               //实际应该给长 101 个空格
  i:=$1;                            //初始返回值
  j:=100;                           //读数据的长度
  While ((i<>0) and (j<>0)) do      //读狗，直到返回正确的值或长度为 0
   begin
     i:=DRead(j,0,buffer^);        //读狗
     dec(j);
     label1.caption:=' '+inttostr(j+1);  //读到的字节数-记数
   end;
  if i>0 then                       //未找到狗，出错误提示
   begin
     Application.MessageBox('没有找到狗!','错误',0);
     exit;
   end;
   Rewrite(F);                      //建立数据文件，未做错误检测
   Writeln(F,'狗的数据：');          //名头
   str1:=Ansistring(Buffer^);        //字符转换
   for i:=1 to j+1 do              //变成字符。
   begin
    ch_hi:=ord(str1[i]) and $F0 shr 4;   //取字节的高位 4 个二进制位
    ch_LO:=ord(str1[i]) and $0F;         //取字节的低位 4 个二进制位
    if ch_hi>9 then write(f,chr(ch_hi+55)) else write(f,chr(ch_hi+48));  //换成字符
    if ch_Lo>9 then write(f,chr(ch_Lo+55)) else write(f,chr(ch_Lo+48));  //换成字符
    if i mod 16=0 then writeln(f) else write(f,' ');   //写到文件里
   end;
   closefile(F);                   //关闭文件
   freemem(buffer);                //翻译内存空间
```

```asm
0044F01C  PUSH EBP
0044F01D  MOV EBP,ESP
0044F01F  ADD ESP,-1E0                   ;申请的变量空间
0044F025  PUSH EBX
0044F026  PUSH ESI
0044F027  PUSH EDI
0044F028  XOR ECX,ECX
0044F02A  MOV DWORD PTR SS:[EBP-1DC],ECX
0044F030  MOV DWORD PTR SS:[EBP-1E0],ECX
0044F036  MOV DWORD PTR SS:[EBP-8],ECX
0044F039  MOV EBX,EAX
0044F03B  XOR EAX,EAX
0044F03D  PUSH EBP
0044F03E  PUSH wq.0044F254
0044F043  PUSH DWORD PTR FS:[EAX]
0044F046  MOV DWORD PTR FS:[EAX],ESP
0044F049  MOV DWORD PTR DS:[452BD8],1                ;初始变量 WW:=1
0044F053  MOV EDX,wq.0044F26C                 ;ASCII "DogData.txt"
0044F058  LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F05E  CALL wq.00402C34                  ；Assignfile(F,'DogData.txt')
0044F063  MOV EAX,65                        ;GetMem(Buffer,101);
0044F068  CALL wq.004026FC                  ;执行函数 GetMEM
0044F06D  MOV DWORD PTR SS:[EBP-4],EAX      ；返回值
0044F070  MOV EAX,DWORD PTR SS:[EBP-4]      ；buffer^进行了初始化
0044F073  MOV EDX,wq.0044F280               ;  ASCII "          "
0044F078  CALL wq.0040436C                  ;格式串
0044F07D  MOV EDI,1                         ;i:=$1
0044F082  MOV ESI,64                        ;j:=100
0044F087  JMP SHORT wq.0044F0D3
0044F089  /MOV EAX,DWORD PTR SS:[EBP-4]     while (i<>0 and j<>0) do
0044F08C  |MOV EAX,DWORD PTR DS:[EAX]
0044F08E  |PUSH EAX
0044F08F  |PUSH 0
0044F091  |PUSH ESI
0044F092  |CALL <JMP.&win32.DRead>           ;读狗
0044F097  |MOV EDI,EAX                       ；i:=DRead(*,*,*)返回值
0044F099  |DEC ESI                           ；dec(j)
0044F09A  |LEA EAX,DWORD PTR DS:[ESI+1]
0044F09D  |XOR EDX,EDX
0044F09F  |PUSH EDX                          ;  /Arg2 => 00000000
0044F0A0  |PUSH EAX                          ;  |Arg1
0044F0A1  |LEA EAX,DWORD PTR SS:[EBP-1E0]    ;  |
0044F0A7  |CALL wq.0040815C                ;  \\Inttostr(j)
0044F0AC  |MOV ECX,DWORD PTR SS:[EBP-1E0]
0044F0B2  |LEA EAX,DWORD PTR SS:[EBP-1DC]
0044F0B8  |MOV EDX,wq.0044F2F0
0044F0BD  |CALL wq.00404624                ;串合并
0044F0C2  |MOV EDX,DWORD PTR SS:[EBP-1DC]
0044F0C8  |MOV EAX,DWORD PTR DS:[EBX+300]
0044F0CE  |CALL wq.0042E38C                ;
0044F0D3   TEST EDI,EDI                    ;i=0 不
0044F0D5  |JE SHORT wq.0044F0DB
0044F0D7  |TEST ESI,ESI                    ;j=0 不
0044F0D9  \\JNZ SHORT wq.0044F089
0044F0DB  TEST EDI,EDI                    ;if (i>0) then
0044F0DD  JBE SHORT wq.0044F0FC
0044F0DF  PUSH 0                          ;i>0 了,则进入此,要出错了
0044F0E1  MOV ECX,wq.0044F2F4
0044F0E6  MOV EDX,wq.0044F2FC
0044F0EB  MOV EAX,DWORD PTR DS:[450FD4]
0044F0F0  MOV EAX,DWORD PTR DS:[EAX]
0044F0F2  CALL wq.0044DACC                  ;Applicatin.Messagebox('','',0);
0044F0F7  JMP wq.0044F22E                   ;exit;
0044F0FC  LEA EAX,DWORD PTR SS:[EBP-1D8]      ;   Rewrite(F);                  //名头
0044F102  CALL wq.004029D0                  ;   Writeln(F,'狗的数据：');
0044F107  CALL wq.00402864                  ;   str1:=Ansistring(Buffer^);      //字符转换
0044F10C  MOV EDX,wq.0044F310
0044F111  LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F117  CALL wq.004049C8
0044F11C  CALL wq.004030A8                  ;字符
0044F121  CALL wq.00402864
0044F126  LEA EAX,DWORD PTR SS:[EBP-8]
0044F129  MOV EDX,DWORD PTR SS:[EBP-4]
0044F12C  MOV EDX,DWORD PTR DS:[EDX]
0044F12E  CALL wq.004043B0
0044F133  INC ESI
0044F134  TEST ESI,ESI
0044F136  JBE wq.0044F216
0044F13C  MOV EDI,1                         ;生成字符处了
0044F141  /MOV EAX,DWORD PTR SS:[EBP-8]
0044F144  |MOV AL,BYTE PTR DS:[EAX+EDI-1]
0044F148  |MOV EBX,EAX
0044F14A  |AND BL,0F0                       ;and $F0
0044F14D  |AND EBX,0FF
0044F153  |SHR EBX,4                        ;Shr 4
0044F156  |AND AL,0F                        ;and $0F
0044F158  |MOV BYTE PTR SS:[EBP-9],AL       ;变到变量里
0044F15B  |CMP BL,9                         if ch_hi>9
0044F15E  |JBE SHORT wq.0044F17E
0044F160  |XOR EDX,EDX
0044F162  |MOV DL,BL
0044F164  |ADD EDX,37                       ;ch_hi+55
0044F167  |LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F16D  |CALL wq.0040306C
0044F172  |CALL wq.00402CF0
0044F177  |CALL wq.00402864
0044F17C  |JMP SHORT wq.0044F19A
0044F17E  |XOR EDX,EDX
0044F180  |MOV DL,BL
0044F182  |ADD EDX,30                       ;ch_hi+48
0044F185  |LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F18B  |CALL wq.0040306C
0044F190  |CALL wq.00402CF0
0044F195  |CALL wq.00402864
0044F19A  |CMP BYTE PTR SS:[EBP-9],9         ;if ch_lo>9
0044F19E  |JBE SHORT wq.0044F1BF
0044F1A0  |XOR EDX,EDX
0044F1A2  |MOV DL,BYTE PTR SS:[EBP-9]
0044F1A5  |ADD EDX,37                       ;ch_lo+55
0044F1A8  |LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F1AE  |CALL wq.0040306C
0044F1B3  |CALL wq.00402CF0
0044F1B8  |CALL wq.00402864
0044F1BD  |JMP SHORT wq.0044F1DC
0044F1BF  |XOR EDX,EDX
0044F1C1  |MOV DL,BYTE PTR SS:[EBP-9]
0044F1C4  |ADD EDX,30                       ;ch_lo+48
0044F1C7  |LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F1CD  |CALL wq.0040306C
0044F1D2  |CALL wq.00402CF0
0044F1D7  |CALL wq.00402864
0044F1DC  |MOV EAX,EDI                      ;i mod 16
0044F1DE  |AND EAX,0F
0044F1E1  |TEST EAX,EAX                      ;if (i mod 16=0)
0044F1E3  |JNZ SHORT wq.0044F1F7
0044F1E5  |LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F1EB  |CALL wq.004030A8
0044F1F0  |CALL wq.00402864
0044F1F5  |JMP SHORT wq.0044F20E
0044F1F7  |MOV DL,20
0044F1F9  |LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F1FF  |CALL wq.0040306C
0044F204  |CALL wq.00402CF0
0044F209  |CALL wq.00402864
0044F20E  |INC EDI
0044F20F  |DEC ESI
0044F210  \\JNZ wq.0044F141
0044F216  LEA EAX,DWORD PTR SS:[EBP-1D8]
0044F21C  CALL wq.00402CFC
0044F221  CALL wq.00402864
0044F226  MOV EAX,DWORD PTR SS:[EBP-4]
0044F229  CALL wq.0040271C
0044F22E  XOR EAX,EAX
0044F230  POP EDX
0044F231  POP ECX
0044F232  POP ECX
```



```
0044F233  MOV DWORD PTR FS:[EAX],EDX  

0044F236  PUSH wq.0044F25B  

0044F23B  LEA EAX,DWORD PTR SS:[EBP-1E0]  

0044F241  MOV EDX,2  

0044F246  CALL wq.0040433C  

0044F24B  LEA EAX,DWORD PTR SS:[EBP-8]  

0044F24E  CALL wq.00404318  

0044F253  RETN  

0044F254  JMP wq.00403D1C  

0044F259  JMP SHORT wq.0044F23B  

0044F25B  POP EDI  

0044F25C  POP ESI  

0044F25D  POP EBX  

0044F25E  MOV ESP,EBP  

0044F260  POP EBP  

0044F261  RETN
```

`4dd466d0e1e0cc123afbe504da2462ca`
