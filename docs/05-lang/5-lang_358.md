
# Syncronize Backup V1.35

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-214.htm`](https://www.pediy.com/kssd/pediy08/pediy8-214.htm)

【破文日期】 2006 年 6 月 9 日（今天是高考的第三天，愿所有考生都能进入理想的学府）

【破文作者】 冷血书生

【作者主页】 `http://blog.csdn.net/xueleng/`

【 E-mail 】 `colddoctor@126.com`

【文章题目】 Syncronize Backup V1.35 算法分析

【软件名称】 Syncronize Backup 1.35

【下载地址】 `http://www.skycn.com/soft/27192.html`

-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------

【加密方式】 序列号

【破解工具】 `OD`

【破解难度】 +++初级+++ 中级 高级 超难

【破解平台】 Win9x/NT/2000/XP/XP SP2

-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------

【软件简介】

Syncronize Backup (专业版)可以有效的保护您的数据。有两种模式：备份和同步。简洁易用。30 天试用。技术特性：界面友好；完全自订制任务；详细的任务报告；快速数据拷贝，省时省力。

-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------

【破解过程】

Microsoft Visual Basic 5.0 / 6.0 编写，下断点 `bp __vbaStrComp`，然后返回到程序领空，来到下面代码处：

```
005E6126     /0F85 95000000      jnz Syncroni.005E61C1                            ; 不相等就跳到下面弹出提示
005E612C     |8D8D 64FFFFFF      lea ecx,dword ptr ss:[ebp-9C]
005E6132     |C785 64FFFFFF A60>mov dword ptr ss:[ebp-9C],0A6
005E613C     |51                push ecx
005E613D     |E8 3EBFFBFF       call Syncroni.005A2080
005E6142     |8B3D 28144000     mov edi,dword ptr ds:[<&MSVBVM60.__vbaStrMove>]  ; MSVBVM60.__vbaStrMove
005E6148     |8BD0              mov edx,eax
005E614A     |8D4D E4           lea ecx,dword ptr ss:[ebp-1C]
005E614D     |FFD7              call edi
005E614F     |B8 04000280       mov eax,80020004
005E6154     |8D55 98           lea edx,dword ptr ss:[ebp-68]
005E6157     |8945 A0           mov dword ptr ss:[ebp-60],eax
005E615A     |8945 B0           mov dword ptr ss:[ebp-50],eax
005E615D     |8945 C0           mov dword ptr ss:[ebp-40],eax
005E6160     |8B45 E4           mov eax,dword ptr ss:[ebp-1C]
005E6163     |8945 D0           mov dword ptr ss:[ebp-30],eax
005E6166     |8D45 A8           lea eax,dword ptr ss:[ebp-58]
005E6169     |52                push edx
005E616A     |8D4D B8           lea ecx,dword ptr ss:[ebp-48]
005E616D     |50                push eax
005E616E     |51                push ecx
005E616F     |8D55 C8           lea edx,dword ptr ss:[ebp-38]
005E6172     |BB 0A000000       mov ebx,0A
005E6177     |6A 10             push 10
005E6179     |52                push edx
005E617A     |895D 98           mov dword ptr ss:[ebp-68],ebx
005E617D     |895D A8           mov dword ptr ss:[ebp-58],ebx
005E6180     |895D B8           mov dword ptr ss:[ebp-48],ebx
005E6183     |8975 E4           mov dword ptr ss:[ebp-1C],esi
005E6186     |C745 C8 08000000 mov dword ptr ss:[ebp-38],8
005E618D     |FF15 2C114000     call dword ptr ds:[<&MSVBVM60.#595>]            ; MSVBVM60.rtcMsgBox
005E6193     |8B35 8C144000     mov esi,dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]  ; MSVBVM60.__vbaFreeStr
005E6199     |8D4D E4           lea ecx,dword ptr ss:[ebp-1C]
005E619C     |FFD6              call esi
005E619E     |8D45 98           lea eax,dword ptr ss:[ebp-68]
005E61A1     |8D4D A8           lea ecx,dword ptr ss:[ebp-58]
005E61A4     |50                push eax
005E61A5     |8D55 B8           lea edx,dword ptr ss:[ebp-48]
005E61A8     |51                push ecx
005E61A9     |8D45 C8           lea eax,dword ptr ss:[ebp-38]
005E61AC     |52                push edx
005E61AD     |50                push eax
005E61AE     |6A 04             push 4
005E61B0     |FF15 58104000     call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>] ; MSVBVM60.__vbaFreeVarList
005E61B6     |83C4 14           add esp,14
005E61B9     |FF15 50104000     call dword ptr ds:[<&MSVBVM60.__vbaEnd>]         ; MSVBVM60.__vbaEnd
005E61BF     |EB 11             jmp short Syncroni.005E61D2
005E61C1     \8B3D 28144000     mov edi,dword ptr ds:[<&MSVBVM60.__vbaStrMove>]   ; MSVBVM60.__vbaStrMove
005E61C7      8B35 8C144000     mov esi,dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]   ; MSVBVM60.__vbaFreeStr
005E61CD      BB 0A000000       mov ebx,0A
005E61D2      8B4D 08           mov ecx,dword ptr ss:[ebp+8]
005E61D5      66:0FB649 34      movzx cx,byte ptr ds:[ecx+34]
005E61DA      66:83C1 01        add cx,1
005E61DE      0F80 D2030000     jo Syncroni.005E65B6
005E61E4      FF15 64124000     call dword ptr ds:[<&MSVBVM60.__vbaUI1I2>]       ; MSVBVM60.__vbaUI1I2
005E61EA      8B4D 08           mov ecx,dword ptr ss:[ebp+8]
005E61ED      51                push ecx
005E61EE      8B11              mov edx,dword ptr ds:[ecx]
005E61F0      8841 34           mov byte ptr ds:[ecx+34],al
005E61F3      FF92 04030000     call dword ptr ds:[edx+304]
005E61F9      50                push eax
005E61FA      8D45 DC           lea eax,dword ptr ss:[ebp-24]
005E61FD      50                push eax
005E61FE      FF15 28114000     call dword ptr ds:[<&MSVBVM60.__vbaObjSet>]      ; MSVBVM60.__vbaObjSet
005E6204      8B08              mov ecx,dword ptr ds:[eax]
005E6206      8D55 E4           lea edx,dword ptr ss:[ebp-1C]
005E6209      52                push edx
005E620A      50                push eax
005E620B      8985 5CFFFFFF     mov dword ptr ss:[ebp-A4],eax
005E6211      FF91 A0000000     call dword ptr ds:[ecx+A0]
005E6217      85C0              test eax,eax
005E6219      DBE2              fclex
005E621B      7D 18             jge short Syncroni.005E6235
005E621D      8B8D 5CFFFFFF     mov ecx,dword ptr ss:[ebp-A4]
005E6223      68 A0000000       push 0A0
005E6228      68 68EB4100       push Syncroni.0041EB68
005E622D      51                push ecx
005E622E      50                push eax
005E622F      FF15 D4104000     call dword ptr ds:[<&MSVBVM60.__vbaHresultCheckOb>; MSVBVM60.__vbaHresultCheckObj
005E6235      8B55 E4           mov edx,dword ptr ss:[ebp-1C]                   ; 机器码转移
005E6238      8D4D E0           lea ecx,dword ptr ss:[ebp-20]
005E623B      C745 E4 00000000 mov dword ptr ss:[ebp-1C],0
005E6242      FFD7              call edi
005E6244      8D55 E0           lea edx,dword ptr ss:[ebp-20]
005E6247      52                push edx
005E6248      E8 C3190000      call Syncroni.005E7C10                           ; 算法 CALL，跟进
005E624D      8BD0              mov edx,eax                                    ; 真码
005E624F      8D4D E8           lea ecx,dword ptr ss:[ebp-18]
005E6252      FFD7              call edi
005E6254      8D4D E0           lea ecx,dword ptr ss:[ebp-20]
005E6257      FFD6              call esi
005E6259      8D4D DC           lea ecx,dword ptr ss:[ebp-24]
005E625C      FF15 88144000     call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>]     ; MSVBVM60.__vbaFreeObj
005E6262      8B45 08           mov eax,dword ptr ss:[ebp+8]
005E6265      50                push eax
005E6266      8B08              mov ecx,dword ptr ds:[eax]
005E6268      FF91 08030000     call dword ptr ds:[ecx+308]
005E626E      8D55 DC           lea edx,dword ptr ss:[ebp-24]
005E6271      50                push eax
005E6272      52                push edx
005E6273      FF15 28114000     call dword ptr ds:[<&MSVBVM60.__vbaObjSet>]      ; MSVBVM60.__vbaObjSet
005E6279      8B08              mov ecx,dword ptr ds:[eax]
005E627B      8D55 E4           lea edx,dword ptr ss:[ebp-1C]
005E627E      52                push edx
005E627F      50                push eax
005E6280      8985 5CFFFFFF     mov dword ptr ss:[ebp-A4],eax
005E6286      FF91 A0000000     call dword ptr ds:[ecx+A0]
```



