

# Acoustica2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50062.htm`](https://www.pediy.com/kssd/pediy05/pediy50062.htm)

音乐处理 acoustica2.0 注册码破解及注册机   
  
工具：TRW2000，Keymake，w32dasm   
软件介绍：acoustica2.0，一个功能强大的声音处理程序，能实现声音文件的录制、编辑及一些特效，象 Cooledit 等软件一样。   
  
1、运行程序，弹出注册框，输入 name：esoft，Company：dzzx，Key code:123456，按 Register。出来一个对话框，只有一个标题不知道是什么意思。软件要重新运行软件才检查注册成功与否。关闭软件，我门来找它把注册信息放的哪里，经过查找注册码存放在注册表里面，而且是明文形式的。   
  
HKEY\_CURRENT\_USER\\Software\\Acon AS\\Acoustica\\2.0\\RegisterInfo   
name：esoft   
Company：dzzx   
Key code:123456   
  
2、启动 trw，load 软件，在入口处被 trw 断下，下断点   
bpx RegQueryValueExa do "db \*(esp+8)"   
按 F5 几次后，在 trw 数据窗口的右上角区域可以看到存放注册信息的几个键的名称 name、Company、Key 等字样。   
bd \*   
pmodule   
这时按 F10 单步跟踪，不久来到下面：   
  
:00446D73 83C418                  add esp, 00000018   
:00446D76 8D8D60FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF60\]   
:00446D7C A1168B4A00              mov eax, dword ptr \[004A8B16\]   
:00446D81 6A50                    push 00000050   
:00446D83 6829A04900              push 0049A029   
:00446D88 51                      push ecx   
  
\* Possible StringData Ref from Data Obj ->"Company"   
                                  |   
:00446D89 6821A04900              push 0049A021   
  
\* Possible StringData Ref from Data Obj ->"RegisterInfo"   
                                  |   
:00446D8E 6814A04900              push 0049A014   
:00446D93 50                      push eax   
:00446D94 E807CEFCFF              call 00413BA0   
:00446D99 83C418                  add esp, 00000018   
:00446D9C 8D9510FFFFFF            lea edx, dword ptr \[ebp+FFFFFF10\]   
:00446DA2 8B0D168B4A00            mov ecx, dword ptr \[004A8B16\]   
:00446DA8 6A50                    push 00000050   
:00446DAA 683BA04900              push 0049A03B   
:00446DAF 52                      push edx   
  
\* Possible StringData Ref from Data Obj ->"Key"   
                                  |   
:00446DB0 6837A04900              push 0049A037   
  
\* Possible StringData Ref from Data Obj ->"RegisterInfo"   
                                  |   
:00446DB5 682AA04900              push 0049A02A   
:00446DBA 51                      push ecx   
:00446DBB E8E0CDFCFF              call 00413BA0   
:00446DC0 83C418                  add esp, 00000018   
  
\* Possible Ref to Menu: MenuID\_0064, Item: "Draw Freehand Volume Curve..."   
                                  |   
  
\* Possible Reference to String Resource ID=00001: "Enter an arbitary volume curve"   
                                  |   
:00446DC3 BB01000000              mov ebx, 00000001   
:00446DC8 8D7DB0                  lea edi, dword ptr \[ebp\-50\]   
:00446DCB EB16                    jmp 00446DE3   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00446DEF(C)   
|   
:00446DCD 0FBE07                  movsx eax, byte ptr \[edi\]   
:00446DD0 8BD3                    mov edx, ebx   
:00446DD2 83E20F                  and edx, 0000000F   
:00446DD5 0FBE8C1500FFFFFF        movsx ecx, byte ptr \[ebp+edx\-00000100\]   
:00446DDD F7E9                    imul ecx   
:00446DDF 03F0                    add esi, eax   
:00446DE1 43                      inc ebx   
:00446DE2 47                      inc edi   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00446DCB(U)   
|   
:00446DE3 8D45B0                  lea eax, dword ptr \[ebp\-50\]   
:00446DE6 50                      push eax   
:00446DE7 E870520300              call 0047C05C   
:00446DEC 59                      pop ecx   
:00446DED 3BD8                    cmp ebx, eax   
:00446DEF 76DC                    jbe 00446DCD   
  
\* Possible Ref to Menu: MenuID\_0064, Item: "Draw Freehand Volume Curve..."   
                                  |   
  
\* Possible Reference to String Resource ID=00001: "Enter an arbitary volume curve"   
                                  |   
:00446DF1 BB01000000              mov ebx, 00000001   
:00446DF6 8DBD60FFFFFF            lea edi, dword ptr \[ebp+FFFFFF60\]   
:00446DFC EB16                    jmp 00446E14   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00446E23(C)   
|   
:00446DFE 0FBE07                  movsx eax, byte ptr \[edi\]   
:00446E01 8BD3                    mov edx, ebx   
:00446E03 83E20F                  and edx, 0000000F   
:00446E06 0FBE8C15F0FEFFFF        movsx ecx, byte ptr \[ebp+edx\-00000110\]   
:00446E0E F7E9                    imul ecx   
:00446E10 03F0                    add esi, eax   
:00446E12 43                      inc ebx   
:00446E13 47                      inc edi   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00446DFC(U)   
|   
:00446E14 8D8560FFFFFF            lea eax, dword ptr \[ebp+FFFFFF60\]   
:00446E1A 50                      push eax   
:00446E1B E83C520300              call 0047C05C   
:00446E20 59                      pop ecx   
:00446E21 3BD8                    cmp ebx, eax   
:00446E23 76D9                    jbe 00446DFE   
:00446E25 8BC6                    mov eax, esi   
:00446E27 B9A0860100              mov ecx, 000186A0   
:00446E2C 33D2                    xor edx, edx   
:00446E2E F7F1                    div ecx   
:00446E30 52                      push edx   
  
