
# e 软代理服务器

> 原文：`www.pediy.com/kssd/pediy06/pediy6534.htm`

【软件名称】 e 软代理服务器

【下载地址】 `www.esoftsoft.com/`

【声明】 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

这是一个代理服务器软件，无壳，`delphi` 程序。

程序采用机器码/授权文件方式注册，随便输入一些注册信息后点击验证按纽，提示错误信息。

`delphi` 的当然先用 `dede` 来看看了，`dede` 分析后找到对授权文件验证按纽点击事件的处理的地方：

### 1

`TFrmMain.SpeedButton13Click`

```
0059CC48   55                 push    ebp  
0059CC49   8BEC               mov     ebp, esp  
0059CC4B   6A00               push    $00  
0059CC4D   53                 push    ebx  
0059CC4E   8BD8               mov     ebx, eax  
0059CC50   33C0               xor     eax, eax  
0059CC52   55                 push    ebp  
* Possible String Reference to: '閝 u?腽[Y]?  
|  
0059CC53   68E2CC5900         push    $0059CCE2  
***** TRY  
|  
0059CC58   64FF30             push    dword ptr fs:[eax]  
0059CC5B   648920             mov     fs:[eax], esp  
0059CC5E   8BC3               mov     eax, ebx  
|  
0059CC60   E833FEFFFF         call    0059CA98  
0059CC65   84C0               test    al, al  
0059CC67   741B               jz      0059CC84  
0059CC69   6A30               push    $30  
* Possible String Reference to: '[X 软软件]提醒'  
|  
0059CC6B   68F0CC5900         push    $0059CCF0  
* Possible String Reference to: '你的注册码是正确的，恭喜你'  
|  
0059CC70   6800CD5900         push    $0059CD00  
0059CC75   8BC3               mov     eax, ebx  
* Reference to: Controls.TWinControl.GetHandle(TWinControl):HWND;  
|           or: QComCtrls.TTrackBar.GetHandle(TTrackBar):QClxSliderH;  
|           or: QComCtrls.TCustomViewControl.GetHandle(TCustomViewControl):QListViewH;  
|           or: QComCtrls.TCustomViewControl.ViewportHandle(TCustomViewControl):QWidgetH;  
|           or: QComCtrls.TCustomHeaderControl.GetHandle(TCustomHeaderControl):QHeaderH;  
|           or: QComCtrls.TCustomSpinEdit.GetHandle(TCustomSpinEdit):QClxSpinBoxH;  
|  
0059CC77   E87C23EBFF         call    0044EFF8  
0059CC7C   50                 push    eax  
* Reference to: user32.MessageBoxA()  
|  
0059CC7D   E8AAAEE6FF         call    00407B2C  
0059CC82   EB48               jmp     0059CCCC  
0059CC84   8D55FC             lea     edx, [ebp-$04]  
* Reference to control MemoRegister : TMemo  
|  
0059CC87   8B835C050000       mov     eax, [ebx+$055C]  
* Reference to: Controls.TControl.GetText(TControl):TCaption;  
|  
0059CC8D   E856BBEAFF         call    004487E8  
0059CC92   837DFC00           cmp     dword ptr [ebp-$04], +$00  
0059CC96   751B               jnz     0059CCB3  
0059CC98   6A30               push    $30  
* Possible String Reference to: '[X 软软件]提醒'  
|  
0059CC9A   68F0CC5900         push    $0059CCF0  
* Possible String Reference to: '对不起，你还没有注册，你的使用将受?  
|                             揭欢ǖ南拗疲?  
|  
0059CC9F   681CCD5900       push    $0059CD1C  
0059CCA4   8BC3               mov     eax, ebx  
* Reference to: Controls.TWinControl.GetHandle(TWinControl):HWND;  
|           or: QComCtrls.TTrackBar.GetHandle(TTrackBar):QClxSliderH;  
|           or: QComCtrls.TCustomViewControl.GetHandle(TCustomViewControl):QListViewH;  
|           or: QComCtrls.TCustomViewControl.ViewportHandle(TCustomViewControl):QWidgetH;  
|           or: QComCtrls.TCustomHeaderControl.GetHandle(TCustomHeaderControl):QHeaderH;  
|           or: QComCtrls.TCustomSpinEdit.GetHandle(TCustomSpinEdit):QClxSpinBoxH;  
|  
0059CCA6   E84D23EBFF       call    0044EFF8  
0059CCAB   50                 push    eax  
* Reference to: user32.MessageBoxA()  
|  
0059CCAC   E87BAEE6FF       call    00407B2C  
0059CCB1   EB19               jmp     0059CCCC  
0059CCB3   6A30               push    $30  
* Possible String Reference to: '[X 软软件]提醒'  
|  
0059CCB5   68F0CC5900       push    $0059CCF0  
* Possible String Reference to: '对不起，您的注册码是不正确的。你的?  
|                             褂媒艿揭欢ǖ南拗疲?  
|  
0059CCBA   6850CD5900       push    $0059CD50  
0059CCBF   8BC3               mov     eax, ebx  
* Reference to: Controls.TWinControl.GetHandle(TWinControl):HWND;  
|           or: QComCtrls.TTrackBar.GetHandle(TTrackBar):QClxSliderH;  
|           or: QComCtrls.TCustomViewControl.GetHandle(TCustomViewControl):QListViewH;  
|           or: QComCtrls.TCustomViewControl.ViewportHandle(TCustomViewControl):QWidgetH;  
|           or: QComCtrls.TCustomHeaderControl.GetHandle(TCustomHeaderControl):QHeaderH;  
|           or: QComCtrls.TCustomSpinEdit.GetHandle(TCustomSpinEdit):QClxSpinBoxH;  
|  
0059CCC1   E83223EBFF       call    0044EFF8  
0059CCC6   50                 push    eax  
* Reference to: user32.MessageBoxA()  
|  
0059CCC7   E860AEE6FF       call    00407B2C  
0059CCCC   33C0               xor     eax, eax  
0059CCCE   5A                 pop     edx  
0059CCCF   59                 pop     ecx  
0059CCD0   59                 pop     ecx  
0059CCD1   648910             mov     fs:[eax], edx  
```

