
# 是男人就上一百层

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-110.htm`](https://www.pediy.com/kssd/pediy08/pediy8-110.htm)

;===========================================分析说明====================================================

【软件名称】是男人就上一百层

【下载地址】单击下载附件

【应用平台】Win9x/NT/2000/XP

【软件大小】503K

【软件限制】未注册时，没有任何功能限制。

【保护方式】序列号

【破 解 者】neohost

【破解难度】1/10

【破解声明】只是为了熟悉逆向工程，所以注册机源码及注册检验模拟源码仅研究用请勿用于提供破解下载。

【破解工具】`W32DASM`,`ResHacker`,Windows 自带计算器

【注册机下载】仅提供源代码，支持共享软件

【软件简介】魔王转世你从地狱救到你的朋友,但是现在你要从地狱出来。

;============================================分析步骤===================================================

【分析过程】

这仅仅是个静态分析，没有调试。

1、 直接用 `w32dasm` 反汇编，无壳；

2、 在 `ResHacker` 看到注册窗口的注册码编辑框的资源 ID 为 `1003D`==`03EBH`

3、 在 `W32DASM` 查找输入函数 `GetWindowText` 和 `GetDlgItemText`,顺利地找到了 `GetDlgItemText`,

    找到 `nIDDlgItem`（要获取字符串的控件的 ID）参数为 `03EB` 的那一个,这个 `GetDlgItemText` 应该就是获取输入的注册码的地方了.

;======================================================================================================

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040685B(C)

|

```
:00406766 6800010000           push 00000100                  ;nMaxCount=100H
:0040676B 8D85FCFDFFFF         lea eax, dword ptr [ebp+FFFFFDFC]     ;eax=ebp-204H
:00406771 50                  push eax                           ;lpString=ebp-204H,注册码地址
                                                                     ;此后 lpString 即代表注册码地址,不再说明
```

* Possible Reference to Dialog: `DialogID_008C`, `CONTROL_ID`:03EB, ""

```
:00406772 68EB030000         push 000003EB                  ;nIDDlgItem=03EB (user name edit)
:00406777 8B4508             mov eax, dword ptr [ebp+08]
:0040677A 50                  push eax                         ;hDlg=eax=[ebp+08]
```

* Reference To: `USER32.GetDlgItemTextA`, Ord:00EFh

```
:0040677B FF1568744100       Call dword ptr [00417468]         ;GetDlgItemTextA([ebp+08],03EB,ebp-204H,100H);
:00406781 8D85FCFDFFFF       lea eax, dword ptr [ebp+FFFFFDFC]     ;eax=ebp-204H,即=lpString
:00406787 50                  push eax                           
:00406788 E8C0010000         call 0040694D                      ;eax=0040694D(lpString)
                                                                     ;调用函数 0040694D,1 个参数为注册码，返回值放入 eax
:0040678D 83C404             add esp, 00000004
:00406790 85C0               test eax, eax
:00406792 0F8537000000       jne 004067CF                       ;若 eax!=0,则跳到 004067CF,改这儿暴破应该也行的.
```

............................

............................                             ;若 eax 等于 0 则

* Reference To: `USER32.LoadStringA`, Ord:0186h

```
:004067AC FF151C744100       Call dword ptr [0041741C]         ;LoadString(失败)
```

............................

* Reference To: `USER32.MessageBoxA`, Ord:0197h

```
:004067C4 FF1518744100       Call dword ptr [00417418]         ;MessageBox(失败)
```

............................

;======================================================================================================

4、 由 `00406788` 到 `00406792` 可推测函数 `0040694D` 是注册码验证函数的,参数类型为 `char*` ,返回值为 `bool`.

    暂时推测为 `bool RegStringCheck(char*)`，进入 CALL:

;======================================================================================================

```
:00406788 E8C0010000        call 0040694D 即  eax=RegStringCheck(lpString)
```

* Referenced by a CALL at Addresses:

|:0040599F   , :00405ECA   , :00406788   

|

```
:0040694D 55                  push ebp
:0040694E 8BEC                 mov ebp, esp
:00406950 83EC04               sub esp, 00000004                 ;定义局部变量 var
:00406953 53                  push ebx
:00406954 56                  push esi
:00406955 57                  push edi
:00406956 8B4508               mov eax, dword ptr [ebp+08]         ;eax 指向第一个参数即 lpString
:00406959 33C9                 xor ecx, ecx
:0040695B 8A4807               mov cl, byte ptr [eax+07]           ;cl=lpString[7],即取注册码第 8 个字符到 cl.
:0040695E 85C9                 test ecx, ecx
:00406960 0F8407000000         je 0040696D                          ;如果 ecx 为 0 则跳到 0040696d
:00406966 33C0                 xor eax, eax                          
:00406968 E945010000           jmp 00406AB2                          ;否则跳到 00406AB2,函数结束并返回 eax=0,注册失败
                                                                     ;由此判断，注册码的长度不能大于 7 位. 