```
005E628C      85C0                 test eax,eax

005E628E      DBE2                 fclex

005E6290      7D 18               jge short Syncroni.005E62AA

005E6292      8B8D 5CFFFFFF      mov ecx,dword ptr ss:\[ebp-A4\]

005E6298      68 A0000000        push 0A0

005E629D      68 68EB4100        push Syncroni.0041EB68

005E62A2      51                 push ecx

005E62A3      50                 push eax

005E62A4      FF15 D4104000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaHresultCheckOb>; MSVBVM60.\_\_vbaHresultCheckObj

005E62AA      8B55 E8           mov edx,dword ptr ss:\[ebp-18\]

005E62AD      8B45 E4           mov eax,dword ptr ss:\[ebp-1C\]

005E62B0      52                 push edx

005E62B1      50                 push eax

005E62B2      FF15 E0114000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaStrCmp>\]       ; MSVBVM60.\_\_vbaStrCmp

005E62B8      F7D8               neg eax                                                   ; 真假比较

005E62BA      1BC0               sbb eax,eax

005E62BC      8D4D E4           lea ecx,dword ptr ss:\[ebp-1C\]

005E62BF      F7D8               neg eax

005E62C1      F7D8               neg eax

005E62C3      66:8985 54FFFFFF  mov word ptr ss:\[ebp-AC\],ax

005E62CA      FFD6               call esi

005E62CC      8D4D DC           lea ecx,dword ptr ss:\[ebp-24\]

005E62CF      FF15 88144000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeObj>\]     ; MSVBVM60.\_\_vbaFreeObj

005E62D5      66:83BD 54FFFFFF >cmp word ptr ss:\[ebp-AC\],0                           ; 与 0 比较

005E62DD      0F84 85000000      je Syncroni.005E6368                                   ; 相等就跳到下面注册成功

005E62E3      8D8D 64FFFFFF      lea ecx,dword ptr ss:\[ebp-9C\]

005E62E9      C785 64FFFFFF A70>mov dword ptr ss:\[ebp-9C\],0A7

005E62F3      51                 push ecx

005E62F4      E8 87BDFBFF       call Syncroni.005A2080

005E62F9      8BD0               mov edx,eax

005E62FB      8D4D E4           lea ecx,dword ptr ss:\[ebp-1C\]

005E62FE      FFD7               call edi

005E6300      B8 04000280        mov eax,80020004

005E6305      8D55 98           lea edx,dword ptr ss:\[ebp-68\]

005E6308      8945 A0           mov dword ptr ss:\[ebp-60\],eax

005E630B      8945 B0           mov dword ptr ss:\[ebp-50\],eax

005E630E      8945 C0           mov dword ptr ss:\[ebp-40\],eax

005E6311      8B45 E4           mov eax,dword ptr ss:\[ebp-1C\]

005E6314      8945 D0           mov dword ptr ss:\[ebp-30\],eax

005E6317      8D45 A8           lea eax,dword ptr ss:\[ebp-58\]

005E631A      52                 push edx

005E631B      8D4D B8           lea ecx,dword ptr ss:\[ebp-48\]

005E631E      50                 push eax

005E631F      51                 push ecx

005E6320      8D55 C8           lea edx,dword ptr ss:\[ebp-38\]

005E6323      6A 40               push 40

005E6325      52                 push edx

005E6326      895D 98           mov dword ptr ss:\[ebp-68\],ebx

005E6329      895D A8           mov dword ptr ss:\[ebp-58\],ebx

005E632C      895D B8           mov dword ptr ss:\[ebp-48\],ebx

005E632F      C745 E4 00000000  mov dword ptr ss:\[ebp-1C\],0

005E6336      C745 C8 08000000  mov dword ptr ss:\[ebp-38\],8

005E633D      FF15 2C114000      call dword ptr ds:\[<&MSVBVM60.#595>\]               ; MSVBVM60.rtcMsgBox

005E6343      8D4D E4           lea ecx,dword ptr ss:\[ebp-1C\]                       ; 注册失败

005E6346      FFD6               call esi

005E6348      8D45 98           lea eax,dword ptr ss:\[ebp-68\]

005E634B      8D4D A8           lea ecx,dword ptr ss:\[ebp-58\]

005E634E      50                 push eax

005E634F      8D55 B8           lea edx,dword ptr ss:\[ebp-48\]

005E6352      51                 push ecx

005E6353      8D45 C8           lea eax,dword ptr ss:\[ebp-38\]

005E6356      52                 push edx

005E6357      50                 push eax

005E6358      6A 04               push 4

005E635A      FF15 58104000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeVarList>\]  ; MSVBVM60.\_\_vbaFreeVarList

005E6360      83C4 14           add esp,14

005E6363      E9 E2010000        jmp Syncroni.005E654A

005E6368      BA 98E74100        mov edx,Syncroni.0041E798                              ; UNICODE

"System\\CurrentControlSet\\Control\\Class\\{4C38E96B-E422-14CE-BFC4-08002BM10358}"

005E636D      8D4D E4           lea ecx,dword ptr ss:\[ebp-1C\]

005E6370      FF15 74134000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaStrCopy>\]      ; MSVBVM60.\_\_vbaStrCopy

005E6376      8D4D E4           lea ecx,dword ptr ss:\[ebp-1C\]

005E6379      8D95 60FFFFFF     lea edx,dword ptr ss:\[ebp-A0\]

005E637F      51                 push ecx

005E6380      52                 push edx
```