很容易就能找出注册是否成功的判断点：

```
0059CC60   E833FEFFFF         call    0059CA98  
0059CC65   84C0               test    al, al  
0059CC67   741B               jz      0059CC84  
0059CC65   84C0               test    al, al  
```

`al` 为 0 则失败，`al` 不是零就验证通过了。要爆破，把

```
0059CC67   741B               jz      0059CC84  
```

这句 `nop` 掉就可以了。

呵呵，这样也太简单了点吧 :)

### 2

下面来看看这个软件的授权文件验证算法：

进入验证 `call 0059CA98` 看看(以下牵涉到软件作者 `name` 以及 `phone` 的私人信息一律略过)

```
0059CA98   55                 push    ebp  
0059CA99   8BEC               mov     ebp, esp  
0059CA9B   33C9               xor     ecx, ecx  
0059CA9D   51                 push    ecx  
0059CA9E   51                 push    ecx  
0059CA9F   51                 push    ecx  
0059CAA0   51                 push    ecx  
0059CAA1   51                 push    ecx  
0059CAA2   51                 push    ecx  
0059CAA3   53                 push    ebx  
0059CAA4   8BD8               mov     ebx, eax  
0059CAA6   33C0               xor     eax, eax  
0059CAA8   55                 push    ebp  
* Possible String Reference to: '轸 v?胫嬅[嬪]?  
|  
0059CAA9   685FCB5900         push    $0059CB5F  
***** TRY  
|  
0059CAAE   64FF30             push    dword ptr fs:[eax]  
0059CAB1   648920             mov     fs:[eax], esp  
0059CAB4   8B0D04FB5300       mov     ecx, [$0053FB04]  
* Possible String Reference to: '作者的名字'  
|  
0059CABA   BA78CB5900         mov     edx, $0059CB78  
* Reference to control DCP_blowfish1 : TDCP_blowfish  
|  
0059CABF   8B834C050000       mov     eax, [ebx+$054C]  
|  
0059CAC5   E86EFEF9FF         call    0053C938  
0059CACA   8D55F4             lea     edx, [ebp-$0C]  
* Reference to control MemoRegister : TMemo  
|  
0059CACD   8B835C050000       mov     eax, [ebx+$055C]  
* Reference to: Controls.TControl.GetText(TControl):TCaption;  
|  
0059CAD3   E810BDEAFF         call    004487E8                 取得输入的注册信息  
0059CAD8   8B45F4             mov     eax, [ebp-$0C]  
0059CADB   8D55F8             lea     edx, [ebp-$08]  
* Reference to: SysUtils.Trim(AnsiString):AnsiString;overload;  
|  
0059CADE   E819C7E6FF         call    004091FC  
0059CAE3   8B55F8             mov     edx, [ebp-$08]  
0059CAE6   8D4DFC             lea     ecx, [ebp-$04]  
* Reference to control DCP_blowfish1 : TDCP_blowfish  
|  
0059CAE9   8B834C050000       mov     eax, [ebx+$054C]  
0059CAEF   8B18               mov     ebx, [eax]  
0059CAF1   FF5358             call    dword ptr [ebx+$58]  
```



