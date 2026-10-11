
# Lc3/Lc4 破解加注册机编写篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy6532.htm`](https://www.pediy.com/kssd/pediy06/pediy6532.htm)

作者: `KernelKiller`

使用工具: `W32Dasm` `DebuggerKiller1.1` (内部测试版) `vc++6.0`

系统平台: `Windows 2000 Server`

软件名字: 世界著名破解 `WINNT` 密码工具 `Lc3/Lc4` (从来都是它去破解别人..而今天却要被我给破了)

日期 : `2004-06-17`

破解声明: 作者只是教大家破解和从复杂的反汇编里面找注册算法并做出自己的注册机。注:破别人收费软件总是不太好(最好不要破中国人的).

个人声明: `Lc3/Lc4` 此软件已经是作者 `N` 年前破好,`N` 年前写好的注册机了. 为什么还要翻老帐重写教程？当年是用世界著名的内核调试器 `Softice`，可现在作者用的是自己设计的内核调试器 `DebuggerKiller1.1`(内部测试版)爆破软件。 感觉就是不一样!!!! 爽!!

## 内容

哎~！`Lc3/Lc4` 虽然是几年前的东西了,但现在依然是最强的破 `NT` 密码工具(速度一流)。可惜软件限制只能用 `15` 次,好东西只能用 `15` 次太少了,就让这个限制见鬼去吧!!! 作者我将带领你们一步一步从反汇编代码里找到爆破地方,和注册算法。 现在 `Lc5` 已经出来了，大家看了这篇文章就自己做个 `Lc5` 注册机吧。作者祝福你们！！

## 破解动态分析:Lc3

他的注册方法是，程序给出序列号，用户输入开锁码。 现在我们输入一串假开锁码`"KernelKiller"` ,选择 `OK`， 程序弹出错误对话框`"You have entered an invalid code,Please try again "` 注册失败。 好。大概注册过程分析就这样

加载内核调试 `DebuggerKiller1.1` (`Ctrl + Y`)

用命令 `addr 进程名` 加载到进程空间 ，为了让大家习惯采用和 `Softice` 一样命令。

设置断点 `bpx GetWindowTextA`

输入一串假开锁码`"KernelKiller"` ,选择 `OK`

好我的内核调试器截获断点。现在我们来到 `GetWindowTextA` 首地址，因为现在是在系统的 `GetWindowTextA` 函数里，所以我们要跳出这个函数进入 `Lc3` 代码，可惜我没有设计像 `Softice` 那样的 `F11` 跳出 `CALL`

只好找到 `RET` 在这里设置断，按 `F5` 跳来到这个断点 `RET`，单步跟踪，来到 `Lc3` 代码，并到了下面类似代码，我们就退出调试器，`X` 进行静态分析

## 破解静态分析:Lc3

现在进行静态分析，打开 `W32Dasm` 加载 `Lc3` 进行反汇编分析来到

