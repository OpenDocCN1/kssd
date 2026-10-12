
# Syser Debugger v1.96 破解过程

> 原文：[`www.pediy.com/kssd/pediy10/63397.html`](https://www.pediy.com/kssd/pediy10/63397.html)

## 标 题: 【原创】Syser Debugger v1.96 破解过程  

作 者: `vessial`  

时 间: 2008-04-19,14:52  

链 接: `http://bbs.pediy.com/showthread.php?t=48741`  

欢迎交流讨论:)  

`http://hi.baidu.com/vessial`  

本破解过程针对 `Syser Debugger 1.96.1900.0957`  

准备工作参照我前边对 `v1.92` 的分析文章，注册过程是在 `syser.sys` 这个驱动里面做的，  

这个驱动比较大，根本以往的经验，它有对时间的判断，所以我们在 `KeQuerySystemTime` 函数  

下断点，分析过程如下.  

```
.text:F4C4E0F9                     mov     ecx, P  

.text:F4C4E0FF                     push    ebx                 ; KeyHandle  

.text:F4C4E100                     lea     eax, [ebp+var_288]  

.text:F4C4E106                     push    eax                 ; int  

.text:F4C4E107                     lea     eax, [ebp+SourceString]  

.text:F4C4E10D                     push    eax                 ; SourceString  

.text:F4C4E10E                     mov     word ptr [ebp+var_288], bx  

.text:F4C4E115                     call    sub_F4C66E4A  

.text:F4C4E11A                     lea     eax, [ebp+var_288] ; //eax store the Serial Number UNICODE  

.text:F4C4E120                     push    eax  

.text:F4C4E121                     lea     eax, [ebp+var_188]  

.text:F4C4E127                     push    eax  

.text:F4C4E128                     call    sub_F4C4517E  

.text:F4C4E12D                     lea     eax, [ebp+var_188] ; //eax store the Serial Number ASCII  

.text:F4C4E133                     lea     ecx, [esi+0C20h]  

.text:F4C4E139                     push    eax  

.text:F4C4E13A                     call    sub_F4C48704  

.text:F4C4E13F                     push    offset aMjdobnf ;// "MjdObnf" minus one= "LicName"  

.text:F4C4E144                     lea     eax, [ebp+SourceString]  

.text:F4C4E14A                     push    eax                  //Read LicName from Registry  

.text:F4C4E14B                     call    sub_F4C4520E  

.text:F4C4E150                     cmp     [ebp+SourceString], bx  

.text:F4C4E157                     jz      short loc_F4C4E171  

.text:F4C4E159                     lea     eax, [ebp+SourceString]  

.text:F4C4E15F                     lea     ecx, [ebp+SourceString]  

.text:F4C4E171                     mov     ecx, P  

.text:F4C4E177                     push    ebx                 ; KeyHandle  

.text:F4C4E178                     lea     eax, [ebp+var_288]  

.text:F4C4E17E                     push    eax                 ; int  

.text:F4C4E17F                     lea     eax, [ebp+SourceString]  

.text:F4C4E185                     push    eax                 ; SourceString  

.text:F4C4E186                     mov     word ptr [ebp+var_288], bx  

.text:F4C4E18D                     call    sub_F4C66E4A  

.text:F4C4E192                     lea     eax, [ebp+var_288]  

.text:F4C4E198                     push    eax  

.text:F4C4E199                     lea     eax, [ebp+var_188]  

.text:F4C4E19F                     push    eax  

.text:F4C4E1A0                     call    sub_F4C4517E  

.text:F4C4E1A5                     lea     eax, [ebp+var_188] ; Read Registry LicName  

.text:F4C4E1AB                     lea     ecx, [esi+0C30h]  

.text:F4C4E1B1                     push    eax  

.text:F4C4E1B2                     call    sub_F4C48704    ; Read Registry SN  

.text:F4C4E1B7                     push    offset aJotuujnf ; "JotuUjnf" minus one= "InstTime"  

.text:F4C4E1BC                     lea     eax, [ebp+SourceString]//Read InstTime from Registry  

.text:F4C4E1C2                     push    eax  

.text:F4C4E1C3                     call    sub_F4C4520E  

.text:F4C4E1E9                     push    edi  

.text:F4C4E1EA                     lea     eax, [esi+0C00h]  

.text:F4C4E1F0                     push    ebx                 ; KeyHandle  

.text:F4C4E1F1                     mov     [eax], ebx  

.text:F4C4E1F3                     mov     [eax+4], ebx  

.text:F4C4E1F6                     push    eax                 ; int  

.text:F4C4E1F7                     lea     edi, [esi+0BF8h]  

.text:F4C4E1FD                     lea     eax, [ebp+SourceString]  

.text:F4C4E203                     mov     [edi], ebx  

.text:F4C4E205                     mov     [edi+4], ebx  

.text:F4C4E208                     mov     ecx, P  

.text:F4C4E20E                     push    eax                 ; SourceString  

.text:F4C4E20F                     call    sub_F4C669A8  

.text:F4C4E214                     push    edi                 ; CurrentTime  

.text:F4C4E215                     call    ds:KeQuerySystemTime //得到当前的时间  

.text:F4C4E21B                     mov     ecx, esi  

.text:F4C4E21D                     call    Process_SN //这里是关键的利用 LicName 生成和比较注册码的函数  

.text:F4C4E222                     cmp     al, bl  

.text:F4C4E224                     mov     [esi+0C08h], al  

.text:F4C4E22A                     jnz     loc_F4C4E30D  

.text:F4C4E230                     mov     eax, dword_F4D60B3C  

.text:F4C4E235                     mov     edi, [eax]  

.text:F4C4E237                     lea     eax, [ebp+var_8]  

.text:F4C4E23A                     push    eax  

.text:F4C4E23B                     lea     ecx, [esi+0C30h]  

.text:F4C4E241                     call    sub_F4C4516A  

.text:F4C4E246                     push    eax  

.text:F4C4E247                     lea     ecx, [esi+0C20h]  

.text:F4C4E24D                     call    sub_F4C4516A  

.text:F4C4E252                     mov     ecx, dword_F4D60B3C  

.text:F4C4E258                     push    eax  

.text:F4C4E259                     call    dword ptr [edi+0C8h]  

.text:F4C4DD54 Process_SN        proc near                    ; CODE XREF: sub_F4C4E0B8+165p  

.text:F4C4DD54  

.text:F4C4DD54 var_300          = byte ptr -300h  

.text:F4C4DD54 var_200          = word ptr -200h  

.text:F4C4DD54  

.text:F4C4DD54                     push    ebp  

.text:F4C4DD55                     mov     ebp, esp  

.text:F4C4DD57                     sub     esp, 300h  

.text:F4C4DD5D                     push    ebx  

.text:F4C4DD5E                     push    esi  

.text:F4C4DD5F                     mov     esi, ecx  

.text:F4C4DD61                     push    edi  

.text:F4C4DD62                     lea     ecx, [esi+0C20h]  

.text:F4C4DD68                     lea     edi, [esi+0C30h]  

.text:F4C4DD6E                     call    sub_F4C4516A  

.text:F4C4DD73                     push    eax  

.text:F4C4DD74                     mov     ecx, edi  

.text:F4C4DD76                     call    sub_F4C4516A  

.text:F4C4DD7B                     push    eax  

.text:F4C4DD7C                     call    sub_F4CD40FA    ; process LicName and SN //Get in  ---->  

.text:F4C4DD81                     xor     ebx, ebx                                                 |    

.text:F4C4DD83                     cmp     eax, ebx                 |   

.text:F4C4DD85                     mov     [esi+0C0Ch], eax            |   

.text:F4C4DD8B                     jz      loc_F4C4DE30            |   

                            |                 

.text:F4CD40FA                     push    ebp          <-----------------------------------------                                            

.text:F4CD40FB                     mov     ebp, esp  

.text:F4CD40FD                     sub     esp, 110h  

.text:F4CD4103                     push    esi  

.text:F4CD4104                     push    edi  

.text:F4CD4105                     mov     edi, [ebp+arg_4]  

.text:F4CD4108                     push    edi  

.text:F4CD4109                     lea     ecx, [ebp+var_10]  

.text:F4CD410C                     call    sub_F4C45E32  

.text:F4CD4111                     mov     esi, [ebp+var_C]  

.text:F4CD4114                     test    esi, esi  

.text:F4CD4116                     jz      short loc_F4CD411E  

.text:F4CD4118                     push    esi  

.text:F4CD4119                     call    sub_F4C4A6CC  

.text:F4CD411E  

.text:F4CD411E loc_F4CD411E:                       ; CODE XREF: sub_F4CD40FA+1Cj  

.text:F4CD411E                     and     byte_F4D60B50, 0  

.text:F4CD4125                     test    esi, esi  

.text:F4CD4127                     jnz     short loc_F4CD412E  

.text:F4CD4129                     mov     esi, offset byte_F4D60B50  
```