\* Possible String Reference to: '作者的名字，软件信息，网站等等'

|

0059CAF4   6894CB5900          push    $0059CB94

0059CAF9   8D45EC              lea     eax, \[ebp\-$14\]

|

0059CAFC   E8B75AFAFF          call    005425B8                                取网卡 mac 地址

0059CB01   FF75EC              push    dword ptr \[ebp\-$14\]

0059CB04   8D45E8              lea     eax, \[ebp\-$18\]

\* Reference to : `TDCP_blowfish._PROC_00542348()`

|

0059CB07   E83C58FAFF          call    00542348                                取硬盘物理序列号

0059CB0C   FF75E8              push    dword ptr \[ebp\-$18\]

0059CB0F   8D45F0              lea     eax, \[ebp\-$10\]

0059CB12   BA03000000          mov     edx, $00000003

|

0059CB17   E82C81E6FF          call    00404C48

0059CB1C   8B55F0              mov     edx, \[ebp\-$10\]

0059CB1F   8B45FC              mov     eax, \[ebp\-$04\]

\* Reference to: `System.@LStrCmp;`                                          比较

|

0059CB22   E8A581E6FF          call    00404CCC

0059CB27   7504                jnz     0059CB2D

0059CB29   B301                mov     bl, $01

0059CB2B   EB02                jmp     0059CB2F

0059CB2D   33DB                xor     ebx, ebx

0059CB2F   33C0                xor     eax, eax

0059CB31   5A                  pop     edx

0059CB32   59                  pop     ecx

0059CB33   59                  pop     ecx

0059CB34   648910              mov     fs:\[eax\], edx

\*\*\*\*\*\* FINALLY

dede 就是好，代码的流程大概清楚了，粗略的跟踪一下知道验证过程是：

读取输入的注册信息(做某些变换?)然后通过 `blowfish`(`encrypt` 还是 `decrypt`？)算法处理后

与特定的字符串：

`"'作者的名字，软件信息，网站等等'+'网卡 mac'+'硬盘序列号'"` 进行比较，如果一样则验

证通过，注册成功。

这里用到了 `blowfish` 算法，要做的就是找出算法的 `key` 还有是 `encrypt` 还是 `decrypt`。

粗略跟踪知道输入信息是在：

0059CAF1  |. FF53 58       CALL DWORD PTR DS:\[EBX+58\]                ;

进行处理的，步过此 `call` 后就得到了后面和特定字符串比较的数据，进入 `call`：

0053CCE0   . 53            PUSH EBX

0053CCE1   . 56            PUSH ESI

0053CCE2   . 57            PUSH EDI

0053CCE3   . 8BD9          MOV EBX,ECX

0053CCE5   . 8BFA          MOV EDI,EDX

0053CCE7   . 8BF0          MOV ESI,EAX

0053CCE9   . 8BD3          MOV EDX,EBX

0053CCEB   . 8BC7          MOV EAX,EDI

0053CCED   . E8 16F5FFFF   CALL EsoftPro.0053C208                      对输入数据进行 base64 解码

0053CCF2   . 8B03          MOV EAX,DWORD PTR DS:\[EBX\]

0053CCF4   . E8 8F7EECFF   CALL EsoftPro.00404B88

0053CCF9   . 50            PUSH EAX