```

;======================================================================================================

5、 由 `lpString[7]`!=0 则注册失败可知，第 8 个字符必须为结束符即，注册码长不得大于 7 位.

;======================================================================================================

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406960(C)

|

```
:0040696D C745FC00000000      mov [ebp-04], 00000000               ;ebp-04 即局部变量置为 0，即 var=0
:00406974 E903000000          jmp 0040697C                          ;跳到 0040697C 执行
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004069AC(U)

|

```
:00406979 FF45FC             inc [ebp-04]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406974(U)

|

```
:0040697C 837DFC07           cmp dword ptr [ebp-04], 00000007     
:00406980 0F8D2B000000       jnl 004069B1                          ;假如 var>=7,则跳到 004069B1
:00406986 8B45FC             mov eax, dword ptr [ebp-04]          ;eax=var;
:00406989 8B4D08             mov ecx, dword ptr [ebp+08]          ;ecx=lpString
:0040698C 8A0408             mov al, byte ptr [eax+ecx]           ;al=[ecx+eax]
                                                                     ;即 al=lpString[var],取注册码第 var+1 个字符                                              
:0040698F 50                  push eax
:00406990 E822010000         call 00406AB7                         ;eax=00406ab7(lpString[var])
```

;======================================================================================================

6、 由此推测函数 `00406ab7` 的参数为 `char`,假设为 `00406AB7(char cReg)`，进入函数分析

;======================================================================================================

* Referenced by a CALL at Addresses:

|:00406990   , :004069B8   , :004069CB   , :004069EB   , :00406A06   

|:00406A19   , :00406A3A   , :00406A55   , :00406A67   , :00406A88   

|

```
:00406AB7 55                  push ebp
:00406AB8 8BEC                 mov ebp, esp
:00406ABA 53                  push ebx
:00406ABB 56                  push esi
:00406ABC 57                  push edi
:00406ABD 33C0                 xor eax, eax
:00406ABF 8A4508               mov al, byte ptr [ebp+08]            ;ebp+08 即参数字符 cReg 的地址,字符放入 al,此次调用，cReg=lpString[var]
:00406AC2 83F861               cmp eax, 00000061                    
:00406AC5 0F8C0B000000         jl 00406AD6                          ;若 cReg<'a',则直接跳到 00406AD6
:00406ACB 33C0                 xor eax, eax                          
:00406ACD 8A4508               mov al, byte ptr [ebp+08]            
:00406AD0 83E820               sub eax, 00000020
:00406AD3 884508               mov byte ptr [ebp+08], al            ;否则将 cReg 减去 20H 再执行 00406AD6
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406AC5(C)

|



7. 根据上面分析函数 `00406AB7` 的返回值类型为 `int` 型，参数类型为 `char`，作用是把注册码的某个字符变形。用 C 语言模拟：

```c
int DealRegString(char cReg)
{
    if (cReg >= 'a')
        cReg -= 0x20; // 32
    if (cReg >= 'A')
        cReg -= 7;
    cReg -= 0x30; // 48
    return cReg;
}
```