```assembly
.text:F4CD412E

.text:F4CD412E loc_F4CD412E:                             ; CODE XREF: sub_F4CD40FA+2Dj

.text:F4CD412E                 push    esi

.text:F4CD412F                 call    sub_F4CD3DB8

.text:F4CD4134                 test    al, al

.text:F4CD4136                 jnz     short loc_F4CD4152

.text:F4CD4138                 lea     ecx, [ebp+var_10]

.text:F4CD413B                 call    sub_F4C44C8C

.text:F4CD4140

.text:F4CD4140 loc_F4CD4140:                             ; CODE XREF: sub_F4CD40FA+93j

.text:F4CD4140                                         ; sub_F4CD40FA+BAj ...

.text:F4CD4140                 xor     esi, esi

.text:F4CD4142

.text:F4CD4142 loc_F4CD4142:                             ; CODE XREF: sub_F4CD40FA+A9j

.text:F4CD4142                                         ; sub_F4CD40FA+D0j

.text:F4CD4142                 lea     ecx, [ebp+var_10]

.text:F4CD4145                 call    sub_F4C44C60

.text:F4CD414A                 pop     edi

.text:F4CD414B                 mov     eax, esi

.text:F4CD414D                 pop     esi

.text:F4CD414E                 leave

.text:F4CD414F                 retn    8

.text:F4CD4152 ; ---------------------------------------------------------------------------

.text:F4CD4152

.text:F4CD4152 loc_F4CD4152:                             ; CODE XREF: sub_F4CD40FA+3Cj

.text:F4CD4152                 mov     esi, [ebp+arg_0]

.text:F4CD4155                 push    100h

.text:F4CD415A                 push    esi             ; SN

.text:F4CD415B                 lea     eax, [ebp+var_110]

.text:F4CD4161                 push    eax

.text:F4CD4162                 call    sub_F4C42F18

.text:F4CD4167                 cmp     [ebp+var_110], 0

.text:F4CD416E                 jz      short loc_F4CD417E

.text:F4CD4170                 lea     eax, [ebp+var_110]

.text:F4CD4176

.text:F4CD4176 loc_F4CD4176:                             ; CODE XREF: sub_F4CD40FA+82j

.text:F4CD4176                 inc     byte ptr [eax]  ; LicName every byte plus one //LicName 每个字符加 1

.text:F4CD4178                 inc     eax

.text:F4CD4179                 cmp     byte ptr [eax], 0

.text:F4CD417C                 jnz     short loc_F4CD4176

.text:F4CD417E

.text:F4CD417E loc_F4CD417E:                             ; CODE XREF: sub_F4CD40FA+74j

.text:F4CD417E                 lea     eax, [ebp+var_90]

.text:F4CD4184                 push    eax

.text:F4CD4185                 push    esi             ; LicName

.text:F4CD4186                 call    Gen_regcode   //产生注册码的地方

.text:F4CD418B                 test    al, al

.text:F4CD418D                 jz      short loc_F4CD4140

.text:F4CD418F                 push    edi

.text:F4CD4190                 lea     eax, [ebp+var_90]

.text:F4CD4196                 push    eax                          //存入的注册码，和你输入的注册码，在这儿下断点就可以得真正的注册码了.

.text:F4CD4197                 call    Compare_reg     ; Compare SN     //比较注册码

.text:F4CD419C                 test    eax, eax

.text:F4CD419E                 jnz     short loc_F4CD41A5

.text:F4CD41A0                 push    3

.text:F4CD41A2

.text:F4CD41A2 loc_F4CD41A2:                             ; CODE XREF: sub_F4CD40FA+101j

.text:F4CD41A2                 pop     esi

.text:F4CD41A3                 jmp     short loc_F4CD4142

.text:F4CD41A5 ; ---------------------------------------------------------------------------

.text:F4CD41A5

.text:F4CD41A5 loc_F4CD41A5:                             ; CODE XREF: sub_F4CD40FA+A4j

.text:F4CD41A5                 lea     eax, [ebp+var_90]

.text:F4CD41AB                 push    eax

.text:F4CD41AC                 push    esi

.text:F4CD41AD                 call    sub_F4CD3F3E

.text:F4CD41B2                 test    al, al

.text:F4CD41B4                 jz      short loc_F4CD4140

.text:F4CD41B6                 push    edi

.text:F4CD41B7                 lea     eax, [ebp+var_90]

.text:F4CD41BD                 push    eax

.text:F4CD41BE                 call    Compare_reg

.text:F4CD41C3                 test    eax, eax

.text:F4CD41C5                 jnz     short loc_F4CD41CF

.text:F4CD41C7                 xor     esi, esi

.text:F4CD41C9                 inc     esi

.text:F4CD41CA                 jmp     loc_F4CD4142

.text:F4CD41CF ; ---------------------------------------------------------------------------

.text:F4CD41CF

.text:F4CD41CF loc_F4CD41CF:                             ; CODE XREF: sub_F4CD40FA+CBj

.text:F4CD41CF                 lea     eax, [ebp+var_90]

.text:F4CD41D5                 push    eax

.text:F4CD41D6                 push    esi

.text:F4CD41D7                 call    sub_F4CD4066

.text:F4CD41DC                 test    al, al

.text:F4CD41DE                 jz      loc_F4CD4140

.text:F4CD41E4                 push    edi

.text:F4CD41E5                 lea     eax, [ebp+var_90]

.text:F4CD41EB                 push    eax

.text:F4CD41EC                 call    Compare_reg

.text:F4CD41F1                 test    eax, eax

.text:F4CD41F3                 jnz     loc_F4CD4140

.text:F4CD41F9                 push    2

.text:F4CD41FB                 jmp     short loc_F4CD41A2

.text:F4CD41FB sub_F4CD40FA    endp

.text:F4CD3FD2 Gen_regcode    proc near                 ; CODE XREF: sub_F4CD40FA+8Cp

.text:F4CD3FD2

.text:F4CD3FD2 var_18            = dword ptr -18h

.text:F4CD3FD2 var_14            = dword ptr -14h

.text:F4CD3FD2 var_10            = dword ptr -10h

.text:F4CD3FD2 var_C             = dword ptr -0Ch

.text:F4CD3FD2 var_8             = dword ptr -8

.text:F4CD3FD2 var_4             = dword ptr -4

.text:F4CD3FD2 arg_0             = dword ptr  8

.text:F4CD3FD2 arg_4             = dword ptr  0Ch

.text:F4CD3FD2

.text:F4CD3FD2                 push    ebp

.text:F4CD3FD3                 mov     ebp, esp

.text:F4CD3FD5                 sub     esp, 18h

.text:F4CD3FD8                 mov     eax, [ebp+arg_0]

.text:F4CD3FDB                 push    ebx

.text:F4CD3FDC                 xor     ebx, ebx

.text:F4CD3FDE                 cmp     eax, ebx

.text:F4CD3FE0                 jz      short loc_F4CD405E

.text:F4CD3FE2                 cmp     [eax], bl

.text:F4CD3FE4                 jz      short loc_F4CD405E

.text:F4CD3FE6                 push    eax

.text:F4CD3FE7                 lea     ecx, [ebp+var_18]

.text:F4CD3FEA                 mov     [ebp+var_18], offset off_F4CD8408

.text:F4CD3FF1                 mov     [ebp+var_14], ebx

.text:F4CD3FF4                 mov     [ebp+var_10], ebx

.text:F4CD3FF7                 mov     [ebp+var_C], ebx

.text:F4CD3FFA                 call    sub_F4C48704

.text:F4CD3FFF                 cmp     [ebp+var_14], ebx

.text:F4CD4002                 jz      short loc_F4CD400C

.text:F4CD4004                 push    [ebp+var_14]

.text:F4CD4007                 call    sub_F4C4A6A6

.text:F4CD400C

.text:F4CD400C loc_F4CD400C:                             ; CODE XREF: sub_F4CD3FD2+30j

.text:F4CD400C                 push    esi

.text:F4CD400D                 xor     esi, esi

.text:F4CD400F                 cmp     [ebp+var_10], ebx

.text:F4CD4012                 mov     [ebp+var_8], 0D6C2CBC0h

.text:F4CD4019                 mov     [ebp+var_4], 0D7C0D6DCh

.text:F4CD4020                 jle     short loc_F4CD4040

.text:F4CD4022

.text:F4CD4022 loc_F4CD4022:                             ; CODE XREF: sub_F4CD3FD2+6Cj

.text:F4CD4022                 mov     eax, [ebp+var_14]

.text:F4CD4025                 mov     cl, [eax+esi]

.text:F4CD4028                 xor     edx, edx
```