0053CCFA   . 8BC3          MOV EAX,EBX

0053CCFC   . E8 D780ECFF   CALL EsoftPro.00404DD8

0053CD01   . 50            PUSH EAX

0053CD02   . 8BC3          MOV EAX,EBX

0053CD04   . E8 CF80ECFF   CALL EsoftPro.00404DD8

0053CD09   . 8BD0          MOV EDX,EAX

0053CD0B   . 8BC6          MOV EAX,ESI

0053CD0D   . 59            POP ECX

0053CD0E   . 8B18          MOV EBX,DWORD PTR DS:\[EAX\]

0053CD10   . FF93 80000000 CALL DWORD PTR DS:\[EBX+80\]                对解码后的数据进一步的变换

0053CD16   . 5F            POP EDI

0053CD17   . 5E            POP ESI

0053CD18   . 5B            POP EBX

0053CD19   . C3            RETN

base64 解码：

0053C0D4  /$ 56            PUSH ESI

0053C0D5  |. 57            PUSH EDI

0053C0D6  |. 55            PUSH EBP

0053C0D7  |. 83C4 F0       ADD ESP,-10

0053C0DA  |. 890C24        MOV DWORD PTR SS:\[ESP\],ECX

0053C0DD  |. 8BF8          MOV EDI,EAX

0053C0DF  |. 895424 08     MOV DWORD PTR SS:\[ESP+8\],EDX

0053C0E3  |. 33F6          XOR ESI,ESI

0053C0E5  |. 33ED          XOR EBP,EBP

0053C0E7  |. 33C0          XOR EAX,EAX

0053C0E9  |. 8B1424        MOV EDX,DWORD PTR SS:\[ESP\]

0053C0EC  |. 85D2          TEST EDX,EDX

0053C0EE  |. 79 03         JNS SHORT EsoftPro.0053C0F3

0053C0F0  |. 83C2 03       ADD EDX,3

0053C0F3  |> C1FA 02       SAR EDX,2

0053C0F6  |. 85D2          TEST EDX,EDX

0053C0F8  |. 0F8E 02010000 JLE EsoftPro.0053C200

0053C0FE  |. 895424 0C     MOV DWORD PTR SS:\[ESP+C\],EDX

0053C102  |> BA 04000000   /MOV EDX,4                       ;

0053C107  |. 8D4424 04     |LEA EAX,DWORD PTR SS:\[ESP+4\]

0053C10B  |> 33C9          |/XOR ECX,ECX                     ;

0053C10D  |. 8A0C37        ||MOV CL,BYTE PTR DS:\[EDI+ESI\]

0053C110  |. 83F9 3D       ||CMP ECX,3D                      ;  Switch (cases 2B..7A)

0053C113  |. 7F 14         ||JG SHORT EsoftPro.0053C129

0053C115  |. 74 52         ||JE SHORT EsoftPro.0053C169

0053C117  |. 83E9 2B       ||SUB ECX,2B

0053C11A  |. 74 43         ||JE SHORT EsoftPro.0053C15F

0053C11C  |. 83E9 04       ||SUB ECX,4

0053C11F  |. 74 43         ||JE SHORT EsoftPro.0053C164

0053C121  |. 49            ||DEC ECX

0053C122  |. 83E9 0A       ||SUB ECX,0A

0053C125  |. 72 2B         ||JB SHORT EsoftPro.0053C152

0053C127  |. EB 43         ||JMP SHORT EsoftPro.0053C16C

0053C129  |> 83C1 BF       ||ADD ECX,-41

0053C12C  |. 83E9 1A       ||SUB ECX,1A

0053C12F  |. 72 0A         ||JB SHORT EsoftPro.0053C13B

0053C131  |. 83C1 FA       ||ADD ECX,-6

0053C134  |. 83E9 1A       ||SUB ECX,1A

0053C137  |. 72 0C         ||JB SHORT EsoftPro.0053C145

0053C139  |. EB 31         ||JMP SHORT EsoftPro.0053C16C