为方便，此后所有的 `00406AB7` 都用 `DealRegString` 作替换。`DealRegString` 结束返回到前面的 `RegStringCheck(lpString)` 中。

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004069AC(C)
|
:00406979 FF45FC          inc [ebp-04]                  ; var 增加 1，与后面的分析结合知 var 为循环的计数器
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406974(U)
|
:0040697C 837DFC07        cmp dword ptr [ebp-04], 00000007
:00406980 0F8D2B000000    jnl 004069B1                  ; 假如 var>=7,则循环结束,跳到 004069B1
:00406986 8B45FC          mov eax, dword ptr [ebp-04]   ; eax=var;
:00406989 8B4D08          mov ecx, dword ptr [ebp+08]   ; ecx=lpString
:0040698C 8A0408          mov al, byte ptr [eax+ecx]    ; al=lpStrng[var],取注册码第 var+1 个字符
:0040698F 50              push eax
:00406990 E822010000      call 00406AB7                 ; eax=DealRegString(lpString[var])
                                                            ; 对注册码的第 var+1 个字符进行处理，结果放入 eax
:00406995 83C404          add esp, 00000004
:00406998 33C9            xor ecx, ecx
:0040699A 8AC8            mov cl, al                    ; cl=al,函数 DealRegString(char)的返回值保存到 cl
:0040699C 83F924          cmp ecx, 00000024
:0040699F 0F8E07000000    jle 004069AC                  ; 如果 cl<=24H，则跳到 004069AC
:004069A5 33C0            xor eax, eax                  ; 否则，返回值设为 0
:004069A7 E906010000      jmp 00406AB2                  ; 返回 0，结束函数，注册失败.

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040699F(C)
|
:004069AC E9C8FFFFFF      jmp 00406979                  ; 跳到 00406979 循环
```

8. 从上面一段代码 `00406979` 到 `004069AC` 的分析，循环对注册码的 7 个字符用函数 `DealRegString(char)` 进行处理。若某个字符的处理结果大于 `24H`，则注册码无效，注册失败。

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406980(C)
|
:004069B1 8B4508          mov eax, dword ptr [ebp+08]   ; eax=lpString,注册码地址放到 eax
:004069B4 8A4005          mov al, byte ptr [eax+05]    ; al=lpString[5],第 6 个字符放入 al
:004069B7 50              push eax
:004069B8 E8FA000000      call DealRegString(char)
:004069BD 83C404          add esp, 00000004
:004069C0 33DB            xor ebx, ebx
:004069C2 8AD8            mov bl, al                    ; 以上几句的作用 bl＝DealRegString(lpString[5]),处理第 6 个字符,结果放入 bl
:004069C4 8B4508          mov eax, dword ptr [ebp+08]
:004069C7 8A4002          mov al, byte ptr [eax+02]
:004069CA 50              push eax
:004069CB E8E7000000      call DealRegString(char)
:004069D0 83C404          add esp, 00000004
:004069D3 33C9            xor ecx, ecx
:004069D5 8AC8            mov cl, al                    ; 以上几句的作用 cl＝DealRegString(lpString[2]),处理第 3 个字符,结果放入 cl
* Possible Reference to String Resource ID=00036: "1 M"
|
:004069D7 BE24000000      mov esi, 00000024             ; esi=24H
:004069DC 8D44591C        lea eax, dword ptr [ecx+2*ebx+1C] ; eax=ecx+2*ebx+1CH
:004069E0 99              cdq                           ; 用 eax 的符号位填充 edx
:004069E1 F7FE            idiv esi                      ; (edx:eax)/esi
:004069E3 8BDA            mov ebx, edx                  ; 余数放入 ebx,
                                                            ; 以上几句作用 ecx+2*ebx+1CH 除以 24H,余数放入 ebx.
:004069E5 8B4508          mov eax, dword ptr [ebp+08]
:004069E8 8A00            mov al, byte ptr [eax]
:004069EA 50              push eax
:004069EB E8C7000000      call DealRegString(char)
:004069F0 83C404          add esp, 00000004
:004069F3 33C9            xor ecx, ecx
:004069F5 8AC8            mov cl, al                    ; 以上几句的作用 cl＝DealRegString(lpString[0]),处理第 1 个字符,结果放入 cl
:004069F7 3BD9            cmp ebx, ecx
:004069F9 0F85AC000000    jne 00406AAB                  ; 若 ebx!=ecx，则跳到 00406AAB,即函数返回 0,函数结束，注册失败
:004069FF 8B4508          mov eax, dword ptr [ebp+08]
:00406A02 8A4004          mov al, byte ptr [eax+04]
:00406A05 50              push eax
:00406A06 E8AC000000      call DealRegString(char)
:00406A0B 83C404          add esp, 00000004
:00406A0E 33DB            xor ebx, ebx
:00406A10 8AD8            mov bl, al                    ; 以上几句的作用 bl＝DealRegString(lpString[4]),处理第 5 个字符,结果放入 bl
:00406A12 8B4508          mov eax, dword ptr [ebp+08]
:00406A15 8A4001          mov al, byte ptr [eax+01]
:00406A18 50              push eax
:00406A19 E899000000      call DealRegString(char)
:00406A1E 83C404          add esp, 00000004
:00406A21 33C9            xor ecx, ecx
:00406A23 8AC8            mov cl, al                    ; 以上几句的作用 cl＝DealRegString(lpString[1]),处理第 2 个字符,结果放入 cl
* Possible Reference to String Resource ID=00036: "1 M"
|
:00406A25 BE24000000      mov esi, 00000024             ; 此句到 00406A71 与 004069D7 到 00406A23 极为类似
:00406A2A 8D44591C        lea eax, dword ptr [ecx+2*ebx+1C] ; 以后只作简单说明
```