```
:004201E7 8D8D00FFFFFF            LEA ECX, DWORD PTR [EBP+FFFFFF00]    ~~~~~~~~~~~~~~~~~~~~~~~~
:004201ED 53                      PUSH EBX
:004201EE E853E90300              CALL 0045EB46
:004201F3 8D8DA4FEFFFF            LEA ECX, DWORD PTR [EBP+FFFFFEA4]
:004201F9 8B85A4FEFFFF            MOV EAX, DWORD PTR [EBP+FFFFFEA4]
:004201FF 8B80B8000000            MOV EAX, DWORD PTR [EAX+000000B8]
:00420205 FFD0                    CALL EAX                              ; 不需要关心
:00420207 83F801                  CMP EAX, 00000001
:0042020A 0F859E000000            JNE 004202AE
:00420210 8D8504FFFFFF            LEA EAX, DWORD PTR [EBP+FFFFFF04]
:00420216 50                      PUSH EAX
:00420217 8B4DE0                  MOV ECX, DWORD PTR [EBP-20]
:0042021A E827E90300              CALL 0045EB46
:0042021F 8B45E4                  MOV EAX, DWORD PTR [EBP-1C]           ~~~~~~~~~~~~~~~~~~~~~~~~~
:00420222 8B880C010000            MOV ECX, DWORD PTR [EAX+0000010C];获得程序的序列号
:00420228 8D55C8                  LEA EDX, DWORD PTR [EBP-38] ;一个缓冲区 相当于 char cpBuf[BUF_MAX];用来保存真实的开锁码
:0042022B 52                      PUSH EDX  ;作为参数压栈 ;缓冲区
:0042022C 51                      PUSH ECX  ;作为参数压栈  ;序列号
:0042022D E8DED9FEFF              CALL 0040DC10  ;调用这个函数功能计算真实的开锁码。待会我才解释 GetUnLockCode
:00420232 83C408                  ADD ESP, 00000008  ;堆栈平衡
:00420235 8B45E4                  MOV EAX, DWORD PTR [EBP-1C]
:00420238 8B8810010000            MOV ECX, DWORD PTR [EAX+00000110]   ;获得我输入的开锁码
:0042023E 8D55C8                  LEA EDX, DWORD PTR [EBP-38]         ;获得程序真实的开锁码
:00420241 52                      PUSH EDX ;作为参数压栈  //我输入的开锁码
:00420242 51                      PUSH ECX ;作为参数压栈  //程序真实的开锁码
:00420243 E884700200              CALL 004472CC   ;调用这个函数功能比较 2 个参数是不是相等 相当于 strcmp
:00420248 83C408                  ADD ESP, 00000008 ;堆栈平衡
:0042024B 85C0                    TEST EAX, EAX     ;比较返回值 1 失败,0 成功
:0042024D 7533                    JNE 00420282      ;1 失败就跳 否则执行下面进入程序提示注册成功
:0042024F 8B4DE4                  MOV ECX, DWORD PTR [EBP-1C]         ~~~~~~~~~~~~~~~~~~~~~~~~~
:00420252 C7811401000000000000    MOV DWORD PTR [EBX+00000114], 00000000
                                  ;[EBX+00000114] 地址是 1 代表要注册,清 0 代表注册成功
:0042025C 8B8110010000            MOV EAX, DWORD PTR [ECX+00000110]
                                  ;[ECX+00000110] 地址是用户输入的开锁码
:00420262 50                      PUSH EAX
:00420263 6820486D00              PUSH 006D4820
:00420268 6800486D00              PUSH 006D4800
                                  ; 不需要关心
:0042026D E8C0D80400              CALL 0046DB32   ;写注册表
:00420272 33C0                    XOR EAX, EAX    ;清 0   作为对话框风格参数
:00420274 50                      PUSH EAX        ;对话框风格参数
:00420275 50                      PUSH EAX        ;对话框风格参数
:00420276 68C0496D00              PUSH 006D49C0   ;要显示的字符串
:0042027B E8009B0400              CALL 00469D80  ;弹出对话框
:00420280 EB0E                    JMP 00420290          ~~~~~~~~~~~~~~~~~~~~~~~

省略 ...............................

:0042024D(C)    ;是错误跳转到这里

:00420282 33C0                    XOR EAX, EAX
:00420284 50                      PUSH EAX  ;对话框风格参数
:00420285 50                      PUSH EAX  ;对话框风格参数
:00420286 68004A6D00              PUSH 006D4A00  ;要显示的字符串
:0042028B E8F09A0400              CALL 00469D80 ;调用这个函数抬出对话框 提示注册失败。
```

(`Ctrl + Y`)，设置 `Ring3` 断点 `Bpx 0042024B`

来到断点把 `EAX` 改成 `0` 注册成功

//好。到这里你应该知道怎么硬破解了。我就不多说啦。 下面我来研究注册算法，做自己的注册机

## 注册算法分析:Lc3

`call 0040DC10` 调用这个函数是获得真实的开锁码

`GetUnlockCode(char *cpSerial, char *cpUnlockCode)` 返回真实的开锁码到 `cpUnlockCode` 里

