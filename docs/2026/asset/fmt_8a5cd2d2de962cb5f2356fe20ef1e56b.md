
```text
.text:00017982 mov [ebp+var_8], eax
.text:00017985 pop eax
.text:00017986 mov eax, [ebp+var_8]
.text:00017989
.text:00017989 loc_17989:
.text:00017989 pop edi
.text:0001798A pop esi
.text:0001798B pop ebx
.text:0001798C leave
.text:0001798D retn 8
.text:0001798D sub_17852 endp
```

## 2、回调函数安装

`sub_180FC`调用`sub_18056`完成`CmpCallback`（`sub_17F38`）回调函数的安装，该函数监视`KiFastCallEntry`的hook点(`dword_18FA8`)指令是否被改变，如有改变调用函数`sub_17EE6`进行恢复。

```text
.text:000180FC sub_180FC proc near
.text:000180FC
.text:000180FC mov edi, edi
.text:000180FE push ebp
.text:000180FF mov ebp, esp
.text:00018101 sub esp, 0Ch
.text:00018104 cmp dword_18F4C, 0 //是否执行过inline hook安装
.text:0001810B jz short locret_1813A
.text:0001810D push 0 ; CSDVersion
.text:0001810F lea eax, [ebp+BuildNumber]
.text:00018112 push eax ; BuildNumber
.text:00018113 lea eax, [ebp+MinorVersion]
.text:00018116 push eax ; MinorVersion
.text:00018117 lea eax, [ebp+MajorVersion]
.text:0001811A push eax ; MajorVersion
.text:0001811B call PsGetVersion
.text:00018120 mov eax, dword_18F50
.text:00018125 add eax, 0FFFFFFFBh
.text:00018128 mov dword_18FA8, eax //保存inline hook点地址
.text:0001812D mov eax, [eax+1]
.text:00018130 mov dword_18F98, eax //保存hook点jmp指令的操作数
.text:00018135 call sub_18056 //安装CmpCallback
.text:0001813A
.text:0001813A locret_1813A:
.text:0001813A leave
.text:0001813B retn
.text:0001813B sub_180FC endp
```

函数`sub_1832C`结尾处调用`PsSetCreateProcessNotifyRoutine(NotifyRoutine, 0)`完成`CreateProcess`回调函数安装,该函数在进程创建时做相关规则检查。

## 三、流程分析

此处通过调试一个注入保护进程的例子，尝试分析保险箱进程保护机制。(代码来自syser debugger)

### 1、KiFastCallEntry分析