0053C13B  |> 8A0C37        ||MOV CL,BYTE PTR DS:\[EDI+ESI\]    ;  Cases 41 ('A'),42 ('B'),43 ('C'),44 ('D'),45 ('E'),46 ('F'),47 ('G'),48 ('H'),49 ('I'),4A ('J'),4B ('K'),4C ('L'),4D ('M'),4E ('N'),4F ('O'),50 ('P'),51 ('Q'),52 ('R'),53 ('S'),54 ('T')... of switch 0053C110

0053C13E  |. 80E9 41       ||SUB CL,41

0053C141  |. 8808          ||MOV BYTE PTR DS:\[EAX\],CL

0053C143  |. EB 27         ||JMP SHORT EsoftPro.0053C16C

0053C145  |> 8A0C37        ||MOV CL,BYTE PTR DS:\[EDI+ESI\]    ;  Cases 61 ('a'),62 ('b'),63 ('c'),64 ('d'),65 ('e'),66 ('f'),67 ('g'),68 ('h'),69 ('i'),6A ('j'),6B ('k'),6C ('l'),6D ('m'),6E ('n'),6F ('o'),70 ('p'),71 ('q'),72 ('r'),73 ('s'),74 ('t')... of switch 0053C110

0053C148  |. 80E9 61       ||SUB CL,61

0053C14B  |. 80C1 1A       ||ADD CL,1A

0053C14E  |. 8808          ||MOV BYTE PTR DS:\[EAX\],CL

0053C150  |. EB 1A         ||JMP SHORT EsoftPro.0053C16C

0053C152  |> 8A0C37        ||MOV CL,BYTE PTR DS:\[EDI+ESI\]    ;  Cases 30 ('0'),31 ('1'),32 ('2'),33 ('3'),34 ('4'),35 ('5'),36 ('6'),37 ('7'),38 ('8'),39 ('9') of switch 0053C110

0053C155  |. 80E9 30       ||SUB CL,30

0053C158  |. 80C1 34       ||ADD CL,34

0053C15B  |. 8808          ||MOV BYTE PTR DS:\[EAX\],CL

0053C15D  |. EB 0D         ||JMP SHORT EsoftPro.0053C16C

0053C15F  |> C600 3E       ||MOV BYTE PTR DS:\[EAX\],3E         ;  Case 2B ('+') of switch 0053C110

0053C162  |. EB 08         ||JMP SHORT EsoftPro.0053C16C

0053C164  |> C600 3F       ||MOV BYTE PTR DS:\[EAX\],3F         ;  Case 2F ('/') of switch 0053C110

0053C167  |. EB 03         ||JMP SHORT EsoftPro.0053C16C

0053C169  |> C600 FF       ||MOV BYTE PTR DS:\[EAX\],0FF        ;  Case 3D ('=') of switch 0053C110

0053C16C  |> 46            ||INC ESI                             ;  Default case of switch 0053C110

0053C16D  |. 40            ||INC EAX

0053C16E  |. 4A            ||DEC EDX

0053C16F  |. 75 9A         |\JNZ SHORT EsoftPro.0053C10B

0053C171  |. 8A4424 04     |MOV AL,BYTE PTR SS:\[ESP+4\]

0053C175  |. C1E0 02       |SHL EAX,2

0053C178  |. 33D2          |XOR EDX,EDX

0053C17A  |. 8A5424 05     |MOV DL,BYTE PTR SS:\[ESP+5\]

0053C17E  |. C1EA 04       |SHR EDX,4

0053C181  |. 0AC2          |OR AL,DL

0053C183  |. 8B5424 08     |MOV EDX,DWORD PTR SS:\[ESP+8\]

0053C187  |. 88042A        |MOV BYTE PTR DS:\[EDX+EBP\],AL

0053C18A  |. 8D45 01       |LEA EAX,DWORD PTR SS:\[EBP+1\]

0053C18D  |. 8A5424 06     |MOV DL,BYTE PTR SS:\[ESP+6\]

0053C191  |. 80FA FF       |CMP DL,0FF

0053C194  |. 74 27         |JE SHORT EsoftPro.0053C1BD

0053C196  |. 807C24 07 FF  |CMP BYTE PTR SS:\[ESP+7\],0FF



# 汇编代码分析