```
////////////////////////////////////////////////////////////////////////////////////////////
//////////////////////////   跟进 call Syncroni.005E7C10  //////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////
```

```
005E7C6A      FF15 74134000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaStrCopy>\]      ; MSVBVM60.\_\_vbaStrCopy

005E7C70      8B7D 08           mov edi,dword ptr ss:\[ebp+8\]

005E7C73      8B1D 44104000      mov ebx,dword ptr ds:\[<&MSVBVM60.\_\_vbaLenBstr>\]   ; MSVBVM60.\_\_vbaLenBstr

005E7C79      8B07               mov eax,dword ptr ds:\[edi\]                           ;  机器码转移

005E7C7B      50                 push eax

005E7C7C      FFD3               call ebx

005E7C7E      8BC8               mov ecx,eax                                           ; EAX=B

005E7C80      FF15 FC114000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaI2I4>\]          ; MSVBVM60.\_\_vbaI2I4

005E7C86      8985 6CFFFFFF      mov dword ptr ss:\[ebp-94\],eax

005E7C8C      BE 01000000        mov esi,1                                              ; ESI=1

005E7C91      66:3BB5 6CFFFFFF   cmp si,word ptr ss:\[ebp-94\]                          ; 计算开始

005E7C98      0F8F 64010000      jg Syncroni.005E7E02

005E7C9E      0FBFD6             movsx edx,si

005E7CA1      8D4D C4           lea ecx,dword ptr ss:\[ebp-3C\]

005E7CA4      8D45 84           lea eax,dword ptr ss:\[ebp-7C\]

005E7CA7      51                 push ecx

005E7CA8      52                 push edx

005E7CA9      8D4D B4           lea ecx,dword ptr ss:\[ebp-4C\]

005E7CAC      50                 push eax

005E7CAD      51                 push ecx

005E7CAE      C745 CC 01000000   mov dword ptr ss:\[ebp-34\],1

005E7CB5      C745 C4 02000000   mov dword ptr ss:\[ebp-3C\],2

005E7CBC      897D 8C           mov dword ptr ss:\[ebp-74\],edi

005E7CBF      C745 84 08400000   mov dword ptr ss:\[ebp-7C\],4008

005E7CC6      FF15 A8114000      call dword ptr ds:\[<&MSVBVM60.#632>\]                ; MSVBVM60.rtcMidCharVar

005E7CCC      8D55 B4           lea edx,dword ptr ss:\[ebp-4C\]

005E7CCF      8D45 D4           lea eax,dword ptr ss:\[ebp-2C\]

005E7CD2      52                 push edx

005E7CD3      50                 push eax

005E7CD4      FF15 04134000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaStrVarVal>\]     ; MSVBVM60.\_\_vbaStrVarVal

005E7CDA      50                 push eax

005E7CDB      FF15 7C104000      call dword ptr ds:\[<&MSVBVM60.#516>\]                ; MSVBVM60.rtcAnsiValueBstr

005E7CE1      66:05 0200         add ax,2                                               ; AX+ 2

005E7CE5      0F80 39020000      jo Syncroni.005E7F24

005E7CEB      0FBFC8             movsx ecx,ax                                           ; 结果转移

005E7CEE      894D E0           mov dword ptr ss:\[ebp-20\],ecx

005E7CF1      8D4D D4           lea ecx,dword ptr ss:\[ebp-2C\]

005E7CF4      FF15 8C144000      call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeStr>\]       ; MSVBVM60.\_\_vbaFreeStr

005E7CFA      8D55 B4           lea edx,dword ptr ss:\[ebp-4C\]

005E7CFD      8D45 C4           lea eax,dword ptr ss:\[ebp-3C\]
```