### 机器码分析

:00406A2E `99`  `cdq`

:00406A2F `F7FE`  `idiv` `esi`

:00406A31 `8BDA`  `mov` `ebx,` `edx` ;以上几句用 `ecx+2*ebx+1CH` 除以 `24H`,余数放入 `ebx`.

:00406A33 `8B4508`  `mov` `eax,` `dword ptr [ebp+08]`

:00406A36 `8A4006`  `mov` `al,` `byte ptr [eax+06]`

:00406A39 `50`  `push` `eax`

:00406A3A `E878000000`  `call` `DealRegString(char)`

:00406A3F `83C404`  `add` `esp,` `00000004`

:00406A42 `33C9`  `xor` `ecx,` `ecx`

:00406A44 `8AC8`  `mov` `cl,` `al` ;以上几句的作用 `cl＝DealRegString(lpString[6])`,处理第 7 个字符,结果放入 `cl`

:00406A46 `3BD9`  `cmp` `ebx,` `ecx`

:00406A48 `0F855D000000`  `jne` `00406AAB` ;若 `ebx` 不等于 `ecx`，则函数返回 0，注册失败

\* Possible Reference to String Resource `ID=00036`: `"1 M"`

:00406A4E `8B4508`  `mov` `eax,` `dword ptr [ebp+08]`

:00406A51 `8A4006`  `mov` `al,` `byte ptr [eax+06]`

:00406A54 `50`  `push` `eax`

:00406A55 `E85D000000`  `call` `DealRegString(char)`

:00406A5A `83C404`  `add` `esp,` `00000004`

:00406A5D `33DB`  `xor` `ebx,` `ebx`

:00406A5F `8AD8`  `mov` `bl,` `al` ;以上几句的作用 `bl＝DealRegString(lpString[6])`,处理第 7 个字符,结果放入 `bl`

:00406A61 `8B4508`  `mov` `eax,` `dword ptr [ebp+08]`

:00406A64 `8A00`  `mov` `al,` `byte ptr [eax]`

:00406A66 `50`  `push` `eax`

:00406A67 `E84B000000`  `call` `DealRegString(char)`

:00406A6C `83C404`  `add` `esp,` `00000004`

:00406A6F `33C9`  `xor` `ecx,` `ecx`

:00406A71 `8AC8`  `mov` `cl,` `al` ;以上几句的作用 `cl＝DealRegString(lpString[0])`,处理第 1 个字符,结果放入 `cl`

\* Possible Reference to String Resource `ID=00001`: `"NS-TOWE/q 玱?"`