```asm
.text:F4CD402A             lea     eax, [ebp+var_4+3]  

.text:F4CD402D  

.text:F4CD402D loc_F4CD402D:                    ; CODE XREF: sub_F4CD3FD2+66j  

.text:F4CD402D             xor     byte ptr [ebp+edx+var_8], cl ; LicName XOR with 0xD6C2BC0h ,0x0D7C0D6DCh  

.text:F4CD4031             add     [eax], cl  

.text:F4CD4033             inc     edx  

.text:F4CD4034             dec     eax  

.text:F4CD4035             cmp     edx, 8  

.text:F4CD4038             jl      short loc_F4CD402D  

.text:F4CD403A             inc     esi  

.text:F4CD403B             cmp     esi, [ebp+var_10]  

.text:F4CD403E             jl      short loc_F4CD4022  

.text:F4CD4040  

.text:F4CD4040 loc_F4CD4040:                    ; CODE XREF: sub_F4CD3FD2+4Ej  

.text:F4CD4040             push    ebx  

.text:F4CD4041             push    1  

.text:F4CD4043             push    8  

.text:F4CD4045             push    [ebp+arg_4]  

.text:F4CD4048             lea     eax, [ebp+var_8]  

.text:F4CD404B             push    eax  

.text:F4CD404C             call    sub_F4CD3DEE    ; Process LicName //最终将在这儿产生注册码:) -----------------  

.text:F4CD4051             lea     ecx, [ebp+var_18]                  |   

.text:F4CD4054             call    sub_F4C44C60                       |           

.text:F4CD4059             mov     al, 1                              |   

.text:F4CD405B             pop     esi                              |   

.text:F4CD405C             jmp     short loc_F4CD4060                       |   

.text:F4CD405E ; ---------------------------------------------------------------------------                       |   

.text:F4CD405E                             |   

.text:F4CD405E loc_F4CD405E:                    ; CODE XREF: sub_F4CD3FD2+Ej                              |  

.text:F4CD405E                                 ; sub_F4CD3FD2+12j                |  

.text:F4CD405E             xor     al, al                         |       

.text:F4CD4060                            |   

.text:F4CD4060 loc_F4CD4060:                    ; CODE XREF: sub_F4CD3FD2+8Aj                              |  

.text:F4CD4060             pop     ebx                                                     |  

.text:F4CD4061             leave                           |  

.text:F4CD4062             retn    8                        |   

.text:F4CD4062 sub_F4CD3FD2    endp                                   |  

                            |   

                        |  

text:F4283DEE                       |  

.text:F4283DEE             push    ebp                  <------------------------------------------------------  

.text:F4283DEF             mov     ebp, esp  

.text:F4283DF1             sub     esp, 18h  

.text:F4283DF4             push    ebx  

.text:F4283DF5             mov     ebx, [ebp+arg_C]  

.text:F4283DF8             push    esi  

.text:F4283DF9             push    edi  

.text:F4283DFA             xor     edi, edi  

.text:F4283DFC             cmp     [ebp+arg_10], edi  

.text:F4283DFF             lea     esi, [ebx+ebx]  

.text:F4283E02             mov     [ebp+var_4], esi  

.text:F4283E05             mov     [ebp+var_C], edi  

.text:F4283E08             jz      short loc_F4283E18  

.text:F4283E0A             push    [ebp+arg_10]  

.text:F4283E0D             call    sub_F41F2F04  

.text:F4283E12             add     [ebp+var_4], eax  

.text:F4283E15             mov     [ebp+var_C], eax  

.text:F4283E18  

.text:F4283E18 loc_F4283E18:                    ; CODE XREF: sub_F4283DEE+1Aj  

.text:F4283E18             mov     eax, [ebp+arg_8]  

.text:F4283E1B             cdq  

.text:F4283E1C             idiv    ebx  

.text:F4283E1E             test    eax, eax  

.text:F4283E20             mov     ebx, edx  

.text:F4283E22             mov     [ebp+var_18], ebx  

.text:F4283E25             jle     loc_F4283EB4  

.text:F4283E2B             mov     ecx, [ebp+arg_0]  

.text:F4283E2E             mov     [ebp+var_8], ecx  

.text:F4283E31             mov     ecx, [ebp+arg_4]  

.text:F4283E34             add     esi, ecx  

.text:F4283E36             mov     edi, eax  

.text:F4283E38             mov     [ebp+var_10], esi  

.text:F4283E3B             mov     [ebp+var_14], edi  

.text:F4283E3E  

.text:F4283E3E loc_F4283E3E:                    ; CODE XREF: sub_F4283DEE+C4j  

.text:F4283E3E             cmp     [ebp+arg_10], 0  

.text:F4283E42             jz      short loc_F4283E4D  

.text:F4283E44             push    [ebp+arg_10]  

.text:F4283E47             push    esi  

.text:F4283E48             call    sub_F41F2EE8  

.text:F4283E4D  

.text:F4283E4D loc_F4283E4D:                    ; CODE XREF: sub_F4283DEE+54j  

.text:F4283E4D             and     [ebp+arg_8], 0  

.text:F4283E51             cmp     [ebp+arg_C], 0  

.text:F4283E55             jle     short loc_F4283EA0  

.text:F4283E57  

.text:F4283E57 loc_F4283E57:                    ; CODE XREF: sub_F4283DEE+ADj  

.text:F4283E57             mov     eax, [ebp+var_8]  

.text:F4283E5A             mov     ecx, [ebp+arg_8]  

.text:F4283E5D             mov     cl, [eax+ecx]  

.text:F4283E60             movzx   eax, cl  

.text:F4283E63             cdq  

.text:F4283E64             push    10h  

.text:F4283E66             pop     ebx  

.text:F4283E67             idiv    ebx  

.text:F4283E69             dec     esi  

.text:F4283E6A             cmp     dl, 9  

.text:F4283E6D             movzx   eax, dl  

.text:F4283E70             ja      short loc_F4283E77  

.text:F4283E72             add     eax, 30h  

.text:F4283E75             jmp     short loc_F4283E7A  

.text:F4283E77 ; ---------------------------------------------------------------------------  

.text:F4283E77  

.text:F4283E77 loc_F4283E77:                    ; CODE XREF: sub_F4283DEE+82j  

.text:F4283E77             add     eax, 37h  

.text:F4283E7A  

.text:F4283E7A loc_F4283E7A:                    ; CODE XREF: sub_F4283DEE+87j  

.text:F4283E7A             shr     cl, 4  

.text:F4283E7D             mov     [esi], al  

.text:F4283E7F             dec     esi  

.text:F4283E80             cmp     cl, 9  

.text:F4283E83             movzx   eax, cl  

.text:F4283E86             ja      short loc_F4283E8D  

.text:F4283E88             add     eax, 30h  

.text:F4283E8B             jmp     short loc_F4283E90  

.text:F4283E8D ; ---------------------------------------------------------------------------  

.text:F4283E8D  

.text:F4283E8D loc_F4283E8D:                    ; CODE XREF: sub_F4283DEE+98j  

.text:F4283E8D             add     eax, 37h  

.text:F4283E90  

.text:F4283E90 loc_F4283E90:                    ; CODE XREF: sub_F4283DEE+9Dj  

.text:F4283E90             inc     [ebp+arg_8]  

.text:F4283E93             mov     [esi], al  

.text:F4283E95             mov     eax, [ebp+arg_8]  

.text:F4283E98             cmp     eax, [ebp+arg_C]  

.text:F4283E9B             jl      short loc_F4283E57  

.text:F4283E9D             mov     ebx, [ebp+var_18]  

.text:F4283EA0  

.text:F4283EA0 loc_F4283EA0:                    ; CODE XREF: sub_F4283DEE+67j  

.text:F4283EA0             mov     esi, [ebp+var_10]  

.text:F4283EA3             add     esi, [ebp+var_4]  
```