```
005E7D00      52                push edx  

005E7D01      50                push eax  

005E7D02      6A 02             push 2  

005E7D04      FF15 58104000     call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>]  ; MSVBVM60.__vbaFreeVarList  

005E7D0A      8B0F              mov ecx,dword ptr ds:[edi]  

005E7D0C      83C4 0C           add esp,0C  

005E7D0F      51                push ecx  

005E7D10      FFD3              call ebx  

005E7D12      8BD0              mov edx,eax  

005E7D14      8B45 E0           mov eax,dword ptr ss:[ebp-20]  

005E7D17      83C2 10           add edx,10                              ; 机器码长度+10  

005E7D1A      0F80 04020000     jo Syncroni.005E7F24  

005E7D20      0FAFD0            imul edx,eax                            ; edx*eax  

005E7D23      0F80 FB010000     jo Syncroni.005E7F24  

005E7D29      03D0              add edx,eax                             ; edx+eax  

005E7D2B      8B07              mov eax,dword ptr ds:[edi]  

005E7D2D      0F80 F1010000     jo Syncroni.005E7F24  

005E7D33      50                push eax  

005E7D34      8955 E0           mov dword ptr ss:[ebp-20],edx           ; edx 为上面计算结果  

005E7D37      FFD3              call ebx  

005E7D39      8B55 E0           mov edx,dword ptr ss:[ebp-20]  

005E7D3C      8B4D E4           mov ecx,dword ptr ss:[ebp-1C]  

005E7D3F      03C2              add eax,edx                             ; eax+edx  

005E7D41      898D 7CFFFFFF     mov dword ptr ss:[ebp-84],ecx  

005E7D47      0F80 D7010000     jo Syncroni.005E7F24  

005E7D4D      8945 E0           mov dword ptr ss:[ebp-20],eax           ; eax 为最后计算结果  

005E7D50      8D45 84           lea eax,dword ptr ss:[ebp-7C]  

005E7D53      8D4D C4           lea ecx,dword ptr ss:[ebp-3C]  

005E7D56      8D55 E0           lea edx,dword ptr ss:[ebp-20]  

005E7D59      50                push eax  

005E7D5A      51                push ecx  

005E7D5B      C785 74FFFFFF 080>mov dword ptr ss:[ebp-8C],8  

005E7D65      8955 8C           mov dword ptr ss:[ebp-74],edx  

005E7D68      C745 84 03400000  mov dword ptr ss:[ebp-7C],4003  

005E7D6F      FF15 E8134000     call dword ptr ds:[<&MSVBVM60.#613>]        ; MSVBVM60.rtcVarStrFromVar  

005E7D75      8D55 C4           lea edx,dword ptr ss:[ebp-3C]  

005E7D78      8D45 D4           lea eax,dword ptr ss:[ebp-2C]  

005E7D7B      52                push edx  

005E7D7C      50                push eax  

005E7D7D      FF15 04134000     call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>]    ; MSVBVM60.__vbaStrVarVal  

005E7D83      50                push eax                          ; 最后结果转化为 10 进制后压栈  

005E7D84      FF15 A8124000     call dword ptr ds:[<&MSVBVM60.#713>]        ; MSVBVM60.rtcStrReverse  

005E7D8A      8D4D B4           lea ecx,dword ptr ss:[ebp-4C]     ; 逆向  

005E7D8D      8D55 A4           lea edx,dword ptr ss:[ebp-5C]  

005E7D90      51                push ecx  

005E7D91      52                push edx  

005E7D92      8945 BC           mov dword ptr ss:[ebp-44],eax     ; 逆向后结果  

005E7D95      C745 B4 08000000  mov dword ptr ss:[ebp-4C],8  

005E7D9C      FF15 60114000     call dword ptr ds:[<&MSVBVM60.#520>]        ; MSVBVM60.rtcTrimVar  

005E7DA2      8D85 74FFFFFF     lea eax,dword ptr ss:[ebp-8C]  

005E7DA8      8D4D A4           lea ecx,dword ptr ss:[ebp-5C]  

005E7DAB      50                push eax  

005E7DAC      51                push ecx  

005E7DAD      8D55 94           lea edx,dword ptr ss:[ebp-6C]  

005E7DB0      52                push edx  

005E7DB1      FF15 08134000     call dword ptr ds:[<&MSVBVM60.__vbaVarCat>]       ; MSVBVM60.__vbaVarCat  

005E7DB7      50                push eax  

005E7DB8      FF15 48104000     call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>]   ; MSVBVM60.__vbaStrVarMove  

005E7DBE      8BD0              mov edx,eax  

005E7DC0      8D4D E4           lea ecx,dword ptr ss:[ebp-1C]  

005E7DC3      FF15 28144000     call dword ptr ds:[<&MSVBVM60.__vbaStrMove>]      ; MSVBVM60.__vbaStrMove  

005E7DC9      8D4D D4           lea ecx,dword ptr ss:[ebp-2C]  

005E7DCC      FF15 8C144000     call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]      ; MSVBVM60.__vbaFreeStr  

005E7DD2      8D45 94           lea eax,dword ptr ss:[ebp-6C]  

005E7DD5      8D4D A4           lea ecx,dword ptr ss:[ebp-5C]  

005E7DD8      50                push eax  

005E7DD9      8D55 B4           lea edx,dword ptr ss:[ebp-4C]  

005E7DDC      51                push ecx  

005E7DDD      8D45 C4           lea eax,dword ptr ss:[ebp-3C]  

005E7DE0      52                push edx  

005E7DE1      50                push eax  

005E7DE2      6A 04             push 4  

005E7DE4      FF15 58104000     call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>]  ; MSVBVM60.__vbaFreeVarList  

005E7DEA      B8 01000000       mov eax,1  

005E7DEF      83C4 14           add esp,14  

005E7DF2      66:03C6           add ax,si  

005E7DF5      0F80 29010000     jo Syncroni.005E7F24  

005E7DFB      8BF0              mov esi,eax  

005E7DFD    ^ E9 8FFEFFFF       jmp Syncroni.005E7C91                   ; 循环计算  

005E7E02      8B55 E4           mov edx,dword ptr ss:[ebp-1C]  

005E7E05      8D4D DC           lea ecx,dword ptr ss:[ebp-24]  

005E7E08      FF15 74134000     call dword ptr ds:[<&MSVBVM60.__vbaStrCopy>]      ; MSVBVM60.__vbaStrCopy  

005E7E0E      8B4D DC           mov ecx,dword ptr ss:[ebp-24]  

005E7E11      51                push ecx  

005E7E12      FFD3              call ebx  

005E7E14      83F8 08           cmp eax,8                         ; 长度与 8 比较  

005E7E17      7C 39             jl short Syncroni.005E7E52        ; 小于就跳  

005E7E19      8B35 7C144000     mov esi,dword ptr ds:[<&MSVBVM60.__vbaMidStmtBstr>; MSVBVM60.__vbaMidStmtBstr  

005E7E1F      8D55 DC           lea edx,dword ptr ss:[ebp-24]  

005E7E22      52                push edx  

005E7E23      6A 05             push 5  

005E7E25      6A 01             push 1  

005E7E27      68 74724200       push Syncroni.00427274  

005E7E2C      6A 00             push 0  

005E7E2E      FFD6              call esi                          ; 第五，十，十五位使用“-”代替  

005E7E30      8D45 DC           lea eax,dword ptr ss:[ebp-24]  

005E7E33      50                push eax  

005E7E34      6A 0A             push 0A  

005E7E36      6A 01             push 1  

005E7E38      68 74724200       push Syncroni.00427274  

005E7E3D      6A 00             push 0  

005E7E3F      FFD6              call esi                          ;  

005E7E41      8D4D DC           lea ecx,dword ptr ss:[ebp-24]  

005E7E44      51                push ecx  

005E7E45      6A 0F             push 0F  

005E7E47      6A 01             push 1  

005E7E49      68 74724200       push Syncroni.00427274  

005E7E4E      6A 00             push 0  

005E7E50      FFD6              call esi  

005E7E52      8B35 10144000     mov esi,dword ptr ds:[<&MSVBVM60.#617>]         ; MSVBVM60.rtcLeftCharVar  

005E7E58      8D45 84           lea eax,dword ptr ss:[ebp-7C]  

005E7E5B      6A 10             push 10  

005E7E5D      8D4D C4           lea ecx,dword ptr ss:[ebp-3C]  

005E7E60      8D55 DC           lea edx,dword ptr ss:[ebp-24]  

005E7E63      50                push eax  

005E7E64      51                push ecx  

005E7E65      8955 8C           mov dword ptr ss:[ebp-74],edx  

005E7E68      C745 84 08400000  mov dword ptr ss:[ebp-7C],4008  

005E7E6F      FFD6              call esi  

005E7E71      8D55 C4           lea edx,dword ptr ss:[ebp-3C]  

005E7E74      52                push edx  
```