:00406A73 `BE24000000`  `mov` `esi,` `00000024`

:00406A78 `8D44591C`  `lea` `eax,` `dword ptr [ecx+2*ebx+1C]`

:00406A7C `99`  `cdq`

:00406A7D `F7FE`  `idiv` `esi`

:00406A7F `8BDA`  `mov` `ebx,` `edx` ;以上几句作用 `ecx+2*ebx+1CH` 除以 `24H`,余数放入 `ebx`.

:00406A81 `8B4508`  `mov` `eax,` `dword ptr [ebp+08]`

:00406A84 `8A4003`  `mov` `al,` `byte ptr [eax+03]`

:00406A87 `50`  `push` `eax`

:00406A88 `E82A000000`  `call` `DealRegString(char)`

:00406A8D `83C404`  `add` `esp,` `00000004`

:00406A90 `33C9`  `xor` `ecx,` `ecx`

:00406A92 `8AC8`  `mov` `cl,` `al` ;以上几句的作用 `cl＝DealRegString(lpString[3])`,处理第 4 个字符,结果放入 `cl`

:00406A94 `3BD9`  `cmp` `ebx,` `ecx`

:00406A96 `0F850F000000`  `jne` `00406AAB` ;若 `ebx` 不等于 `ecx` 则函数返回 0，注册失败

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:004069F9(C), :00406A48(C), :00406A96(C)`

:00406A9C `B801000000`  `mov` `eax,` `00000001` ;返回值设为 1,注册成功。函数返回 `bool` 类型

:00406AA1 `E90C000000`  `jmp` `00406AB2`

:00406AA6 `E907000000`  `jmp` `00406AB2`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00406968(U), :004069A7(U), :00406AA1(U), :00406AA6(U), :00406AAD(U)`

:00406AAB `33C0`  `xor` `eax,` `eax` ;返回值设为 0,注册失败

:00406AAD `E900000000`  `jmp` `00406AB2`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00406968(U), :004069A7(U), :00406AA1(U), :00406AA6(U), :00406AAD(U)`

:00406AB2 `5F`  `pop` `edi` ;以下几句，函数返回 `eax`,函数结束

:00406AB3 `5E`  `pop` `esi`

:00406AB4 `5B`  `pop` `ebx`

:00406AB5 `C9`  `leave`

:00406AB6 `C3`  `ret`

;======================================================================================================

### 9. 注册码检验函数的 C 语言模拟

由此，注册码检验函数已经分析出来，用 C 语言模拟出来

```c
bool RegStringCheck(char* strReg) {
    int iVar;        
    int al,bl,cl; //模拟寄存器
    if(*(strReg+7))
        return false
    iVar=0;
    while(iVar<7) {
        if(DealRegChar(strReg[iVar])>0x24)
            return false;
        iVar++;
    }
    bl=DealRegChar(strReg[5]);
    cl=DealRegChar(strReg[2]);
    bl=(cl+2*bl+0x1C)%0x24;
    cl=DealRegChar(strReg[0]);
    if(bl!=cl)
        return false;
    bl=DealRegChar(strReg[4]);
    cl=DealRegChar(strReg[1]);
    bl=(cl+2*bl+0x1C)%0x24;
    cl=DealRegChar(strReg[6]);
    if(bl!=cl)
        return false;
    bl=DealRegChar(strReg[6]);
    cl=DealRegChar(strReg[0]);
    bl=(cl+2*bl+0x1C)%0x24;
    cl=DealRegChar(strReg[3]);
    if(bl!=cl)
        return false;
    return true;
}

其中的 DealRegChar 上面已经分析出来了。

int DealRegString(char cReg) {
    if(cReg>='a')
        cReg-=0x20; //32
    if(cReg>='A')
        cReg-=7;
    cReg-=0x30;     //48
    return cReg;
}
```

;======================================================================================================

### 10. 注册机编写与算法分析

最后一步，写注册机

由 `RegStringCheck` 中的代码：

```c
if(DealRegChar(strReg[iVar])>0x24)
    return false;