```
0053C19B  |. 75 20          |JNZ SHORT EsoftPro.0053C1BD  

0053C19D  |. 8A4424 05      |MOV AL,BYTE PTR SS:[ESP+5]  

0053C1A1  |. C1E0 04        |SHL EAX,4  

0053C1A4  |. 81E2 FF000000  |AND EDX,0FF  

0053C1AA  |. C1EA 02        |SHR EDX,2  

0053C1AD  |. 0AC2           |OR AL,DL  

0053C1AF  |. 8B5424 08      |MOV EDX,DWORD PTR SS:[ESP+8]  

0053C1B3  |. 88442A 01      |MOV BYTE PTR DS:[EDX+EBP+1],AL  

0053C1B7  |. 8D45 02        |LEA EAX,DWORD PTR SS:[EBP+2]  

0053C1BA  |. 45             |INC EBP  

0053C1BB  |. EB 38          |JMP SHORT EsoftPro.0053C1F5  

0053C1BD  |> 80FA FF        |CMP DL,0FF  

0053C1C0  |. 74 33          |JE SHORT EsoftPro.0053C1F5  

0053C1C2  |. 8A4424 05      |MOV AL,BYTE PTR SS:[ESP+5]  

0053C1C6  |. C1E0 04        |SHL EAX,4  

0053C1C9  |. 81E2 FF000000 |AND EDX,0FF  

0053C1CF  |. C1EA 02        |SHR EDX,2  

0053C1D2  |. 0AC2           |OR AL,DL  

0053C1D4  |. 8B5424 08      |MOV EDX,DWORD PTR SS:[ESP+8]  

0053C1D8  |. 88442A 01      |MOV BYTE PTR DS:[EDX+EBP+1],AL  

0053C1DC  |. 8B4424 08      |MOV EAX,DWORD PTR SS:[ESP+8]  

0053C1E0  |. 8A5424 06      |MOV DL,BYTE PTR SS:[ESP+6]  

0053C1E4  |. C1E2 06        |SHL EDX,6  

0053C1E7  |. 0A5424 07      |OR DL,BYTE PTR SS:[ESP+7]  

0053C1EB  |. 885428 02      |MOV BYTE PTR DS:[EAX+EBP+2],DL  

0053C1EF  |. 8D45 03        |LEA EAX,DWORD PTR SS:[EBP+3]  

0053C1F2  |. 83C5 02        |ADD EBP,2  

0053C1F5  |> 45             |INC EBP  

0053C1F6  |. FF4C24 0C      |DEC DWORD PTR SS:[ESP+C]  

0053C1FA  |.⁰F85 02FFFFFF \\JNZ EsoftPro.0053C102  

0053C200  |> 83C4 10        |ADD ESP,10  

0053C203  |. 5D             |POP EBP  

0053C204  |. 5F             |POP EDI  

0053C205  |. 5E             |POP ESI  

0053C206  |\\. C3             |RETN  

3,  
```

## 进一步的变换

