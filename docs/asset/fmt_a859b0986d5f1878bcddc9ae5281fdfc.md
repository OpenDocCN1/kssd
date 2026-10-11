
# CheckFiles V1.8

> 原文：[`www.pediy.com/kssd/pediy06/pediy6143.htm`](https://www.pediy.com/kssd/pediy06/pediy6143.htm)

【软件名称】：`CheckFiles V1.8` 

【软件大小】：209 KB 

【软件类别】：国外软件 / 共享版 / 文件管理 

【加入时间】：2003-07-02 07:50:05 

【开 发 商】：`http://www.lightlink.com/ym/chkfiles.htm` 

【软件简介】：该软件可以跟踪指定目录下的文件那些是新增加的，哪些已经被删除，哪些是已经修改过的或者是损坏了的。 

【作 者】：`cyclotron` 

【破解过程】：用 `Fi2.45` 检查，`VC 5.0` 编写，无壳。于是用 `W32Dasm` 反汇编后查找错误信息，找到关键点如下： 

```
:00408483 E8017B0100        call 0041FF89  
:00408488 8BC8              mov ecx, eax  
:0040848A E8217C0100        call 004200B0  
/\* 取用户名位数 \*/  
:0040848F 85C0              test eax, eax  
:00408491 7518              jne 004084AB  
:00408493 50                push eax 
\* Possible StringData Ref from Data Obj ->"CheckFiles Registration"  
                           |  
:00408494 6848ED4200        push 0042ED48 
\* Possible StringData Ref from Data Obj ->"You need to enter a user name."  
                           |  
:00408499 6828ED4200        push 0042ED28  
:0040849E 8BCE              mov ecx, esi  
:004084A0 E87DA20100        call 00422722  
:004084A5 5F                pop edi  
:004084A6 5E                pop esi  
:004084A7 83C420            add esp, 00000020  
:004084AA C3                ret 
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00408491(C)  
|  
:004084AB 83F814            cmp eax, 00000014  
/\* 用户名是否在 20 位以内？ \*/  
:004084AE 7E19              jle 004084C9  
:004084B0 6A00              push 00000000 
\* Possible StringData Ref from Data Obj ->"CheckFiles Registration"  
                           |  
:004084B2 6848ED4200        push 0042ED48 
\* Possible StringData Ref from Data Obj ->"The user name must be 20 characters "  
                             ->"or less."  
                           |  
:004084B7 68F8EC4200        push 0042ECF8  
:004084BC 8BCE              mov ecx, esi  
:004084BE E85FA20100        call 00422722  
:004084C3 5F                pop edi  
:004084C4 5E                pop esi  
:004084C5 83C420            add esp, 00000020  
:004084C8 C3                ret 
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004084AE(C)  
|  
:004084C9 8D44240C          lea eax, dword ptr [esp+0C]  
:004084CD 6A17              push 00000017  
:004084CF 50                push eax 
\* Possible Reference to Dialog: DialogID_0087, CONTROL_ID:040F, ""  
                           |  
:004084D0 680F040000        push 0000040F  
:004084D5 8BCE              mov ecx, esi  
:004084D7 E8AD7A0100        call 0041FF89  
:004084DC 8BC8              mov ecx, eax  
:004084DE E8CD7B0100        call 004200B0  
/\* 取试炼码位数 \*/  
:004084E3 85C0              test eax, eax  
:004084E5 7518              jne 004084FF  
:004084E7 50                push eax 
\* Possible StringData Ref from Data Obj ->"CheckFiles Registration"  
                           |  
:004084E8 6848ED4200        push 0042ED48 
\* Possible StringData Ref from Data Obj ->"You need to enter a registration "  
                             ->"number."  
                           |  
:004084ED 68CCEC4200        push 0042ECCC  
:004084F2 8BCE              mov ecx, esi  
:004084F4 E829A20100        call 00422722  
:004084F9 5F                pop edi  
:004084FA 5E                pop esi  
:004084FB 83C420            add esp, 00000020  
:004084FE C3                ret 
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004084E5(C)  
|  
:004084FF 8D4C2408          lea ecx, dword ptr [esp+08]  
:00408503 8D54240C          lea edx, dword ptr [esp+0C]  
/\* 取试炼码地址 \*/  
:00408507 51                push ecx 
\* Possible StringData Ref from Data Obj ->"%lu"  
                           |  
:00408508 6854E24200        push 0042E254  
:0040850D 52                push edx  
:0040850E E8AD100000        call 004095C0  
/\* 判断试炼码是否全是数字，若是则转为 16 进制，不是则给出错误信息 \*/  
:00408513 83C40C            add esp, 0000000C  
:00408516 83F801            cmp eax, 00000001  
:00408519 7419              je 00408534  
:0040851B 6A00              push 00000000 
\* Possible StringData Ref from Data Obj ->"CheckFiles Registration"  
                           |  
:0040851D 6848ED4200        push 0042ED48 
\* Possible StringData Ref from Data Obj ->"You need to enter a valid registration "  
                             ->"number."  
                           |  
:00408522 689CEC4200        push 0042EC9C  
:00408527 8BCE              mov ecx, esi  
:00408529 E8F4A10100        call 00422722  
:0040852E 5F                pop edi  
:0040852F 5E                pop esi  
:00408530 83C420            add esp, 00000020  
:00408533 C3                ret 
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00408519(C)  
|  
:00408534 8B442408          mov eax, dword ptr [esp+08]  
/\* 16 进制值送 eax \*/  
:00408538 85C0              test eax, eax  
:0040853A 7519              jne 00408555  
:0040853C 6A00              push 00000000 
\* Possible StringData Ref from Data Obj ->"CheckFiles Registration"  
                           |  
:0040853E 6848ED4200        push 0042ED48 
\* Possible StringData Ref from Data Obj ->"You need to enter a valid registartion "  
                             ->"number."  
                           |  
:00408543 686CEC4200        push 0042EC6C  
:00408548 8BCE              mov ecx, esi  
:0040854A E8D3A10100        call 00422722  
:0040854F 5F                pop edi  
:00408550 5E                pop esi  
:00408551 83C420            add esp, 00000020  
:00408554 C3                ret 
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0040853A(C)  
|  
:00408555 68282D4300        push 00432D28  
/\* 用户名地址入栈 \*/  
:0040855A E8A18AFFFF        call 00401000  
/\* 算法 call \*/  
:0040855F 8B4C240C          mov ecx, dword ptr [esp+0C]  
:00408563 83C404            add esp, 00000004  
:00408566 3BC8              cmp ecx, eax  
/\* 关键比较 \*/  
:00408568 7419              je 00408583  
/\* 一定要跳 \*/  
:0040856A 6A00              push 00000000 
\* Possible StringData Ref from Data Obj ->"CheckFiles Registration"  
                           |  
:0040856C 6848ED4200        push 0042ED48 
\* Possible StringData Ref from Data Obj ->"Sorry, this registration number "  
                             ->"is not valid."  
                           |  
:00408571 683CEC4200        push 0042EC3C  
:00408576 8BCE              mov ecx, esi  
:00408578 E8A5A10100        call 00422722  
:0040857D 5F                pop edi  
:0040857E 5E                pop esi  
:0040857F 83C420            add esp, 00000020  
:00408582 C3                ret 
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00408568(C)  
| 
\* Possible StringData Ref from Data Obj ->"ww"  
                           |  
:00408583 68FCE34200        push 0042E3FC 
\* Possible StringData Ref from Data Obj ->"chkfiles.ser"  
                           |  
:00408588 685CE24200        push 0042E25C  
:0040858D E86E120000        call 00409800  
:00408592 8BF8              mov edi, eax  
:00408594 83C408            add esp, 00000008  
```