```

知 `DealRegChar` 返回值必须<=24H

如果 `cReg>='a'`，则 `cReg-20H-30H-7<=24H====>cReg<=7BH`

如果 `cReg>='A'`，则 `cReg-30H-7<=24H=====>cReg<=5BH`

如果 `cReg<'A'`，则 `cReg-30H<=24H=====>cReg<=54H='T'`

`00H-->'T'`和`'A'-->'a'`和`'a'--->7BH` 都是满足条件的

即从 `00H-->7BH` 都满足条件。

如果要保证返回值>=0 的话，还有 `cReg>=30H='0'`

由 `DealRegString` 的代码知，此函数忽略大小写

并且函数 `RegStringCheck` 只判断 `DealRegString` 的结果

所以整个验证过程中，是忽略大小写的

所以为了方便只取 `30H--->5BH`

它们的 `DealRegChar` 函数返回值从 0 到 24H(0 到 36)

设注册码七个字符处理后的 7 个返回值为数组 `x[7]`

则,则数组 `x[7]`满足

```c
(2*x[5]+x[2]+28)%36==x[0];
(2*x[4]+x[1]+28)%36==x[6];
(2*x[6]+x[0]+28)%36==x[3];
x0,x1,x2,x3,x4,x5,x6<=36;
x0,x1,x2,x3,x4,x5,x6>=0;
```

我找不出更好的算法，只好穷取

```c
#include <stdio.h>