```
(0053CD10   . FF93 80000000 CALL DWORD PTR DS:[EBX+80])

00540CBC  /. 55              |PUSH EBP  
00540CBD  |. 8BEC            |MOV EBP,ESP  
00540CBF  |. 83C4 E8         |ADD ESP,-18  
00540CC2  |. 53              |PUSH EBX  
00540CC3  |. 56              |PUSH ESI  
00540CC4  |. 57              |PUSH EDI  
00540CC5  |. 894D FC         |MOV DWORD PTR SS:[EBP-4],ECX  
00540CC8  |. 8BD8            |MOV EBX,EAX  
00540CCA  |. 807B 30 00      |CMP BYTE PTR DS:[EBX+30],0  
00540CCE  |. 75 16           |JNZ SHORT EsoftPro.00540CE6  
00540CD0  |. B9 440D5400     |MOV ECX,EsoftPro.00540D44             ;  ASCII "Cipher not initialized"  
00540CD5  |. B2 01           |MOV DL,1  
00540CD7  |. A1 08C65300     |MOV EAX,DWORD PTR DS:[53C608]  
00540CDC  |. E8 A7C4ECFF     |CALL EsoftPro.0040D188  
00540CE1  |. E8 AA35ECFF     |CALL EsoftPro.00404290  
00540CE6  |> 8BF2            |MOV ESI,EDX  
00540CE8  |. 8B45 FC         |MOV EAX,DWORD PTR SS:[EBP-4]  
00540CEB  |. 8945 F8         |MOV DWORD PTR SS:[EBP-8],EAX  
00540CEE  |. 8B45 08         |MOV EAX,DWORD PTR SS:[EBP+8]  
00540CF1  |. 85C0            |TEST EAX,EAX  
00540CF3  |. 76 3E           |JBE SHORT EsoftPro.00540D33  
00540CF5  |. 8945 E8         |MOV DWORD PTR SS:[EBP-18],EAX  
00540CF8  |> 8A06            |MOV AL,BYTE PTR DS:[ESI]  
00540CFA  |. 8845 F7         |MOV BYTE PTR SS:[EBP-9],AL  
00540CFD  |. 8D4D EF         |LEA ECX,DWORD PTR SS:[EBP-11]  
00540D00  |. 8D53 40         |LEA EDX,DWORD PTR DS:[EBX+40]  
00540D03  |. 8BC3            |MOV EAX,EBX  
00540D05  |. 8B38            |MOV EDI,DWORD PTR DS:[EAX]  
00540D07  |. FF57 6C         |CALL DWORD PTR DS:[EDI+6C]               blowfish_en  
00540D0A  |. 8A06            |MOV AL,BYTE PTR DS:[ESI]  
00540D0C  |. 3245 EF         |XOR AL,BYTE PTR SS:[EBP-11]  
00540D0F  |. 8B55 F8         |MOV EDX,DWORD PTR SS:[EBP-8]  
00540D12  |. 8802            |MOV BYTE PTR DS:[EDX],AL  
00540D14  |. 8D53 40         |LEA EDX,DWORD PTR DS:[EBX+40]  
00540D17  |. 8D43 41         |LEA EAX,DWORD PTR DS:[EBX+41]  
00540D1A  |. B9 07000000     |MOV ECX,7  
00540D1F  |. E8 681CECFF     |CALL EsoftPro.0040298C  
00540D24  |. 8A45 F7         |MOV AL,BYTE PTR SS:[EBP-9]  
00540D27  |. 8843 47         |MOV BYTE PTR DS:[EBX+47],AL  
00540D2A  |. 46              |INC ESI  
00540D2B  |. FF45 F8         |INC DWORD PTR SS:[EBP-8]  
00540D2E  |. FF4D E8         |DEC DWORD PTR SS:[EBP-18]  
00540D31  |.⁷⁵ C5            |JNZ SHORT EsoftPro.00540CF8  
00540D33  |> 5F              |POP EDI  
00540D34  |. 5E              |POP ESI  
00540D35  |. 5B              |POP EBX  
00540D36  |. 8BE5            |MOV ESP,EBP  
00540D38  |. 5D              |POP EBP  
00540D39  |\\. C2 0400         |RETN 4  
```

分析这段代码知道变换过程是这样的：

设 `basedecode` 后得到的字符串为

`"1234"`

先对 `hex` 串 `'FA6A1510698C8467'` 运用 `blowfish` 加密得到 `hex` 串:

`'4D72933486AEF1F0'`

则 `ascii('1')⁰x4d=0x7c`，`0x7c` 就是变换后的结果。

然后把 `ascii('1')` 添加到 `'FA6A1510698C8467'` 后面，得到 `'FA6A1510698C846731'`

再对 `'6A1510698C846731'` 运用 `blowfish` 加密。

加密结果的的头两位再与 `ascii('2')` 异或，得到变换结果的第 2 位，

如此循环处理 `len(base6 解码后的字符串)` 次，依次连接变换后的结果，得到最后用来与特定字串

`"'作者的名字，软件信息，网站等等'+'网卡 mac'+'硬盘序列号'"` 比较的字串.

变换部分清楚了，那剩下的就是找到 `blowfish` 计算的 `key` 了.

通过对 `hex` 串 `'FA6A1510698C8467'` 地址设置内存断点跟踪，来到这儿：

