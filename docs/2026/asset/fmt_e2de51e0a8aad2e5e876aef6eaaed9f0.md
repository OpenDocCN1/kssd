
# RealVNC Enterprice Edition 4.5算法分析

- **文章标题**：[原创]RealVNC Enterprice Edition 4.5算法分析
- **文章作者**：wuwenyao
- **作者邮箱**：wuwenyao.gm@gmail.com
- **软件名称**：RealVNC Enterprice Edition 4.5
- **下载地址**：[314K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2j5h3I4$3L8X3y4Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3](<https://bbs.kanxue.com/elink@0d0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2j5h3I4$3L8X3y4Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3>).
- **加壳方式**：未加壳
- **编写语言**：C
- **使用工具**：OllyDbg, peid, IDA
- **操作平台**：win32
- **软件介绍**：远程控制软件,有免费版本,企业版需要注册
- **作者声明**：本文只是分析算法，不提供注册机，如果要使用，请向开发商购买。

## 详细过程

最近用到远程控制，试了几个VNC版本，发现还是最原始的RealVNC好用，不过企业是收费的：（，花了几天分析了一下算法。

安装后，查看菜单，是使用`vncconfig.exe -noconsole -license`弹出注册码窗口，随便输入一个注册码，会弹出信息窗口提示注册错误。

OD载入`vncconfig.exe`，参数`-noconsole -license`，F9运行后，下`MessageBoxW`断点，F7,F8,CTRL+F9几次就可以找到验证注册码的部分了，这里不再详述过程。

下面来看看算法过程：

### 注册码验证主体

```asm
.text:004084B0 ; =============== S U B R O U T I N E =======================================

.text:004084B0
.text:004084B0
.text:004084B0 sub_4084B0      proc near               ; DATA XREF: .rdata:00459DC0o
.text:004084B0
.text:004084B0 var_2C          = dword ptr -2Ch
.text:004084B0 var_28          = dword ptr -28h
.text:004084B0 Args            = byte ptr -24h
.text:004084B0 var_20          = dword ptr -20h
.text:004084B0 Memory          = dword ptr -1Ch
.text:004084B0 var_18          = dword ptr -18h
.text:004084B0 var_14          = dword ptr -14h
.text:004084B0 var_10          = dword ptr -10h
.text:004084B0 var_C           = dword ptr -0Ch
.text:004084B0 var_4           = dword ptr -4
.text:004084B0
.text:004084B0                 push    -1
.text:004084B2                 push    offset sub_451258
.text:004084B7                 mov     eax, large fs:0
.text:004084BD                 push    eax
.text:004084BE                 sub     esp, 20h
.text:004084C1                 push    ebx
.text:004084C2                 push    ebp
.text:004084C3                 push    esi
.text:004084C4                 push    edi
.text:004084C5                 mov     eax, dword_472404
.text:004084CA                 xor     eax, esp
.text:004084CC                 push    eax
.text:004084CD                 lea     eax, [esp+40h+var_C]
.text:004084D1                 mov     large fs:0, eax
.text:004084D7                 mov     edi, ecx
.text:004084D9                 mov     [esp+40h+var_2C], edi
.text:004084DD                 xor     ebx, ebx          ;ebx=0
.text:004084DF                 push    ebx
.text:004084E0                 lea     ecx, [edi+3Ch]
.text:004084E3                 call    sub_41FDA0      ; 取注册码
.text:004084E8                 push    eax
.text:004084E9                 call    sub_416560      ; 注册码处理
.text:004084EE                 mov     ebp, eax        ; C结构指针,C[0]指向通过验证的注册码特征码
.text:004084F0                 add     esp, 4
.text:004084F3                 mov     [esp+40h+var_28], ebp ; C指针放入v_28
.text:004084F7                 mov     [esp+40h+Memory], ebx
.text:004084FB                 mov     [esp+40h+var_18], ebx
.text:004084FF                 cmp     ebp, ebx            ;C结构指针不为0,跳转,继续
.text:00408501                 mov     [esp+40h+var_4], ebx
.text:00408505                 jnz     short loc_40851C
.text:00408507                 push    -1
.text:00408509                 push    offset aTheSuppliedL_0 ; "The supplied license key is invalid. Yo"...
.text:0040850E                 lea     ecx, [esp+48h+Memory]
.text:00408512                 call    sub_40CBC0
.text:00408517                 jmp     loc_40870C
.text:0040851C ; ---------------------------------------------------------------------------
.text:0040851C
.text:0040851C loc_40851C:                             ; CODE XREF: sub_4084B0+55j
.text:0040851C                 mov     ecx, ebp
.text:0040851E                 call    sub_4165C0      ; C与当前时候校验
.text:00408523                 test    al, al                 ;返回不为0,跳转,继续
.text:00408525                 jz      short loc_40853C
.text:00408527                 push    -1
.text:00408529                 push    offset aTheSuppliedTri ; "The supplied trial license key has expi"...
.text:0040852E                 lea     ecx, [esp+48h+Memory]
.text:00408532                 call    sub_40CBC0
.text:00408537                 jmp     loc_4086F1
.text:0040853C ; ---------------------------------------------------------------------------
.text:0040853C
.text:0040853C loc_40853C:                             ; CODE XREF: sub_4084B0+75j
.text:0040853C                 mov     ecx, ds:dword_45AEF0   ;固定值,时间低位
.text:00408542                 mov     eax, ds:dword_45AEF4   ;固定值,时间高位
.text:00408547                 sub     ecx, 1E28500h   ; 366天(一年)
.text:0040854D                 sbb     eax, ebx
.text:0040854F                 cmp     eax, [ebp+1Ch]  ; [45AEF4]与C[0x1C]比较
.text:00408552                 jl      short loc_408570 ; 小于继续
.text:00408554                 jg      short loc_40855B
.text:00408556                 cmp     ecx, [ebp+18h]  ; [45AEF0]-0x1E28500与C[0x18]比较
.text:00408559                 jb      short loc_408570 ; 小于继续
.text:0040855B
.text:0040855B loc_40855B:                             ; CODE XREF: sub_4084B0+A4j
.text:0040855B                 push    -1
.text:0040855D                 push    offset aTheSuppliedL_1 ; "The supplied license key is not valid f"...
.text:00408562                 lea     ecx, [esp+48h+Memory]
.text:00408566                 call    sub_40CBC0
.text:0040856B                 jmp     loc_4086F1
.text:00408570 ; ---------------------------------------------------------------------------
.text:00408570
.text:00408570 loc_408570:                             ; CODE XREF: sub_4084B0+A2j
.text:00408570                                         ; sub_4084B0+A9j
.text:00408570                 mov     esi, [ebp+0]            ;C[0]
.text:00408573                 mov     edi, [edi+50h]         ;特征码0
.text:00408576                 mov     ecx, esi
.text:00408578                 mov     eax, edi
.text:0040857A                 lea     ebx, [ebx+0]
.text:00408580
.text:00408580 loc_408580:                             ; CODE XREF: sub_4084B0+EAj
.text:00408580                 mov     dl, [eax]       ; 比较特征码
.text:00408582                 cmp     dl, [ecx]
.text:00408584                 jnz     short loc_4085A0
.text:00408586                 cmp     dl, bl             ;判断结束符0
.text:00408588                 jz      short loc_40859C
.text:0040858A                 mov     dl, [eax+1]
.text:0040858D                 cmp     dl, [ecx+1]
.text:00408590                 jnz     short loc_4085A0
.text:00408592                 add     eax, 2
.text:00408595                 add     ecx, 2
.text:00408598                 cmp     dl, bl
.text:0040859A                 jnz     short loc_408580
.text:0040859C
.text:0040859C loc_40859C:                             ; CODE XREF: sub_4084B0+D8j
.text:0040859C                 xor     eax, eax      ;相等,则eax置0
.text:0040859E                 jmp     short loc_4085A5
.text:004085A0 ; ---------------------------------------------------------------------------
.text:004085A0
.text:004085A0 loc_4085A0:                             ; CODE XREF: sub_4084B0+D4j
.text:004085A0                                         ; sub_4084B0+E0j
.text:004085A0                 sbb     eax, eax
.text:004085A2                 sbb     eax, 0FFFFFFFFh;不等,则eax置1
.text:004085A5
.text:004085A5 loc_4085A5:                             ; CODE XREF: sub_4084B0+EEj
```