\* Possible StringData Ref from Data Obj ->"AC200-%d"   
                                  |   
:00446E31 683CA04900              push 0049A03C   
:00446E36 8D85A0FEFFFF            lea eax, dword ptr \[ebp+FFFFFEA0\]   
:00446E3C 50                      push eax   
:00446E3D E8CA890300              call 0047F80C（算注册码，形式为 AC200-xxxxx 后面是五位）   
:00446E42 83C40C                  add esp, 0000000C（在此处 D ECX 看到注册码）   
:00446E45 8D9510FFFFFF            lea edx, dword ptr \[ebp+FFFFFF10\]   
:00446E4B 52                      push edx   
:00446E4C 8D8DA0FEFFFF            lea ecx, dword ptr \[ebp+FFFFFEA0\]   
:00446E52 51                      push ecx   
  
\* Reference To: KERNEL32.lstrcmpA, Ord:0000h   
                                  |   
:00446E53 E89E470400              Call 0048B5F6（这里调用 lstrcmpA 函数比较注册码）   
:00446E58 85C0                    test eax, eax（若相同则 EAX 的值为 0）   
:00446E5A 0F94C0                  sete al   
:00446E5D 83E001                  and eax, 00000001   
:00446E60 5F                      pop edi   
:00446E61 5E                      pop esi   
:00446E62 5B                      pop ebx   
:00446E63 8BE5                    mov esp, ebp   
:00446E65 5D                      pop ebp   
:00446E66 C3                      ret   
  
  
:00446E67 90                      nop   
  
上面的这段代码是验证注册码的部分，程序在启动时和注册时都要调用它。   
  
:00447100 E81BFCFFFF              call 00446D20（这里是启动时的调用）   
:00447105 59                      pop ecx   
:00447106 84C0                    test al, al（未注册时 AL 的值是 0）   
:00447108 7411                    je 0044711B（这里不跳就爆破，不提示注册也没有时间限制）   
:0044710A 8B8514FDFFFF            mov eax, dword ptr \[ebp+FFFFFD14\]   
:00447110 64A300000000            mov dword ptr fs:\[00000000\], eax   
:00447116 E91C050000              jmp 00447637   
…………   
:00447599 E882F7FFFF              call 00446D20（这里是注册时的调用）   
:0044759E 59                      pop ecx   
:0044759F 84C0                    test al, al   
:004475A1 7440                    je 004475E3   
  
\* Possible Reference to Dialog: DialogID\_0072, CONTROL\_ID:00C8, ""   
                                  |   
:004475A3 68C8000000              push 000000C8   
:004475A8 8D9FB6000000            lea ebx, dword ptr \[edi+000000B6\]   
:004475AE 53                      push ebx   
:004475AF 68B4230000              push 000023B4   
:004475B4 8D4704                  lea eax, dword ptr \[edi+04\]   
:004475B7 50                      push eax   
:004475B8 E8F8B40200              call 00472AB5   
:004475BD 83C410                  add esp, 00000010   
:004475C0 8BD3                    mov edx, ebx   
:004475C2 8B4F66                  mov ecx, dword ptr \[edi+66\]   
:004475C5 8B01                    mov eax, dword ptr \[ecx\]   
:004475C7 6A00                    push 00000000   
  
\* Possible StringData Ref from Data Obj ->"Acoustica"   
                                  |   
:004475C9 6807A14900              push 0049A107   
:004475CE 52                      push edx   
:004475CF 8B500C                  mov edx, dword ptr \[eax+0C\]   
:004475D2 52                      push edx   
:004475D3 8B4868                  mov ecx, dword ptr \[eax+68\]   
:004475D6 51                      push ecx   
:004475D7 E83AD60100              call 00464C16（只有标题的对话框）   
:004475DC 83C414                  add esp, 00000014   
:004475DF 33DB                    xor ebx, ebx   
:004475E1 EB40                    jmp 00447623   
  
3、注册机：   
使用“注册机编写器（Keymaker）”之“另类注册机”功能   
1、程序名称：acoustica.exe   
2、添加数据：   
　　中断地址：446E42   
    中断次数：2   
　　第一字节：83   
　　指令长度：3   
3、选择内存方式 ECX。   
  
通过对程序的分析我们知道其注册码形式为 AC200-xxxxx（后面是五位），当我们不填 name 和 Company 时，在 Keycode 框中直接填入 AC200-0，奇异的事情发生了，竟然能够注册成功，这也算是程序的一个漏洞吧。   
  
esoft2001.51.net   
2002 年 4 月 5 日  4dd466d0e1e0cc123afbe504da2462ca