```
005E7E75     FF15 48104000     call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>]   ; MSVBVM60.__vbaStrVarMove

005E7E7B     8B1D 28144000     mov ebx,dword ptr ds:[<&MSVBVM60.__vbaStrMove>]   ; MSVBVM60.__vbaStrMove

005E7E81     8BD0              mov edx,eax

005E7E83     8D4D DC           lea ecx,dword ptr ss:[ebp-24]                     ; 真码转移

005E7E86     FFD3              call ebx

005E7E88     8B3D 38104000     mov edi,dword ptr ds:[<&MSVBVM60.__vbaFreeVar>]   ; MSVBVM60.__vbaFreeVar

005E7E8E     8D4D C4           lea ecx,dword ptr ss:[ebp-3C]

005E7E91     FFD7              call edi

005E7E93     8D4D 84           lea ecx,dword ptr ss:[ebp-7C]

005E7E96     6A 0E             push 0E

005E7E98     8D55 C4           lea edx,dword ptr ss:[ebp-3C]

005E7E9B     8D45 DC           lea eax,dword ptr ss:[ebp-24]

005E7E9E     51                push ecx

005E7E9F     52                push edx

005E7EA0     8945 8C           mov dword ptr ss:[ebp-74],eax

005E7EA3     C745 84 08400000  mov dword ptr ss:[ebp-7C],4008

005E7EAA     FFD6              call esi

005E7EAC     8D45 C4           lea eax,dword ptr ss:[ebp-3C]

005E7EAF     50                push eax

005E7EB0     FF15 48104000     call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>]   ; MSVBVM60.__vbaStrVarMove

005E7EB6     8BD0              mov edx,eax

005E7EB8     8D4D D8           lea ecx,dword ptr ss:[ebp-28]

005E7EBB     FFD3              call ebx

005E7EBD     8D4D C4           lea ecx,dword ptr ss:[ebp-3C]

005E7EC0     FFD7              call edi

005E7EC2     68 0E7F5E00       push Syncroni.005E7F0E

005E7EC7     EB 34             jmp short Syncroni.005E7EFD

005E7EC9     F645 FC 04        test byte ptr ss:[ebp-4],4

005E7ECD     74 09             je short Syncroni.005E7ED8

005E7ECF     8D4D D8           lea ecx,dword ptr ss:[ebp-28]

005E7ED2     FF15 8C144000     call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]      ; MSVBVM60.__vbaFreeStr

005E7ED8     8D4D D4           lea ecx,dword ptr ss:[ebp-2C]

005E7EDB     FF15 8C144000     call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]      ; MSVBVM60.__vbaFreeStr

005E7EE1     8D4D 94           lea ecx,dword ptr ss:[ebp-6C]

005E7EE4     8D55 A4           lea edx,dword ptr ss:[ebp-5C]

005E7EE7     51                push ecx

005E7EE8     8D45 B4           lea eax,dword ptr ss:[ebp-4C]

005E7EEB     52                push edx

005E7EEC     8D4D C4           lea ecx,dword ptr ss:[ebp-3C]

005E7EEF     50                push eax

005E7EF0     51                push ecx

005E7EF1     6A 04             push 4

005E7EF3     FF15 58104000     call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>]  ; MSVBVM60.__vbaFreeVarList

005E7EF9     83C4 14           add esp,14

005E7EFC     C3                retn

005E7EFD     8B35 8C144000     mov esi,dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]   ; MSVBVM60.__vbaFreeStr

005E7F03     8D4D E4           lea ecx,dword ptr ss:[ebp-1C]

005E7F06     FFD6              call esi

005E7F08     8D4D DC           lea ecx,dword ptr ss:[ebp-24]

005E7F0B     FFD6              call esi

005E7F0D     C3                retn
```

### 算法总结

1. `[(机器码长度+10)*(机器码+2)]+(机器码+2) + 机器码长度`
2. 结果 `1` 转化为十进制后逆向
3. 结果 `2` 第五，十，十五位使用 `-` 代替，但注册码只取前面十四位

---

### 破解心得

没啥心得，纯属娱乐一下~~

---

### 破解声明

我是一只小菜鸟，偶得一点心得，愿与大家分享 :)

### 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

---

文章写于

`2006-06-09 10:42:27` 感谢 `rdsnow` 大哥空间上的破文生成器，呵呵

`4dd466d0e1e0cc123afbe504da2462ca`
