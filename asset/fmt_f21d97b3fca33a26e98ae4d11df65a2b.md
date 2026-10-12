
004093E0    90                    NOP

**第二种情况：**

**解码/加密：**

```
00406348   /75 49                JNZ SHORT 00406393

0040634A  |FF15 3E554100   CALL DWORD PTR DS:[41553E]               ; 这里进去解码

00406350  |026B E8            ADD CH,BYTE PTR DS:[EBX-18]

00406353 |77 CD               JA SHORT 00406322

00406369 |99                  CDQ

……

00406382 |FF15 4E554100   CALL DWORD PTR DS:[41554E]               ; 这里进去把解码后的代码加密回去
```

**解码代码：**

```
……
00D00003    9C                 PUSHFD
00D00004    90                 NOP
00D00005    90                 NOP
00D00006    90                 NOP
00D00007    60                 PUSHAD
00D00008    8B4424 24          MOV EAX,DWORD PTR SS:[ESP+24]
00D0000C    8B08               MOV ECX,DWORD PTR DS:[EAX]
00D0000E    8D78 04            LEA EDI,DWORD PTR DS:[EAX+4]
00D00011    897C24 24          MOV DWORD PTR SS:[ESP+24],EDI
00D00015    81E9 D46AE877      SUB ECX,77E86AD4
00D0001B    FC                 CLD

……

00D00028    8A07               MOV AL,BYTE PTR DS:[EDI]
00D0002A    C0C8 42            ROR AL,42

00D0002D    90                 NOP
00D0002E    90                 NOP
00D0002F    90                 NOP
00D00030    90                 NOP
00D00031    04 D0              ADD AL,0D0
00D00033    02C1               ADD AL,CL
00D00035    FEC8               DEC AL
00D00037    04 09              ADD AL,9
00D00039    FEC8               DEC AL

00D0003B    90                 NOP
00D0003C    90                 NOP
00D0003D    90                 NOP
00D0003E    90                 NOP
00D0003F    90                 NOP
00D00040    90                 NOP
00D00041    34 2C              XOR AL,2C
00D00043    C0C0 DA            ROL AL,0DA

00D00046    90                 NOP
00D00047    90                 NOP
00D00048    90                 NOP
00D00049    90                 NOP
00D0004A    90                 NOP
00D0004B    90                 NOP
00D0004C    90                 NOP
00D0004D    90                 NOP
00D0004E    90                 NOP
00D0004F    FEC8               DEC AL
00D00051    90                 NOP
00D00052    AA                 STOS BYTE PTR ES:[EDI]                 ; 解码
00D00053    49                 DEC ECX
00D00054  ^ 75 D2              JNZ SHORT 00D00028
```

**加密代码：**

```
……
00D10009    9C                 PUSHFD
00D1000A    60                 PUSHAD
00D1000B    8B4424 24          MOV EAX,DWORD PTR SS:[ESP+24]
00D1000F    8B08               MOV ECX,DWORD PTR DS:[EAX]
00D10011    8D78 04            LEA EDI,DWORD PTR DS:[EAX+4]
00D10014    897C24 24          MOV DWORD PTR SS:[ESP+24],EDI
00D10018    8D7F F6            LEA EDI,DWORD PTR DS:[EDI-A]
00D1001B    81E9 F67FBB0E      SUB ECX,0EBB7FF6
00D10021    2BF9               SUB EDI,ECX
00D10023    FC                 CLD
00D10024    8A07               MOV AL,BYTE PTR DS:[EDI]
00D10026    90                 NOP
00D10027    FEC0               INC AL

00D10029    90                 NOP
00D1002A    90                 NOP
00D1002B    90                 NOP
00D1002C    90                 NOP
00D1002D    90                 NOP
00D1002E    90                 NOP
00D1002F    90                 NOP
00D10030    90                 NOP
00D10031    90                 NOP
00D10032    C0C8 DA            ROR AL,0DA

00D10035    34 2C              XOR AL,2C

00D10037    90                 NOP
00D10038    90                 NOP
00D10039    90                 NOP
00D1003A    90                 NOP
00D1003B    90                 NOP
00D1003C    90                 NOP
00D1003D    FEC0               INC AL
00D1003F    2C 09              SUB AL,9
00D10041    FEC0               INC AL
00D10043    2AC1               SUB AL,CL
00D10045    2C D0              SUB AL,0D0

00D10047    90                 NOP
00D10048    90                 NOP
00D10049    90                 NOP
00D1004A    90                 NOP
00D1004B   C0C0 42            ROL AL,42

00D1004E    AA                 STOS BYTE PTR ES:[EDI]
00D1004F    49                 DEC ECX
00D10050  ^ 75 D2              JNZ SHORT 00D10024
```