void RegGen() {
    int i[7],j[7],k;
    for(i[0]=0;i[0]<=36;i[0]++) {
        for(i[1]=0;i[1]<=36;i[1]++) {
            for(i[2]=0;i[2]<=36;i[2]++) {
                for(i[3]=0;i[3]<=36;i[3]++) {
                    for(i[4]=0;i[4]<=36;i[4]++) {
                        for(i[5]=0;i[5]<=36;i[5]++) {
                            for(i[6]=0;i[6]<=36;i[6]++) {
                                if((2*i[5]+i[2]+28)%36==i[0]&&
                                (2*i[4]+i[1]+28)%36==i[6]&&
                                (2*i[6]+i[0]+28)%36==i[3])
                                {
                                    for(k=0;k<7;k++) {
                                        if(i[k]<=16)
                                            j[k]=i[k]+48;
                                        else
                                            j[k]=i[k]+55;
                                    }
                                    printf("%c%c%c%c%c%c%c\n",j[0],j[1],j[2],j[3],j[4],j[5],j[6]);
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}

int main( int argc, char *argv[] ) {
    RegGen();
    return 0;
}
```

这个算法不能产生所有的注册码。仅能产生从 30H,到 80H 之间的注册码

其中，从`'a'`到 80H 之间的不需要参与穷举，只需要把生成的注册码中从`'A'`-到 60H 这间的字符+20H 即可

如注册码 `00<OLYY`，从`'A'`到 60H 间的字符有`'O'`,`'L'`,`'Y'`,`'Y'`，`'O'+20H='o'`,`'L'+20H='l'`,`'Y'+20H='y'`



所以`00<oLYY`,`00<oLyy`...都是合法的注册码。类似的`'\'[']+20H='{'`,`'\]'+20H='}'`.......

### 穷举法说明
穷举是最没有技术含量的。说它是注册机都有点过分。哪个兄弟有更好的注册机算法请一定要写出来。以下是几个生成的注册码：

- `00<O3YY`
- `00<O<@@`
- `00<O<Y@`
- `00<OL@Y`
- `00<OLYY`
- `00<OU@@`
- `00<OUY@`
- `00<S4@0`
- `00<S4Y0`

### C++ 注册机示例
附网友 `xuanqiang` 早些年写的版本(C++)

```cpp
//keygenOf100up
//only 4 study and research

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <iostream.h>
#include <string.h>

//generate the key
char * keyGen()
{
  int keyNum[7];
  // initialize random generator
  srand ( time(NULL) );
  // generate key by some random numbers
  keyNum[2]=rand()%0x24;
  keyNum[5]=rand()%0x24;
  keyNum[0]=(keyNum[2]+keyNum[5]*2+0x1c)%0x24;
  keyNum[1]=rand()%0x24;
  keyNum[4]=rand()%0x24;
  keyNum[6]=(keyNum[1]+keyNum[4]*2+0x1c)%0x24;
  keyNum[3]=(keyNum[0]+keyNum[6]*2+0x1c)%0x24;
  //translate keyNum 2 ASCII
  char * sn=new char [8];
  sn[7]='\0';
  for(int i=0;i<7;i++)
  {
    if(keyNum[i]<10)
      sn[i]=keyNum[i]+0x30;
    else
      sn[i]=keyNum[i]+0x37;
  }
  return sn;
}

void main ()
{
  int i;
  char * sn;
  cout<<"keygen for 是男人就上 100 层"<<endl;
  cout<<"only 4 study and research"<<endl;
  cout<<"Press Enter to generate again, Ctrl+C to exit"<<endl;
  while (1)
  {
    sn=keyGen();
    cout<<sn;
    delete sn;
    cin.get()>>i;
  }
}
```

要重新测试注册码，只需要把`c:\windows\`下的`ns-tower.ini`改一下或者删掉就行了。

======THE END=======

;===========================================QQ：41086722================================================

欢迎交流

### 另一个 C++ 版本
附一个我早些年写的版本

```cpp
//keygenOf100up
//only 4 study and research

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <iostream.h>
#include <string.h>

//generate the key
char * keyGen()
{
  int keyNum[7];
  // initialize random generator
  srand ( time(NULL) );
  // generate key by some random numbers
  keyNum[2]=rand()%0x24;
  keyNum[5]=rand()%0x24;
  keyNum[0]=(keyNum[2]+keyNum[5]*2+0x1c)%0x24;
  keyNum[1]=rand()%0x24;
  keyNum[4]=rand()%0x24;
  keyNum[6]=(keyNum[1]+keyNum[4]*2+0x1c)%0x24;
  keyNum[3]=(keyNum[0]+keyNum[6]*2+0x1c)%0x24;
  //translate keyNum 2 ASCII
  char * sn=new char [8];
  sn[7]='\0';
  for(int i=0;i<7;i++)
  {
    if(keyNum[i]<10)
      sn[i]=keyNum[i]+0x30;
    else
      sn[i]=keyNum[i]+0x37;
  }
  return sn;
}

void main ()
{
  int i;
  char * sn;
  cout<<"keygen for 是男人就上 100 层"<<endl;
  cout<<"only 4 study and research"<<endl;
  cout<<"Press Enter to generate again, Ctrl+C to exit"<<endl;
  while (1)
  {
    sn=keyGen();
    cout<<sn;
    delete sn;
    cin.get()>>i;
  }
}
```

### 代码勘误与建议
进步了，谢楼主。指出楼主几个笔误：

1.
```asm
:004069C4 8B4508             mov eax, dword ptr [ebp+08]
:004069C7 8A4002             mov al, byte ptr [eax+02]
:004069CA 50                 push eax
:004069CB E8E7000000         call DealRegString(char)
:004069D0 83C404             add esp, 00000004
:004069D3 33C9               xor ecx, ecx
:004069D5 8AC8               mov cl, al ;以上几句的作用 cl＝DealRegString(lpString[5]),处理第 6 个字符,结果放入 cl
```
----这里应该是`DealRegString(lpString[2])`，处理第 3 个字符

2.
```c
bool RegStringCheck(char* strReg)
{
    int     iVar;
    int     al,bl,cl; //模拟寄存器
    if(strReg+1)    ----这个判断是不是应该为 if(*(strReg+7))
        return false    ----这里少了分号 :)
    ...
}
```

3.

如果`cReg>='a'`，则`cReg-20H-30H-7<=24H====>cReg<=81H` ----应为`7Bh`

如果`cReg>='A'`，则`cReg-30H-7<=24H=====>cReg<=61H='a'` ----应为`5Bh`

如果`cReg<'A'`，则`cReg-30H<=24H=====>cReg<=54H='T'`

呵呵，楼主有些粗心哦。