```
.text:F4283EA6             mov     eax, [ebp+arg_C]  
.text:F4283EA9             add     [ebp+var_8], eax  
.text:F4283EAC             dec     [ebp+var_14]  
.text:F4283EAF             mov     [ebp+var_10], esi  
.text:F4283EB2             jnz     short loc_F4283E3E  
.text:F4283EB4  
.text:F4283EB4 loc_F4283EB4:                  ; CODE XREF: sub_F4283DEE+37j  
.text:F4283EB4             test    ebx, ebx  
.text:F4283EB6             jz      short loc_F4283F26  
.text:F4283EB8             mov     ecx, [ebp+arg_4]  
.text:F4283EBB             and     [ebp+arg_10], 0  
.text:F4283EBF             mov     eax, edi  
.text:F4283EC1             imul    eax, [ebp+var_4]  
.text:F4283EC5             add     ecx, eax  
.text:F4283EC7             test    ebx, ebx  
.text:F4283EC9             lea     esi, [ecx+ebx*2]  
.text:F4283ECC             jle     short loc_F4283F22  
.text:F4283ECE             mov     ecx, [ebp+arg_0]  
.text:F4283ED1             mov     eax, edi  
.text:F4283ED3             imul    eax, [ebp+arg_C]  
.text:F4283ED7             add     eax, ecx  
.text:F4283ED9             mov     [ebp+arg_C], eax  
.text:F4283EDC  
.text:F4283EDC loc_F4283EDC:                  ; CODE XREF: sub_F4283DEE+132j  
.text:F4283EDC             mov     eax, [ebp+arg_C]  
.text:F4283EDF             mov     ecx, [ebp+arg_10]  
.text:F4283EE2             mov     cl, [eax+ecx]  
.text:F4283EE5             movzx   eax, cl  
.text:F4283EE8             cdq  
.text:F4283EE9             push    10h  
.text:F4283EEB             pop     ebx  
.text:F4283EEC             idiv    ebx  
.text:F4283EEE             dec     esi  
.text:F4283EEF             cmp     dl, 9  
.text:F4283EF2             movzx   eax, dl  
.text:F4283EF5             ja      short loc_F4283EFC  
.text:F4283EF7             add     eax, 30h  
.text:F4283EFA             jmp     short loc_F4283EFF  
.text:F4283EFC ; ---------------------------------------------------------------------------  
.text:F4283EFC  
.text:F4283EFC loc_F4283EFC:                  ; CODE XREF: sub_F4283DEE+107j  
.text:F4283EFC             add     eax, 37h  
.text:F4283EFF  
.text:F4283EFF loc_F4283EFF:                  ; CODE XREF: sub_F4283DEE+10Cj  
.text:F4283EFF             shr     cl, 4  
.text:F4283F02             mov     [esi], al  
.text:F4283F04             dec     esi  
.text:F4283F05             cmp     cl, 9  
.text:F4283F08             movzx   eax, cl  
.text:F4283F0B             ja      short loc_F4283F12  
.text:F4283F0D             add     eax, 30h  
.text:F4283F10             jmp     short loc_F4283F15  
.text:F4283F12 ; ---------------------------------------------------------------------------  
.text:F4283F12  
.text:F4283F12 loc_F4283F12:                  ; CODE XREF: sub_F4283DEE+11Dj  
.text:F4283F12             add     eax, 37h  
.text:F4283F15  
.text:F4283F15 loc_F4283F15:                  ; CODE XREF: sub_F4283DEE+122j  
.text:F4283F15             inc     [ebp+arg_10]  
.text:F4283F18             mov     ebx, [ebp+var_18]  
.text:F4283F1B             cmp     [ebp+arg_10], ebx  
.text:F4283F1E             mov     [esi], al  
.text:F4283F20             jl      short loc_F4283EDC  
.text:F4283F22  
.text:F4283F22 loc_F4283F22:                  ; CODE XREF: sub_F4283DEE+DEj  
.text:F4283F22             and     [ebp+var_C], 0  
.text:F4283F26  
.text:F4283F26 loc_F4283F26:                  ; CODE XREF: sub_F4283DEE+C8j  
.text:F4283F26             imul    edi, [ebp+var_4]  
.text:F4283F2A             lea     eax, [edi+ebx*2]  
.text:F4283F2D             sub     eax, [ebp+var_C]  
.text:F4283F30             pop     edi  
.text:F4283F31             add     eax, [ebp+arg_4]  
.text:F4283F34             pop     esi  
.text:F4283F35             and     byte ptr [eax], 0  
.text:F4283F38             pop     ebx  
.text:F4283F39             leave  
.text:F4283F3A             retn    14h  
.text:F4283F3A sub_F4283DEE    endp  
.text:F4283F3A  
```

注册算法后续补上:)
