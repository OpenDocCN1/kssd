
# EnCryptPE v2 Code injection

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-565.htm`](https://www.pediy.com/kssd/pediy07/pediy7-565.htm)

## 目标、工具与任务

- **目标**: 随便你自己去加个就行了
- **工具**: `Olydbg1.1`(diy 版)、`LORDPE`, `WINHEX`
- **任务**: 不脱壳注入 pj 代码
- **操作平台**: `Windows 2003 server`
- **作者**: `LOVEBOOM[DFCG][FCG][US]`
- **相关链接**: `www.encryptpe.com`

## 简要说明

EPE V2 的反调试和加密功能比 2003 可说是强了很多。这次我来一次不脱壳直接通过注入代码修改原程序流程。通过代码注入绕过 EPE 的注册认证。这个问题我已经和老王说过了，不久后应该会解决这个问题的:\_)。我这篇只讲 service 的加壳方式，此方法也可以用于普通的加密方式，因为考虑有关问题，我不打算写出来。我选用 `epe2004810` 做为目标，关于这个可以很方便的注入的原因老王说了是为了方便有的客户加多层壳。看来做什么，怎么做都是有难的地方。

## 详细过程

首先强烈建议各位对这个壳有一定了解，对 PE 知识有点了解，知道怎么用 OD 修改保存代码，并且会脱 `EPE V22004810` 的朋友再看这篇文章，至于 epev2 的脱壳，可以参考 `FLY` 或是我的 for 2005314 的文章，情况是差不多的。这篇文章里不会告诉你如何去找 CRC 处，CRC 处在我的 2005 的那篇文章里有写😊.因为这个壳是通过注入壳的 DLL 到 explorer 时里，所以如果你的 patch 后程序运行之前你的系统已经加载了 DLL，这个注入方法将会失败（当然，用其它方法还是可以注入的，方法也不难，自己想想就会了。）。

用 OD 载入目标，设置忽略全部异常，按 `epe v20053.14` 的方法可以很快速的定位到 CRC 检测和两个注册检测点，找到的结果：

### CRC 的两个地方

```asm
711E3792    B0 00           MOV AL,0       ;crc check 这里是修改后的 
......
711E501D    0F95C0          SETNE AL       ;crc check
711E5020    5F              POP EDI
711E5021    5E              POP ESI
```

### 注册的两个地方

```asm
711E2C4B    8B83 00030000   MOV EAX,DWORD PTR DS:[EBX+300]
711E2C51    8378 0C 01      CMP DWORD PTR DS:[EAX+C],1     ;这里必须为 0([EAX+C]==0 跳过注册)
711E2C55    75 22           JNZ SHORT 711E2C79       ;如果不要注册则跳
......
711E2C85    8B83 24030000   MOV EAX,DWORD PTR DS:[EBX+324]
711E2C8B    8378 30 64      CMP DWORD PTR DS:[EAX+30],64  ;这里不能为 64 正常应该是 65([EAX+30]<>64 跳过注册)
711E2C8F    75 0A           JNZ SHORT 711E2C9B       
```

Ok 再次载入目标，`F9` 运行几秒钟，`F12` 暂停然后执行到解压壳代码处，因为壳是 DLL 的所以要向上找到判断是否已经解压的代码直接向上找到：

```asm
71236C90 >  807C24 08 01    CMP BYTE PTR SS:[ESP+8],1
71236C95    0F85 91010000   JNZ 71236E2C                  ; 这里判断，如果解压完了则跳
```

好了，先记下这个地址，向下找到跳过壳入口的代码：

```asm
71236E2B    61              POPAD                              ; 这里开始改成我们自己的 patch 代码
71236E2C  ^ E9 73E5FAFF    JMP 711E53A4
```

跟踪发现如果 DLL 没有载入的话每次都会强行生成一个生的 DLL 文件，下面我们就来看看它是怎么样生成的,重来一遍分析一下：

```asm
0044E000 >  60              PUSHAD                            ; EP
0044E001    9C              PUSHFD
0044E002    64:FF35 0000000> PUSH DWORD PTR FS:[0]
0044E009    E8 7A010000     CALL 0044E188                      ; 这里 f7 跟进
……
```

进来后分析如下：

```asm
0044E188    5D              POP EBP                            ; CDD1+5cr.0044E00E
0044E189    68 04010000     PUSH 104                           ; /Buffer Size ==104
0044E18E    89E8            MOV EAX,EBP                        ; |
0044E190    83C0 4C         ADD EAX,4C                         ; |
0044E193    50              PUSH EAX                           ; |Buffer
0044E194    89E8            MOV EAX,EBP                        ; |
0044E196    83C0 28         ADD EAX,28                         ; |
0044E199    FF10            CALL DWORD PTR DS:[EAX]            ; \API Function Get SystemDirectoryA
0044E19B    83F8 04         CMP EAX,4                          ; 如果路径长度小于 4 则 over
0044E19E    0F8C 1F010000    JL <ExitProcess>
0044E1A4    89C1            MOV ECX,EAX
0044E1A6    89E8            MOV EAX,EBP
0044E1A8    83C0 4C         ADD EAX,4C
0044E1AB    8A5401 FF       MOV DL,BYTE PTR DS:[ECX+EAX-1]     ; 判断路径是否以\结束
0044E1AF    80FA 5C         CMP DL,5C
0044E1B2    74 06           JE SHORT <@F>
0044E1B4    41              INC ECX                            ; 如果不是则加上\
0044E1B5    C64401 FF 5C    MOV BYTE PTR DS:[ECX+EAX-1],5C
0044E1BA >  89C3            MOV EBX,EAX
0044E1BC    81C3 05010000    ADD EBX,105
0044E1C2    01C8            ADD EAX,ECX
0044E1C4    31C9            XOR ECX,ECX
0044E1C6    8A1419          MOV DL,BYTE PTR DS:[ECX+EBX]       ; 连接成完整的 DLL 名
0044E1C9    881401          MOV BYTE PTR DS:[ECX+EAX],DL
0044E1CC    80FA 00         CMP DL,0
0044E1CF    74 03           JE SHORT <Do next>                 ; 生成了完整的 DLL 名则跳
0044E1D1    41              INC ECX
0044E1D2  ^ EB F2           JMP SHORT 0044E1C6
0044E1D4 >  6A 00           PUSH 0                              ; /hTemplateFile = NULL
0044E1D6    68 80000000     PUSH 80                             ; |Attributes = NORMAL
0044E1DB    6A 04           PUSH 4                              ; |Mode = OPEN_ALWAYS
0044E1DD    6A 00           PUSH 0                              ; |pSecurity = NULL
0044E1DF    6A 00           PUSH 0                              ; |ShareMode = 0
0044E1E1    68 000000C0     PUSH C0000000                       ; |Access = GENERIC_READ|GENERIC_WRITE
0044E1E6    89E8            MOV EAX,EBP                          ; |
0044E1E8    83C0 4C         ADD EAX,4C                           ; |
0044E1EB    50              PUSH EAX                              ; |FileName = "C:\WINDOWS\system32\V22004810.EPE"
0044E1EC    89E8            MOV EAX,EBP                          ; |
0044E1EE    83C0 2C         ADD EAX,2C                           ; |
0044E1F1    FF10            CALL DWORD PTR DS:[EAX]            ; \CreateFileA
0044E1F3    83F8 FF         CMP EAX,-1                           ; 如果返回－1 则表明 DLL 已经加载，如果文件不存在则会新建一个
0044E1F6    0F84 86000000    JE <LoadLibrary Proc>               ; 如果目标 DLL 已经加载则跳去直接 GetProcess
0044E1FC    89C3            MOV EBX,EAX
0044E1FE    6A 00           PUSH 0                              ; /MapName = NULL
0044E200    89E8            MOV EAX,EBP                          ; |
0044E202    05 71010000     ADD EAX,171                          ; |
0044E207    8B08            MOV ECX,DWORD PTR DS:[EAX]           ; |
0044E209    51              PUSH ECX                              ; |MaximumSizeLow = 6B200
0044E20A    6A 00           PUSH 0                              ; |MaximumSizeHigh = 0
0044E20C    6A 04           PUSH 4                              ; |Protection = PAGE_READWRITE
0044E20E    6A 00           PUSH 0                              ; |pSecurity = NULL
0044E210    53              PUSH EBX                              ; |hFile = 00000014 (window)
0044E211    89E8            MOV EAX,EBP                          ; |
0044E213    83C0 30         ADD EAX,30                           ; |
0044E216    FF10            CALL DWORD PTR DS:[EAX]            ; \CreateFileMappingA
0044E218    83F8 00         CMP EAX,0
0044E21B    74 5D           JE SHORT <CloseHandleProc>          ; 如果操作失败则跳去直接 GetProcess
0044E21D    89C6            MOV ESI,EAX                          ; hMem
0044E21F    89E8            MOV EAX,EBP
0044E221    05 71010000     ADD EAX,171
0044E226    8B08            MOV ECX,DWORD PTR DS:[EAX]
0044E228    51              PUSH ECX                              ; /MapSize = 6B200 (438784.)
0044E229    6A 00           PUSH 0                              ; |OffsetLow = 0
0044E22B    6A 00           PUSH 0                              ; |OffsetHigh = 0
0044E22D    68 1F000F00     PUSH 0F001F                         ; |AccessMode = F001F
0044E232    56              PUSH ESI                              ; |hMapObject = 0000001C
0044E233    89E8            MOV EAX,EBP                          ; |
0044E235    83C0 34         ADD EAX,34                           ; |
0044E238    FF10            CALL DWORD PTR DS:[EAX]            ; \MapViewOfFile
0044E23A    83F8 00         CMP EAX,0
0044E23D    74 33           JE SHORT <CloseHandleProc>          ; 如果操作失败则跳
0044E23F    50              PUSH EAX
0044E240    89C7            MOV EDI,EAX
0044E242    89E8            MOV EAX,EBP
0044E244    05 71010000     ADD EAX,171
```



0044E249    8B08                 MOV ECX,DWORD PTR DS:[EAX]  

0044E24B    83F9 00              CMP ECX,0  

0044E24E    74 1B                JE SHORT <UnmapViewOfFileProc>               ; 如果大小为 0 则跳  

0044E250    51                   PUSH ECX                                      ; 开始循环解压出代码  

0044E251    8B48 04              MOV ECX,DWORD PTR DS:[EAX+4]  

0044E254    8A70 08              MOV DH,BYTE PTR DS:[EAX+8]                     ; 这里关键取出关键 KEY（28）  

0044E257    83C0 09              ADD EAX,9  

0044E25A    01C8                 ADD EAX,ECX  

0044E25C    59                   POP ECX  

0044E25D >   8A10                 MOV DL,BYTE PTR DS:[EAX]                       ; eax==开始解压的地址,44E2CC  

0044E25F    32D6                 XOR DL,DH                                       ; Decrypt =KEY (28) xor Encrypted code  

0044E261    8817                 MOV BYTE PTR DS:[EDI],DL                       ; 保存正确的代码到 mem 中  

0044E263    40                   INC EAX  

0044E264    47                   INC EDI  

0044E265    49                   DEC ECX  

0044E266    83F9 00              CMP ECX,0  

0044E269  ^ 75 F2                JNZ SHORT <Loop Decrypt>  

0044E26B >   89E8                 MOV EAX,EBP  

0044E26D    83C0 38              ADD EAX,38  

0044E270    FF10                 CALL DWORD PTR DS:[EAX]                        ; UnmapViewOfFile  

0044E272 >   56                   PUSH ESI                                      ; /hObject==1c  

0044E273    89E8                 MOV EAX,EBP                                   ; |  

0044E275    83C0 3C              ADD EAX,3C                                    ; |  

0044E278    FF10                 CALL DWORD PTR DS:[EAX]                        ; \\CloseHandle  

0044E27A >   53                   PUSH EBX                                      ; /hObject  

0044E27B    89E8                 MOV EAX,EBP                                   ; |  

0044E27D    83C0 3C              ADD EAX,3C                                    ; |  

0044E280    FF10                 CALL DWORD PTR DS:[EAX]                        ; \\CloseHandle  

0044E282 >   89E8                 MOV EAX,EBP  

0044E284    83C0 4C              ADD EAX,4C  

0044E287    50                   PUSH EAX                                      ; /FileName = "C:\\WINDOWS\\system32\\V22004810.EPE"  

0044E288    89E8                 MOV EAX,EBP                                   ; |  

0044E28A    83C0 40              ADD EAX,40                                    ; |  

0044E28D    FF10                 CALL DWORD PTR DS:[EAX]                        ; \\LoadLibraryA  

0044E28F    83F8 00              CMP EAX,0                                     ; 如果载入 DLL 失败则 OVER  

0044E292    74 2F                JE SHORT <ExitProcess>  

0044E294    89C3                 MOV EBX,EAX                                   ; EAX=hmodule  

0044E296    89E8                 MOV EAX,EBP  

0044E298    05 62010000          ADD EAX,162                                   ; 准备获取 Encrypt_Init 的地址  

0044E29D    50                   PUSH EAX                                      ; /ProcNameOrOrdinal = "EncryptPE_Init"  

0044E29E    53                   PUSH EBX                                      ; |hModule = 71120000 (V2200481)  

0044E29F    89E8                 MOV EAX,EBP                                   ; |  

0044E2A1    83C0 44              ADD EAX,44                                    ; |  

0044E2A4    FF10                 CALL DWORD PTR DS:[EAX]                        ; \\GetProcAddress  

0044E2A6    83F8 00              CMP EAX,0                                     ; 如果获取函数失败则 over  

0044E2A9    74 18                JE SHORT <ExitProcess>  

0044E2AB    89C3                 MOV EBX,EAX  

0044E2AD    89E8                 MOV EAX,EBP  

0044E2AF    05 71010000          ADD EAX,171  

0044E2B4    8B08                 MOV ECX,DWORD PTR DS:[EAX]  

0044E2B6    8B50 04              MOV EDX,DWORD PTR DS:[EAX+4]  

0044E2B9    83C0 09              ADD EAX,9  

0044E2BC    01D0                 ADD EAX,EDX  

0044E2BE    01C8                 ADD EAX,ECX  

0044E2C0    50                   PUSH EAX                                      ; /Decrypt address  

0044E2C1    FFD3                 CALL EBX                                      ; \\Call DecryptCode  

0044E2C3 >   6A 00                PUSH 0                                       ; /Exit Code ==0  

0044E2C5    89E8                 MOV EAX,EBP                                   ; |  

0044E2C7    83C0 48              ADD EAX,48                                    ; |  

0044E2CA    FF10                 CALL DWORD PTR DS:[EAX]                        ; \\Exit Process  

0044E2CC    65:72 78             JB SHORT 0044E347                               ; 这里开始就是壳程序加密后的代码存放处  

0044E2CF    282A                 SUB BYTE PTR DS:[EDX],CH  

0044E2D1    2828                 SUB BYTE PTR DS:[EAX],CH  

通过分析现在已经得到 `KEY` 为 `28`，现在通过计算可以知道跳去壳 `OEP` 的代码存放在 `address:4b84f7` `RAW:6AAF7` 处，判断是否已经加载的代码大概在: `address:4B8363` `RAW:6A963` 处.下面就剩下写 `patch` 代码了：  

```
71236E2B    E8 00000000         CALL 71236E30                                  ; 用于后面定位具体地址  
71236E30    58                  POP EAX  
71236E31    8BD8                MOV EBX,EAX  
71236E33    2D 9E360500         SUB EAX,5369E  
71236E38    E8 30000000         CALL 71236E6D                                  ; 去除 711E3792 处的 CRC 检测  
71236E3D    8BC3                MOV EAX,EBX  
71236E3F    2D 131E0500         SUB EAX,51E13  
71236E44    E8 24000000         CALL 71236E6D                                  ; 去除 711E501D 处的 CRC 检测  
71236E49    2D D2230000         SUB EAX,23D2  
71236E4E    8BD3                MOV EDX,EBX  
71236E50    83C2 58             ADD EDX,58  
71236E53    E8 20000000         CALL 71236E78                                  ; patch 代码去除注册判断点一  
71236E58    83C0 3A             ADD EAX,3A  
71236E5B    8BD3                MOV EDX,EBX  
71236E5D    83C2 68             ADD EDX,68  
71236E60    E8 13000000         CALL 71236E78                                  ; PATCH 代码去除注册判断点二  
71236E65    61                  POPAD  
71236E66  ^ E9 39E5FAFF         JMP 711E53A4                                   ; 修改完毕，执行原壳代码  
71236E6B    90                  NOP  
71236E6C    90                  NOP  
71236E6D    66:C700 B000        MOV WORD PTR DS:[EAX],0B0                      ; 两个 crc 处都改为 mov bl,0  
71236E72    C640 02 90          MOV BYTE PTR DS:[EAX+2],90  
71236E76    C3                  RETN  
71236E77    90                  NOP  
71236E78    2BD0                SUB EDX,EAX                                     ; 计算跳转的距离  
71236E7A    83EA 06             SUB EDX,6  
71236E7D    66:C700 90E8        MOV WORD PTR DS:[EAX],0E890  
71236E82    8950 02             MOV DWORD PTR DS:[EAX+2],EDX                    ; 修改地址  
71236E85    C3                  RETN  
71236E86    90                  NOP  
71236E87    90                  NOP  
71236E88    8B83 00030000       MOV EAX,DWORD PTR DS:[EBX+300]                   ; patch 注册点一  
71236E8E    C740 0C 0000000>    MOV DWORD PTR DS:[EAX+C],0  
71236E95    C3                  RETN  
71236E96    90                  NOP  
71236E97    90                  NOP  
71236E98    8B83 24030000       MOV EAX,DWORD PTR DS:[EBX+324]                   ; patch 注册点二  
71236E9E    C740 30 6500000>    MOV DWORD PTR DS:[EAX+30],65  
71236EA5    C3                  RETN  
```

写完这代码再写几行加密代码，然后把代码保存到相关位置即可。  

加密代码:  

```
      mov ecx,size  
    mov edi,offset startaddr  
  @@:  
    mov al,byte ptr [edi]  
    xor al,28h  
    stosb  
    dec ecx  
    jnz @B  
```

计算后和得出加密后的数据，然后用 `winhex` 放到相关位置，保存，再运行试试，如何，是的你成功了。  

Greetz:  

Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!  

By `loveboom[DFCG][FCG][US]`  

`http://blog.csdn.net/bmd2chen`  

Email:`loveboom#163.com`  

Date:`5/22/2005 9:51:00 PM`  

`4dd466d0e1e0cc123afbe504da2462ca`
