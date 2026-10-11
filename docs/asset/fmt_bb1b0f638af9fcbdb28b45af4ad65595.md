
# 爆破 HASP 保护的国外某专业软件

> 原文：`www.pediy.com/kssd/pediy10/72500.html`

【文章标题】: 爆破 HASP 保护的国外某专业软件  

【文章作者】: qqchunming  

【作者邮箱】: qqchunming@163.com  

【软件名称】: 商业软件保密  

【下载地址】: 下载不到呵呵  

【加壳方式】: 无  

【保护方式】: USB 加密狗，`HASP4.95`  

【编写语言】: `VC`  

【使用工具】: `peid`, `OD`, `Win32Dasm`  

【操作平台】: `XP`  

【软件介绍】: 商业软件不做介绍了。  

新手新人第一次破文，请论坛各位老师多多指点。以下是详细过程：  

\--------------------------------------------------------------------------------------------------------  

1、用 `PEID` 检测，未加壳，`VC` 编写  

2、由于是商用软件，因此本文将把模块名称屏蔽掉，姑且称该程序主程序为 `APP`，两个关键 dll：读狗 DLL 记为 `HLK_DLL`，用户 DLL 记为 `user_dll`，其他 dll 一律记为 `XXX_dll`。跟踪程序后才发现，该程序主模块完全是调用 `user_dll` 加载信息，所有其他 dll 模块完全由 `user_dll` 调用，包括 `HLK_dll`  

3、用 `ollydbg` 载入，中断在入口点，由于没有多少可用信息，节约论坛空间，节约大家时间，不拷贝任何代码  

4、在当前程序领空上单击右键选择 `search for->all interModular calls`，以便寻找蛛丝马迹，无奈找不到任何像是与狗相关的函数，诸如 `HASP`、`HLK`、`Logon`、`LogIn`、`LogOut` 等与狗相关的函数一个都没有  

5、没有线索只能按 `F8` 单步执行，这步要有足够耐心，自己也记不清按了多少 `F8` 了，中间还有 N 次重来，按了将近一个上午，终于来到下面代码处，大家没必要看全部代码，只需注意带注释的那个 CALL。  

\--------------------------------------------------------------------------------------------------------  

```
004015FF  |.  8D95 ACFBFFFF         lea edx,[local.277]  
00401605  |.  52                   push edx  
00401606  |.  8B85 F8FBFFFF        mov eax,[local.258]  
0040160C  |.  50                   push eax  
0040160D  |.  8B8D A8FBFFFF        mov ecx,[local.278]  
00401613  |.  51                   push ecx  
00401614  |.  68 D4E54000          push APP.0040E5D4                            ;  ASCII "EN"  
00401619  |.  8B55 E8              mov edx,[local.6]  
0040161C  |.  52                   push edx  
0040161D  |.  E8 8E230000         call <jmp.&user_dll.sui_In>                ;这个 CALL 只要一执行，就死掉，所以必须跟进  
00401622  |.  8945 EC             mov [local.5],eax  
00401625  |.  8B45 EC             mov eax,[local.5]  
00401628  |>  8B4D F4             mov ecx,[local.3]  
0040162B  |.  64:890D 00000000     mov fs:[0],ecx  
00401632  |.  8B4D E4             mov ecx,[local.7]  
00401635  |.  E8 C72D0000        call APP.00404401  
0040163A  |.  8BE5                mov esp,ebp  
0040163C  |.  5D                  pop ebp  
0040163D  \.  C3                  retn  
```

\--------------------------------------------------------------------------------------------------------  

6、跟进后出现一下代码，看第一行，就是 0040161D 处调用的函数，再看右面的注释，心跳加快，`InitSystem`，从名字上你都能猜想它会干什么，对，读狗的相关代码就应该在这儿初始化（当然还要初始化其他信息），继续跟进  

\--------------------------------------------------------------------------------------------------------  

```
004039B0   $- FF25 C8554000        jmp ds:[<&user_In>]             ;  InitSystem  
004039B6   $- FF25 C4554000        jmp ds:[<&user_Star>]          ;  StartProgram  
004039BC   $- FF25 C0554000        jmp ds:[<&user_Mai>]         
004039C2   $- FF25 AC544000        jmp ds:[<&XXX_DLL.#3>]   
004039C8   $- FF25 A8544000        jmp ds:[<&XXX_DLL.#47>]  
```

\--------------------------------------------------------------------------------------------------------  

7、跟进后，代码是一系列的 `IsBadReadPtr`，由此判断该处是初始化内存工作，离读狗还用有一段距离。部分代码如下，还有更多的 `IsBadReadPtr`，没必要就不贴了，贴一点提示思路即可。  

一直按 `F8`，痛苦的是这段代码中有个循环，要循环 20 多次，稍不留神就结束循环，立刻进入另一个关键 CALL，程序又死掉。所以在这儿又费了我很多时间。  

\--------------------------------------------------------------------------------------------------------  