### 第三种：`Anti Unpack`

这个不知道算不算，但主程序里有一处检测：

```
00409837    B8 ABA44300        MOV EAX,43A4AB                   ; 检测是否被脱壳
0040983C    2D 910A0300        SUB EAX,30A91
00409841    FFD0               CALL EAX                         ; 这里进去就是具体的方法
```

看看具体的：

```
00409A1A    B8 1BBAD5FA        MOV EAX,FAD5BA1B
00409A1F    05 BFE06A05        ADD EAX,56AE0BF
00409A24    BB 4655A308        MOV EBX,8A35546
00409A29    81EB 5D54A308      SUB EBX,8A3545D                  ; EBX=0E9
00409A2F    2A18               SUB BL,BYTE PTR DS:[EAX]          ; 也就是检测 409ada 处是否为 0E9,如果不是后面就会乱跳
00409A31    58                 POP EAX
00409A32    C1C3 16            ROL EBX,16
00409A35    03C3               ADD EAX,EBX
00409A37    FFE0               JMP EAX                          ; 如果正确返回上面执行正确的代码
```

到这里壳已经分析完毕。转第二步了\_^.\_

### 第二步：脱壳

分析完就可以脱壳了，这里要写的修复代码也比较多哦。我用脚本、修改壳代码和自己写代码完成脱壳过程。

过程为：脚本修复 `IAT` 到 `stolen code` -> 申请空间 -> 写代码 -> 改 `EIP` -> 设断 -> 修改代码 -> 脚本清除“垃圾”代码 -> 去除 `anti-unpack`。

先写点脚本修复 `IAT` 并到 `OEP Stolen code` 处：脚本出下：

```
/*
//////////////////////////////////////////////////
    PESpin v1.1 Stolen Code Finder v0.1
    Author:    loveboom
    Email : loveboom#163.com
    OS    : WinXP sp1,Ollydbg 1.1,OllyScript v0.92
    Date  : 2005-3-9
        Action: 修复 IAT，停在 stolen code 处.
    Config: Ignore all exceptions
    Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
*/

var addr
var addr1

start:

   Msgyn "Config:Ignore all exceptions,continue?"

   cmp $RESULT,1

   je lbl1

   ret

lbl1:

  gpa "LoadLibraryA","kernel32.dll"        //在 LoadLibraryA+B 处下断
  mov addr,$RESULT
  add addr,B
  bp addr
  esto

lbl2:

  cmp eip,addr
  jne lblabort
  bc addr
  mov addr,esp
  add addr,c
  mov addr,[addr]
  bp addr
  esto
  bc addr

lbl3:

  find eip,#0FBA67FF07#        //find command 'bt [edi-1],7'
  cmp $RESULT,0
  je lblabort
  mov addr,$RESULT
  fill addr,1,F8        //修改为 clc 清除 CF
  inc addr
  mov [addr],90909090

lblnext1:

  find addr,#0F31#        //find command 'RDTSC'
  cmp $RESULT,0
  je lblabort
  find $RESULT,#FF6424FC#        //find command 'JMP DWORD PTR SS:[ESP-4]'
  cmp $RESULT,0
  je lblabort
  mov addr1,$RESULT
  bp addr1

lblfind1:

  find addr,#FF6424FC#        //find command 'JMP DWORD PTR SS:[ESP-4]'
  cmp $RESULT,0
  je lblabort
  go $RESULT
  sto
  sti

lblfind2:

  find eip,#807FFFEA#        //find command'CMP BYTE PTR DS:[EDI-1],0EA'
  cmp $RESULT,0
  je lblabort
  find $RESULT,#FE4FFF83C7042BC78947FC#

/*

find commands:

    FE4F FF         DEC BYTE PTR DS:[EDI-1]
    83C7 04         ADD EDI,4
    2BC7            SUB EAX,EDI
    8947 FC         MOV DWORD PTR DS:[EDI-4],EAX

*/

  cmp $RESULT,0
  je lblabort
  fill $RESULT,b,90
  mov addr,$RESULT
  bp addr

lblloop1:

  run

lblcheck:

  cmp eip,addr
  jne lbl4
  exec                //fix iat
    mov word ptr [edi-1],25FF
    mov [edi+1],edx
    mov [edx],eax
  ende
  jmp lblloop1

lbl4:

  bc addr
  bc addr1
  find eip,#E801000000??83C404#        //find commands:'call $+1 add esp,4'
  cmp $RESULT,0
  je lblerrver
  go $RESULT
  find $RESULT,#61#
  cmp $RESULT,0
  je lblerrver
  go $RESULT
  sto
  cmt eip,"Stolen code."

lblend:

  msg "Script finished,script by loveboom[DFCG][FCG][US].Thank you for using my script!"
```