```
005408CD  |. 8D4B 38        |LEA ECX,DWORD PTR DS:[EBX+38]  
005408D0  |. 8D53 38        |LEA EDX,DWORD PTR DS:[EBX+38]  
005408D3  |. 8BC3           |MOV EAX,EBX  
005408D5  |. 8B30           |MOV ESI,DWORD PTR DS:[EAX]  
005408D7  |. FF56 6C        |CALL DWORD PTR DS:[ESI+6C]          blowfish_en  
005408DA  |. 8BC3           |MOV EAX,EBX  
005408DC  |. 8B10           |MOV EDX,DWORD PTR DS:[EAX]  
005408DE  |. FF52 48        |CALL DWORD PTR DS:[EDX+48]  
005408E1  |. EB 16          |JMP SHORT EsoftPro.005408F9  
```

原来 `'FA6A1510698C8467'` 是用 `blowfish` 加密 64 位全零串得到的. 那剩下的就是找出 `key` 了。

对 `blowfish` 的 `pbox` 数据地址设置内存访问断点，来到这里：

```
005408A4  |. 8B4D FC        |MOV ECX,DWORD PTR SS:[EBP-4]       ; |  
005408A7  |. 8BC3           |MOV EAX,EBX                         ; |  
005408A9  |. E8 1EC0FFFF    |CALL EsoftPro.0053C8CC              ; \\检查密钥长度  
005408AE  |. 8BD7           |MOV EDX,EDI  
005408B0  |. 8B4D FC        |MOV ECX,DWORD PTR SS:[EBP-4]  
005408B3  |. 8BC3           |MOV EAX,EBX  
005408B5  |. 8B38           |MOV EDI,DWORD PTR DS:[EAX]  
005408B7  |. FF57 5C        |CALL DWORD PTR DS:[EDI+5C]          ;  密钥初始化  
005408BA  |. 85F6           |TEST ESI,ESI  
005408BC  |. 75 25          |JNZ SHORT EsoftPro.005408E3  
005408BE  |. 8D43 38        |LEA EAX,DWORD PTR DS:[EBX+38]  
005408C1  |. 33C9           |XOR ECX,ECX  
005408C3  |. BA 08000000    |MOV EDX,8  
005408C8  |. E8 EF28ECFF    |CALL EsoftPro.004031BC  
005408CD  |. 8D4B 38        |LEA ECX,DWORD PTR DS:[EBX+38]  
005408D0  |. 8D53 38        |LEA EDX,DWORD PTR DS:[EBX+38]  
```

得到 `key` 是:

`'BA0E57F2906C60C62BBB8C7EEBB53B0E'`

(`in hex`)

继续对这个 `key` 的地址设置内存断点，分析得到：

```
005406B1    . 8B53 38      |MOV EDX,DWORD PTR DS:[EBX+38]  
005406B4    . 8910         |MOV DWORD PTR DS:[EAX],EDX  
005406B6    . 8D83 8C000000 |LEA EAX,DWORD PTR DS:[EBX+8C]  
005406BC    . 8B53 34      |MOV EDX,DWORD PTR DS:[EBX+34]  
005406BF    . 8910         |MOV DWORD PTR DS:[EAX],EDX  
005406C1    . 8BC3         |MOV EAX,EBX  
005406C3    . E8 20F5FFFF  |CALL EsoftPro.0053FBE8           ;  md5  
005406C8    . 8BD6         |MOV EDX,ESI  
005406CA    . 8D43 40      |LEA EAX,DWORD PTR DS:[EBX+40]  
005406CD    . B9 10000000  |MOV ECX,10  
005406D2    . E8 B522ECFF  |CALL EsoftPro.0040298C  
```

加密 `key` 是通过对字符串 `'作者的名字’` 做 `md5` 计算得到的。

如果有兴趣，也可以看看机器码的计算过程，可以发现，

机器码是对 `'ghq1978'` 进行 `tiger` 计算，得到 `key`:

`'534E72C523DDF7A89079166D71563965547BB63B0B4FCDD4'`

(`in hex`)

然后用这个 `key`，字符串 `"'ghq1978'+'网卡 mac'+'硬盘序列号'"`

进行和 3 处一样的变换，把结果用 `base64` 编码输出就是软件显示的机器码.

整个授权文件的验证过程烦了点，主要是那些循环太讨厌了。

但看清楚了过程，要写注册机还是比较简单的。

水平有限，分析得不对的地方，敬请诸位大侠赐教！

`by ikki`

`4dd466d0e1e0cc123afbe504da2462ca`