```
00EF5940 >  55                   push ebp       ;InitSystem 函数  
00EF5941   8BEC                  mov ebp,esp  
00EF5943   81EC 280C0000         sub esp,0C28  
00EF5949   A1 6C2EF100           mov eax,ds:[F12E6C]  
00EF594E   8945 E0               mov ss:[ebp-20],eax  
00EF5951   57                   push edi  
00EF5952   8B45 0C               mov eax,ss:[ebp+C]  
00EF5955   50                   push eax  
00EF5956   68 6848F100          push user_dll.00F14868  
00EF595B   E8 34F70000          call <jmp.&MSVCR71.strcpy>  
00EF5960   83C4 08               add esp,8  
00EF5963   B9 583AF100           mov ecx,user_dll.00F13A58  
00EF5968   85C9                 test ecx,ecx  
00EF596A   74 46                je short user_dll.00EF59B2  
00EF596C   6A 01                push 1  
00EF596E   68 583AF100          push user_dll.00F13A58  
00EF5973   FF15 1072F000       call ds:[<&KERNEL32.IsBadWritePtr>]           ; kernel32.IsBadWritePtr  
00EF5979   85C0                 test eax,eax  
00EF597B   75 35                jnz short user_dll.00EF59B2  
00EF597D   837D 10 00           cmp dword ptr ss:[ebp+10],0  
00EF5981   74 26                je short user_dll.00EF59A9  
00EF5983   6A 01                push 1  
00EF5985   8B55 10               mov edx,ss:[ebp+10]  
00EF5988   52                   push edx  
00EF5989   FF15 1472F000       call ds:[<&KERNEL32.IsBadReadPtr>]           ; kernel32.IsBadReadPtr  
00EF598F   85C0                 test eax,eax  
00EF5991   75 16                jnz short user_dll.00EF59A9  
00EF5993   68 04010000          push 104  
00EF5998   8B45 10               mov eax,ss:[ebp+10]  
00EF599B   50                   push eax  
00EF599C   68 583AF100          push user_dll.00F13A58  
00EF59A1   FF15 1872F000       call ds:[<&KERNEL32.lstrcpynW>]              ; kernel32.lstrcpynW  
```

\--------------------------------------------------------------------------------------------------------  

8、上面代码执行结束后迅速来到下面代码，注释处的 CALL 执行就死，跟进  

\--------------------------------------------------------------------------------------------------------  

```
00EF5D8A   6A 03                push 3  
00EF5D8C   68 10000100          push 10010                         
00EF5D91   8B15 9C3EF100        mov edx,ds:[F13E9C]                
00EF5D97   83C2 12              add edx,12  
00EF5D9A   52                   push edx  
00EF5D9B   68 D80CF100          push user_dll.00F10CD8             
00EF5DA0   A1 9C3EF100          mov eax,ds:[F13E9C]  
00EF5DA5   8B88 2C0F0000        mov ecx,ds:[eax+F2C]  
00EF5DAB   51                   push ecx  
00EF5DAC   E8 45E70000         call <jmp.&XXX_dll.#66_UserMessage>      
00EF5DB1   EB 15                jmp short user_dll.00EF5DC8  
00EF5DB3   E8 580B0000         call user_dll.00EF6910      ;该 CALL 执行就死掉，读狗函数在这里  
00EF5DB8   85C0                 test eax,eax  
00EF5DBA   74 0C                je short user_dll.00EF5DC8  
```

\--------------------------------------------------------------------------------------------------------  

9、跟进后一直按 `F8`，遇到 CALL `user_dll.00XXXXXX` 字样的要格外小心，果然执行到某个 CALL 的时候死掉，只好记住地址，重设断点，重新来，继续跟进死掉的 CALL。由于这儿一直是寻找读狗函数，所以不再贴代码了，大家只看思路，无用代码太多，会扰乱思路.  

10、不厌其烦的重复上述过程突然发现下面代码，相信仅仅函数名足以让你兴奋  

\-------------------------------------------------------------------------------------------------------  

```
00EF6EB8   51                   push ecx  
00EF6EB9   68 680AF100         push user_dll.00F10A68             
00EF6EBE   8B15 9C3EF100       mov edx,ds:[F13E9C]               
00EF6EC4   83C2 12            add edx,12  
00EF6EC7   52                   push edx  
00EF6EC8   E8 85D70000        call <jmp.&HLK_dll.#13_hlk_LogIn>  ;LogIn，登录狗  
00EF6ECD   85C0                 test eax,eax        ;判断返回值，如是 0  
00EF6ECF   0F84 B2000000       je user_dll.00EF6F87      ;跳走，死掉  
00EF6ED5   C705 6C3EF100 01000000 mov dword ptr ds:[F13E6C],1  
00EF6EDF   E8 5CD70000        call <jmp.&HLK_dll.#10_hlk_Avail>   ;读另一个数据，为 ReadDong 做准备，虽然  
00EF6EE4   85C0                 test eax,eax        ;不知道读什么，但返回值如是 0  
00EF6EE6   0F84 9B000000       je user_dll.00EF6F87      ;跳走，死掉  
00EF6EEC   8D85 5CFFFFFF       lea eax,ss:[ebp-A4]  
00EF6EF2   50                   push eax  
```
