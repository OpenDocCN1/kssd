
# 齿轮参数计算 2.1 破解分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-341.htm`](https://www.pediy.com/kssd/pediy07/pediy7-341.htm)

**【破文标题】：** 齿轮参数计算 2.1 破解分析  
**【破文作者】：** KuNgBiM\[DFCG\]  
**【作者邮箱】：** `gb_1227@163.com`  
**【软件名称】：** 齿轮参数计算 2.1  
**【软件大小】：** 4761 KB  
**【软件类别】：** 国产软件/共享版/辅助设计  
**【整理时间】：** 2005-8-10  
**【开发 商】：** [`speed-long.com/`](http://speed-long.com/)  
**【下载地址】：** [`www.onlinedown.net/soft/32336.htm`](http://www.onlinedown.net/soft/32336.htm)  

**【软件简介】：** 机械工程师的得力助手,中、小企业及个体私营业主值得依靠的技术力量，软件主要功能如下：  

1. 外啮合圆柱齿轮设计计算，几何计算，强度校核及测绘；  
2. 内啮合圆柱齿轮设计计算，几何计算，强度校核；  
3. 国家标准尺寸公差与配合查阅与选用；  
4. 渐开线反函数的计算；  
5. 齿厚上下偏差的确定；  
6. 减速器传动比的分配；  
7. 标准螺旋齿轮传动设计（汽车变速器里程表齿轮计算）。  

**【保护方式】：** 注册码 + 功能限制  
**【编译语言】：** Microsoft Visual Basic 5.0 / 6.0  
**【调试环境】：** `WinXP`、`PEiD`、`IDA`  
**【破解日期】：** 2005-08-11  
**【破解目的】：** 研究算法分析  
**【作者声明】：** 初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  

—————————————————————————————————  

**【破解过程】：**  

**侦测：** 用 `PEiD` 查壳，无壳，Microsoft Visual Basic 5.0 / 6.0 编译。  

我这几天在学用 `IDA` 调试程序，所以以下是 `IDA` 反编译出的代码：（BTW：`IDA` 太强大了，我也要学习学习！）  

```asm
.text:00555ED0             push    ebp
.text:00555ED1             mov     ebp, esp
.text:00555ED3             sub     esp, 8          ; 整数相减
.text:00555ED6             push    offset loc_406316
.text:00555EDB             mov     eax, large fs:0
.text:00555EE1             push    eax
.text:00555EE2             mov     large fs:0, esp
.text:00555EE9             sub     esp, 58h        ; 整数相减
.text:00555EEC             push    ebx
.text:00555EED             push    esi
.text:00555EEE             push    edi
.text:00555EEF             mov     [ebp-8], esp
.text:00555EF2             mov     dword ptr [ebp-4], offset dword_4053A8
.text:00555EF9             mov     eax, dword_5815D0
.text:00555EFE             xor     esi, esi        ; 异或清零
.text:00555F00             cmp     eax, esi        ; 比较二个运算单元
.text:00555F02             mov     [ebp-14h], esi
.text:00555F05             mov     [ebp-18h], esi
.text:00555F08             mov     [ebp-1Ch], esi
.text:00555F0B             mov     [ebp-20h], esi
.text:00555F0E             mov     [ebp-30h], esi
.text:00555F11             mov     [ebp-40h], esi
.text:00555F14             jnz     short loc_555F26 ; 不等于 0 时则跳 (ZF=0)
.text:00555F14
.text:00555F16             push    offset dword_5815D0
.text:00555F1B             push    offset dword_434B30
.text:00555F20             call    ds:__vbaNew2    ; Indirect Call Near Procedure
.text:00555F20
.text:00555F26
.text:00555F26  loc_555F26:                       ; CODE XREF: .text:00555F14j
.text:00555F26             mov     edi, dword_5815D0
.text:00555F2C             lea     ecx, [ebp-20h]  ; 载入有效位址
.text:00555F2F             push    ecx
.text:00555F30             push    edi
.text:00555F31             mov     eax, [edi]
.text:00555F33             call    dword ptr [eax+14h] ; Indirect Call Near Procedure
.text:00555F33
.text:00555F36             cmp     eax, esi        ; 比较二个运算单元
.text:00555F38             fnclex                  ; 清除异常 (no wait)
.text:00555F3A             jge     short loc_555F4B ; 大于或等于则跳 (SF=OF)
.text:00555F3A
.text:00555F3C             push    14h
.text:00555F3E             push    offset dword_434B20
.text:00555F43             push    edi
.text:00555F44             push    eax
.text:00555F45             call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure
.text:00555F45
.text:00555F4B
.text:00555F4B  loc_555F4B:                       ; CODE XREF: .text:00555F3Aj
.text:00555F4B             mov     eax, [ebp-20h]
.text:00555F4E             lea     ecx, [ebp-18h]  ; 载入有效位址
.text:00555F51             push    ecx
.text:00555F52             push    eax
.text:00555F53             mov     edx, [eax]
.text:00555F55             mov     edi, eax
.text:00555F57             call    dword ptr [edx+50h] ; Indirect Call Near Procedure
.text:00555F57
.text:00555F5A             cmp     eax, esi        ; 比较二个运算单元
.text:00555F5C             fnclex                  ; 清除异常 (no wait)
.text:00555F5E             jge     short loc_555F6F ; 大于或等于则跳 (SF=OF)
.text:00555F5E
.text:00555F60             push    50h
.text:00555F62             push    offset dword_434BF0
.text:00555F67             push    edi
.text:00555F68             push    eax
.text:00555F69             call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure
.text:00555F69
.text:00555F6F
.text:00555F6F  loc_555F6F:                       ; CODE XREF: .text:00555F5Ej
.text:00555F6F             mov     edx, [ebp-18h]
.text:00555F72             push    edx
.text:00555F73             push    offset aGear21_ljg ; "\\\gear21.ljg"    ★注册验证文件★（保存注册码的 keyfile）
.text:00555F78             call    ds:__vbaStrCat  ; Indirect Call Near Procedure
.text:00555F78
.text:00555F7E             mov     edi, ds:__vbaStrMove
.text:00555F84             mov     edx, eax
.text:00555F86             lea     ecx, [ebp-1Ch]  ; 载入有效位址
.text:00555F89             call    edi ; __vbaStrMove ; Indirect Call Near Procedure
.text:00555F89
.text:00555F8B             mov     ebx, ds:__vbaFileOpen
.text:00555F91             push    eax
.text:00555F92             push    1
.text:00555F94             push    0FFFFFFFFh
.text:00555F96             push    2
.text:00555F98             call    ebx ; __vbaFileOpen ; Indirect Call Near Procedure
.text:00555F98
.text:00555F9A             lea     eax, [ebp-1Ch]  ; 载入有效位址
.text:00555F9D             lea     ecx, [ebp-18h]  ; 载入有效位址
.text:00555FA0             push    eax
.text:00555FA1             push    ecx
.text:00555FA2             push    2
.text:00555FA4             call    ds:__vbaFreeStrList ; Indirect Call Near Procedure
.text:00555FA4
.text:00555FAA             add     esp, 0Ch        ; 加法运算
.text:00555FAD             lea     ecx, [ebp-20h]  ; 载入有效位址
.text:00555FB0             call    ds:__vbaFreeObj ; Indirect Call Near Procedure
.text:00555FB0
.text:00555FB6             mov     edx, [ebp+8]
.text:00555FB9             mov     eax, [edx+3Ch]
.text:00555FBC             push    eax
.text:00555FBD             push    1
.text:00555FBF             push    offset dword_438A04
.text:00555FC4             call    ds:__vbaPrintFile ; Indirect Call Near Procedure
.text:00555FC4
.text:00555FCA             add     esp, 0Ch        ; 加法运算
.text:00555FCD             push    1
.text:00555FCF             call    ds:__vbaFileClose ; Indirect Call Near Procedure
.text:00555FCF
.text:00555FD5             cmp     dword_5815D0, esi ; 比较二个运算单元
.text:00555FDB             jnz     short loc_555FED ; 不等于 0 时则跳 (ZF=0)
.text:00555FDB
.text:00555FDD             push    offset dword_5815D0
.text:00555FE2             push    offset dword_434B30
.text:00555FE7             call    ds:__vbaNew2    ; Indirect Call Near Procedure
.text:00555FE7
.text:00555FED
```



```asm
.text:00555FED     loc_555FED:                     ; CODE XREF: .text:00555FDBj  

.text:00555FED     mov      esi, dword_5815D0  

.text:00555FF3     lea      edx, [ebp-20h]   ; 载入有效位址  

.text:00555FF6     push     edx  

.text:00555FF7     push     esi  

.text:00555FF8     mov      ecx, [esi]  

.text:00555FFA     call     dword ptr [ecx+14h] ; Indirect Call Near Procedure  

.text:00555FFA  

.text:00555FFD     test     eax, eax         ; 两操作数作与运算,逻辑比较  

.text:00555FFF     fnclex                  ; 清除异常 (no wait)  

.text:00556001     jge      short loc_556012 ; 大于或等于则跳 (SF=OF)  

.text:00556001  

.text:00556003     push     14h  

.text:00556005     push     offset dword_434B20  

.text:0055600A     push     esi  

.text:0055600B     push     eax  

.text:0055600C     call     ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  

.text:0055600C  

.text:00556012  

.text:00556012     loc_556012:                     ; CODE XREF: .text:00556001j  

.text:00556012     mov      eax, [ebp-20h]  

.text:00556015     lea      edx, [ebp-18h]   ; 载入有效位址  

.text:00556018     push     edx  

.text:00556019     push     eax  

.text:0055601A     mov      ecx, [eax]  

.text:0055601C     mov      esi, eax  

.text:0055601E     call     dword ptr [ecx+50h] ; Indirect Call Near Procedure  

.text:0055601E  

.text:00556021     test     eax, eax         ; 两操作数作与运算,逻辑比较  

.text:00556023     fnclex                  ; 清除异常 (no wait)  

.text:00556025     jge      short loc_556036 ; 大于或等于则跳 (SF=OF)  

.text:00556025  

.text:00556027     push     50h  

.text:00556029     push     offset dword_434BF0  

.text:0055602E     push     esi  

.text:0055602F     push     eax  

.text:00556030     call     ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  

.text:00556030  

.text:00556036  

.text:00556036     loc_556036:                     ; CODE XREF: .text:00556025j  

.text:00556036     mov      eax, [ebp-18h]  

.text:00556039     push     eax  

.text:0055603A     push     offset aGear21_flg ; "\\\gear21.flg"    ★注册项目特征文件★（保存所注册功能的特征文件）  

.text:0055603F     call     ds:__vbaStrCat  ; Indirect Call Near Procedure  

.text:0055603F  

.text:00556045     mov      edx, eax  

.text:00556047     lea      ecx, [ebp-1Ch]   ; 载入有效位址  

.text:0055604A     call     edi ; __vbaStrMove ; Indirect Call Near Procedure  

.text:0055604A  

.text:0055604C     push     eax  

.text:0055604D     push     1  

.text:0055604F     push     0FFFFFFFFh  

.text:00556051     push     2  

.text:00556053     call     ebx ; __vbaFileOpen ; Indirect Call Near Procedure  

.text:00556053  

.text:00556055     lea      ecx, [ebp-1Ch]   ; 载入有效位址  

.text:00556058     lea      edx, [ebp-18h]   ; 载入有效位址  

.text:0055605B     push     ecx  

.text:0055605C     push     edx  

.text:0055605D     push     2  

.text:0055605F     call     ds:__vbaFreeStrList ; Indirect Call Near Procedure  

.text:0055605F  

.text:00556065     add      esp, 0Ch         ; 加法运算  

.text:00556068     lea      ecx, [ebp-20h]   ; 载入有效位址  

.text:0055606B     call     ds:__vbaFreeObj ; Indirect Call Near Procedure  

.text:0055606B  

.text:00556071     mov      eax, [ebp+8]  

.text:00556074     mov      cx, [eax+44h]  

.text:00556078     push     ecx  

.text:00556079     push     1  

.text:0055607B     push     offset dword_43B1B0  

.text:00556080     call     ds:__vbaPrintFile ; Indirect Call Near Procedure  

.text:00556080  

.text:00556086     add      esp, 0Ch         ; 加法运算  

.text:00556089     push     1  

.text:0055608B     call     ds:__vbaFileClose ; Indirect Call Near Procedure  

.text:0055608B  

.text:00556091     push     0  

.text:00556093     lea      edx, [ebp-30h]   ; 载入有效位址  

.text:00556096     push     offset aScripting_file ; "Scripting.FileSystemObject"  

.text:0055609B     push     edx  

.text:0055609C     call     ds:rtcCreateObject2 ; Indirect Call Near Procedure  

.text:0055609C  

.text:005560A2     lea      eax, [ebp-30h]   ; 载入有效位址  

.text:005560A5     push     eax  

.text:005560A6     call     ds:__vbaObjVar  ; Indirect Call Near Procedure  

.text:005560A6  

.text:005560AC     lea      ecx, [ebp-14h]   ; 载入有效位址  

.text:005560AF     push     eax  

.text:005560B0     push     ecx  

.text:005560B1     call     ds:__vbaObjSetAddref ; Indirect Call Near Procedure  

.text:005560B1  

.text:005560B7     lea      ecx, [ebp-30h]   ; 载入有效位址  

.text:005560BA     call     ds:__vbaFreeVar ; Indirect Call Near Procedure  

.text:005560BA  

.text:005560C0     mov      eax, dword_5815D0  

.text:005560C5     test     eax, eax         ; 两操作数作与运算,逻辑比较  

.text:005560C7     jnz      short loc_5560D9 ; 不等于 0 时则跳 (ZF=0)  

.text:005560C7  

.text:005560C9     push     offset dword_5815D0  

.text:005560CE     push     offset dword_434B30  

.text:005560D3     call     ds:__vbaNew2    ; Indirect Call Near Procedure  

.text:005560D3  

.text:005560D9  

.text:005560D9     loc_5560D9:                     ; CODE XREF: .text:005560C7j  

.text:005560D9     mov      esi, dword_5815D0  

.text:005560DF     lea      eax, [ebp-20h]   ; 载入有效位址  

.text:005560E2     push     eax  

.text:005560E3     push     esi  

.text:005560E4     mov      edx, [esi]  

.text:005560E6     call     dword ptr [edx+14h] ; Indirect Call Near Procedure  

.text:005560E6  

.text:005560E9     test     eax, eax         ; 两操作数作与运算,逻辑比较  

.text:005560EB     fnclex                  ; 清除异常 (no wait)  

.text:005560ED     jge      short loc_5560FE ; 大于或等于则跳 (SF=OF)  

.text:005560ED  

.text:005560EF     push     14h  

.text:005560F1     push     offset dword_434B20  

.text:005560F6     push     esi  

.text:005560F7     push     eax  

.text:005560F8     call     ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  

.text:005560F8  

.text:005560FE  

.text:005560FE     loc_5560FE:                     ; CODE XREF: .text:005560EDj  

.text:005560FE     mov      eax, [ebp-20h]  

.text:00556101     lea      edx, [ebp-18h]   ; 载入有效位址  

.text:00556104     push     edx  

.text:00556105     push     eax  

.text:00556106     mov      ecx, [eax]  

.text:00556108     mov      esi, eax  

.text:0055610A     call     dword ptr [ecx+50h] ; Indirect Call Near Procedure
```



```asm
.text:0055610A

.text:0055610D
            test    eax, eax        ; 两操作数作与运算,逻辑比较

.text:0055610F
            fnclex                ; 清除异常 (no wait)

.text:00556111
            jge     short loc_556122 ; 大于或等于则跳 (SF=OF)

.text:00556111

.text:00556113
            push    50h

.text:00556115
            push    offset dword_434BF0

.text:0055611A
            push    esi

.text:0055611B
            push    eax

.text:0055611C
            call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:0055611C

.text:00556122

.text:00556122
            loc_556122:                 ; CODE XREF: .text:00556111j

.text:00556122
            mov     eax, [ebp-18h]

.text:00556125
            mov     ebx, ds:__vbaStrCat

.text:0055612B
            push    eax

.text:0055612C
            push    offset aGear21_dll ; "\\\gear21.dll"    ★功能限制特征文件★

.text:00556131
            call    ebx ; __vbaStrCat ; Indirect Call Near Procedure

.text:00556131

.text:00556133
            sub     esp, 10h        ; 整数相减

.text:00556136
            mov     ecx, 8

.text:0055613B
            mov     edx, esp

.text:0055613D
            mov     [ebp-30h], ecx

.text:00556140
            mov     [ebp-28h], eax

.text:00556143
            push    1

.text:00556145
            mov     [edx], ecx

.text:00556147
            mov     ecx, [ebp-2Ch]

.text:0055614A
            push    offset aFileexists ; "fileexists"

.text:0055614F
            mov     [edx+4], ecx

.text:00556152
            mov     ecx, [ebp-14h]

.text:00556155
            push    ecx

.text:00556156
            mov     [edx+8], eax

.text:00556159
            mov     eax, [ebp-24h]

.text:0055615C
            mov     [edx+0Ch], eax

.text:0055615F
            lea     edx, [ebp-40h]  ; 载入有效位址

.text:00556162
            push    edx

.text:00556163
            call    ds:__vbaLateMemCallLd ; Indirect Call Near Procedure

.text:00556163

.text:00556169
            add     esp, 20h        ; 加法运算

.text:0055616C
            push    eax

.text:0055616D
            call    ds:__vbaBoolVarNull ; Indirect Call Near Procedure

.text:0055616D

.text:00556173
            mov     edi, ds:__vbaFreeStr

.text:00556179
            lea     ecx, [ebp-18h]  ; 载入有效位址

.text:0055617C
            mov     esi, eax

.text:0055617E
            call    edi ; __vbaFreeStr ; Indirect Call Near Procedure

.text:0055617E

.text:00556180
            lea     ecx, [ebp-20h]  ; 载入有效位址

.text:00556183
            call    ds:__vbaFreeObj ; Indirect Call Near Procedure

.text:00556183

.text:00556189
            lea     eax, [ebp-40h]  ; 载入有效位址

.text:0055618C
            lea     ecx, [ebp-30h]  ; 载入有效位址

.text:0055618F
            push    eax

.text:00556190
            push    ecx

.text:00556191
            push    2

.text:00556193
            call    ds:__vbaFreeVarList ; Indirect Call Near Procedure

.text:00556193

.text:00556199
            add     esp, 0Ch        ; 加法运算

.text:0055619C
            test    si, si          ; 两操作数作与运算,逻辑比较

.text:0055619F
            jz      loc_55623D      ; 等于 0 则跳 (ZF=1)

.text:0055619F

.text:005561A5
            mov     eax, dword_5815D0

.text:005561AA
            test    eax, eax        ; 两操作数作与运算,逻辑比较

.text:005561AC
            jnz     short loc_5561BE ; 不等于 0 时则跳 (ZF=0)

.text:005561AC

.text:005561AE
            push    offset dword_5815D0

.text:005561B3
            push    offset dword_434B30

.text:005561B8
            call    ds:__vbaNew2    ; Indirect Call Near Procedure

.text:005561B8

.text:005561BE

.text:005561BE
            loc_5561BE:                 ; CODE XREF: .text:005561ACj

.text:005561BE
            mov     esi, dword_5815D0

.text:005561C4
            lea     eax, [ebp-20h]  ; 载入有效位址

.text:005561C7
            push    eax

.text:005561C8
            push    esi

.text:005561C9
            mov     edx, [esi]

.text:005561CB
            call    dword ptr [edx+14h] ; Indirect Call Near Procedure

.text:005561CB

.text:005561CE
            test    eax, eax        ; 两操作数作与运算,逻辑比较

.text:005561D0
            fnclex                ; 清除异常 (no wait)

.text:005561D2
            jge     short loc_5561E3 ; 大于或等于则跳 (SF=OF)

.text:005561D2

.text:005561D4
            push    14h

.text:005561D6
            push    offset dword_434B20

.text:005561DB
            push    esi

.text:005561DC
            push    eax

.text:005561DD
            call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:005561DD

.text:005561E3

.text:005561E3
            loc_5561E3:                 ; CODE XREF: .text:005561D2j

.text:005561E3
            mov     eax, [ebp-20h]

.text:005561E6
            lea     edx, [ebp-18h]  ; 载入有效位址

.text:005561E9
            push    edx

.text:005561EA
            push    eax

.text:005561EB
            mov     ecx, [eax]

.text:005561ED
            mov     esi, eax

.text:005561EF
            call    dword ptr [ecx+50h] ; Indirect Call Near Procedure

.text:005561EF

.text:005561F2
            test    eax, eax        ; 两操作数作与运算,逻辑比较

.text:005561F4
            fnclex                ; 清除异常 (no wait)

.text:005561F6
            jge     short loc_556207 ; 大于或等于则跳 (SF=OF)

.text:005561F6

.text:005561F8
            push    50h

.text:005561FA
            push    offset dword_434BF0

.text:005561FF
            push    esi

.text:00556200
            push    eax

.text:00556201
            call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:00556201

.text:00556207

.text:00556207
            loc_556207:                 ; CODE XREF: .text:005561F6j

.text:00556207
            mov     eax, [ebp-18h]

.text:0055620A
            push    eax

.text:0055620B
            push    offset aGear21_dll ; "\\\gear21.dll"    ★功能限制特征文件★

.text:00556210
            call    ebx ; __vbaStrCat ; Indirect Call Near Procedure

.text:00556210

.text:00556212
            lea     ecx, [ebp-30h]  ; 载入有效位址

.text:00556215
            mov     [ebp-28h], eax

.text:00556218
            push    ecx

.text:00556219
            mov     dword ptr [ebp-30h], 8

.text:00556220
            call    ds:rtcKillFiles ; Indirect Call Near Procedure

.text:00556220

.text:00556226
            lea     ecx, [ebp-18h]  ; 载入有效位址

.text:00556229
            call    edi ; __vbaFreeStr ; Indirect Call Near Procedure

.text:00556229

.text:0055622B
            lea     ecx, [ebp-20h]  ; 载入有效位址

.text:0055622E
            call    ds:__vbaFreeObj ; Indirect Call Near Procedure

.text:0055622E

.text:00556234
            lea     ecx, [ebp-30h]  ; 载入有效位址

.text:00556237
            call    ds:__vbaFreeVar ; Indirect Call Near Procedure

.text:00556237

.text:0055623D
```



```asm
.text:0055623D     loc_55623D:                             ; CODE XREF: .text:0055619Fj  

.text:0055623D            lea     edx, [ebp-14h]   ; 载入有效位址  

.text:00556240            push    0  

.text:00556242            push    edx  

.text:00556243            call    ds:__vbaObjSetAddref ; Indirect Call Near Procedure  

.text:00556243     

.text:00556249            push    offset loc_55628A  

.text:0055624E            jmp     short loc_556280 ; 跳  

.text:0055624E     

.text:00556250     ; ---------------------------------------------------------------------------  

.text:00556250            lea     eax, [ebp-1Ch]   ; 载入有效位址  

.text:00556253            lea     ecx, [ebp-18h]   ; 载入有效位址  

.text:00556256            push    eax  

.text:00556257            push    ecx  

.text:00556258            push    2  

.text:0055625A            call    ds:__vbaFreeStrList ; Indirect Call Near Procedure  

.text:0055625A     

.text:00556260            add     esp, 0Ch         ; 加法运算  

.text:00556263            lea     ecx, [ebp-20h]   ; 载入有效位址  

.text:00556266            call    ds:__vbaFreeObj ; Indirect Call Near Procedure  

.text:00556266     

.text:0055626C            lea     edx, [ebp-40h]   ; 载入有效位址  

.text:0055626F            lea     eax, [ebp-30h]   ; 载入有效位址  

.text:00556272            push    edx  

.text:00556273            push    eax  

.text:00556274            push    2  

.text:00556276            call    ds:__vbaFreeVarList ; Indirect Call Near Procedure  

.text:00556276     

.text:0055627C            add     esp, 0Ch         ; 加法运算  

.text:0055627F            retn                   ; 返回程序  

.text:0055627F     

.text:00556280     ; ---------------------------------------------------------------------------  

.text:00556280     

.text:00556280     loc_556280:                             ; CODE XREF: .text:0055624Ej  

.text:00556280            lea     ecx, [ebp-14h]   ; 载入有效位址  

.text:00556283            call    ds:__vbaFreeObj ; Indirect Call Near Procedure  

.text:00556283     

.text:00556289            retn                   ; 返回程序  

.text:00556289     

.text:0055628A     ; ---------------------------------------------------------------------------  

.text:0055628A     

.text:0055628A     loc_55628A:                             ; DATA XREF: .text:00556249o  

.text:0055628A            mov     ecx, [ebp-10h]  

.text:0055628D            pop     edi  

.text:0055628E            pop     esi  

.text:0055628F            xor     eax, eax         ; 异或清零  

.text:00556291            mov     large fs:0, ecx  

.text:00556298            pop     ebx  

.text:00556299            mov     esp, ebp  

.text:0055629B            pop     ebp  

.text:0055629C            ret     4                ; 返回程序  

.text:0055629C     

.text:0055629C     ; ---------------------------------------------------------------------------  

.text:0055629F            align 4  

.text:005562A0            push    ebp  

.text:005562A1            mov     ebp, esp  

.text:005562A3            sub     esp, 0Ch         ; 整数相减  

.text:005562A6            push    offset loc_406316  

.text:005562AB            mov     eax, large fs:0  

.text:005562B1            push    eax  

.text:005562B2            mov     large fs:0, esp  

.text:005562B9            sub     esp, 11Ch        ; 整数相减  

.text:005562BF            push    ebx  

.text:005562C0            push    esi  

.text:005562C1            push    edi  

.text:005562C2            mov     [ebp-0Ch], esp  

.text:005562C5            mov     dword ptr [ebp-8], offset dword_4053B8  

.text:005562CC            mov     esi, [ebp+8]  

.text:005562CF            mov     eax, esi  

.text:005562D1            and     eax, 1           ; 与运算  

.text:005562D4            mov     [ebp-4], eax  

.text:005562D7            and     esi, 0FFFFFFFEh ; 与运算  

.text:005562DA            push    esi  

.text:005562DB            mov     [ebp+8], esi  

.text:005562DE            mov     ecx, [esi]  

.text:005562E0            call    dword ptr [ecx+4] ; Indirect Call Near Procedure  

.text:005562E0     

.text:005562E3            mov     edx, [esi]  

.text:005562E5            xor     edi, edi         ; 异或清零  

.text:005562E7            push    esi  

.text:005562E8            mov     [ebp-1Ch], edi  

.text:005562EB            mov     [ebp-18h], edi  

.text:005562EE            mov     [ebp-20h], edi  

.text:005562F1            mov     [ebp-24h], edi  

.text:005562F4            mov     [ebp-28h], edi  

.text:005562F7            mov     [ebp-2Ch], edi  

.text:005562FA            mov     [ebp-30h], edi  

.text:005562FD            mov     [ebp-34h], edi  

.text:00556300            mov     [ebp-38h], edi  

.text:00556303            mov     [ebp-48h], edi  

.text:00556306            mov     [ebp-58h], edi  

.text:00556309            mov     [ebp-68h], edi  

.text:0055630C            mov     [ebp-78h], edi  

.text:0055630F            mov     [ebp-88h], edi  

.text:00556315            mov     [ebp-98h], edi  

.text:0055631B            mov     [ebp-0A8h], edi  

.text:00556321            mov     [ebp-0B8h], edi  

.text:00556327            mov     [ebp-0C8h], edi  

.text:0055632D            mov     [ebp-0D8h], edi  

.text:00556333            mov     [ebp-0FCh], edi  

.text:00556339            mov     [ebp-100h], edi  

.text:0055633F            call    dword ptr [edx+304h] ; Indirect Call Near Procedure  

.text:0055633F     

.text:00556345            mov     ebx, ds:__vbaObjSet  

.text:0055634B            push    eax  

.text:0055634C            lea     eax, [ebp-30h]   ; 载入有效位址  

.text:0055634F            push    eax  

.text:00556350            call    ebx ; __vbaObjSet ; Indirect Call Near Procedure  

.text:00556350     

.text:00556352            mov     ecx, [eax]  

.text:00556354            lea     edx, [ebp-0FCh] ; 载入有效位址  

.text:0055635A            push    edx  

.text:0055635B            push    eax  

.text:0055635C            mov     [ebp-104h], eax  

.text:00556362            call    dword ptr [ecx+0E0h] ; Indirect Call Near Procedure  

.text:00556362     

.text:00556368            cmp     eax, edi         ; 比较二个运算单元  

.text:0055636A            fnclex                 ; 清除异常 (no wait)  

.text:0055636C            jge     short loc_556386 ; 大于或等于则跳 (SF=OF)  

.text:0055636C     

.text:0055636E            mov     ecx, [ebp-104h]  

.text:00556374            push    0E0h  

.text:00556379            push    offset dword_437A44  

.text:0055637E            push    ecx  

.text:0055637F            push    eax  

.text:00556380            call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  

.text:00556380     

.text:00556386     

.text:00556386     loc_556386:                             ; CODE XREF: .text:0055636Cj  

.text:00556386            mov     edx, [esi]  

.text:00556388            push    esi
```



```
.text:00556389                 call    dword ptr [edx+300h] ; Indirect Call Near Procedure

.text:0055638F                 push    eax

.text:00556390                 lea     eax, [ebp-34h]  ; 载入有效位址

.text:00556393                 push    eax

.text:00556394                 call    ebx ; __vbaObjSet ; Indirect Call Near Procedure

.text:00556396                 mov     ecx, [eax]

.text:00556398                 lea     edx, [ebp-100h] ; 载入有效位址

.text:0055639E                 push    edx

.text:0055639F                 push    eax

.text:005563A0                 mov     [ebp-10Ch], eax

.text:005563A6                 call    dword ptr [ecx+0E0h] ; Indirect Call Near Procedure

.text:005563AC                 cmp     eax, edi        ; 比较二个运算单元

.text:005563AE                 fnclex                  ; 清除异常 (no wait)

.text:005563B0                 jge     short loc_5563CA ; 大于或等于则跳 (SF=OF)

.text:005563B2                 mov     ecx, [ebp-10Ch]

.text:005563B8                 push    0E0h

.text:005563BD                 push    offset dword_437A44

.text:005563C2                 push    ecx

.text:005563C3                 push    eax

.text:005563C4                 call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:005563CA                 loc_5563CA:             ; CODE XREF: .text:005563B0j

.text:005563CA                 mov     eax, 1

.text:005563CF                 xor     edx, edx        ; 异或清零

.text:005563D1                 cmp     [ebp-100h], ax  ; 比较二个运算单元

.text:005563D8                 setnz   dl              ; 计数器置 1 (ZF=0)

.text:005563DB                 neg     edx             ; 寄存器反向互补

.text:005563DD                 xor     ecx, ecx        ; 异或清零

.text:005563DF                 cmp     [ebp-0FCh], ax  ; 比较二个运算单元

.text:005563E6                 lea     eax, [ebp-30h]  ; 载入有效位址

.text:005563E9                 setnz   cl              ; 计数器置 1 (ZF=0)

.text:005563EC                 neg     ecx             ; 寄存器反向互补

.text:005563EE                 and     edx, ecx        ; 与运算

.text:005563F0                 mov     [ebp-114h], edx

.text:005563F6                 lea     edx, [ebp-34h]  ; 载入有效位址

.text:005563F9                 push    edx

.text:005563FA                 push    eax

.text:005563FB                 push    2

.text:005563FD                 call    ds:__vbaFreeObjList ; Indirect Call Near Procedure

.text:00556403                 add     esp, 0Ch        ; 加法运算

.text:00556406                 cmp     [ebp-114h], di  ; 比较二个运算单元

.text:0055640D                 jz      loc_5564A8      ; 等于 0 则跳 (ZF=1)

.text:00556413                 call    ds:rtcBeep      ; Indirect Call Near Procedure

.text:00556419                 mov     esi, ds:__vbaVarDup

.text:0055641F                 mov     ecx, 80020004h

.text:00556424                 mov     [ebp-70h], ecx

.text:00556427                 mov     eax, 0Ah

.text:0055642C                 mov     [ebp-60h], ecx

.text:0055642F                 mov     ebx, 8

.text:00556434                 lea     edx, [ebp-0D8h] ; 载入有效位址

.text:0055643A                 lea     ecx, [ebp-58h]  ; 载入有效位址

.text:0055643D                 mov     [ebp-78h], eax

.text:00556440                 mov     [ebp-68h], eax

.text:00556443                 mov     dword ptr [ebp-0D0h], offset dword_43BCD4

.text:0055644D                 mov     [ebp-0D8h], ebx

.text:00556453                 call    esi ; __vbaVarDup ; Indirect Call Near Procedure

.text:00556455                 lea     edx, [ebp-0C8h] ; 载入有效位址

.text:0055645B                 lea     ecx, [ebp-48h]  ; 载入有效位址

.text:0055645E                 mov     dword ptr [ebp-0C0h], offset dword_43BCB8

.text:00556468                 mov     [ebp-0C8h], ebx

.text:0055646E                 call    esi ; __vbaVarDup ; Indirect Call Near Procedure

.text:00556470                 lea     ecx, [ebp-78h]  ; 载入有效位址

.text:00556473                 lea     edx, [ebp-68h]  ; 载入有效位址

.text:00556476                 push    ecx

.text:00556477                 lea     eax, [ebp-58h]  ; 载入有效位址

.text:0055647A                 push    edx

.text:0055647B                 push    eax

.text:0055647C                 lea     ecx, [ebp-48h]  ; 载入有效位址

.text:0055647F                 push    30h

.text:00556481                 push    ecx

.text:00556482                 call    ds:rtcMsgBox    ; Indirect Call Near Procedure

.text:00556488                 lea     edx, [ebp-78h]  ; 载入有效位址

.text:0055648B                 lea     eax, [ebp-68h]  ; 载入有效位址

.text:0055648E                 push    edx

.text:0055648F                 lea     ecx, [ebp-58h]  ; 载入有效位址

.text:00556492                 push    eax

.text:00556493                 lea     edx, [ebp-48h]  ; 载入有效位址

.text:00556496                 push    ecx

.text:00556497                 push    edx

.text:00556498                 push    4

.text:0055649A                 call    ds:__vbaFreeVarList ; Indirect Call Near Procedure

.text:005564A0                 add     esp, 14h        ; 加法运算

.text:005564A3                 jmp     loc_556E87      ; 跳

.text:005564A8                 ; ---------------------------------------------------------------------------

.text:005564A8                 loc_5564A8:             ; CODE XREF: .text:0055640Dj

.text:005564A8                 mov     eax, [esi]

.text:005564AA                 push    esi

.text:005564AB                 call    dword ptr [eax+330h] ; Indirect Call Near Procedure

.text:005564B1                 lea     ecx, [ebp-30h]  ; 载入有效位址

.text:005564B4                 push    eax

.text:005564B5                 push    ecx

.text:005564B6                 call    ebx             ; Indirect Call Near Procedure

.text:005564B8                 mov     edx, [eax]

.text:005564BA                 lea     ecx, [ebp-20h]  ; 载入有效位址

.text:005564BD                 push    ecx

.text:005564BE                 push    eax

.text:005564BF                 mov     [ebp-104h], eax

.text:005564C5                 call    dword ptr [edx+50h] ; Indirect Call Near Procedure

.text:005564C8                 cmp     eax, edi        ; 比较二个运算单元

.text:005564CA                 fnclex                  ; 清除异常 (no wait)

.text:005564CC                 jge     short loc_5564E3 ; 大于或等于则跳 (SF=OF)

.text:005564CE                 mov     edx, [ebp-104h]

.text:005564D4                 push    50h

.text:005564D6                 push    offset dword_436A50

.text:005564DB                 push    edx

.text:005564DC                 push    eax

.text:005564DD                 call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:005564E3                 loc_5564E3:             ; CODE XREF: .text:005564CCj

.text:005564E3                 mov     eax, [esi]
```



```
.text:005564E5 push esi
.text:005564E6 call dword ptr [eax+330h] ; Indirect Call Near Procedure

.text:005564EC lea ecx, [ebp-34h] ; 载入有效位址
.text:005564EF push eax
.text:005564F0 push ecx
.text:005564F1 call ebx ; Indirect Call Near Procedure

.text:005564F3 mov edx, [eax]
.text:005564F5 lea ecx, [ebp-24h] ; 载入有效位址
.text:005564F8 push ecx
.text:005564F9 push eax
.text:005564FA mov [ebp-10Ch], eax
.text:00556500 call dword ptr [edx+50h] ; Indirect Call Near Procedure

.text:00556503 cmp eax, edi ; 比较二个运算单元
.text:00556505 fnclex ; 清除异常 (no wait)
.text:00556507 jge short loc_55651E ; 大于或等于则跳 (SF=OF)

.text:00556509 mov edx, [ebp-10Ch]
.text:0055650F push 50h
.text:00556511 push offset dword_436A50
.text:00556516 push edx
.text:00556517 push eax
.text:00556518 call ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:0055651E loc_55651E: ; CODE XREF: .text:00556507j
.text:0055651E mov eax, [ebp-24h]
.text:00556521 push eax
.text:00556522 push offset dword_434914
.text:00556527 call ds:__vbaStrCmp ; Indirect Call Near Procedure

.text:0055652D mov ecx, [ebp-20h]
.text:00556530 mov edi, eax
.text:00556532 neg edi ; 寄存器反向互补
.text:00556534 sbb edi, edi ; 带借位减法
.text:00556536 push ecx
.text:00556537 inc edi ; 计数器自加 1
.text:00556538 push offset dword_436AF4
.text:0055653D neg edi ; 寄存器反向互补
.text:0055653F call ds:__vbaStrCmp ; Indirect Call Near Procedure

.text:00556545 neg eax ; 寄存器反向互补
.text:00556547 sbb eax, eax ; 带借位减法
.text:00556549 lea edx, [ebp-24h] ; 载入有效位址
.text:0055654C inc eax ; 计数器自加 1
.text:0055654D push edx
.text:0055654E neg eax ; 寄存器反向互补
.text:00556550 or edi, eax ; 或运算
.text:00556552 lea eax, [ebp-20h] ; 载入有效位址
.text:00556555 push eax
.text:00556556 push 2
.text:00556558 call ds:__vbaFreeStrList ; Indirect Call Near Procedure

.text:0055655E lea ecx, [ebp-34h] ; 载入有效位址
.text:00556561 lea edx, [ebp-30h] ; 载入有效位址
.text:00556564 push ecx
.text:00556565 push edx
.text:00556566 push 2
.text:00556568 call ds:__vbaFreeObjList ; Indirect Call Near Procedure

.text:0055656E add esp, 18h ; 加法运算
.text:00556571 test di, di ; 两操作数作与运算,逻辑比较
.text:00556574 jz loc_556615 ; 等于 0 则跳 (ZF=1)

.text:0055657A mov esi, ds:__vbaVarDup
.text:00556580 mov ecx, 80020004h
.text:00556585 mov [ebp-70h], ecx
.text:00556588 mov eax, 0Ah
.text:0055658D mov [ebp-60h], ecx
.text:00556590 mov edi, 8
.text:00556595 lea edx, [ebp-0D8h] ; 载入有效位址
.text:0055659B lea ecx, [ebp-58h] ; 载入有效位址
.text:0055659E mov [ebp-78h], eax
.text:005565A1 mov [ebp-68h], eax
.text:005565A4 mov dword ptr [ebp-0D0h], offset dword_43B714
.text:005565AE mov [ebp-0D8h], edi
.text:005565B4 call esi ; __vbaVarDup ; Indirect Call Near Procedure

.text:005565B6 lea edx, [ebp-0C8h] ; 载入有效位址
.text:005565BC lea ecx, [ebp-48h] ; 载入有效位址
.text:005565BF mov dword ptr [ebp-0C0h], offset dword_43BCE0
.text:005565C9 mov [ebp-0C8h], edi
.text:005565CF call esi ; __vbaVarDup ; Indirect Call Near Procedure

.text:005565D1 lea eax, [ebp-78h] ; 载入有效位址
.text:005565D4 lea ecx, [ebp-68h] ; 载入有效位址
.text:005565D7 push eax
.text:005565D8 lea edx, [ebp-58h] ; 载入有效位址
.text:005565DB push ecx
.text:005565DC push edx
.text:005565DD lea eax, [ebp-48h] ; 载入有效位址
.text:005565E0 push 0
.text:005565E2 push eax
.text:005565E3 call ds:rtcMsgBox ; Indirect Call Near Procedure

.text:005565E9 lea ecx, [ebp-78h] ; 载入有效位址
.text:005565EC lea edx, [ebp-68h] ; 载入有效位址
.text:005565EF push ecx
.text:005565F0 lea eax, [ebp-58h] ; 载入有效位址
.text:005565F3 push edx
.text:005565F4 lea ecx, [ebp-48h] ; 载入有效位址
.text:005565F7 push eax
.text:005565F8 push ecx
.text:005565F9 push 4
.text:005565FB call ds:__vbaFreeVarList ; Indirect Call Near Procedure

.text:00556601 add esp, 14h ; 加法运算
.text:00556604 call ds:rtcBeep ; Indirect Call Near Procedure

.text:0055660A call ds:__vbaEnd ; Indirect Call Near Procedure

.text:00556610 jmp loc_556E85 ; 跳

.text:00556615 ; ---------------------------------------------------------------------------

.text:00556615 loc_556615: ; CODE XREF: .text:00556574j
.text:00556615 mov edx, [esi]
.text:00556617 push esi
.text:00556618 call dword ptr [edx+320h] ; Indirect Call Near Procedure

.text:0055661E push eax
.text:0055661F lea eax, [ebp-30h] ; 载入有效位址
.text:00556622 push eax
.text:00556623 call ebx ; Indirect Call Near Procedure

.text:00556625 mov edi, eax
.text:00556627 lea edx, [ebp-20h] ; 载入有效位址
.text:0055662A push edx
```



```
.text:0055662B                 push    edi  

.text:0055662C                 mov     ecx, [edi]  

.text:0055662E                 call    dword ptr [ecx+0A0h] ; Indirect Call Near Procedure  

.text:0055662E     

.text:00556634                 test    eax, eax        ; 两操作数作与运算,逻辑比较  

.text:00556636                 fnclex                  ; 清除异常 (no wait)  

.text:00556638                 jge     short loc_55664C ; 大于或等于则跳 (SF=OF)  

.text:00556638     

.text:0055663A                 push    0A0h  

.text:0055663F                 push    offset dword_438F3C  

.text:00556644                 push    edi  

.text:00556645                 push    eax  

.text:00556646                 call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  

.text:00556646     

.text:0055664C     

.text:0055664C     loc_55664C:                      ; CODE XREF: .text:00556638j  

.text:0055664C                 mov     eax, [ebp-20h]  

.text:0055664F                 lea     ecx, [ebp-58h]  ; 载入有效位址  

.text:00556652                 mov     [ebp-40h], eax  

.text:00556655                 lea     eax, [ebp-48h]  ; 载入有效位址  

.text:00556658                 push    eax  

.text:00556659                 push    ecx  

.text:0055665A                 mov     dword ptr [ebp-20h], 0  

.text:00556661                 mov     dword ptr [ebp-48h], 8  

.text:00556668                 call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:00556668     

.text:0055666E                 mov     edx, [esi]  

.text:00556670                 push    esi  

.text:00556671                 call    dword ptr [edx+31Ch] ; Indirect Call Near Procedure  

.text:00556671     

.text:00556677                 push    eax  

.text:00556678                 lea     eax, [ebp-34h]  ; 载入有效位址  

.text:0055667B                 push    eax  

.text:0055667C                 call    ebx             ; Indirect Call Near Procedure  

.text:0055667C     

.text:0055667E                 mov     edi, eax  

.text:00556680                 lea     edx, [ebp-24h]  ; 载入有效位址  

.text:00556683                 push    edx  

.text:00556684                 push    edi  

.text:00556685                 mov     ecx, [edi]  

.text:00556687                 call    dword ptr [ecx+0A0h] ; Indirect Call Near Procedure  

.text:00556687     

.text:0055668D                 test    eax, eax        ; 两操作数作与运算,逻辑比较  

.text:0055668F                 fnclex                  ; 清除异常 (no wait)  

.text:00556691                 jge     short loc_5566A5 ; 大于或等于则跳 (SF=OF)  

.text:00556691     

.text:00556693                 push    0A0h  

.text:00556698                 push    offset dword_438F3C  

.text:0055669D                 push    edi  

.text:0055669E                 push    eax  

.text:0055669F                 call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  

.text:0055669F     

.text:005566A5     

.text:005566A5     loc_5566A5:                      ; CODE XREF: .text:00556691j  

.text:005566A5                 mov     eax, [ebp-24h]  

.text:005566A8                 lea     ecx, [ebp-78h]  ; 载入有效位址  

.text:005566AB                 mov     [ebp-60h], eax  

.text:005566AE                 lea     eax, [ebp-68h]  ; 载入有效位址  

.text:005566B1                 push    eax  

.text:005566B2                 push    ecx  

.text:005566B3                 mov     dword ptr [ebp-24h], 0  

.text:005566BA                 mov     dword ptr [ebp-68h], 8  

.text:005566C1                 call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:005566C1     

.text:005566C7                 mov     edx, [esi]  

.text:005566C9                 push    esi  

.text:005566CA                 call    dword ptr [edx+318h] ; Indirect Call Near Procedure  

.text:005566CA     

.text:005566D0                 push    eax  

.text:005566D1                 lea     eax, [ebp-38h]  ; 载入有效位址  

.text:005566D4                 push    eax  

.text:005566D5                 call    ebx             ; Indirect Call Near Procedure  

.text:005566D5     

.text:005566D7                 mov     edi, eax  

.text:005566D9                 lea     edx, [ebp-28h]  ; 载入有效位址  

.text:005566DC                 push    edx  

.text:005566DD                 push    edi  

.text:005566DE                 mov     ecx, [edi]  

.text:005566E0                 call    dword ptr [ecx+0A0h] ; Indirect Call Near Procedure  

.text:005566E0     

.text:005566E6                 test    eax, eax        ; 两操作数作与运算,逻辑比较  

.text:005566E8                 fnclex                  ; 清除异常 (no wait)  

.text:005566EA                 jge     short loc_5566FE ; 大于或等于则跳 (SF=OF)  

.text:005566EA     

.text:005566EC                 push    0A0h  

.text:005566F1                 push    offset dword_438F3C  

.text:005566F6                 push    edi  

.text:005566F7                 push    eax  

.text:005566F8                 call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  

.text:005566F8     

.text:005566FE     

.text:005566FE     loc_5566FE:                      ; CODE XREF: .text:005566EAj  

.text:005566FE                 mov     eax, [ebp-28h]  

.text:00556701                 lea     ecx, [ebp-0A8h] ; 载入有效位址  

.text:00556707                 mov     [ebp-90h], eax  

.text:0055670D                 lea     eax, [ebp-98h]  ; 载入有效位址  

.text:00556713                 push    eax  

.text:00556714                 push    ecx  

.text:00556715                 mov     dword ptr [ebp-28h], 0  

.text:0055671C                 mov     dword ptr [ebp-98h], 8  

.text:00556726                 call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:00556726     

.text:0055672C                 mov     edi, ds:__vbaVarCat  

.text:00556732                 lea     edx, [ebp-58h]  ; 载入有效位址  

.text:00556735                 lea     eax, [ebp-78h]  ; 载入有效位址  

.text:00556738                 push    edx  

.text:00556739                 lea     ecx, [ebp-88h]  ; 载入有效位址  

.text:0055673F                 push    eax  

.text:00556740                 push    ecx  

.text:00556741                 call    edi ; __vbaVarCat ; Indirect Call Near Procedure  

.text:00556741     

.text:00556743                 push    eax  

.text:00556744                 lea     edx, [ebp-0A8h] ; 载入有效位址  

.text:0055674A                 lea     eax, [ebp-0B8h] ; 载入有效位址  

.text:00556750                 push    edx  

.text:00556751                 push    eax  

.text:00556752                 call    edi ; __vbaVarCat ; Indirect Call Near Procedure  

.text:00556752     

.text:00556754                 push    eax  

.text:00556755                 call    ds:__vbaStrVarMove ; Indirect Call Near Procedure  

.text:00556755     

.text:0055675B                 mov     edx, eax  

.text:0055675D                 lea     ecx, [ebp-2Ch]  ; 载入有效位址  

.text:00556760                 call    ds:__vbaStrMove ; Indirect Call Near Procedure  

.text:00556760     

.text:00556766                 mov     edx, eax  

.text:00556768                 lea     ecx, [esi+3Ch]  ; 载入有效位址  

.text:0055676B                 call    ds:__vbaStrCopy ; Indirect Call Near Procedure  

.text:0055676B     

.text:00556771                 lea     ecx, [ebp-2Ch]  ; 载入有效位址  

.text:00556774                 call    ds:__vbaFreeStr ; Indirect Call Near Procedure  

.text:00556774     
```



```
.text:0055677A                 lea     ecx, [ebp-38h]   ; 载入有效位址
.text:0055677D                 lea     edx, [ebp-34h]   ; 载入有效位址
.text:00556780                 push    ecx
.text:00556781                 lea     eax, [ebp-30h]   ; 载入有效位址
.text:00556784                 push    edx
.text:00556785                 push    eax
.text:00556786                 push    3
.text:00556788                 call    ds:__vbaFreeObjList ; Indirect Call Near Procedure
.text:00556788
.text:0055678E                 lea     ecx, [ebp-0B8h] ; 载入有效位址
.text:00556794                 lea     edx, [ebp-0A8h] ; 载入有效位址
.text:0055679A                 push    ecx
.text:0055679B                 lea     eax, [ebp-88h]   ; 载入有效位址
.text:005567A1                 push    edx
.text:005567A2                 lea     ecx, [ebp-98h]   ; 载入有效位址
.text:005567A8                 push    eax
.text:005567A9                 lea     edx, [ebp-78h]   ; 载入有效位址
.text:005567AC                 push    ecx
.text:005567AD                 lea     eax, [ebp-58h]   ; 载入有效位址
.text:005567B0                 push    edx
.text:005567B1                 lea     ecx, [ebp-68h]   ; 载入有效位址
.text:005567B4                 push    eax
.text:005567B5                 lea     edx, [ebp-48h]   ; 载入有效位址
.text:005567B8                 push    ecx
.text:005567B9                 push    edx
.text:005567BA                 push    8
.text:005567BC                 call    ds:__vbaFreeVarList ; Indirect Call Near Procedure
.text:005567BC
.text:005567C2                 mov     eax, [esi]
.text:005567C4                 add     esp, 34h         ; 加法运算
.text:005567C7                 push    esi
.text:005567C8                 call    dword ptr [eax+304h] ; Indirect Call Near Procedure
.text:005567C8
.text:005567CE                 lea     ecx, [ebp-30h]   ; 载入有效位址
.text:005567D1                 push    eax
.text:005567D2                 push    ecx
.text:005567D3                 call    ebx              ; Indirect Call Near Procedure
.text:005567D3
.text:005567D5                 mov     edi, eax
.text:005567D7                 lea     eax, [ebp-0FCh] ; 载入有效位址
.text:005567DD                 push    eax
.text:005567DE                 push    edi
.text:005567DF                 mov     edx, [edi]
.text:005567E1                 call    dword ptr [edx+0E0h] ; Indirect Call Near Procedure
.text:005567E1
.text:005567E7                 test    eax, eax         ; 两操作数作与运算,逻辑比较
.text:005567E9                 fnclex                   ; 清除异常 (no wait)
.text:005567EB                 jge     short loc_5567FF ; 大于或等于则跳 (SF=OF)
.text:005567EB
.text:005567ED                 push    0E0h
.text:005567F2                 push    offset dword_437A44
.text:005567F7                 push    edi
.text:005567F8                 push    eax
.text:005567F9                 call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure
.text:005567F9
.text:005567FF
.text:005567FF     loc_5567FF:                          ; CODE XREF: .text:005567EBj
.text:005567FF                 mov     ecx, [esi]
.text:00556801                 push    esi
.text:00556802                 call    dword ptr [ecx+300h] ; Indirect Call Near Procedure
.text:00556802
.text:00556808                 lea     edx, [ebp-34h]   ; 载入有效位址
.text:0055680B                 push    eax
.text:0055680C                 push    edx
.text:0055680D                 call    ebx              ; Indirect Call Near Procedure
.text:0055680D
.text:0055680F                 mov     edi, eax
.text:00556811                 lea     ecx, [ebp-100h] ; 载入有效位址
.text:00556817                 push    ecx
.text:00556818                 push    edi
.text:00556819                 mov     eax, [edi]
.text:0055681B                 call    dword ptr [eax+0E0h] ; Indirect Call Near Procedure
.text:0055681B
.text:00556821                 test    eax, eax         ; 两操作数作与运算,逻辑比较
.text:00556823                 fnclex                   ; 清除异常 (no wait)
.text:00556825                 jge     short loc_556839 ; 大于或等于则跳 (SF=OF)
.text:00556825
.text:00556827                 push    0E0h
.text:0055682C                 push    offset dword_437A44
.text:00556831                 push    edi
.text:00556832                 push    eax
.text:00556833                 call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure
.text:00556833
.text:00556839
.text:00556839     loc_556839:                          ; CODE XREF: .text:00556825j
.text:00556839                 mov     edi, 1
.text:0055683E                 xor     edx, edx         ; 异或清零
.text:00556840                 cmp     [ebp-100h], di   ; 比较二个运算单元
.text:00556847                 lea     ecx, [ebp-34h]   ; 载入有效位址
.text:0055684A                 push    ecx
.text:0055684B                 setz    dl               ; 计数器置 0 (ZF=1)
.text:0055684E                 neg     edx              ; 寄存器反向互补
.text:00556850                 xor     eax, eax         ; 异或清零
.text:00556852                 cmp     [ebp-0FCh], di   ; 比较二个运算单元
.text:00556859                 setz    al               ; 计数器置 0 (ZF=1)
.text:0055685C                 neg     eax              ; 寄存器反向互补
.text:0055685E                 and     edx, eax         ; 与运算
.text:00556860                 mov     edi, edx
.text:00556862                 lea     edx, [ebp-30h]   ; 载入有效位址
.text:00556865                 push    edx
.text:00556866                 push    2
.text:00556868                 call    ds:__vbaFreeObjList ; Indirect Call Near Procedure
.text:00556868
.text:0055686E                 add     esp, 0Ch         ; 加法运算
.text:00556871                 test    di, di           ; 两操作数作与运算,逻辑比较
.text:00556874                 jz      short loc_55688F ; 等于 0 则跳 (ZF=1)
.text:00556874
.text:00556876                 mov     word ptr [esi+44h], 3
.text:0055687C                 mov     dword ptr [ebp-1Ch], 40000000h
.text:00556883                 mov     dword ptr [ebp-18h], 417FCA03h
.text:0055688A                 jmp     loc_556A16       ; 跳
.text:0055688A
.text:0055688F     ; ---------------------------------------------------------------------------
.text:0055688F
.text:0055688F     loc_55688F:                          ; CODE XREF: .text:00556874j
.text:0055688F                 mov     eax, [esi]
.text:00556891                 push    esi
.text:00556892                 call    dword ptr [eax+304h] ; Indirect Call Near Procedure
.text:00556892
.text:00556898                 lea     ecx, [ebp-30h]   ; 载入有效位址
.text:0055689B                 push    eax
.text:0055689C                 push    ecx
.text:0055689D                 call    ebx              ; Indirect Call Near Procedure
.text:0055689D
.text:0055689F                 mov     edi, eax
.text:005568A1                 lea     eax, [ebp-0FCh] ; 载入有效位址
.text:005568A7                 push    eax
.text:005568A8                 push    edi
.text:005568A9                 mov     edx, [edi]
.text:005568AB                 call    dword ptr [edx+0E0h] ; Indirect Call Near Procedure
.text:005568AB
```



```asm
.text:005568B1             test    eax, eax        ; 两操作数作与运算,逻辑比较  
.text:005568B3             fnclex                  ; 清除异常 (no wait)  
.text:005568B5             jge     short loc_5568C9 ; 大于或等于则跳 (SF=OF)  
.text:005568B5    
.text:005568B7             push    0E0h  
.text:005568BC             push    offset dword_437A44  
.text:005568C1             push    edi  
.text:005568C2             push    eax  
.text:005568C3             call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  
.text:005568C3    
.text:005568C9    
.text:005568C9     loc_5568C9:                     ; CODE XREF: .text:005568B5j  
.text:005568C9             mov     ecx, [esi]  
.text:005568CB             push    esi  
.text:005568CC             call    dword ptr [ecx+300h] ; Indirect Call Near Procedure  
.text:005568CC    
.text:005568D2             lea     edx, [ebp-34h]  ; 载入有效位址  
.text:005568D5             push    eax  
.text:005568D6             push    edx  
.text:005568D7             call    ebx             ; Indirect Call Near Procedure  
.text:005568D7    
.text:005568D9             mov     edi, eax  
.text:005568DB             lea     ecx, [ebp-100h] ; 载入有效位址  
.text:005568E1             push    ecx  
.text:005568E2             push    edi  
.text:005568E3             mov     eax, [edi]  
.text:005568E5             call    dword ptr [eax+0E0h] ; Indirect Call Near Procedure  
.text:005568E5    
.text:005568EB             test    eax, eax        ; 两操作数作与运算,逻辑比较  
.text:005568ED             fnclex                  ; 清除异常 (no wait)  
.text:005568EF             jge     short loc_556903 ; 大于或等于则跳 (SF=OF)  
.text:005568EF    
.text:005568F1             push    0E0h  
.text:005568F6             push    offset dword_437A44  
.text:005568FB             push    edi  
.text:005568FC             push    eax  
.text:005568FD             call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  
.text:005568FD    
.text:00556903    
.text:00556903     loc_556903:                     ; CODE XREF: .text:005568EFj  
.text:00556903             xor     edx, edx        ; 异或清零  
.text:00556905             lea     ecx, [ebp-34h]  ; 载入有效位址  
.text:00556908             cmp     [ebp-100h], dx  ; 比较二个运算单元  
.text:0055690F             push    ecx  
.text:00556910             setz    dl              ; 计数器置 0 (ZF=1)  
.text:00556913             neg     edx             ; 寄存器反向互补  
.text:00556915             xor     eax, eax        ; 异或清零  
.text:00556917             cmp     word ptr [ebp-0FCh], 1 ; 比较二个运算单元  
.text:0055691F             setz    al              ; 计数器置 0 (ZF=1)  
.text:00556922             neg     eax             ; 寄存器反向互补  
.text:00556924             and     edx, eax        ; 与运算  
.text:00556926             mov     edi, edx  
.text:00556928             lea     edx, [ebp-30h]  ; 载入有效位址  
.text:0055692B             push    edx  
.text:0055692C             push    2  
.text:0055692E             call    ds:__vbaFreeObjList ; Indirect Call Near Procedure  
.text:0055692E    
.text:00556934             add     esp, 0Ch        ; 加法运算  
.text:00556937             test    di, di          ; 两操作数作与运算,逻辑比较  
.text:0055693A             jz      short loc_556955 ; 等于 0 则跳 (ZF=1)  
.text:0055693A    
.text:0055693C             mov     word ptr [esi+44h], 1  
.text:00556942             mov     dword ptr [ebp-1Ch], 80000000h  
.text:00556949             mov     dword ptr [ebp-18h], 41653157h  
.text:00556950             jmp     loc_556A16      ; 跳  
.text:00556950    
.text:00556955     ; ---------------------------------------------------------------------------  
.text:00556955    
.text:00556955     loc_556955:                     ; CODE XREF: .text:0055693Aj  
.text:00556955             mov     eax, [esi]  
.text:00556957             push    esi  
.text:00556958             call    dword ptr [eax+304h] ; Indirect Call Near Procedure  
.text:00556958    
.text:0055695E             lea     ecx, [ebp-30h]  ; 载入有效位址  
.text:00556961             push    eax  
.text:00556962             push    ecx  
.text:00556963             call    ebx             ; Indirect Call Near Procedure  
.text:00556963    
.text:00556965             mov     edi, eax  
.text:00556967             lea     eax, [ebp-0FCh] ; 载入有效位址  
.text:0055696D             push    eax  
.text:0055696E             push    edi  
.text:0055696F             mov     edx, [edi]  
.text:00556971             call    dword ptr [edx+0E0h] ; Indirect Call Near Procedure  
.text:00556971    
.text:00556977             test    eax, eax        ; 两操作数作与运算,逻辑比较  
.text:00556979             fnclex                  ; 清除异常 (no wait)  
.text:0055697B             jge     short loc_55698F ; 大于或等于则跳 (SF=OF)  
.text:0055697B    
.text:0055697D             push    0E0h  
.text:00556982             push    offset dword_437A44  
.text:00556987             push    edi  
.text:00556988             push    eax  
.text:00556989             call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  
.text:00556989    
.text:0055698F    
.text:0055698F     loc_55698F:                     ; CODE XREF: .text:0055697Bj  
.text:0055698F             mov     ecx, [esi]  
.text:00556991             push    esi  
.text:00556992             call    dword ptr [ecx+300h] ; Indirect Call Near Procedure  
.text:00556992    
.text:00556998             lea     edx, [ebp-34h]  ; 载入有效位址  
.text:0055699B             push    eax  
.text:0055699C             push    edx  
.text:0055699D             call    ebx             ; Indirect Call Near Procedure  
.text:0055699D    
.text:0055699F             mov     ebx, eax  
.text:005569A1             lea     ecx, [ebp-100h] ; 载入有效位址  
.text:005569A7             push    ecx  
.text:005569A8             push    ebx  
.text:005569A9             mov     eax, [ebx]  
.text:005569AB             call    dword ptr [eax+0E0h] ; Indirect Call Near Procedure  
.text:005569AB    
.text:005569B1             test    eax, eax        ; 两操作数作与运算,逻辑比较  
.text:005569B3             fnclex                  ; 清除异常 (no wait)  
.text:005569B5             jge     short loc_5569C9 ; 大于或等于则跳 (SF=OF)  
.text:005569B5    
.text:005569B7             push    0E0h  
.text:005569BC             push    offset dword_437A44  
.text:005569C1             push    ebx  
.text:005569C2             push    eax  
.text:005569C3             call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure  
.text:005569C3    
.text:005569C9    
.text:005569C9     loc_5569C9:                     ; CODE XREF: .text:005569B5j  
.text:005569C9             xor     edx, edx        ; 异或清零  
.text:005569CB             cmp     word ptr [ebp-100h], 1 ; 比较二个运算单元  
.text:005569D3             lea     ecx, [ebp-34h]  ; 载入有效位址  
.text:005569D6             setz    dl              ; 计数器置 0 (ZF=1)  
.text:005569D9             xor     eax, eax        ; 异或清零  
```



```
.text:005569DB                 push    ecx  
.text:005569DC                 neg     edx               ; 寄存器反向互补  
.text:005569DE                 cmp     [ebp-0FCh], ax  ; 比较二个运算单元  
.text:005569E5                 setz    al                ; 计数器置 0 (ZF=1)  
.text:005569E8                 neg     eax               ; 寄存器反向互补  
.text:005569EA                 and     edx, eax          ; 与运算  
.text:005569EC                 mov     edi, edx  
.text:005569EE                 lea     edx, [ebp-30h]    ; 载入有效位址  
.text:005569F1                 push    edx  
.text:005569F2                 push    2  
.text:005569F4                 call    ds:__vbaFreeObjList ; Indirect Call Near Procedure  
.text:005569FA                 add     esp, 0Ch          ; 加法运算  
.text:005569FD                 test    di, di            ; 两操作数作与运算,逻辑比较  
.text:00556A00                 jz      short loc_556A16 ; 等于 0 则跳 (ZF=1)  
.text:00556A02                 mov     word ptr [esi+44h], 2  
.text:00556A08                 mov     dword ptr [ebp-1Ch], 80000000h  
.text:00556A0F                 mov     dword ptr [ebp-18h], 41753157h  
.text:00556A16     loc_556A16:                            ; CODE XREF: .text:0055688Aj  
.text:00556A16                            ; .text:00556950j ...  
.text:00556A16                 mov     edx, offset aC  ; "c:\\"    ★取 C 盘 ID 作为机器码运算特征码★  
.text:00556A1B                 lea     ecx, [ebp-20h]    ; 载入有效位址  
.text:00556A1E                 call    ds:__vbaStrCopy ; Indirect Call Near Procedure  
.text:00556A24                 lea     eax, [ebp-20h]    ; 载入有效位址  
.text:00556A27                 push    eax  
.text:00556A28                 call    sub_488540        ; ★机器码计算 CALL★有兴趣的朋友可以跟进看看！  
.text:00556A2D                 fabs                ; 取绝对值  
.text:00556A2F                 fadd    ds:dbl_401410     ; 准备加法运算  
.text:00556A35                 lea     ecx, [ebp-48h]    ; 载入有效位址  
.text:00556A38                 lea     edx, [ebp-58h]    ; 载入有效位址  
.text:00556A3B                 push    ecx  
.text:00556A3C                 push    edx  
.text:00556A3D                 fadd    qword ptr [ebp-1Ch] ; 准备加法运算  
.text:00556A40                 mov     dword ptr [ebp-48h], 5  
.text:00556A47                 fstp    qword ptr [ebp-40h] ; 准备储存和出栈  
.text:00556A4A                 fnstsw  ax                ; 储存写入状态 (no wait)  
.text:00556A4C                 test    al, 0Dh           ; 两操作数作与运算,逻辑比较  
.text:00556A4E                 jnz     loc_556F16        ; 不等于 0 时则跳 (ZF=0)  
.text:00556A54                 call    ds:rtcVarStrFromVar ; Indirect Call Near Procedure  
.text:00556A5A                 lea     eax, [ebp-58h]    ; 载入有效位址  
.text:00556A5D                 lea     ecx, [ebp-68h]    ; 载入有效位址  
.text:00556A60                 push    eax  
.text:00556A61                 push    ecx  
.text:00556A62                 call    ds:rtcTrimVar     ; Indirect Call Near Procedure  
.text:00556A68                 lea     edx, [ebp-68h]    ; 载入有效位址  
.text:00556A6B                 mov     dword ptr [ebp-0FCh], 0  
.text:00556A75                 push    edx  
.text:00556A76                 call    ds:__vbaStrVarMove ; Indirect Call Near Procedure  
.text:00556A7C                 mov     ebx, ds:__vbaStrMove  
.text:00556A82                 mov     edx, eax  
.text:00556A84                 lea     ecx, [ebp-24h]    ; 载入有效位址  
.text:00556A87                 call    ebx ; __vbaStrMove ; Indirect Call Near Procedure  
.text:00556A89                 lea     eax, [ebp-0FCh]   ; 载入有效位址  
.text:00556A8F                 lea     ecx, [ebp-24h]    ; 载入有效位址  
.text:00556A92                 push    eax  
.text:00556A93                 push    ecx  
.text:00556A94                 lea     edi, [esi+40h]    ; 载入有效位址  
.text:00556A97                 call    sub_488E70        ; ★注册码计算 CALL★跟进！  
.text:00556A9C                 mov     edx, eax          ; ★真码出现★(此处可以做内存注册机)  
.text:00556A9E                 lea     ecx, [ebp-28h]    ; 载入有效位址  
.text:00556AA1                 call    ebx ; __vbaStrMove ; Indirect Call Near Procedure  
.text:00556AA3                 mov     edx, eax  
.text:00556AA5                 mov     ecx, edi  
.text:00556AA7                 call    ds:__vbaStrCopy ; Indirect Call Near Procedure  
.text:00556AAD                 lea     edx, [ebp-28h]    ; 载入有效位址  
.text:00556AB0                 lea     eax, [ebp-24h]    ; 载入有效位址  
.text:00556AB3                 push    edx  
.text:00556AB4                 lea     ecx, [ebp-20h]    ; 载入有效位址  
.text:00556AB7                 push    eax  
.text:00556AB8                 push    ecx  
.text:00556AB9                 push    3  
.text:00556ABB                 call    ds:__vbaFreeStrList ; Indirect Call Near Procedure  
.text:00556AC1                 mov     ebx, ds:__vbaFreeVarList  
.text:00556AC7                 lea     edx, [ebp-68h]    ; 载入有效位址  
.text:00556ACA                 lea     eax, [ebp-58h]    ; 载入有效位址  
.text:00556ACD                 push    edx  
.text:00556ACE                 lea     ecx, [ebp-48h]    ; 载入有效位址  
.text:00556AD1                 push    eax  
.text:00556AD2                 push    ecx  
.text:00556AD3                 push    3  
.text:00556AD5                 call    ebx ; __vbaFreeVarList ; Indirect Call Near Procedure  
.text:00556AD7                 mov     edx, [esi+3Ch]  
.text:00556ADA                 mov     eax, [edi]  
.text:00556ADC                 add     esp, 20h          ; 加法运算  
.text:00556ADF                 push    edx  
.text:00556AE0                 push    eax  
.text:00556AE1                 call    ds:__vbaStrCmp    ; Indirect Call Near Procedure  
.text:00556AE7                 test    eax, eax          ; 两操作数作与运算,逻辑比较  
.text:00556AE9                 jnz     loc_556D99        ; 不等于 0 时则跳 (ZF=0)  
.text:00556AEF                 movsx   eax, word ptr [esi+44h] ; 先符号扩展,再传送  
.text:00556AF3                 dec     eax               ; 运算前自减 1  
.text:00556AF4                 jz      loc_556CAF        ; 等于 0 则跳 (ZF=1)  
.text:00556AFA                 dec     eax               ; 运算前自减 1  
.text:00556AFB                 jz      loc_556BDA        ; 等于 0 则跳 (ZF=1)  
.text:00556B01                 dec     eax               ; 运算前自减 1  
.text:00556B02                 jnz     loc_556E85        ; 不等于 0 时则跳 (ZF=0)  
.text:00556B08                 or      eax, 0FFFFFFFFh ; 或运算  
.text:00556B0B                 push    esi  
.text:00556B0C                 mov     word_58006E, ax  
.text:00556B12                 mov     word_58006C, ax  
.text:00556B18                 mov     word_580070, ax  
.text:00556B1E                 mov     ecx, [esi]  
.text:00556B20                 call    dword ptr [ecx+6F8h] ; Indirect Call Near Procedure  
.text:00556B26                 mov     edi, ds:__vbaVarDup  
.text:00556B2C                 mov     ecx, 80020004h  
.text:00556B31                 mov     [ebp-70h], ecx  
.text:00556B34                 mov     eax, 0Ah  
```



```assembly
.text:00556B39          mov     [ebp-60h], ecx  

.text:00556B3C          lea     edx, [ebp-0D8h]  ; 载入有效位址  

.text:00556B42          lea     ecx, [ebp-58h]   ; 载入有效位址  

.text:00556B45          mov     [ebp-78h], eax  

.text:00556B48          mov     [ebp-68h], eax  

.text:00556B4B          mov     dword ptr [ebp-0D0h], offset dword_43B714  

.text:00556B55          mov     dword ptr [ebp-0D8h], 8  

.text:00556B5F          call    edi ; __vbaVarDup ; Indirect Call Near Procedure  

.text:00556B5F     

.text:00556B61          lea     edx, [ebp-0C8h]  ; 载入有效位址  

.text:00556B67          lea     ecx, [ebp-48h]   ; 载入有效位址  

.text:00556B6A          mov     dword ptr [ebp-0C0h], offset dword_43BD48  

.text:00556B74          mov     dword ptr [ebp-0C8h], 8  

.text:00556B7E          call    edi ; __vbaVarDup ; Indirect Call Near Procedure  

.text:00556B7E     

.text:00556B80          lea     edx, [ebp-78h]   ; 载入有效位址  

.text:00556B83          lea     eax, [ebp-68h]   ; 载入有效位址  

.text:00556B86          push    edx  

.text:00556B87          lea     ecx, [ebp-58h]   ; 载入有效位址  

.text:00556B8A          push    eax  

.text:00556B8B          push    ecx  

.text:00556B8C          lea     edx, [ebp-48h]   ; 载入有效位址  

.text:00556B8F          push    30h  

.text:00556B91          push    edx  

.text:00556B92          call    ds:rtcMsgBox    ; Indirect Call Near Procedure  

.text:00556B92     

.text:00556B98          lea     eax, [ebp-78h]   ; 载入有效位址  

.text:00556B9B          lea     ecx, [ebp-68h]   ; 载入有效位址  

.text:00556B9E          push    eax  

.text:00556B9F          lea     edx, [ebp-58h]   ; 载入有效位址  

.text:00556BA2          push    ecx  

.text:00556BA3          lea     eax, [ebp-48h]   ; 载入有效位址  

.text:00556BA6          push    edx  

.text:00556BA7          push    eax  

.text:00556BA8          push    4  

.text:00556BAA          call    ebx ; __vbaFreeVarList ; Indirect Call Near Procedure  

.text:00556BAA     

.text:00556BAC          mov     eax, dword_5815D0  

.text:00556BB1          add     esp, 14h         ; 加法运算  

.text:00556BB4          test    eax, eax         ; 两操作数作与运算,逻辑比较  

.text:00556BB6          jnz     short loc_556BC8 ; 不等于 0 时则跳 (ZF=0)  

.text:00556BB6     

.text:00556BB8          push    offset dword_5815D0  

.text:00556BBD          push    offset dword_434B30  

.text:00556BC2          call    ds:__vbaNew2    ; Indirect Call Near Procedure  

.text:00556BC2     

.text:00556BC8     

.text:00556BC8     loc_556BC8:                     ; CODE XREF: .text:00556BB6j  

.text:00556BC8          mov     edi, dword_5815D0  

.text:00556BCE          lea     ecx, [ebp-30h]   ; 载入有效位址  

.text:00556BD1          push    esi  

.text:00556BD2          push    ecx  

.text:00556BD3          mov     ebx, [edi]  

.text:00556BD5          jmp     loc_556D7F       ; 跳  

.text:00556BD5     

.text:00556BDA     ; ---------------------------------------------------------------------------  

●●●●●●● 跟进：.text:00556A97        call    sub_488E70 算法 CALL：●●●●●●●  

.text:00488E70     

.text:00488E70     ; ************************ S U B R O U T I N E ************************  

.text:00488E70     

.text:00488E70     ; Attributes: bp-based frame  

.text:00488E70     

.text:00488E70     sub_488E70      proc near                ; CODE XREF: .text:00484172p  

.text:00488E70                                       ; .text:00556A97p ...  

.text:00488E70     

.text:00488E70     var_1DC         = qword ptr -1DCh  

.text:00488E70     var_180         = qword ptr -180h  

.text:00488E70     var_178         = qword ptr -178h  

.text:00488E70     var_170         = qword ptr -170h  

.text:00488E70     var_138         = dword ptr -138h  

.text:00488E70     var_130         = dword ptr -130h  

.text:00488E70     var_12C         = dword ptr -12Ch  

.text:00488E70     var_124         = dword ptr -124h  

.text:00488E70     var_120         = dword ptr -120h  

.text:00488E70     var_11C         = dword ptr -11Ch  

.text:00488E70     var_118         = dword ptr -118h  

.text:00488E70     var_114         = dword ptr -114h  

.text:00488E70     var_110         = dword ptr -110h  

.text:00488E70     var_10C         = dword ptr -10Ch  

.text:00488E70     var_108         = dword ptr -108h  

.text:00488E70     var_100         = dword ptr -100h  

.text:00488E70     var_FC          = dword ptr -0FCh  

.text:00488E70     var_F8          = dword ptr -0F8h  

.text:00488E70     var_F0          = dword ptr -0F0h  

.text:00488E70     var_EC          = dword ptr -0ECh  

.text:00488E70     var_E8          = dword ptr -0E8h  

.text:00488E70     var_E0          = dword ptr -0E0h  

.text:00488E70     var_DC          = dword ptr -0DCh  

.text:00488E70     var_D8          = dword ptr -0D8h  

.text:00488E70     var_D0          = dword ptr -0D0h  

.text:00488E70     var_CC          = dword ptr -0CCh  

.text:00488E70     var_C8          = dword ptr -0C8h  

.text:00488E70     var_C4          = dword ptr -0C4h  

.text:00488E70     var_C0          = dword ptr -0C0h  

.text:00488E70     var_BC          = dword ptr -0BCh  

.text:00488E70     var_B8          = dword ptr -0B8h  

.text:00488E70     var_B4          = dword ptr -0B4h  

.text:00488E70     var_B0          = dword ptr -0B0h  

.text:00488E70     var_AC          = dword ptr -0ACh  

.text:00488E70     var_A0          = dword ptr -0A0h  

.text:00488E70     var_9C          = dword ptr -9Ch  

.text:00488E70     var_94          = dword ptr -94h  

.text:00488E70     var_90          = byte ptr -90h  

.text:00488E70     var_8C          = dword ptr -8Ch  

.text:00488E70     var_84          = dword ptr -84h  

.text:00488E70     var_7C          = dword ptr -7Ch  

.text:00488E70     var_74          = qword ptr -74h  

.text:00488E70     var_6C          = dword ptr -6Ch  

.text:00488E70     var_68          = dword ptr -68h  

.text:00488E70     var_5C          = dword ptr -5Ch  

.text:00488E70     var_58          = byte ptr -58h  

.text:00488E70     var_54          = qword ptr -54h  

.text:00488E70     var_4C          = dword ptr -4Ch  

.text:00488E70     var_3C          = dword ptr -3Ch  

.text:00488E70     var_38          = dword ptr -38h  

.text:00488E70     var_34          = dword ptr -34h  

.text:00488E70     var_30          = dword ptr -30h  

.text:00488E70     var_2C          = dword ptr -2Ch  

.text:00488E70     var_28          = dword ptr -28h  

.text:00488E70     var_24          = dword ptr -24h  

.text:00488E70     var_20          = dword ptr -20h  

.text:00488E70     var_1C          = dword ptr -1Ch  

.text:00488E70     var_18          = dword ptr -18h  

.text:00488E70     var_14          = dword ptr -14h  

.text:00488E70     var_C           = dword ptr -0Ch  

.text:00488E70     var_8           = dword ptr -8  

.text:00488E70     var_4           = dword ptr -4  

.text:00488E70     arg_0           = dword ptr  8  

.text:00488E70     arg_4           = dword ptr  0Ch  

.text:00488E70     

.text:00488E70 000             push    ebp  

.text:00488E71 004             mov     ebp, esp  

.text:00488E73 004             sub     esp, 0Ch         ; Integer Subtraction  

.text:00488E76 010             push    offset loc_406316  
```



```
.text:00488E7B 014                     mov     eax, large fs:0

.text:00488E81 014                     push    eax

.text:00488E82 018                     mov     large fs:0, esp

.text:00488E89 018                     sub     esp, 16Ch       ; Integer Subtraction

.text:00488E8F 184                     push    ebx

.text:00488E90 188                     push    esi

.text:00488E91 18C                     push    edi

.text:00488E92 190                     mov     [ebp+var_C], esp

.text:00488E95 190                     mov     [ebp+var_8], offset dword_4017C8

.text:00488E9C 190                     mov     esi, ds:__vbaAryConstruct2

.text:00488EA2 190                     xor     eax, eax        ; Logical Exclusive OR

.text:00488EA4 190                     mov     [ebp+var_1C], eax

.text:00488EA7 190                     mov     [ebp+var_20], eax

.text:00488EAA 190                     mov     [ebp+var_B4], eax

.text:00488EB0 190                     mov     [ebp+var_B8], eax

.text:00488EB6 190                     mov     [ebp+var_BC], eax

.text:00488EBC 190                     mov     [ebp+var_C0], eax

.text:00488EC2 190                     mov     [ebp+var_D0], eax

.text:00488EC8 190                     mov     [ebp+var_E0], eax

.text:00488ECE 190                     mov     [ebp+var_F0], eax

.text:00488ED4 190                     mov     [ebp+var_100], eax

.text:00488EDA 190                     mov     [ebp+var_110], eax

.text:00488EE0 190                     mov     [ebp+var_114], eax

.text:00488EE6 190                     mov     [ebp+var_118], eax

.text:00488EEC 190                     mov     [ebp+var_11C], eax

.text:00488EF2 190                     mov     [ebp+var_120], eax

.text:00488EF8 190                     mov     [ebp+var_124], eax

.text:00488EFE 190                     push    3

.text:00488F00 194                     lea     eax, [ebp+var_3C] ; Load Effective Address

.text:00488F03 194                     push    offset dword_436C04

.text:00488F08 198                     push    eax

.text:00488F09 19C                     call    esi ; __vbaAryConstruct2 ; Indirect Call Near Procedure

.text:00488F09

.text:00488F0B 000                     push    3

.text:00488F0D 004                     lea     ecx, [ebp+var_58] ; Load Effective Address

.text:00488F10 004                     push    offset dword_436C20

.text:00488F15 008                     push    ecx

.text:00488F16 00C                     call    esi ; __vbaAryConstruct2 ; Indirect Call Near Procedure

.text:00488F16

.text:00488F18 -190                    push    8

.text:00488F1A -18C                    lea     edx, [ebp+var_74] ; Load Effective Address

.text:00488F1D -18C                    push    offset dword_436C3C

.text:00488F22 -188                    push    edx

.text:00488F23 -184                    call    esi ; __vbaAryConstruct2 ; Indirect Call Near Procedure

.text:00488F23

.text:00488F25 -320                    push    3

.text:00488F27 -31C                    lea     eax, [ebp+var_90] ; Load Effective Address

.text:00488F2D -31C                    push    offset dword_436AFC

.text:00488F32 -318                    push    eax

.text:00488F33 -314                    call    esi ; __vbaAryConstruct2 ; Indirect Call Near Procedure

.text:00488F33

.text:00488F35 -4B0                    push    8

.text:00488F37 -4AC                    lea     ecx, [ebp+var_AC] ; Load Effective Address

.text:00488F3D -4AC                    push    offset dword_436B18

.text:00488F42 -4A8                    push    ecx

.text:00488F43 -4A4                    call    esi ; __vbaAryConstruct2 ; Indirect Call Near Procedure

.text:00488F43

.text:00488F45 -640                    mov     esi, [ebp+arg_0]

.text:00488F48 -640                    mov     edx, [esi]

.text:00488F4A -640                    push    edx

.text:00488F4B -63C                    call    ds:__vbaLenBstr ; Indirect Call Near Procedure

.text:00488F4B

.text:00488F51 -63C                    cmp     eax, 0Ah        ; Compare Two Operands

.text:00488F54 -63C                    jl      short loc_488FAA ; Jump if Less (SF!=OF)

.text:00488F54

.text:00488F56 -63C                    lea     eax, [ebp+var_100] ; Load Effective Address

.text:00488F5C -63C                    push    0Ah

.text:00488F5E -638                    lea     ecx, [ebp+var_D0] ; Load Effective Address

.text:00488F64 -638                    push    eax

.text:00488F65 -634                    push    ecx

.text:00488F66 -630                    mov     [ebp+var_F8], esi

.text:00488F6C -630                    mov     [ebp+var_100], 4008h

.text:00488F76 -630                    call    ds:rtcLeftCharVar ; Indirect Call Near Procedure

.text:00488F76

.text:00488F7C -630                    lea     edx, [ebp+var_D0] ; Load Effective Address

.text:00488F82 -630                    push    edx

.text:00488F83 -62C                    call    ds:__vbaStrVarMove ; Indirect Call Near Procedure

.text:00488F83

.text:00488F89 -62C                    mov     ebx, ds:__vbaStrMove

.text:00488F8F -62C                    mov     edx, eax

.text:00488F91 -62C                    lea     ecx, [ebp+var_B8] ; Load Effective Address

.text:00488F97 -62C                    call    ebx ; __vbaStrMove ; Indirect Call Near Procedure

.text:00488F97

.text:00488F99 -62C                    lea     ecx, [ebp+var_D0] ; Load Effective Address

.text:00488F9F -62C                    call    ds:__vbaFreeVar ; Indirect Call Near Procedure

.text:00488F9F

.text:00488FA5 -62C                    jmp     loc_48902D      ; Jump

.text:00488FA5

.text:00488FAA     ; ---------------------------------------------------------------------------

.text:00488FAA

.text:00488FAA     loc_488FAA:                   ; CODE XREF: sub_488E70+E4j

.text:00488FAA -63C                    dec     eax         ; Decrement by 1

.text:00488FAB -63C                    cmp     eax, 8          ; switch 9 cases

.text:00488FAE -63C                    ja      short loc_489027 ; default

.text:00488FAE

.text:00488FB0 -63C                    jmp     ds:off_489BFC[eax*4] ; switch jump

.text:00488FB0

.text:00488FB7

.text:00488FB7     loc_488FB7:                   ; DATA XREF: sub_488E70+D8Co

.text:00488FB7 -63C                    mov     eax, [esi]      ; case 0x0

.text:00488FB9 -63C                    push    eax

.text:00488FBA -638                    push    offset a123456789 ; "123456789"

.text:00488FBF -634                    jmp     short loc_48900F ; Jump

.text:00488FBF

.text:00488FC1     ; ---------------------------------------------------------------------------

.text:00488FC1

.text:00488FC1     loc_488FC1:                   ; CODE XREF: sub_488E70+140j

.text:00488FC1                                         ; DATA XREF: sub_488E70+D8Co

.text:00488FC1 -63C                    mov     ecx, [esi]      ; case 0x1

.text:00488FC3 -63C                    push    ecx

.text:00488FC4 -638                    push    offset a23456789 ; "23456789"

.text:00488FC9 -634                    jmp     short loc_48900F ; Jump

.text:00488FC9

.text:00488FCB     ; ---------------------------------------------------------------------------

.text:00488FCB

.text:00488FCB     loc_488FCB:                   ; CODE XREF: sub_488E70+140j

.text:00488FCB                                         ; DATA XREF: sub_488E70+D8Co

.text:00488FCB -63C                    mov     edx, [esi]      ; case 0x2

.text:00488FCD -63C                    push    edx

.text:00488FCE -638                    push    offset a3456789 ; "3456789"

.text:00488FD3 -634                    jmp     short loc_48900F ; Jump

.text:00488FD3

.text:00488FD5     ; ---------------------------------------------------------------------------
```



```
.text:00488FD5     loc_488FD5:                           ; CODE XREF: sub_488E70+140j
.text:00488FD5                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FD5 -63C                 mov     eax, [esi]      ; case 0x3
.text:00488FD7 -63C                 push    eax
.text:00488FD8 -638                 push    offset a456789  ; "456789"
.text:00488FDD -634                 jmp     short loc_48900F ; Jump

.text:00488FDF     ; ---------------------------------------------------------------

.text:00488FDF     loc_488FDF:                           ; CODE XREF: sub_488E70+140j
.text:00488FDF                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FDF -63C                 mov     ecx, [esi]      ; case 0x4
.text:00488FE1 -63C                 push    ecx
.text:00488FE2 -638                 push    offset a56789   ; "56789"
.text:00488FE7 -634                 jmp     short loc_48900F ; Jump

.text:00488FE9     ; ---------------------------------------------------------------

.text:00488FE9     loc_488FE9:                           ; CODE XREF: sub_488E70+140j
.text:00488FE9                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FE9 -63C                 mov     edx, [esi]      ; case 0x5
.text:00488FEB -63C                 push    edx
.text:00488FEC -638                 push    offset a6789    ; "6789"
.text:00488FF1 -634                 jmp     short loc_48900F ; Jump

.text:00488FF3     ; ---------------------------------------------------------------

.text:00488FF3     loc_488FF3:                           ; CODE XREF: sub_488E70+140j
.text:00488FF3                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FF3 -63C                 mov     eax, [esi]      ; case 0x6
.text:00488FF5 -63C                 push    eax
.text:00488FF6 -638                 push    offset a789     ; "789"
.text:00488FFB -634                 jmp     short loc_48900F ; Jump

.text:00488FFD     ; ---------------------------------------------------------------

.text:00488FFD     loc_488FFD:                           ; CODE XREF: sub_488E70+140j
.text:00488FFD                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FFD -63C                 mov     ecx, [esi]      ; case 0x7
.text:00488FFF -63C                 push    ecx
.text:00489000 -638                 push    offset a89      ; "89"
.text:00489005 -634                 jmp     short loc_48900F ; Jump

.text:00489007     ; ---------------------------------------------------------------

.text:00489007     loc_489007:                           ; CODE XREF: sub_488E70+140j
.text:00489007                                       ; DATA XREF: sub_488E70+D8Co
.text:00489007 -63C                 mov     edx, [esi]      ; case 0x8
.text:00489009 -63C                 push    edx
.text:0048900A -638                 push    offset dword_436BFC

.text:0048900F     loc_48900F:                           ; CODE XREF: sub_488E70+14Fj
.text:0048900F                                       ; sub_488E70+159j ...
.text:0048900F -634                 call    ds:__vbaStrCat  ; Indirect Call Near Procedure

.text:00489015 -638                 mov     ebx, ds:__vbaStrMove
.text:0048901B -638                 mov     edx, eax
.text:0048901D -638                 lea     ecx, [ebp+var_B8] ; Load Effective Address
.text:00489023 -638                 call    ebx ; __vbaStrMove ; Indirect Call Near Procedure

.text:00489025 -638                 jmp     short loc_48902D ; Jump

.text:00489027     ; ---------------------------------------------------------------

.text:00489027     loc_489027:                           ; CODE XREF: sub_488E70+13Ej
.text:00489027 -63C                 mov     ebx, ds:__vbaStrMove ; default

.text:0048902D     loc_48902D:                           ; CODE XREF: sub_488E70+135j
.text:0048902D                                       ; sub_488E70+1B5j
.text:0048902D -63C                 mov     esi, 1

.text:00489032     loc_489032:                           ; CODE XREF: sub_488E70+3BFj
.text:00489032 -63C                 mov     eax, 0Ah
.text:00489037 -63C                 cmp     esi, eax        ; Compare Two Operands
.text:00489039 -63C                 jg      loc_489234      ; Jump if Greater (ZF=0 & SF=OF)

.text:0048903F -63C                 lea     eax, [ebp+var_B8] ; Load Effective Address
.text:00489045 -63C                 lea     ecx, [ebp+var_D0] ; Load Effective Address
.text:0048904B -63C                 mov     [ebp+var_F8], eax
.text:00489051 -63C                 push    ecx
.text:00489052 -638                 lea     edx, [ebp+var_100] ; Load Effective Address
.text:00489058 -638                 push    esi
.text:00489059 -634                 lea     eax, [ebp+var_E0] ; Load Effective Address
.text:0048905F -634                 push    edx
.text:00489060 -630                 push    eax
.text:00489061 -62C                 mov     [ebp+var_C8], 1
.text:0048906B -62C                 mov     [ebp+var_D0], 2
.text:00489075 -62C                 mov     [ebp+var_100], 4008h
.text:0048907F -62C                 call    ds:rtcMidCharVar ; Indirect Call Near Procedure

.text:00489085 -62C                 lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:0048908B -62C                 push    ecx
.text:0048908C -628                 call    ds:rtcIsNumeric ; Indirect Call Near Procedure

.text:00489092 -628                 mov     di, ax
.text:00489095 -628                 lea     edx, [ebp+var_E0] ; Load Effective Address
.text:0048909B -628                 lea     eax, [ebp+var_D0] ; Load Effective Address
.text:004890A1 -628                 push    edx
.text:004890A2 -624                 push    eax
.text:004890A3 -620                 push    2
.text:004890A5 -61C                 call    ds:__vbaFreeVarList ; Indirect Call Near Procedure

.text:004890AB -628                 add     esp, 0Ch        ; Add
.text:004890AE -634                 test    di, di          ; Logical Compare
.text:004890B1 -634                 jz      loc_489201      ; Jump if Zero (ZF=1)

.text:004890B7 -634                 lea     ecx, [ebp+var_B8] ; Load Effective Address
.text:004890BD -634                 lea     edx, [ebp+var_D0] ; Load Effective Address
.text:004890C3 -634                 mov     [ebp+var_F8], ecx
.text:004890C9 -634                 push    edx
.text:004890CA -630                 lea     eax, [ebp+var_100] ; Load Effective Address
.text:004890D0 -630                 push    esi
.text:004890D1 -62C                 lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:004890D7 -62C                 push    eax
.text:004890D8 -628                 push    ecx
.text:004890D9 -624                 mov     [ebp+var_C8], 1
.text:004890E3 -624                 mov     [ebp+var_D0], 2
.text:004890ED -624                 mov     [ebp+var_100], 4008h
.text:004890F7 -624                 call    ds:rtcMidCharVar ; Indirect Call Near Procedure
```



```
.text:004890FD -624             lea     edx, [ebp+var_E0] ; Load Effective Address  
.text:00489103 -624             lea     eax, [ebp+var_C0] ; Load Effective Address  
.text:00489109 -624             push    edx  
.text:0048910A -620             push    eax  
.text:0048910B -61C             call    ds:__vbaStrVarVal ; Indirect Call Near Procedure  
.text:0048910B     
.text:00489111 -61C             push    eax  
.text:00489112 -618             call    ds:rtcR8ValFromBstr ; Indirect Call Near Procedure  
.text:00489112     
.text:00489118 -618             call    ds:__vbaFpI2    ; Indirect Call Near Procedure  
.text:00489118     
.text:0048911E -618             lea     ecx, [ebp+var_C0] ; Load Effective Address  
.text:00489124 -618             mov     [ebp+var_BC], eax  
.text:0048912A -618             call    ds:__vbaFreeStr ; Indirect Call Near Procedure  
.text:0048912A     
.text:00489130 -618             lea     ecx, [ebp+var_E0] ; Load Effective Address  
.text:00489136 -618             lea     edx, [ebp+var_D0] ; Load Effective Address  
.text:0048913C -618             push    ecx  
.text:0048913D -614             push    edx  
.text:0048913E -610             push    2  
.text:00489140 -60C             call    ds:__vbaFreeVarList ; Indirect Call Near Procedure  
.text:00489140     
.text:00489146 -618             mov     eax, [ebp+arg_4]  
.text:00489149 -618             mov     ecx, [ebp+var_BC]  
.text:0048914F -618             add     esp, 0Ch        ; Add  
.text:00489152 -624             mov     ax, [eax]  
.text:00489155 -624             cmp     cx, ax          ; Compare Two Operands  
.text:00489158 -624             jl      short loc_489169 ; Jump if Less (SF!=OF)  
.text:00489158     
.text:0048915A -624             sub     cx, ax          ; Integer Subtraction  
.text:0048915D -624             jo      loc_489C25      ; Jump if Overflow (OF=1)  
.text:0048915D     
.text:00489163 -624             mov     [ebp+var_BC], ecx  
.text:00489169     
.text:00489169     loc_489169:                    ; CODE XREF: sub_488E70+2E8j  
.text:00489169 -624             lea     edx, [ebp+var_100] ; Load Effective Address  
.text:0048916F -624             lea     eax, [ebp+var_D0] ; Load Effective Address  
.text:00489175 -624             lea     ecx, [ebp+var_BC] ; Load Effective Address  
.text:0048917B -624             push    edx  
.text:0048917C -620             push    eax  
.text:0048917D -61C             mov     [ebp+var_F8], ecx  
.text:00489183 -61C             mov     [ebp+var_100], 4002h  
.text:0048918D -61C             call    ds:rtcVarStrFromVar ; Indirect Call Near Procedure  
.text:0048918D     
.text:00489193 -61C             lea     ecx, [ebp+var_D0] ; Load Effective Address  
.text:00489199 -61C             lea     edx, [ebp+var_E0] ; Load Effective Address  
.text:0048919F -61C             push    ecx  
.text:004891A0 -618             push    edx  
.text:004891A1 -614             call    ds:rtcTrimVar   ; Indirect Call Near Procedure  
.text:004891A1     
.text:004891A7 -614             lea     edi, [esi-1]    ; Load Effective Address  
.text:004891AA -614             cmp     edi, 0Ah        ; Compare Two Operands  
.text:004891AD -614             jb      short loc_4891B5 ; Jump if Below (CF=1)  
.text:004891AD     
.text:004891AF -614             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:004891AF     
.text:004891B5     
.text:004891B5     loc_4891B5:                    ; CODE XREF: sub_488E70+33Dj  
.text:004891B5 -614             lea     eax, [ebp+var_E0] ; Load Effective Address  
.text:004891BB -614             push    eax  
.text:004891BC -610             call    ds:__vbaStrVarMove ; Indirect Call Near Procedure  
.text:004891BC     
.text:004891C2 -610             mov     edx, eax  
.text:004891C4 -610             lea     ecx, [ebp+var_C0] ; Load Effective Address  
.text:004891CA -610             call    ebx ; __vbaStrMove ; Indirect Call Near Procedure  
.text:004891CA     
.text:004891CC -610             mov     ecx, [ebp+var_68]  
.text:004891CF -610             mov     edx, eax  
.text:004891D1 -610             lea     ecx, [ecx+edi*4] ; Load Effective Address  
.text:004891D4 -610             call    ds:__vbaStrCopy ; Indirect Call Near Procedure  
.text:004891D4     
.text:004891DA -610             lea     ecx, [ebp+var_C0] ; Load Effective Address  
.text:004891E0 -610             call    ds:__vbaFreeStr ; Indirect Call Near Procedure  
.text:004891E0     
.text:004891E6 -610             lea     edx, [ebp+var_E0] ; Load Effective Address  
.text:004891EC -610             lea     eax, [ebp+var_D0] ; Load Effective Address  
.text:004891F2 -610             push    edx  
.text:004891F3 -60C             push    eax  
.text:004891F4 -608             push    2  
.text:004891F6 -604             call    ds:__vbaFreeVarList ; Indirect Call Near Procedure  
.text:004891F6     
.text:004891FC -610             add     esp, 0Ch        ; Add  
.text:004891FF -61C             jmp     short loc_489220 ; Jump  
.text:004891FF     
.text:00489201     ; ----------------------------------------------------------
.text:00489201     
.text:00489201     loc_489201:                    ; CODE XREF: sub_488E70+241j  
.text:00489201 -634             lea     edi, [esi-1]    ; Load Effective Address  
.text:00489204 -634             cmp     edi, 0Ah        ; Compare Two Operands  
.text:00489207 -634             jb      short loc_48920F ; Jump if Below (CF=1)  
.text:00489207     
.text:00489209 -634             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:00489209     
.text:0048920F     
.text:0048920F     loc_48920F:                    ; CODE XREF: sub_488E70+397j  
.text:0048920F -634             mov     ecx, [ebp+var_68]  
.text:00489212 -634             mov     edx, offset dword_436AF4  
.text:00489217 -634             lea     ecx, [ecx+edi*4] ; Load Effective Address  
.text:0048921A -634             call    ds:__vbaStrCopy ; Indirect Call Near Procedure  
.text:0048921A     
.text:00489220     
.text:00489220     loc_489220:                    ; CODE XREF: sub_488E70+38Fj  
.text:00489220 -634             mov     eax, 1  
.text:00489225 -634             add     eax, esi        ; Add  
.text:00489227 -634             jo      loc_489C25      ; Jump if Overflow (OF=1)  
.text:00489227     
.text:0048922D -634             mov     esi, eax  
.text:0048922F -634             jmp     loc_489032      ; Jump  
.text:0048922F     
.text:00489234     ; ----------------------------------------------------------
.text:00489234     
.text:00489234     loc_489234:                    ; CODE XREF: sub_488E70+1C9j  
.text:00489234 -63C             mov     esi, 1  
.text:00489239 -63C             mov     [ebp+var_18], esi  
.text:0048923C     
.text:0048923C     loc_48923C:                    ; CODE XREF: sub_488E70+78Dj  
.text:0048923C -63C             mov     eax, 9  
.text:00489241 -63C             cmp     esi, eax        ; Compare Two Operands  
.text:00489243 -63C             jg      loc_489602      ; Jump if Greater (ZF=0 & SF=OF)  
.text:00489243     
.text:00489249 -63C             dec     esi             ; Decrement by 1  
.text:0048924A -63C             cmp     esi, 0Ah        ; Compare Two Operands
```



```
.text:0048924D -63C             jb      short loc_489255 ; Jump if Below (CF=1)  

.text:0048924D      

.text:0048924F -63C             call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  

.text:0048924F      

.text:00489255      

.text:00489255     loc_489255:                     ; CODE XREF: sub_488E70+3DDj  

.text:00489255 -63C             mov     edi, [ebp+var_18]  

.text:00489258 -63C             add     edi, 1          ; Add  

.text:0048925B -63C             jo      loc_489C25      ; Jump if Overflow (OF=1)  

.text:0048925B      

.text:00489261 -63C             dec     edi             ; Decrement by 1  

.text:00489262 -63C             cmp     edi, 0Ah        ; Compare Two Operands  

.text:00489265 -63C             jb      short loc_48926D ; Jump if Below (CF=1)  

.text:00489265      

.text:00489267 -63C             call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  

.text:00489267      

.text:0048926D      

.text:0048926D     loc_48926D:                     ; CODE XREF: sub_488E70+3F5j  

.text:0048926D -63C             mov     edx, [ebp+var_68]  

.text:00489270 -63C             mov     eax, [edx+esi*4]  

.text:00489273 -63C             push    eax  

.text:00489274 -638             call    ds:___vbaR8Str    ; Indirect Call Near Procedure  

.text:00489274      

.text:0048927A -638             fmul    ds:dbl_4017C0    ; Multiply Real  

.text:00489280 -638             mov     ecx, [ebp+var_68]  

.text:00489283 -638             mov     edx, [ecx+edi*4]  

.text:00489286 -638             fnstsw  ax              ; Store Status Word (no wait)  

.text:00489288 -638             test    al, 0Dh         ; Logical Compare  

.text:0048928A -638             jnz     loc_489C20      ; Jump if Not Zero (ZF=0)  

.text:0048928A      

.text:00489290 -638             fstp    [ebp+var_170]    ; Store Real and Pop  

.text:00489296 -638             push    edx  

.text:00489297 -634             call    ds:___vbaR8Str    ; Indirect Call Near Procedure  

.text:00489297      

.text:0048929D -634             fadd    [ebp+var_170]    ; Add Real  

.text:004892A3 -634             sub     esp, 8          ; Integer Subtraction  

.text:004892A6 -62C             fnstsw  ax              ; Store Status Word (no wait)  

.text:004892A8 -62C             test    al, 0Dh         ; Logical Compare  

.text:004892AA -62C             jnz     loc_489C20      ; Jump if Not Zero (ZF=0)  

.text:004892AA      

.text:004892B0 -62C             fstp    qword ptr [esp+0] ; Store Real and Pop  

.text:004892B3 -62C             call    ds:___vbaStrR8    ; Indirect Call Near Procedure  

.text:004892B3      

.text:004892B9 -62C             mov     edx, eax  

.text:004892BB -62C             lea     ecx, [ebp+var_C0] ; Load Effective Address  

.text:004892C1 -62C             call    ebx ; ___vbaStrMove ; Indirect Call Near Procedure  

.text:004892C1      

.text:004892C3 -62C             push    eax  

.text:004892C4 -628             call    ds:rtcR8ValFromBstr ; Indirect Call Near Procedure  

.text:004892C4      

.text:004892CA -628             call    ds:___vbaFpI4     ; Indirect Call Near Procedure  

.text:004892CA      

.text:004892D0 -628             mov     edi, eax  

.text:004892D2 -628             and     edi, 80000001h  ; Logical AND  

.text:004892D8 -628             jns     short loc_4892DF ; Jump if Not Sign (SF=0)  

.text:004892D8      

.text:004892DA -628             dec     edi             ; Decrement by 1  

.text:004892DB -628             or      edi, 0FFFFFFFEh ; Logical Inclusive OR  

.text:004892DE -628             inc     edi             ; Increment by 1  

.text:004892DF      

.text:004892DF     loc_4892DF:                     ; CODE XREF: sub_488E70+468j  

.text:004892DF -628             neg     edi             ; Two's Complement Negation  

.text:004892E1 -628             sbb     edi, edi        ; Integer Subtraction with Borrow  

.text:004892E3 -628             lea     ecx, [ebp+var_C0] ; Load Effective Address  

.text:004892E9 -628             inc     edi             ; Increment by 1  

.text:004892EA -628             neg     edi             ; Two's Complement Negation  

.text:004892EC -628             call    ds:___vbaFreeStr ; Indirect Call Near Procedure  

.text:004892EC      

.text:004892F2 -628             test    di, di          ; Logical Compare  

.text:004892F5 -628             jz      short loc_48935D ; Jump if Zero (ZF=1)  

.text:004892F5      

.text:004892F7 -628             cmp     esi, 0Ah        ; Compare Two Operands  

.text:004892FA -628             jb      short loc_489302 ; Jump if Below (CF=1)  

.text:004892FA      

.text:004892FC -628             call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  

.text:004892FC      

.text:00489302      

.text:00489302     loc_489302:                     ; CODE XREF: sub_488E70+48Aj  

.text:00489302 -628             mov     edi, [ebp+var_18]  

.text:00489305 -628             add     edi, 1          ; Add  

.text:00489308 -628             jo      loc_489C25      ; Jump if Overflow (OF=1)  

.text:00489308      

.text:0048930E -628             dec     edi             ; Decrement by 1  

.text:0048930F -628             cmp     edi, 0Ah        ; Compare Two Operands  

.text:00489312 -628             jb      short loc_48931A ; Jump if Below (CF=1)  

.text:00489312      

.text:00489314 -628             call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  

.text:00489314      

.text:0048931A      

.text:0048931A     loc_48931A:                     ; CODE XREF: sub_488E70+4A2j  

.text:0048931A -628             cmp     esi, 9          ; Compare Two Operands  

.text:0048931D -628             jb      short loc_489325 ; Jump if Below (CF=1)  

.text:0048931D      

.text:0048931F -628             call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  

.text:0048931F      

.text:00489325      

.text:00489325     loc_489325:                     ; CODE XREF: sub_488E70+4ADj  

.text:00489325 -628             mov     eax, [ebp+var_68]  

.text:00489328 -628             mov     ecx, [eax+esi*4]  

.text:0048932B -628             push    ecx  

.text:0048932C -624             call    ds:___vbaR8Str    ; Indirect Call Near Procedure  

.text:0048932C      

.text:00489332 -624             fmul    ds:dbl_4017C0    ; Multiply Real  

.text:00489338 -624             mov     edx, [ebp+var_68]  

.text:0048933B -624             fnstsw  ax              ; Store Status Word (no wait)  

.text:0048933D -624             test    al, 0Dh         ; Logical Compare  

.text:0048933F -624             jnz     loc_489C20      ; Jump if Not Zero (ZF=0)  

.text:0048933F      

.text:00489345 -624             mov     eax, [edx+edi*4]  

.text:00489348 -624             fstp    [ebp+var_178]    ; Store Real and Pop  

.text:0048934E -624             push    eax  

.text:0048934F -620             call    ds:___vbaR8Str    ; Indirect Call Near Procedure  

.text:0048934F      

.text:00489355 -620             fadd    [ebp+var_178]    ; Add Real  

.text:0048935B -620             jmp     short loc_4893C7 ; Jump  

.text:0048935B      

.text:0048935D     ; ---------------------------------------------------------------------------  

.text:0048935D      

.text:0048935D     loc_48935D:                     ; CODE XREF: sub_488E70+485j  

.text:0048935D -628             cmp     esi, 0Ah        ; Compare Two Operands  

.text:00489360 -628             jb      short loc_489368 ; Jump if Below (CF=1)
```



```
.text:00489360      
.text:00489362 -628        call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:00489362      
.text:00489368      
.text:00489368      loc_489368:                     ; CODE XREF: sub_488E70+4F0j  
.text:00489368 -628        mov     edi, [ebp+var_18]  
.text:0048936B -628        add     edi, 1           ; Add  
.text:0048936E -628        jo      loc_489C25       ; Jump if Overflow (OF=1)  
.text:0048936E      
.text:00489374 -628        dec     edi              ; Decrement by 1  
.text:00489375 -628        cmp     edi, 0Ah         ; Compare Two Operands  
.text:00489378 -628        jb      short loc_489380 ; Jump if Below (CF=1)  
.text:00489378      
.text:0048937A -628        call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:0048937A      
.text:00489380      
.text:00489380      loc_489380:                     ; CODE XREF: sub_488E70+508j  
.text:00489380 -628        cmp     esi, 9           ; Compare Two Operands  
.text:00489383 -628        jb      short loc_48938B ; Jump if Below (CF=1)  
.text:00489383      
.text:00489385 -628        call    ds:___vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:00489385      
.text:0048938B      
.text:0048938B      loc_48938B:                     ; CODE XREF: sub_488E70+513j  
.text:0048938B -628        mov     edx, [ebp+var_68]  
.text:0048938E -628        mov     eax, [edx+esi*4]  
.text:00489391 -628        push    eax  
.text:00489392 -624        call    ds:___vbaR8Str   ; Indirect Call Near Procedure  
.text:00489392      
.text:00489398 -624        fmul    ds:dbl_4017C0   ; Multiply Real  
.text:0048939E -624        mov     ecx, [ebp+var_68]  
.text:004893A1 -624        mov     edx, [ecx+edi*4]  
.text:004893A4 -624        fnstsw  ax               ; Store Status Word (no wait)  
.text:004893A6 -624        test    al, 0Dh          ; Logical Compare  
.text:004893A8 -624        jnz     loc_489C20       ; Jump if Not Zero (ZF=0)  
.text:004893A8      
.text:004893AE -624        fstp    [ebp+var_180]   ; Store Real and Pop  
.text:004893B4 -624        push    edx  
.text:004893B5 -620        call    ds:___vbaR8Str   ; Indirect Call Near Procedure  
.text:004893B5      
.text:004893BB -620        fadd    [ebp+var_180]   ; Add Real  
.text:004893C1 -620        fsub    ds:dbl_4017B8   ; Subtract Real  
.text:004893C7      
.text:004893C7      loc_4893C7:                     ; CODE XREF: sub_488E70+4EBj  
.text:004893C7 -620        cmp     dword_580000, 0 ; Compare Two Operands  
.text:004893CE -620        jnz     short loc_4893D8 ; Jump if Not Zero (ZF=0)  
.text:004893CE      
.text:004893D0 -620        fdiv    ds:dbl_4017B0   ; Divide Real  
.text:004893D6 -620        jmp     short loc_4893E9 ; Jump  
.text:004893D6      
.text:004893D8      ; ---------------------------------------------------------------------------  
.text:004893D8      
.text:004893D8      loc_4893D8:                     ; CODE XREF: sub_488E70+55Ej  
.text:004893D8 -620        push    dword ptr ds:dbl_4017B0+4  
.text:004893DE -61C        push    dword ptr ds:dbl_4017B0  
.text:004893E4 -618        call    _adj_fdiv_m64   ; Call Procedure  
.text:004893E4      
.text:004893E9      
.text:004893E9      loc_4893E9:                     ; CODE XREF: sub_488E70+566j  
.text:004893E9 -618        fnstsw  ax               ; Store Status Word (no wait)  
.text:004893EB -618        test    al, 0Dh          ; Logical Compare  
.text:004893ED -618        jnz     loc_489C20       ; Jump if Not Zero (ZF=0)  
.text:004893ED      
.text:004893F3 -618        call    ds:___vbaFpI4    ; Indirect Call Near Procedure  
.text:004893F3      
.text:004893F9 -618        mov     ecx, [ebp+var_30]  
.text:004893FC -618        mov     edi, ds:___vbaGenerateBoundsError  
.text:00489402 -618        cmp     esi, 9           ; Compare Two Operands  
.text:00489405 -618        mov     [ecx+esi*4], eax  
.text:00489408 -618        jb      short loc_48940C ; Jump if Below (CF=1)  
.text:00489408      
.text:0048940A -618        call    edi ; ___vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:0048940A      
.text:0048940C      
.text:0048940C      loc_48940C:                     ; CODE XREF: sub_488E70+598j  
.text:0048940C -618        mov     edx, [ebp+var_30]  
.text:0048940F -618        cmp     dword ptr [edx+esi*4], 0Ah ; Compare Two Operands  
.text:00489413 -618        jl      short loc_489494 ; Jump if Less (SF!=OF)  
.text:00489413      
.text:00489415 -618        cmp     esi, 9           ; Compare Two Operands  
.text:00489418 -618        jb      short loc_48941C ; Jump if Below (CF=1)  
.text:00489418      
.text:0048941A -618        call    edi ; ___vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:0048941A      
.text:0048941C      
.text:0048941C      loc_48941C:                     ; CODE XREF: sub_488E70+5A8j  
.text:0048941C -618        mov     eax, [ebp+var_30]  
.text:0048941F -618        lea     edx, [ebp+var_100] ; Load Effective Address  
.text:00489425 -618        push    edx  
.text:00489426 -614        mov     [ebp+var_100], 4003h  
.text:00489430 -614        lea     ecx, [eax+esi*4] ; Load Effective Address  
.text:00489433 -614        lea     eax, [ebp+var_D0] ; Load Effective Address  
.text:00489439 -614        push    eax  
.text:0048943A -610        mov     [ebp+var_F8], ecx  
.text:00489440 -610        call    ds:rtcVarStrFromVar ; Indirect Call Near Procedure  
.text:00489440      
.text:00489446 -610        cmp     esi, 9           ; Compare Two Operands  
.text:00489449 -610        jb      short loc_48944D ; Jump if Below (CF=1)  
.text:00489449      
.text:0048944B -610        call    edi ; ___vbaGenerateBoundsError ; Indirect Call Near Procedure  
.text:0048944B      
.text:0048944D      
.text:0048944D      loc_48944D:                     ; CODE XREF: sub_488E70+5D9j  
.text:0048944D -610        mov     edi, ds:___vbaStrVarMove  
.text:00489453 -610        lea     ecx, [ebp+var_D0] ; Load Effective Address  
.text:00489459 -610        push    ecx  
.text:0048945A -60C        call    edi ; ___vbaStrVarMove ; Indirect Call Near Procedure  
.text:0048945A      
.text:0048945C -60C        mov     edx, eax  
.text:0048945E -60C        lea     ecx, [ebp+var_C0] ; Load Effective Address  
.text:00489464 -60C        call    ebx ; ___vbaStrMove ; Indirect Call Near Procedure  
.text:00489464      
.text:00489466 -60C        mov     edx, eax  
.text:00489468 -60C        mov     eax, [ebp+var_A0]  
.text:0048946E -60C        lea     ecx, [eax+esi*4] ; Load Effective Address  
.text:00489471 -60C        call    ds:___vbaStrCopy ; Indirect Call Near Procedure  
.text:00489471      
.text:00489477 -60C        lea     ecx, [ebp+var_C0] ; Load Effective Address  
.text:0048947D -60C        call    ds:___vbaFreeStr ; Indirect Call Near Procedure  
.text:0048947D      
.text:00489483 -60C        lea     ecx, [ebp+var_D0] ; Load Effective Address  
```



```
.text:00489489 -60C              call    ds:\_\_vbaFreeVar ; Indirect Call Near Procedure  

.text:00489489   

.text:0048948F -60C              jmp     loc\_48955F      ; Jump  

.text:0048948F   

.text:00489494     ; ---------------------------------------------------------------------------  

.text:00489494   

.text:00489494     loc\_489494:                       ; CODE XREF: sub\_488E70+5A3j  

.text:00489494 -618              cmp     esi, 9          ; Compare Two Operands  

.text:00489497 -618              mov     \[ebp+var\_108\], offset dword\_436AF4  

.text:004894A1 -618              mov     \[ebp+var\_110\], 8  

.text:004894AB -618              jb      short loc\_4894AF ; Jump if Below (CF=1)  

.text:004894AB   

.text:004894AD -618              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:004894AD   

.text:004894AF   

.text:004894AF     loc\_4894AF:                       ; CODE XREF: sub\_488E70+63Bj  

.text:004894AF -618              mov     ecx, \[ebp+var\_30\]  

.text:004894B2 -618              lea     eax, \[ebp+var\_100\] ; Load Effective Address  

.text:004894B8 -618              push    eax  

.text:004894B9 -614              mov     \[ebp+var\_100\], 4003h  

.text:004894C3 -614              lea     edx, \[ecx+esi\*4\] ; Load Effective Address  

.text:004894C6 -614              lea     ecx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004894CC -614              push    ecx  

.text:004894CD -610              mov     \[ebp+var\_F8\], edx  

.text:004894D3 -610              call    ds:rtcVarStrFromVar ; Indirect Call Near Procedure  

.text:004894D3   

.text:004894D9 -610              lea     edx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004894DF -610              lea     eax, \[ebp+var\_E0\] ; Load Effective Address  

.text:004894E5 -610              push    edx  

.text:004894E6 -60C              push    eax  

.text:004894E7 -608              call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:004894E7   

.text:004894ED -608              cmp     esi, 9          ; Compare Two Operands  

.text:004894F0 -608              jb      short loc\_4894F4 ; Jump if Below (CF=1)  

.text:004894F0   

.text:004894F2 -608              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:004894F2   

.text:004894F4   

.text:004894F4     loc\_4894F4:                       ; CODE XREF: sub\_488E70+680j  

.text:004894F4 -608              lea     ecx, \[ebp+var\_110\] ; Load Effective Address  

.text:004894FA -608              lea     edx, \[ebp+var\_E0\] ; Load Effective Address  

.text:00489500 -608              push    ecx  

.text:00489501 -604              lea     eax, \[ebp+var\_F0\] ; Load Effective Address  

.text:00489507 -604              push    edx  

.text:00489508 -600              push    eax  

.text:00489509 -5FC              call    ds:\_\_vbaVarCat  ; Indirect Call Near Procedure  

.text:00489509   

.text:0048950F -5FC              mov     edi, ds:\_\_vbaStrVarMove  

.text:00489515 -5FC              push    eax  

.text:00489516 -5F8              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:00489516   

.text:00489518 -5F8              mov     edx, eax  

.text:0048951A -5F8              lea     ecx, \[ebp+var\_C0\] ; Load Effective Address  

.text:00489520 -5F8              call    ebx ; \_\_vbaStrMove ; Indirect Call Near Procedure  

.text:00489520   

.text:00489522 -5F8              mov     ecx, \[ebp+var\_A0\]  

.text:00489528 -5F8              mov     edx, eax  

.text:0048952A -5F8              lea     ecx, \[ecx+esi\*4\] ; Load Effective Address  

.text:0048952D -5F8              call    ds:\_\_vbaStrCopy ; Indirect Call Near Procedure  

.text:0048952D   

.text:00489533 -5F8              lea     ecx, \[ebp+var\_C0\] ; Load Effective Address  

.text:00489539 -5F8              call    ds:\_\_vbaFreeStr ; Indirect Call Near Procedure  

.text:00489539   

.text:0048953F -5F8              lea     edx, \[ebp+var\_F0\] ; Load Effective Address  

.text:00489545 -5F8              lea     eax, \[ebp+var\_E0\] ; Load Effective Address  

.text:0048954B -5F8              push    edx  

.text:0048954C -5F4              lea     ecx, \[ebp+var\_D0\] ; Load Effective Address  

.text:00489552 -5F4              push    eax  

.text:00489553 -5F0              push    ecx  

.text:00489554 -5EC              push    3  

.text:00489556 -5E8              call    ds:\_\_vbaFreeVarList ; Indirect Call Near Procedure  

.text:00489556   

.text:0048955C -5F4              add     esp, 10h        ; Add  

.text:0048955F   

.text:0048955F     loc\_48955F:                       ; CODE XREF: sub\_488E70+61Fj  

.text:0048955F -604              mov     edx, \[ebp+var\_1C\]  

.text:00489562 -604              cmp     esi, 9          ; Compare Two Operands  

.text:00489565 -604              mov     \[ebp+var\_108\], edx  

.text:0048956B -604              mov     \[ebp+var\_110\], 8  

.text:00489575 -604              jb      short loc\_48957D ; Jump if Below (CF=1)  

.text:00489575   

.text:00489577 -604              call    ds:\_\_vbaGenerateBoundsError ; Indirect Call Near Procedure  

.text:00489577   

.text:0048957D   

.text:0048957D     loc\_48957D:                       ; CODE XREF: sub\_488E70+705j  

.text:0048957D -604              mov     eax, \[ebp+var\_A0\]  

.text:00489583 -604              lea     edx, \[ebp+var\_100\] ; Load Effective Address  

.text:00489589 -604              push    edx  

.text:0048958A -600              mov     \[ebp+var\_100\], 4008h  

.text:00489594 -600              lea     ecx, \[eax+esi\*4\] ; Load Effective Address  

.text:00489597 -600              lea     eax, \[ebp+var\_D0\] ; Load Effective Address  

.text:0048959D -600              push    eax  

.text:0048959E -5FC              mov     \[ebp+var\_F8\], ecx  

.text:004895A4 -5FC              call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:004895A4   

.text:004895AA -5FC              lea     ecx, \[ebp+var\_110\] ; Load Effective Address  

.text:004895B0 -5FC              lea     edx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004895B6 -5FC              push    ecx  

.text:004895B7 -5F8              lea     eax, \[ebp+var\_E0\] ; Load Effective Address  

.text:004895BD -5F8              push    edx  

.text:004895BE -5F4              push    eax  

.text:004895BF -5F0              call    ds:\_\_vbaVarCat  ; Indirect Call Near Procedure  

.text:004895BF   

.text:004895C5 -5F0              push    eax  

.text:004895C6 -5EC              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:004895C6   

.text:004895C8 -5EC              mov     edx, eax  

.text:004895CA -5EC              lea     ecx, \[ebp+var\_1C\] ; Load Effective Address  

.text:004895CD -5EC              call    ebx ; \_\_vbaStrMove ; Indirect Call Near Procedure  

.text:004895CD   

.text:004895CF -5EC              lea     ecx, \[ebp+var\_E0\] ; Load Effective Address  

.text:004895D5 -5EC              lea     edx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004895DB -5EC              push    ecx  

.text:004895DC -5E8              push    edx  

.text:004895DD -5E4              push    2  

.text:004895DF -5E0              call    ds:\_\_vbaFreeVarList ; Indirect Call Near Procedure  

.text:004895DF   

.text:004895E5 -5EC              mov     ecx, \[ebp+var\_18\]  

.text:004895E8 -5EC              mov     eax, 1  

.text:004895ED -5EC              add     esp, 0Ch        ; Add  

.text:004895F0 -5F8              add     eax, ecx        ; Add  
```



```
.text:004895F2 -5F8              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004895F2
.text:004895F8 -5F8              mov     [ebp+var_18], eax
.text:004895FB -5F8              mov     esi, eax
.text:004895FD -5F8              jmp     loc_48923C      ; Jump
.text:004895FD
.text:00489602   ; ---------------------------------------------------------------------------
.text:00489602
.text:00489602   loc_489602:                       ; CODE XREF: sub_488E70+3D3j
.text:00489602 -63C              mov     esi, 1
.text:00489607
.text:00489607   loc_489607:                       ; CODE XREF: sub_488E70+850j
.text:00489607 -63C              mov     eax, 12h
.text:0048960C -63C              cmp     esi, eax        ; Compare Two Operands
.text:0048960E -63C              jg      loc_4896C5      ; Jump if Greater (ZF=0 & SF=OF)
.text:0048960E
.text:00489614 -63C              lea     eax, [ebp+var_1C] ; Load Effective Address
.text:00489617 -63C              lea     ecx, [ebp+var_D0] ; Load Effective Address
.text:0048961D -63C              mov     [ebp+var_F8], eax
.text:00489623 -63C              push    ecx
.text:00489624 -638              lea     edx, [ebp+var_100] ; Load Effective Address
.text:0048962A -638              push    esi
.text:0048962B -634              lea     eax, [ebp+var_E0] ; Load Effective Address
.text:00489631 -634              push    edx
.text:00489632 -630              push    eax
.text:00489633 -62C              mov     [ebp+var_C8], 1
.text:0048963D -62C              mov     [ebp+var_D0], 2
.text:00489647 -62C              mov     [ebp+var_100], 4008h
.text:00489651 -62C              call    ds:rtcMidCharVar ; Indirect Call Near Procedure
.text:00489651
.text:00489657 -62C              lea     edi, [esi-1]    ; Load Effective Address
.text:0048965A -62C              cmp     edi, 12h        ; Compare Two Operands
.text:0048965D -62C              jb      short loc_489665 ; Jump if Below (CF=1)
.text:0048965D
.text:0048965F -62C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048965F
.text:00489665
.text:00489665   loc_489665:                       ; CODE XREF: sub_488E70+7EDj
.text:00489665 -62C              lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:0048966B -62C              lea     edx, [ebp+var_C0] ; Load Effective Address
.text:00489671 -62C              push    ecx
.text:00489672 -628              push    edx
.text:00489673 -624              call    ds:__vbaStrVarVal ; Indirect Call Near Procedure
.text:00489673
.text:00489679 -624              push    eax
.text:0048967A -620              call    ds:rtcR8ValFromBstr ; Indirect Call Near Procedure
.text:0048967A
.text:00489680 -620              call    ds:__vbaFpI4    ; Indirect Call Near Procedure
.text:00489680
.text:00489686 -620              mov     ecx, [ebp+var_4C]
.text:00489689 -620              mov     [ecx+edi*4], eax
.text:0048968C -620              lea     ecx, [ebp+var_C0] ; Load Effective Address
.text:00489692 -620              call    ds:__vbaFreeStr ; Indirect Call Near Procedure
.text:00489692
.text:00489698 -620              lea     edx, [ebp+var_E0] ; Load Effective Address
.text:0048969E -620              lea     eax, [ebp+var_D0] ; Load Effective Address
.text:004896A4 -620              push    edx
.text:004896A5 -61C              push    eax
.text:004896A6 -618              push    2
.text:004896A8 -614              call    ds:__vbaFreeVarList ; Indirect Call Near Procedure
.text:004896A8
.text:004896AE -620              mov     eax, 1
.text:004896B3 -620              add     esp, 0Ch        ; Add
.text:004896B6 -62C              add     eax, esi        ; Add
.text:004896B8 -62C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004896B8
.text:004896BE -62C              mov     esi, eax
.text:004896C0 -62C              jmp     loc_489607      ; Jump
.text:004896C0
.text:004896C5   ; ---------------------------------------------------------------------------
.text:004896C5
.text:004896C5   loc_4896C5:                       ; CODE XREF: sub_488E70+79Ej
.text:004896C5 -63C              mov     edi, 1
.text:004896CA
.text:004896CA   loc_4896CA:                       ; CODE XREF: sub_488E70+BE5j
.text:004896CA -63C              mov     eax, 10h
.text:004896CF -63C              cmp     edi, eax        ; Compare Two Operands
.text:004896D1 -63C              jg      loc_489A5A      ; Jump if Greater (ZF=0 & SF=OF)
.text:004896D1
.text:004896D7 -63C              lea     esi, [edi-1]    ; Load Effective Address
.text:004896DA -63C              cmp     esi, 12h        ; Compare Two Operands
.text:004896DD -63C              jb      short loc_4896E5 ; Jump if Below (CF=1)
.text:004896DD
.text:004896DF -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004896DF
.text:004896E5
.text:004896E5   loc_4896E5:                       ; CODE XREF: sub_488E70+86Dj
.text:004896E5 -63C              mov     eax, edi
.text:004896E7 -63C              add     eax, 1          ; Add
.text:004896EA -63C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004896EA
.text:004896F0 -63C              dec     eax             ; Decrement by 1
.text:004896F1 -63C              cmp     eax, 12h        ; Compare Two Operands
.text:004896F4 -63C              mov     [ebp+var_12C], eax
.text:004896FA -63C              jb      short loc_489702 ; Jump if Below (CF=1)
.text:004896FA
.text:004896FC -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004896FC
.text:00489702
.text:00489702   loc_489702:                       ; CODE XREF: sub_488E70+88Aj
.text:00489702 -63C              mov     eax, edi
.text:00489704 -63C              add     eax, 2          ; Add
.text:00489707 -63C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489707
.text:0048970D -63C              dec     eax             ; Decrement by 1
.text:0048970E -63C              cmp     eax, 12h        ; Compare Two Operands
.text:00489711 -63C              mov     [ebp+var_130], eax
.text:00489717 -63C              jb      short loc_48971F ; Jump if Below (CF=1)
.text:00489717
.text:00489719 -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489719
.text:0048971F
.text:0048971F   loc_48971F:                       ; CODE XREF: sub_488E70+8A7j
.text:0048971F -63C              cmp     esi, 12h        ; Compare Two Operands
.text:00489722 -63C              jb      short loc_48972A ; Jump if Below (CF=1)
.text:00489722
.text:00489724 -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489724
.text:0048972A
.text:0048972A   loc_48972A:                       ; CODE XREF: sub_488E70+8B2j
.text:0048972A -63C              mov     eax, edi
.text:0048972C -63C              add     eax, 1          ; Add
.text:0048972F -63C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:0048972F
.text:00489735 -63C              dec     eax             ; Decrement by 1
```



```
.text:00489736 -63C        cmp     eax, 12h        ; Compare Two Operands
.text:00489739 -63C        mov     [ebp+var_138], eax
.text:0048973F -63C        jb      short loc_489747 ; Jump if Below (CF=1)
.text:00489741
.text:00489741 -63C        call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489747
.text:00489747     loc_489747:                   ; CODE XREF: sub_488E70+8CFj
.text:00489747 -63C        mov     ebx, edi
.text:00489749 -63C        add     ebx, 2          ; Add
.text:0048974C -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489752
.text:00489752 -63C        dec     ebx             ; Decrement by 1
.text:00489753 -63C        cmp     ebx, 12h        ; Compare Two Operands
.text:00489756 -63C        jb      short loc_48975E ; Jump if Below (CF=1)
.text:00489758
.text:00489758 -63C        call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048975E
.text:0048975E     loc_48975E:                   ; CODE XREF: sub_488E70+8E6j
.text:0048975E -63C        mov     eax, [ebp+var_4C]
.text:00489761 -63C        mov     ecx, [ebp+var_138]
.text:00489767 -63C        mov     edx, [eax+ecx*4]
.text:0048976A -63C        mov     ecx, [eax+esi*4]
.text:0048976D -63C        add     edx, ecx        ; Add
.text:0048976F -63C        mov     ecx, [eax+ebx*4]
.text:00489772 -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489778
.text:00489778 -63C        add     edx, ecx        ; Add
.text:0048977A -63C        mov     ebx, [eax+esi*4]
.text:0048977D -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489783
.text:00489783 -63C        xor     ecx, ecx        ; Logical Exclusive OR
.text:00489785 -63C        cmp     edx, 14h        ; Compare Two Operands
.text:00489788 -63C        mov     edx, [ebp+var_12C]
.text:0048978E -63C        setnl   cl              ; Set Byte if Not Less or Equal (ZF=0 & SF=OF)
.text:00489791 -63C        mov     edx, [eax+edx*4]
.text:00489794 -63C        add     edx, ebx        ; Add
.text:00489796 -63C        mov     ebx, [ebp+var_130]
.text:0048979C -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004897A2
.text:004897A2 -63C        add     edx, [eax+ebx*4] ; Add
.text:004897A5 -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004897AB
.text:004897AB -63C        xor     ebx, ebx        ; Logical Exclusive OR
.text:004897AD -63C        cmp     edx, 0Ah        ; Compare Two Operands
.text:004897B0 -63C        setl    bl              ; Set Byte if Less (SF!=OF)
.text:004897B3 -63C        or      ecx, ebx        ; Logical Inclusive OR
.text:004897B5 -63C        jnz     short loc_489834 ; Jump if Not Zero (ZF=0)
.text:004897B7
.text:004897B7 -63C        cmp     esi, 12h        ; Compare Two Operands
.text:004897BA -63C        jb      short loc_4897C5 ; Jump if Below (CF=1)
.text:004897BC
.text:004897BC -63C        call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004897C2 -63C        mov     eax, [ebp+var_4C]
.text:004897C5
.text:004897C5     loc_4897C5:                   ; CODE XREF: sub_488E70+94Aj
.text:004897C5 -63C        mov     ecx, edi
.text:004897C7 -63C        add     ecx, 1          ; Add
.text:004897CA -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004897D0
.text:004897D0 -63C        dec     ecx             ; Decrement by 1
.text:004897D1 -63C        cmp     ecx, 12h        ; Compare Two Operands
.text:004897D4 -63C        mov     [ebp+var_130], ecx
.text:004897DA -63C        jb      short loc_4897E5 ; Jump if Below (CF=1)
.text:004897DC
.text:004897DC -63C        call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004897E2 -63C        mov     eax, [ebp+var_4C]
.text:004897E5
.text:004897E5     loc_4897E5:                   ; CODE XREF: sub_488E70+96Aj
.text:004897E5 -63C        mov     ebx, edi
.text:004897E7 -63C        add     ebx, 2          ; Add
.text:004897EA -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004897F0
.text:004897F0 -63C        dec     ebx             ; Decrement by 1
.text:004897F1 -63C        cmp     ebx, 12h        ; Compare Two Operands
.text:004897F4 -63C        jb      short loc_4897FF ; Jump if Below (CF=1)
.text:004897F6
.text:004897F6 -63C        call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004897FC -63C        mov     eax, [ebp+var_4C]
.text:004897FF
.text:004897FF     loc_4897FF:                   ; CODE XREF: sub_488E70+984j
.text:004897FF -63C        cmp     esi, 10h        ; Compare Two Operands
.text:00489802 -63C        jb      short loc_48980D ; Jump if Below (CF=1)
.text:00489804
.text:00489804 -63C        call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048980A -63C        mov     eax, [ebp+var_4C]
.text:0048980D
.text:0048980D     loc_48980D:                   ; CODE XREF: sub_488E70+992j
.text:0048980D -63C        mov     ecx, [ebp+var_130]
.text:00489813 -63C        mov     edx, [eax+ecx*4]
.text:00489816 -63C        mov     ecx, [eax+esi*4]
.text:00489819 -63C        add     edx, ecx        ; Add
.text:0048981B -63C        mov     ecx, [eax+ebx*4]
.text:0048981E -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489824
.text:00489824 -63C        add     edx, ecx        ; Add
.text:00489826 -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:0048982C
.text:0048982C -63C        sub     edx, 0Ah        ; Integer Subtraction
.text:0048982F -63C        jmp     loc_489989      ; Jump
.text:00489834
.text:00489834     ; -------------------------------------------------------------------------
.text:00489834     loc_489834:                   ; CODE XREF: sub_488E70+945j
.text:00489834 -63C        cmp     esi, 12h        ; Compare Two Operands
.text:00489837 -63C        jb      short loc_489842 ; Jump if Below (CF=1)
.text:00489839
.text:00489839 -63C        call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048983F -63C        mov     eax, [ebp+var_4C]
.text:00489842
.text:00489842     loc_489842:                   ; CODE XREF: sub_488E70+9C7j
.text:00489842 -63C        mov     ecx, edi
.text:00489844 -63C        add     ecx, 1          ; Add
.text:00489847 -63C        jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:0048984D
.text:0048984D -63C        dec     ecx             ; Decrement by 1
.text:0048984E -63C        cmp     ecx, 12h        ; Compare Two Operands
```



```
.text:00489851 -63C             mov     [ebp+var_12C], ecx
.text:00489857 -63C             jb      short loc_489862 ; Jump if Below (CF=1)

.text:00489859 -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048985F -63C             mov     eax, [ebp+var_4C]

.text:00489862        loc_489862:                             ; CODE XREF: sub_488E70+9E7j
.text:00489862 -63C             mov     ebx, edi
.text:00489864 -63C             add     ebx, 2           ; Add
.text:00489867 -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:0048986D -63C             dec     ebx              ; Decrement by 1
.text:0048986E -63C             cmp     ebx, 12h         ; Compare Two Operands
.text:00489871 -63C             jb      short loc_48987C ; Jump if Below (CF=1)

.text:00489873 -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489879 -63C             mov     eax, [ebp+var_4C]

.text:0048987C        loc_48987C:                             ; CODE XREF: sub_488E70+A01j
.text:0048987C -63C             mov     ecx, [ebp+var_12C]
.text:00489882 -63C             mov     edx, [eax+ecx*4]
.text:00489885 -63C             mov     ecx, [eax+esi*4]
.text:00489888 -63C             add     edx, ecx         ; Add
.text:0048988A -63C             mov     ecx, [eax+ebx*4]
.text:0048988D -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:00489893 -63C             add     edx, ecx         ; Add
.text:00489895 -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:0048989B -63C             cmp     edx, 14h         ; Compare Two Operands
.text:0048989E -63C             jl      short loc_48991A ; Jump if Less (SF!=OF)

.text:004898A0 -63C             cmp     esi, 12h         ; Compare Two Operands
.text:004898A3 -63C             jb      short loc_4898AE ; Jump if Below (CF=1)

.text:004898A5 -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004898AB -63C             mov     eax, [ebp+var_4C]

.text:004898AE        loc_4898AE:                             ; CODE XREF: sub_488E70+A33j
.text:004898AE -63C             mov     ecx, edi
.text:004898B0 -63C             add     ecx, 1           ; Add
.text:004898B3 -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:004898B9 -63C             dec     ecx              ; Decrement by 1
.text:004898BA -63C             cmp     ecx, 12h         ; Compare Two Operands
.text:004898BD -63C             mov     [ebp+var_130], ecx
.text:004898C3 -63C             jb      short loc_4898CE ; Jump if Below (CF=1)

.text:004898C5 -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004898CB -63C             mov     eax, [ebp+var_4C]

.text:004898CE        loc_4898CE:                             ; CODE XREF: sub_488E70+A53j
.text:004898CE -63C             mov     ebx, edi
.text:004898D0 -63C             add     ebx, 2           ; Add
.text:004898D3 -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:004898D9 -63C             dec     ebx              ; Decrement by 1
.text:004898DA -63C             cmp     ebx, 12h         ; Compare Two Operands
.text:004898DD -63C             jb      short loc_4898E8 ; Jump if Below (CF=1)

.text:004898DF -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004898E5 -63C             mov     eax, [ebp+var_4C]

.text:004898E8        loc_4898E8:                             ; CODE XREF: sub_488E70+A6Dj
.text:004898E8 -63C             cmp     esi, 10h         ; Compare Two Operands
.text:004898EB -63C             jb      short loc_4898F6 ; Jump if Below (CF=1)

.text:004898ED -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004898F3 -63C             mov     eax, [ebp+var_4C]

.text:004898F6        loc_4898F6:                             ; CODE XREF: sub_488E70+A7Bj
.text:004898F6 -63C             mov     ecx, [ebp+var_130]
.text:004898FC -63C             mov     edx, [eax+ecx*4]
.text:004898FF -63C             mov     ecx, [eax+esi*4]
.text:00489902 -63C             add     edx, ecx         ; Add
.text:00489904 -63C             mov     ecx, [eax+ebx*4]
.text:00489907 -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:0048990D -63C             add     edx, ecx         ; Add
.text:0048990F -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:00489915 -63C             sub     edx, 14h         ; Integer Subtraction
.text:00489918 -63C             jmp     short loc_489989 ; Jump

.text:0048991A        ; ---------------------------------------------------------------------------
.text:0048991A        loc_48991A:                             ; CODE XREF: sub_488E70+A2Ej
.text:0048991A -63C             cmp     esi, 12h         ; Compare Two Operands
.text:0048991D -63C             jb      short loc_489928 ; Jump if Below (CF=1)

.text:0048991F -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489925 -63C             mov     eax, [ebp+var_4C]

.text:00489928        loc_489928:                             ; CODE XREF: sub_488E70+AADj
.text:00489928 -63C             mov     ecx, edi
.text:0048992A -63C             add     ecx, 1           ; Add
.text:0048992D -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:00489933 -63C             dec     ecx              ; Decrement by 1
.text:00489934 -63C             cmp     ecx, 12h         ; Compare Two Operands
.text:00489937 -63C             mov     [ebp+var_130], ecx
.text:0048993D -63C             jb      short loc_489948 ; Jump if Below (CF=1)

.text:0048993F -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489945 -63C             mov     eax, [ebp+var_4C]

.text:00489948        loc_489948:                             ; CODE XREF: sub_488E70+ACDj
.text:00489948 -63C             mov     ebx, edi
.text:0048994A -63C             add     ebx, 2           ; Add
.text:0048994D -63C             jo      loc_489C25       ; Jump if Overflow (OF=1)

.text:00489953 -63C             dec     ebx              ; Decrement by 1
.text:00489954 -63C             cmp     ebx, 12h         ; Compare Two Operands
.text:00489957 -63C             jb      short loc_489962 ; Jump if Below (CF=1)

.text:00489959 -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048995F -63C             mov     eax, [ebp+var_4C]
```



`loc_489962: ; CODE XREF: sub_488E70+AE7j`

```
.text:00489962 -63C             cmp     esi, 10h        ; Compare Two Operands
.text:00489965 -63C             jb      short loc_489970 ; Jump if Below (CF=1)
```

`loc_489965`

```
.text:00489967 -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
```

`loc_489967`

```
.text:0048996D -63C             mov     eax, [ebp+var_4C]
```

`loc_489970`

```
.text:00489970 -63C             mov     ecx, [ebp+var_130]
.text:00489976 -63C             mov     edx, [eax+ecx*4]
.text:00489979 -63C             mov     ecx, [eax+esi*4]
.text:0048997C -63C             add     edx, ecx        ; Add
.text:0048997E -63C             mov     ecx, [eax+ebx*4]
.text:00489981 -63C             jo      loc_489C25      ; Jump if Overflow (OF=1)
```

`loc_489981`

```
.text:00489987 -63C             add     edx, ecx        ; Add
```

`loc_489989`

`loc_489989: ; CODE XREF: sub_488E70+9BFj`
`sub_488E70+AA8j`

```
.text:00489989 -63C             mov     eax, [ebp+var_84]
.text:0048998F -63C             jo      loc_489C25      ; Jump if Overflow (OF=1)
```

`loc_48998F`

```
.text:00489995 -63C             mov     [eax+esi*4], edx
.text:00489998 -63C             mov     ecx, [ebp+var_20]
.text:0048999B -63C             cmp     esi, 10h        ; Compare Two Operands
.text:0048999E -63C             mov     [ebp+var_108], ecx
.text:004899A4 -63C             mov     [ebp+var_110], 8
.text:004899AE -63C             jb      short loc_4899B6 ; Jump if Below (CF=1)
```

`loc_4899AE`

```
.text:004899B0 -63C             call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
```

`loc_4899B0`

`loc_4899B6`

```
.text:004899B6 -63C             mov     edx, [ebp+var_84]
.text:004899BC -63C             lea     ecx, [ebp+var_100] ; Load Effective Address
.text:004899C2 -63C             push    ecx
.text:004899C3 -638             mov     [ebp+var_100], 4003h
.text:004899CD -638             lea     eax, [edx+esi*4] ; Load Effective Address
.text:004899D0 -638             lea     edx, [ebp+var_D0] ; Load Effective Address
.text:004899D6 -638             push    edx
.text:004899D7 -634             mov     [ebp+var_F8], eax
.text:004899DD -634             call    ds:rtcVarStrFromVar ; Indirect Call Near Procedure
```

`loc_4899DD`

```
.text:004899E3 -634             lea     eax, [ebp+var_D0] ; Load Effective Address
.text:004899E9 -634             lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:004899EF -634             push    eax
.text:004899F0 -630             push    ecx
.text:004899F1 -62C             call    ds:rtcTrimVar   ; Indirect Call Near Procedure
```

`loc_4899F1`

```
.text:004899F7 -62C             lea     edx, [ebp+var_110] ; Load Effective Address
.text:004899FD -62C             lea     eax, [ebp+var_E0] ; Load Effective Address
.text:00489A03 -62C             push    edx
.text:00489A04 -628             lea     ecx, [ebp+var_F0] ; Load Effective Address
.text:00489A0A -628             push    eax
.text:00489A0B -624             push    ecx
.text:00489A0C -620             call    ds:__vbaVarCat  ; Indirect Call Near Procedure
```

`loc_489A0C`

```
.text:00489A12 -620             push    eax
.text:00489A13 -61C             call    ds:__vbaStrVarMove ; Indirect Call Near Procedure
```

`loc_489A13`

```
.text:00489A19 -61C             mov     ebx, ds:__vbaStrMove
.text:00489A1F -61C             mov     edx, eax
.text:00489A21 -61C             lea     ecx, [ebp+var_20] ; Load Effective Address
.text:00489A24 -61C             call    ebx ; __vbaStrMove ; Indirect Call Near Procedure
```

`loc_489A24`

```
.text:00489A26 -61C             lea     edx, [ebp+var_F0] ; Load Effective Address
.text:00489A2C -61C             lea     eax, [ebp+var_E0] ; Load Effective Address
.text:00489A32 -61C             push    edx
.text:00489A33 -618             lea     ecx, [ebp+var_D0] ; Load Effective Address
.text:00489A39 -618             push    eax
.text:00489A3A -614             push    ecx
.text:00489A3B -610             push    3
.text:00489A3D -60C             call    ds:__vbaFreeVarList ; Indirect Call Near Procedure
```

`loc_489A3D`

```
.text:00489A43 -618             mov     eax, 1
.text:00489A48 -618             add     esp, 10h        ; Add
.text:00489A4B -628             add     eax, edi        ; Add
.text:00489A4D -628             jo      loc_489C25      ; Jump if Overflow (OF=1)
```

`loc_489A4D`

```
.text:00489A53 -628             mov     edi, eax
.text:00489A55 -628             jmp     loc_4896CA      ; Jump
```

`loc_489A55`

`loc_489A5A`

`loc_489A5A: ; CODE XREF: sub_488E70+861j`

```
.text:00489A5A -63C             lea     edx, [ebp+var_20] ; Load Effective Address
.text:00489A5D -63C             lea     eax, [ebp+var_D0] ; Load Effective Address
.text:00489A63 -63C             mov     [ebp+var_F8], edx
.text:00489A69 -63C             push    eax
.text:00489A6A -638             lea     ecx, [ebp+var_100] ; Load Effective Address
.text:00489A70 -638             push    1
.text:00489A72 -634             lea     edx, [ebp+var_E0] ; Load Effective Address
.text:00489A78 -634             mov     edi, 4008h
.text:00489A7D -634             push    ecx
.text:00489A7E -630             push    edx
.text:00489A7F -62C             mov     [ebp+var_C8], 0Fh
.text:00489A89 -62C             mov     [ebp+var_D0], 2
.text:00489A93 -62C             mov     [ebp+var_100], edi
.text:00489A99 -62C             call    ds:rtcMidCharVar ; Indirect Call Near Procedure
```

`loc_489A99`

```
.text:00489A9F -62C             mov     esi, ds:__vbaStrVarMove
.text:00489AA5 -62C             lea     eax, [ebp+var_E0] ; Load Effective Address
.text:00489AAB -62C             push    eax
.text:00489AAC -628             call    esi ; __vbaStrVarMove ; Indirect Call Near Procedure
```

`loc_489AAC`

```
.text:00489AAE -628             mov     edx, eax
.text:00489AB0 -628             lea     ecx, [ebp+var_20] ; Load Effective Address
.text:00489AB3 -628             call    ebx ; __vbaStrMove ; Indirect Call Near Procedure
```

`loc_489AB3`

```
.text:00489AB5 -628             lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:00489ABB -628             lea     edx, [ebp+var_D0] ; Load Effective Address
.text:00489AC1 -628             push    ecx
.text:00489AC2 -624             push    edx
.text:00489AC3 -620             push    2
.text:00489AC5 -61C             call    ds:__vbaFreeVarList ; Indirect Call Near Procedure
```

`loc_489AC5`

```
.text:00489ACB -628             add     esp, 0Ch        ; Add
.text:00489ACE -634             lea     ecx, [ebp+var_100] ; Load Effective Address
.text:00489AD4 -634             lea     edx, [ebp+var_D0] ; Load Effective Address
```



```asm
.text:00489ADA -634                lea     eax, [ebp+var_20] ; Load Effective Address  

.text:00489ADD -634                push    ecx  

.text:00489ADE -630                push    edx  

.text:00489ADF -62C                mov     [ebp+var_F8], eax  

.text:00489AE5 -62C                mov     [ebp+var_100], edi  

.text:00489AEB -62C                call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:00489AEB    



.text:00489AF1 -62C                lea     eax, [ebp+var_D0] ; Load Effective Address  

.text:00489AF7 -62C                push    eax  

.text:00489AF8 -628                call    esi ; __vbaStrVarMove ; Indirect Call Near Procedure  

.text:00489AF8    



.text:00489AFA -628                mov     edx, eax  

.text:00489AFC -628                lea     ecx, [ebp+var_B4] ; Load Effective Address  

.text:00489B02 -628                call    ebx ; __vbaStrMove ; Indirect Call Near Procedure  

.text:00489B02    



.text:00489B04 -628                lea     ecx, [ebp+var_D0] ; Load Effective Address  

.text:00489B0A -628                call    ds:__vbaFreeVar ; Indirect Call Near Procedure  

.text:00489B0A    



.text:00489B10 -628                wait                    ; Wait until BUSY# Pin is Inactive (HIGH)  

.text:00489B11 -628                push    offset loc_489BE0  

.text:00489B16 -624                jmp     short loc_489B57 ; Jump  

.text:00489B16    



.text:00489B18     ; ---------------------------------------------------------------------------  

.text:00489B18 -624                test    byte ptr [ebp-4], 4 ; Logical Compare  

.text:00489B1C -624                jz      short loc_489B2A ; Jump if Zero (ZF=1)  

.text:00489B1C    



.text:00489B1E -624                lea     ecx, [ebp-0B4h] ; Load Effective Address  

.text:00489B24 -624                call    ds:__vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B24    



.text:00489B2A    



.text:00489B2A     loc_489B2A:                         ; CODE XREF: sub_488E70+CACj  

.text:00489B2A -624                lea     ecx, [ebp-0C0h] ; Load Effective Address  

.text:00489B30 -624                call    ds:__vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B30    



.text:00489B36 -624                lea     ecx, [ebp-0F0h] ; Load Effective Address  

.text:00489B3C -624                lea     edx, [ebp-0E0h] ; Load Effective Address  

.text:00489B42 -624                push    ecx  

.text:00489B43 -624                lea     eax, [ebp-0D0h] ; Load Effective Address  

.text:00489B49 -624                push    edx  

.text:00489B4A -624                push    eax  

.text:00489B4B -624                push    3  

.text:00489B4D -624                call    ds:__vbaFreeVarList ; Indirect Call Near Procedure  

.text:00489B4D    



.text:00489B53 -630                add     esp, 10h        ; Add  

.text:00489B56 -640                retn                    ; Return Near from Procedure  

.text:00489B56    



.text:00489B57     ; ---------------------------------------------------------------------------  

.text:00489B57    



.text:00489B57     loc_489B57:                         ; CODE XREF: sub_488E70+CA6j  

.text:00489B57 -624                mov     edi, ds:__vbaFreeStr  

.text:00489B5D -624                lea     ecx, [ebp+var_1C] ; Load Effective Address  

.text:00489B60 -624                call    edi ; __vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B60    



.text:00489B62 -624                lea     ecx, [ebp+var_20] ; Load Effective Address  

.text:00489B65 -624                call    edi ; __vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B65    



.text:00489B67 -624                mov     esi, ds:__vbaAryDestruct  

.text:00489B6D -624                lea     edx, [ebp+var_114] ; Load Effective Address  

.text:00489B73 -624                lea     ecx, [ebp+var_3C] ; Load Effective Address  

.text:00489B76 -624                push    edx  

.text:00489B77 -620                push    0  

.text:00489B79 -61C                mov     [ebp+var_114], ecx  

.text:00489B7F -61C                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489B7F    



.text:00489B81 -61C                lea     ecx, [ebp+var_118] ; Load Effective Address  

.text:00489B87 -61C                lea     eax, [ebp+var_58] ; Load Effective Address  

.text:00489B8A -61C                push    ecx  

.text:00489B8B -618                push    0  

.text:00489B8D -614                mov     [ebp+var_118], eax  

.text:00489B93 -614                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489B93    



.text:00489B95 -614                lea     eax, [ebp+var_11C] ; Load Effective Address  

.text:00489B9B -614                lea     edx, [ebp+var_74] ; Load Effective Address  

.text:00489B9E -614                push    eax  

.text:00489B9F -610                push    0  

.text:00489BA1 -60C                mov     [ebp+var_11C], edx  

.text:00489BA7 -60C                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489BA7    



.text:00489BA9 -60C                lea     edx, [ebp+var_120] ; Load Effective Address  

.text:00489BAF -60C                lea     ecx, [ebp+var_90] ; Load Effective Address  

.text:00489BB5 -60C                push    edx  

.text:00489BB6 -608                push    0  

.text:00489BB8 -604                mov     [ebp+var_120], ecx  

.text:00489BBE -604                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489BBE    



.text:00489BC0 -604                lea     ecx, [ebp+var_124] ; Load Effective Address  

.text:00489BC6 -604                lea     eax, [ebp+var_AC] ; Load Effective Address  

.text:00489BCC -604                push    ecx  

.text:00489BCD -600                push    0  

.text:00489BCF -5FC                mov     [ebp+var_124], eax  

.text:00489BD5 -5FC                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489BD5    



.text:00489BD7 -5FC                lea     ecx, [ebp+var_B8] ; Load Effective Address  

.text:00489BDD -5FC                call    edi ; __vbaFreeStr ; Indirect Call Near Procedure  

.text:00489BDD    



.text:00489BDF -5FC                retn                    ; Return Near from Procedure  

.text:00489BDF    



.text:00489BE0     ; ---------------------------------------------------------------------------  

.text:00489BE0    



.text:00489BE0     loc_489BE0:                         ; DATA XREF: sub_488E70+CA1o  

.text:00489BE0 -5FC                mov     ecx, [ebp-14h]  

.text:00489BE3 -5FC                mov     eax, [ebp-0B4h]  

.text:00489BE9 -5FC                pop     edi  

.text:00489BEA -5FC                pop     esi  

.text:00489BEB -5FC                mov     large fs:0, ecx  

.text:00489BF2 -5FC                pop     ebx  

.text:00489BF3 -5FC                mov     esp, ebp  

.text:00489BF5 -5FC                pop     ebp  

.text:00489BF6 -5FC                retn    8               ; Return Near from Procedure  

.text:00489BF6    



.text:00489BF6     ; ---------------------------------------------------------------------------  

.text:00489BF9 -5FC                align 4  

.text:00489BFC -5FC off_489BFC     dd  offset loc_488FB7    ; DATA XREF: sub_488E70+140r  

.text:00489BFC                  dd  offset loc_488FC1    ; jump table for switch statement  

.text:00489BFC                  dd  offset loc_488FCB  

.text:00489BFC                  dd  offset loc_488FD5  

.text:00489BFC                  dd  offset loc_488FDF  

.text:00489BFC                  dd  offset loc_488FE9  

.text:00489BFC                  dd  offset loc_488FF3
```



`.text:00489BFC`                  `dd offset loc_488FFD`  

`.text:00489BFC`                  `dd offset loc_489007`  

---

**【算法总结】**  

/ 11111100（单项注册功能 1）     \     / 运算码 1 \    转换     / 单项注册功能 1 对应注册码  

申请码 + {- 22222200（单项注册功能 2）     -} = {- 运算码 2 -} --------> {- 单项注册功能 2 对应注册码  

\ 33333300（完整注册功能 [1+2]） /     \ 运算码 3 /              \ 完整注册功能 [1+2] 对应注册码  

例：（本机信息）  

       / 11111100 \     / 1962924897 \    转换     / 315256941958022  

1951813797 + {- 22222200 -} = {- 1974035997 -} --------> {- 315829212908527  

       \ 33333300 /     \ 1985147097 /              \ 327589729281898  

=============================================================================  

`KeyGen` 我就不写了。。。免得作者找偶麻烦！给个内存注册机：  

**【内存注册机】**  

中断地址：`556ADC`  

中断次数：`1`  

第一字节：`83`  

指令长度：`3`  

内存方式 ---> `EAX` ---> 宽字符串  

..........................................................  

注意：  

注册机截出的注册码比如为：`1234567890ABCDE`  

那么软件里应填写的格式为：`12345-67890-ABCDE`  

=============================================================================  

**【注册信息】**  

申请码：`1951813797`  

单项功能 1 注册码：`31525-69419-58022`  

单项功能 2 注册码：`31582-92129-08527`  

完整功能注册码：`32758-97292-81898`  

---------------------------------------------------------------------------  

版权所有（C）2005 `KuNgBiM[DFCG]`        Copyright (C) 2005 `KuNgBiM[DFCG]`  

---------------------------------------------------------------------------  

     Cracked By `KuNgBiM[DFCG]`  

               2005-08-11  

               15:10:18 PM 

`4dd466d0e1e0cc123afbe504da2462ca`
