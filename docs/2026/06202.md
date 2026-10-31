
# EXECryptor2.2.6主程序脱壳[分享]

## 一、破解目标

EXECryptor2.2.6主程序

## 二、破解工具

OllyDbg v1.10，ImportREC 1.6 Final，LordPE

## 三、破解作者

DarkBull#126.com

## 四、破解过程

### 1. 寻找OEP

首先设置OD停在系统断点，忽略所有异常，载入主程序，EP如下：

```
<ModuleEn>  E8 F7FEFFF>CALL EXECrypt.006D890A                ; EP

006D8A13    05 DBAE000>ADD EAX,0AEDB

006D8A18    FFE0       JMP NEAR EAX

006D8A1A    E8 EBFEFFF>CALL EXECrypt.006D890A                ; TLS CallBack

006D8A1F    05 16CF000>ADD EAX,0CF16

006D8A24    FFE0       JMP NEAR EAX

006D8A26    E8 0400000>CALL EXECrypt.006D8A2F

006D8A2B    FFFF       ???

006D8A2D    FFFF       ???

006D8A2F    5E         POP ESI

006D8A30    C3         RET
```

用HideOD隐藏好OD，去除EP处的一次性断点。在CODE段下内存访问断点，F9运行，来到解码处：

```
006CB6F0    56         PUSH ESI

006CB6F1    57         PUSH EDI

006CB6F2    53         PUSH EBX

006CB6F3    31DB       XOR EBX,EBX

006CB6F5    89C6       MOV ESI,EAX

006CB6F7    89D7       MOV EDI,EDX

006CB6F9    0FB606     MOVZX EAX,BYTE PTR DS:[ESI]

006CB6FC    89C2       MOV EDX,EAX

006CB6FE    83E0 1F    AND EAX,1F

006CB701    C1EA 05    SHR EDX,5

006CB704    74 2D      JE SHORT EXECrypt.006CB733

006CB706    4A         DEC EDX

006CB707    74 15      JE SHORT EXECrypt.006CB71E

006CB709    8D5C13 02  LEA EBX,DWORD PTR DS:[EBX+EDX+2]

006CB70D    46         INC ESI

006CB70E    C1E0 08    SHL EAX,8

006CB711    89FA       MOV EDX,EDI

006CB713    0FB60E     MOVZX ECX,BYTE PTR DS:[ESI]

006CB716    46         INC ESI

006CB717    29CA       SUB EDX,ECX

006CB719    4A         DEC EDX

006CB71A    29C2       SUB EDX,EAX

006CB71C    EB 32      JMP SHORT EXECrypt.006CB750

006CB71E    C1E3 05    SHL EBX,5

006CB721    8D5C03 04  LEA EBX,DWORD PTR DS:[EBX+EAX+4]

006CB725    46         INC ESI

006CB726    89FA       MOV EDX,EDI

006CB728    0FB70E     MOVZX ECX,WORD PTR DS:[ESI]

006CB72B    29CA       SUB EDX,ECX

006CB72D    4A         DEC EDX

006CB72E    83C6 02    ADD ESI,2

006CB731    EB 1D      JMP SHORT EXECrypt.006CB750

006CB733    C1E3 04    SHL EBX,4

006CB736    46         INC ESI

006CB737    89C1       MOV ECX,EAX

006CB739    83E1 0F    AND ECX,0F

006CB73C    01CB       ADD EBX,ECX

006CB73E    C1E8 05    SHR EAX,5

006CB741    73 07      JNB SHORT EXECrypt.006CB74A

006CB743    43         INC EBX

006CB744    89F2       MOV EDX,ESI

006CB746    01DE       ADD ESI,EBX

006CB748    EB 06      JMP SHORT EXECrypt.006CB750

006CB74A    85DB       TEST EBX,EBX

006CB74C    74 0E      JE SHORT EXECrypt.006CB75C

006CB74E  ^ EB A9      JMP SHORT EXECrypt.006CB6F9

006CB750    56         PUSH ESI

006CB751    89D6       MOV ESI,EDX

006CB753    89D9       MOV ECX,EBX

006CB755    F3:A4      REP MOVSB                             ; 写入解压后的代码

006CB757    31DB       XOR EBX,EBX

006CB759    5E         POP ESI

006CB75A  ^ EB 9D      JMP SHORT EXECrypt.006CB6F9

006CB75C    89F0       MOV EAX,ESI

006CB75E    5B         POP EBX

006CB75F    5F         POP EDI

006CB760    5E         POP ESI

006CB761    C3         RET
```

然后壳对CODE段`JMP&CALL`的目标地址进行修正，代码如下：