向保险箱中添加保护对象`calc.exe`，使用注入器（[DLL注入器.rar](https://bbs.kanxue.com/attach-download-40068.htm)）向`calc.exe`中注入一个DLL，当我们单步调试注入器时会发现：`WriteProcessMemory`函数会导致保险箱弹出提示：“可疑进程企图注入”。

Syser可以方便的从本地r3跟入r0，使用Syser跟入注入器函数`WriteProcessMemory` --> `ntdll!NtWriteProcessMemory`  `sysenter`。。。跟到`KiFastCallEntry`处：

```text
8053E60C FF0538F6DFFF INC [FFDFF638] //调用记数
8053E612 8BF2 MOV ESI,EDX //栈顶指针
8053E614 8B5F0C MOV EBX,[EDI+0C] //指向SSDT
8053E617 33C9 XOR ECX,ECX //
8053E619 8A0C18 MOV CL,[EAX+EBX] //参数长度
8053E61C 8B3F MOV EDI,[EDI] //ServiceTableBase
8053E61E 8B1C87 MOV EBX,[EDI+EAX*4] //EAX序号，EBX服务地址
8053E621 E9D2DA2501 JMP 8179C0F8 //inline hook指向中间跳转
8053E626 8BFC MOV EDI,ESP //复制参数的目标地址
8053E628 3B35D4995580 CMP ESI,MmUserProbeAddress //地址有效检测
8053E62E 0F83A8010000 JNC 8053E7DC //无效转
8053E634 F3A5 REPZ MOVSD //参数复制
8053E636 FFD3 CALL EBX //调用系统服务
8053E638 8BE5 MOV ESP,EBP //恢复堆栈
```

`8053E621`处就是图1中`KiFastCallEntry`的inline hook点，被覆盖的原指令为：

```text
sub esp,ecx //为复制参数开辟空间
shr ecx,2 //参数个数
```

hook后的指令`JMP 8179C0F8`指向：

```text
8179C0F8 E90D071E78 JMP F997C80A //jmp hooker
```

这是个中间跳转，指向保险箱的`hookport.sys`驱动内的hooker代码；使用中间跳转会导致一些rootkit检测工具无法辨析hook点指向了哪个模块。

### 2、hooker分析

继续跟入中间跳进入hooker代码：

```text
F997C80A 8BFF MOV EDI,EDI
F997C80C 9C PUSHFD
F997C80D 60 PUSHAD
F997C80E 57 PUSH EDI //服务表基址
F997C80F 53 PUSH EBX //服务地址
F997C810 50 PUSH EAX //服务序号
F997C811 E840FFFFFF CALL F997C756 //服务调度函数
F997C816 89442410 MOV [ESP+10],EAX //新的服务地址写回堆栈
F997C81A 61 POPAD
F997C81B 9D POPFD
F997C81C 2BE1 SUB ESP,ECX //补回inline hook处被覆盖的指令
F997C81E C1E902 SHR ECX,02 //补回inline hook处被覆盖的指令
F997C821 FF3550DF97F9 PUSH [F997DF50]
F997C827 C3 RET //push ret方式跳回KiFastCallEntry
```

hooker的核心是`F997C811`处的服务调度函数`CALL F997C756`,该函数根据传入的服务号、服务表基址、服务例程入口地址参数及相关规则来判断返回代理服务例程还是原始服务例程，返回的服务例程地址通过`F997C816`处的指令:`MOV [ESP+10],EAX`存入堆栈中，并在`KiFastCallEntry`的`8053E636 CALL EBX`处得到调用。

`F997C811`处调用的服务调度函数：

```text
F997C756 8BFF MOV EDI,EDI
F997C758 55 PUSH EBP
F997C759 8EBC MOV EBP,ESP
F997C75E 3B052CDF97F9 CMP EAX,[F997DF2C] //是否SSDT基址
F997C764 56 PUSH ESI
F997C765 8B7508 MOV ESI,[EBP+08]
F997C768 7508 JNZ F997C772
F997C76A 3B3528DF97F9 CMP ESI,[F997DF28] //SSDT服务序号是否在范围内
F997C770 7654 JNA F997C7C6
F997C772 3B0524DF97F9 CMP EAX,[F997DF24] //是否SSDTShadow基址
F997C778 7542 JNZ F997C7BC
F997C77A 3B3530DF97F9 CMP ESI,[F997DF30] //SSDTShadow服务序号是否在范围内
F997C780 773A JA F997C7BC
F997C782 A1E4E097F9 MOV EAX,[F997E0E4]
F997C787 83BCB0384E00 CMP [EAX+ESI*4+00004E38],+00 //代理服务表中对应SSDTShadow服务的开关
F997C78F 746B JZF997C7FC
F997C791 33D2 XOR EDX,EDX
F997C793 42 INC EDX
F997C794 8BCE MOV ECX,ESI
F997C796 E8D993FFFF CALL F9975B74 //得到过滤规则表中过滤函数开关(SSDTShadow)
F997C79B 85C0 TEST EAX,EAX
F997C79D 745D JZF997C7FC
F997C79F A1E4E097F9 MOV EAX,[F997E0E4]
F997C7A4 8B4D0C MOV ECX,[EBP+0C]
F997C7A7 898CB04C1F00 MOV [EAX+ESI*4+00001F4C],ECX //保存原SSDTShadow服务地址
F997C7AE A1E4E097F9 MOV EAX,[F997E0E4]
F997C7B3 8B84B0F02E00 MOV EAX,[EAX+ESI*4+00002EF0] //返回SSDTShadow代理服务地址
F997C7BA EB43 JMP F997C7FF
F997C7BC 8B0D3CDF97F9 MOV ECX,ntkrnlpa_KeServiceDescriptor
f997C7C2 3B01 CMP EAX,[ECX]
F997C7C4 7536 JNZ F997C7FC
F997C7C6 A1E4E097F9 MOV EAX,[F997E0E4]
F997C7CB 83BCB0943E00 CMP [EAX+ESI*4+00003E94],+00 //代理服务表中对应SSDT服务的开关
F997C7D3 7427 JZ F997C7FC
F997C7D5 33D2 XOR EDX,EDX
F997C7D7 8BCE MOV EAX,ESI
F997C7D9 E89693FFFF CALL F9975B74 //得到过滤规则表中过滤函数开关(SSDT)
F997C7DE 85C0 TEST EAX,EAX
F997C7E0 741A JZ F997C7FC
F997C7E2 A1E4E097F9 MOV EAX,[F997E0E4]
F997C7E7 8B4D0C MOV ECX,[EBP+0C]
F997C7EA 894CB004 MOV [EAX+ESI*4+04],ECX //保存原SSDT服务地址
F997C7EE A1E4E097F9 MOV EAX,[F997E0E4]
F997C7F3 8B84B0A80F00 MOV EAX,[EAX+ESI*4+0000FA8] //返回SSDT代理服务地址
F997C7FA EB03 JMP F997C7FF
F997C7FC 8B450C MOV EAX,[EBP+0C]
F997C7FF 5E POP ESI
F997F800 5D POP EBP
F997C801 C20C00 RET 000C
```

该服务调度函数本质上是根据代理服务表及过滤规则表中当前系统服务对应的开关来决定返回的是代理服务地址还是原始服务地址。

使用IDA的`hex_rays`分析`hookport.sys`的代理服务表创建函数(`sub_1832c`)及过滤规则表创建函数（`subb_105AA`）可以归纳出该服务调度函数用到的两个数据结构：

```text
typedef struct _PROXY_SERVICE_TABLE{
DWORD SSDTCount;
DWORD OrgSSDTServiceAddress[1001]; //保存原始SSDT服务的地址
DWORD PrxSSDTServiceAddress[1001]; //保存Hook后SSDT代理服务地址
DWORD OrgShadowSSDTServiceAddress[1001]; //保存原始ShadowSSDT服务地址
DWORD PrxShadowSSDTServiceAddress[1001]; //保存Hook后ShadowSSDT代理服务地址
DWORD ReplSwitchForSSDT [1001]; //原始服务是否应被替换的开关(for SSDT)
DWORD ReplSwitchForSSDTShadowSSDT[1001];//同上(for ShadowSSDT)
}SERVICE_EXTEND_TABLE;
```

```text
typedef struct _FILTER_RULE_TABLE{
DWORD Size; //自结构尺寸
DWORD Reserve
DWORD IsFilterFunExist; //过滤函数存在
DWORD FilterFun[0x4B] //过滤函数数组//
PULONG SSDTRuleTableBase; //过滤规则表(for SSDT)
PULONG ShadowSSDTRuleTableBase; //过滤规则表(for ShadowSSDT)
}FILTER_RULE_TABLE;
```

（未完）

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [12.JPG](javascript:void(0);)（24.31kb，2175次下载）
- [DLL注入器.rar](javascript:void(0);)（4.93kb，52次下载）