```
:0040DC10 55                      PUSH EBP
:0040DC11 8BEC                    MOV EBP, ESP
:0040DC13 83EC10                  SUB ESP, 00000010
:0040DC16 83C4F0                  ADD ESP, FFFFFFF0
:0040DC19 8D45F0                  LEA EAX, DWORD PTR [EBP-10]
:0040DC1C 890424                  MOV DWORD PTR [ESP], EAX
:0040DC1F 8B4508                  MOV EAX, DWORD PTR [EBP+08]
:0040DC22 40                      INC EAX
:0040DC23 89442404                MOV DWORD PTR [ESP+04], EAX
:0040DC27 C744240808000000        MOV [ESP+08], 00000008
:0040DC2F E8AC7F0300              CALL 00445BE0
:0040DC34 32C0                    XOR AL, AL
:0040DC36 8845F8                  MOV BYTE PTR [EBP-08], AL
:0040DC39 8D45F0                  LEA EAX, DWORD PTR [EBP-10]
:0040DC3C 890424                  MOV DWORD PTR [ESP], EAX
:0040DC3F C7442404C8FC6C00        MOV [ESP+04], 006CFCC8
:0040DC47 8D45FC                  LEA EAX, DWORD PTR [EBP-04]
:0040DC4A 89442408                MOV DWORD PTR [ESP+08], EAX
:0040DC4E E8D7840300              CALL 0044612A
:0040DC53 8B45FC                  MOV EAX, DWORD PTR [EBP-04]
:0040DC56 35C1985A71              XOR EAX, 715A98C1
:0040DC5B 8945FC                  MOV DWORD PTR [EBP-04], EAX
:0040DC5E 8B45FC                  MOV EAX, DWORD PTR [EBP-04]
:0040DC61 C1E805                  SHR EAX, 05
:0040DC64 8B55FC                  MOV EDX, DWORD PTR [EBP-04]
:0040DC67 C1E21B                  SHL EDX, 1B
:0040DC6A 03C2                    ADD EAX, EDX
:0040DC6C 8945FC                  MOV DWORD PTR [EBP-04], EAX
:0040DC6F 8B450C                  MOV EAX, DWORD PTR [EBP+0C]
:0040DC72 890424                  MOV DWORD PTR [ESP], EAX
:0040DC75 C7442404C0FC6C00        MOV [ESP+04], 006CFCC0
:0040DC7D 8B45FC                  MOV EAX, DWORD PTR [EBP-04]
:0040DC80 89442408                MOV DWORD PTR [ESP+08], EAX
```



:0040DC84 E8D5840300              CALL 0044615E  

:0040DC89 83C410                ADD ESP, 00000010  

:0040DC8C 33C0                  XOR EAX, EAX  

:0040DC8E C9                    LEAVE  

:0040DC8F C3                    RET  

现在是用 VC++6.0 写个自己的 Lc3 注册机。哎 N 年东西了，代码有点乱。保持原来的代码风格  

m_Edit1   EDIT 控件，`m_Edit1` 代表输入程序的序列号  

m_Edit2   EDIT 控件，`m_Edit2` 代表输出的开锁码  

```
void CLc3_4KeyDlg::OnChangeEdit1() 
{ 
    int HighSerial; 
    char Serial[20]; 
    char buf[20]; 
    char UnlockCode[20]; 
    DWORD dwSwitchSerial; 
    DWORD dwBakSwitchSerial; 
    UpdateData(TRUE); 
    memset(Serial,0,20); 
    memset(UnlockCode,0,20); 
    memset(buf,0,20); 
    m_Edit2=""; 
    UpdateData(FALSE); 
    sprintf(buf,"%s",m_Edit1); 
    strncpy(Serial,buf+1,8); 
    sscanf(Serial,"%08x",&dwSwitchSerial); 
    dwSwitchSerial = dwSwitchSerial ^ 0x715A98C1; 
    dwBakSwitchSerial = dwSwitchSerial; 
    _asm{ 
        pushad 
        mov eax,dwSwitchSerial 
        mov ecx,dwBakSwitchSerial 
        shl ecx,1Bh 
        shr eax, 5 
        add ecx, eax 
        mov dwSwitchSerial,ecx 
        popad 
    } 
    _snprintf(UnlockCode,4,"%04x",dwSwitchSerial); 
    HighSerial = dwSwitchSerial << 16; 
    _snprintf(UnlockCode+4,4,"%04x",HighSerial); 
    m_Edit2=UnlockCode; 
    UpdateData(FALSE); 
} 
```
/////////////////////////////////////////////////////////////////////////////////////////////////////  