```
005AC14E    AC         LODSB

005AC14F    D0E8       SHR AL,1

005AC151    80F8 74    CMP AL,74                            ; 是否为JMP&CALL

005AC154    75 0E      JNZ SHORT EXECrypt.005AC164

005AC156    8B06       MOV EAX,DWORD PTR DS:[ESI]

005AC158    0FC8       BSWAP EAX

005AC15A    01C8       ADD EAX,ECX

005AC15C    8906       MOV DWORD PTR DS:[ESI],EAX

005AC15E    83C6 04    ADD ESI,4

005AC161    83E9 04    SUB ECX,4

005AC164    49         DEC ECX

005AC165  ^ 7F E7      JG SHORT EXECrypt.005AC14E

005AC167    59         POP ECX

005AC168    5E         POP ESI

005AC169    C3         RET
```

对`CreateThread`下硬件断点，可以发现壳创建了17个从线程，通过学习simonzhong2000兄的文章，偶对从线程的功能有个大概的了解，在此表示感谢。在EP处下断点，F9来到EP，在OD中去掉忽略所有异常，手动通过最后一个异常（地址：`006D18E7`），根据ESP定律可以找到OEP为`0054E120`，然后DUMP。

### 2. 修复IAT

通过观察，可以确定IAT的起始地址为：`004ED168`，结束地址为：`004ED900`。

IAT需要修复的有两种情况：1.在调用系统函数前写入IAT；2.直接调用系统函数，不写入IAT，需要手动修复。

IAT修复脚本如下：

```
data:

    var base

    var size

    var iats

    var iate

    var fun

    var cnt

code:

    gmi eip,MODULEBASE

    mov base,$RESULT

    gmi eip,MODULESIZE

    mov size,$RESULT

    add size,base

    mov iats,4ED168                                          ; IAT起始地址

    mov iate,4ED900                                          ; IAT结束地址

    exec

    push 004e70f0                                            ; 压入4个参数，防止无效指针。

    push 004e70a0

    push 004e7050

    push 004e7000

    ende

loop1:

    mov fun,[iats]

    cmp fun,base

    jb next

    cmp fun,size

    ja next

    mov eip,fun

    mov esp,0012ffb4

    bphws iats,"w"

    run

    gn [iats]

    cmp $RESULT,0                                            ; 是否为函数地址

    je pause1

    bphwc iats

    inc cnt

    jmp next

pause1:

    pause                                                    ; 手动修复

    bphwc iats

next:

    add iats,4

    cmp iats,iate

    ja end

    jmp loop1

end:

    eval "Already Found {cnt} Function!"

    msg $RESULT

    ret
```

### 3. 跨平台

#### 3.1 加入壳用的IAT

```
FThunk: 000AB173        NbFunc: 00000006

1        000AB173        kernel32.dll        0180        GetModuleHandleA

1        000AB177        kernel32.dll        0253        LoadLibraryA

1        000AB17B        kernel32.dll        01A1        GetProcAddress

1        000AB17F        kernel32.dll        00BA        ExitProcess

1        000AB183        kernel32.dll        0382        VirtualAlloc

1        000AB187        kernel32.dll        0384        VirtualFree

FThunk: 000AB1C3        NbFunc: 00000001

1        000AB1C3        user32.dll        01E0        MessageBoxA
```

#### 3.2 修复TLS Table

```
RVA:000F2000,SIZE:00000018

StartAddressOfRawData:  0x004F1000

EndAddressOfRawData:    0x004F1010

AddressOfIndex:         0x004E74D4

AddressOfCallBacks:     0x004F2010

SizeOfZeroFill:         0x00000000

Characteristics:        0x00000000
```

#### 3.3 修复已初始数据

```
58D9CC        GetModuleHandleA

522EC8        kernel32.dll

522F10        ntdll.dll

58C674        DebugBreak

58be58        ExitProcess

535E10        GlobalAlloc

535E0C        GlobalLock

5A1BD8        hMem

547E90        GetCurrentProcess

5A0310        CreateThread

5A0304        CloseHandle

5A1BE8        hMem

5A02E4        user32.dll

5A1BDC        SetTimer

5961F8        LoadLibraryA

531840        oleaut32.dll

531851        version.dll

531861        gdi32.dll

53186F        ole32.dll

53187D        comctl32.dll

53188E        winspool.dll

53189F        shell32.dll

5318AF        comdlg32.dll
```

将以上地址全部清零，此处感谢南蛮妈妈。

2006.04.24

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [execryptor\_unpacked.part1.rar](javascript:void(0);)

  （0.98MB，78次下载）