破解分析:Lc4  

  和 Lc3 差不多.  

注册算法分析:Lc4  

call 00403EB0  调用这个函数是获得真实的开锁码    
                
GetUnlockCode(char *cpSerial,char *cpUnlockCode) 返回真实的开锁码到 cpUnlockCode 里  

GetUnlockCode: 

:00403EB0 83EC0C                SUB ESP, 0000000C  

:00403EB3 8B442410              MOV EAX, DWORD PTR [ESP+10]  

:00403EB7 6A08                  PUSH 00000008  

:00403EB9 40                    INC EAX  

:00403EBA 50                    PUSH EAX  

:00403EBB 8D4C2408              LEA ECX, DWORD PTR [ESP+08]  

:00403EBF 51                    PUSH ECX  

:00403EC0 E8AB1E0200            CALL 00425D70  

:00403EC5 8D54241C              LEA EDX, DWORD PTR [ESP+1C]  

:00403EC9 52                    PUSH EDX  

:00403ECA 8D442410              LEA EAX, DWORD PTR [ESP+10]  

:00403ECE 6888FA4600            PUSH 0046FA88  

:00403ED3 50                    PUSH EAX  

:00403ED4 C644242000            MOV [ESP+20], 00  

:00403ED9 E865240200            CALL 00426343  

:00403EDE 8B442428              MOV EAX, DWORD PTR [ESP+28]  

:00403EE2 3552AE376F            XOR EAX, 6F37AE52  

:00403EE7 8BC8                  MOV ECX, EAX  

:00403EE9 C1E11B                SHL ECX, 1B  

:00403EEC C1E805                SHR EAX, 05  

:00403EEF 03C8                  ADD ECX, EAX  

:00403EF1 51                    PUSH ECX  

:00403EF2 894C242C              MOV DWORD PTR [ESP+2C], ECX  

:00403EF6 8B4C2430              MOV ECX, DWORD PTR [ESP+30]  

:00403EFA 6880FA4600            PUSH 0046FA80  

:00403EFF 51                    PUSH ECX  

:00403F00 E8EC230200            CALL 004262F1  

:00403F05 33C0                  XOR EAX, EAX  

:00403F07 83C430                ADD ESP, 00000030  

:00403F0A C3                    RET  

现在是用 VC++6.0 写个自己的 Lc4 注册机。哎 N 年东西了，代码有点乱 保持原来的代码风格  

m_Edit3   EDIT 控件，`m_Edit3` 代表输入程序的序列号  

m_Edit4   EDIT 控件，`m_Edit4` 代表输出的开锁码  

```
void CLc3_4KeyDlg::OnChangeEdit3() 
{ 
    int HighSerial; 
    char Serial[20]; 
    char buf[20]; 
    char UnlockCode[20]; 
    DWORD dwSwitchSerial; 
    DWORD dwBakSwitchSerial; 
    UpdateData(TRUE); 
    memset(Serial,0,20); 
    memset(UnlockCode,0,20); 
    memset(buf,0,20); 
    m_Edit4=""; 
    UpdateData(FALSE); 
    sprintf(buf,"%s",m_Edit3); 
    strncpy(Serial,buf+1,8); 
    sscanf(Serial,"%08x",&dwSwitchSerial); 
    dwSwitchSerial = dwSwitchSerial ^ 0x6F37AE52; 
    dwBakSwitchSerial = dwSwitchSerial; 
    _asm{ 
        pushad 
        mov eax,dwSwitchSerial 
        mov ecx,dwBakSwitchSerial 
        shl ecx,1Bh 
        shr eax, 5 
        add ecx, eax 
        mov dwSwitchSerial,ecx 
        popad 
    } 
    _snprintf(UnlockCode,4,"%04x",dwSwitchSerial); 
    HighSerial = dwSwitchSerial << 16; 
    _snprintf(UnlockCode+4,4,"%04x",HighSerial); 
    m_Edit4=UnlockCode; 
    UpdateData(FALSE); 
} 
```
/////////////////////////////////////////////////////////////////////////////////////////////////////

4dd466d0e1e0cc123afbe504da2462ca
