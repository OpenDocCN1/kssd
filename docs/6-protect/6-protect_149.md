
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



# 代码段 1

```
00EF6EF3 E8 54D70000 call <jmp.&HLK\_dll.#15\_hlk\_ReadDong> ;再读
00EF6EF8 85C0 test eax,eax ;判断返回值，如是 0
00EF6EFA 0F84 87000000 je user\_dll.00EF6F87 ;OVER
00EF6F00 B9 2848F100 mov ecx,user\_dll.00F14828 ;判断 ECX 是否为 0，跟踪发现该值是狗的编号
00EF6F05 85C9 test ecx,ecx ;是 0 与否，不影响程序运行
00EF6F07 74 2B je short user\_dll.00EF6F34
00EF6F09 8D55 98 lea edx,ss:\[ebp-68\] ;该地址存放狗的编号
00EF6F0C 85D2 test edx,edx
00EF6F0E 74 16 je short user\_dll.00EF6F26
00EF6F10 6A 06 push 6
00EF6F12 8D45 98 lea eax,ss:\[ebp-68\]
00EF6F15 50 push eax
00EF6F16 68 2848F100 push user\_dll.00F14828
00EF6F1B FF15 E875F000 call ds:\[<&MSVCR71.strncpy>\] ; MSVCR71.strncpy
```

## 11. 绕过狗检查

聪明的你肯定知道该做什么了，对，只要把 `00EF6ECF`、`00EF6EE6`、`00EF6EFA` 三处的 `je` 改为 `jne` 即可跳过狗的检查。我毫不犹豫地就改了，运行，然而并没有出现预想的兴奋的事情。出现一个错误提示“没有发现可选文件”。

## 12. 可选文件分析

“可选文件”是什么意思？到软件安装目录查看，有个 `OPTION` 文件夹，里面有很多文件。用 `UltraEdit` 打开，看不懂。

## 13. 狗数据与解密

将改文件夹中的所有文件删除，带狗运行，出现和 `11` 步同样的提示；恢复文件，运行正常。由此可见，解密那些文件需要狗中的数据。大家知道 `HASP` 狗用的是 `AES`，难道狗中存放着 `AES` 密码？？带着这样的疑问跟踪程序，来到下面代码：

```
00EF723F 8B8D 68F6FFFF mov ecx,ss:\[ebp-998\] ;压入的三个参数意义就不说明了，因为没必要
00EF7245 51 push ecx
00EF7246 8D95 64F8FFFF lea edx,ss:\[ebp-79C\]
00EF724C 52 push edx
00EF724D 8D85 BCFAFFFF lea eax,ss:\[ebp-544\]
00EF7253 50 push eax
00EF7254 E8 47180000 call user\_dll.00EF8AA0 ;该函数执行完后，EDX 会出现一些明码，明码信息
```

大概为 `APPLICAITON=XXX;OPTION=XXX.....` 是不是很像文件启动需要检查的选项？所以要弄懂解密算法，必须跟进。

## 14. 解密过程追踪

跟进后，部分代码如下，这部分代码很关键，所以贴的比较多。但是有很多是内核函数，很容易读懂。

```
00EF8B30 8D8D 68FDFFFF lea ecx,ss:\[ebp-298\]
00EF8B36 51 push ecx
00EF8B37 8B55 0C mov edx,ss:\[ebp+C\]
00EF8B3A 83C2 2C add edx,2C
00EF8B3D 52 push edx
00EF8B3E 68 5038F100 push user\_dll.00F13850
00EF8B43 E8 14BD0000 call <jmp.&XXX\_DLL.#100\_ExtendPathName> ; 获取文件完整路径
00EF8B48 6A 00 push 0
00EF8B4A 8D85 70FFFFFF lea eax,ss:\[ebp-90\]
00EF8B50 50 push eax
00EF8B51 8D8D 68FDFFFF lea ecx,ss:\[ebp-298\]
00EF8B57 51 push ecx
00EF8B58 E8 63BC0000 call <jmp.&XXX\_DLL.#47\_MakeSString>
00EF8B5D 50 push eax
00EF8B5E FF15 0872F000 call ds:\[<&KERNEL32.OpenFile>\] ; 打开文件
00EF8B64 8985 54FDFFFF mov ss:\[ebp-2AC\],eax
00EF8B6A 83BD 54FDFFFF FF cmp dword ptr ss:\[ebp-2AC\],-1
00EF8B71 74 55 je short user\_dll.00EF8BC8
00EF8B73 8B95 50FDFFFF mov edx,ss:\[ebp-2B0\]
00EF8B79 52 push edx
00EF8B7A 8B85 60FDFFFF mov eax,ss:\[ebp-2A0\] ;存放读入数据的缓冲区，关键（后面破解用上）
00EF8B80 50 push eax
00EF8B81 8B8D 54FDFFFF mov ecx,ss:\[ebp-2AC\]
00EF8B87 51 push ecx
00EF8B88 FF15 F071F000 call ds:\[<&KERNEL32.\_hread>\] ; 读文件，ebp-2A0 为存放读入数据的缓冲区
00EF8B8E 3B85 50FDFFFF cmp eax,ss:\[ebp-2B0\]
00EF8B94 75 19 jnz short user\_dll.00EF8BAF
00EF8B96 8B55 10 mov edx,ss:\[ebp+10\]
00EF8B99 52 push edx
00EF8B9A 8B85 64FDFFFF mov eax,ss:\[ebp-29C\]
00EF8BA0 50 push eax
00EF8BA1 8B8D 60FDFFFF mov ecx,ss:\[ebp-2A0\] ;存放解密后的数据，和读入缓冲区地址相同
00EF8BA7 51 push ecx
00EF8BA8 E8 B1BA0000 call <jmp.&HLK\_dll.#12\_hlk\_Crypt> ;对读入的数据进行解密，须弄懂算法
00EF8BAD EB 0A jmp short user\_dll.00EF8BB9
00EF8BAF C785 64FDFFFF 00000000 mov dword ptr ss:\[ebp-29C\],0
00EF8BB9 8B95 54FDFFFF mov edx,ss:\[ebp-2AC\]
00EF8BBF 52 push edx
00EF8BC0 FF15 F471F000 call ds:\[<&KERNEL32.\_lclose>\] ; 关闭文件
00EF8BC6 EB 0A jmp short user\_dll.00EF8BD2
00EF8BC8 C785 64FDFFFF 00000000 mov dword ptr ss:\[ebp-29C\],0 ;后面的函数不做解释了，只需知道解密后的
00EF8BD2 8B85 60FDFFFF mov eax,ss:\[ebp-2A0\] ;数据存放在 ebp-2A0
00EF8BD8 0385 64FDFFFF add eax,ss:\[ebp-29C\]
00EF8BDE C600 00 mov byte ptr ds:\[eax\],0
00EF8BE1 8B4D 08 mov ecx,ss:\[ebp+8\]
00EF8BE4 8B95 60FDFFFF mov edx,ss:\[ebp-2A0\]
00EF8BEA 8911 mov ds:\[ecx\],edx
00EF8BEC 8B85 64FDFFFF mov eax,ss:\[ebp-29C\]
00EF8BF2 8B4D FC mov ecx,ss:\[ebp-4\]
00EF8BF5 E8 E8C30000 call user\_dll.00F04FE2
00EF8BFA 8BE5 mov esp,ebp
00EF8BFC 5D pop ebp
00EF8BFD C3 retn
```

## 15. 利用自身程序解密

为了得到解密后的数据，跟进 `00EF8BA8` 处的 `call <jmp.&HLK\_dll.#12\_hlk\_Crypt>`，看看到底是什么算法。可是乐观的情绪又受打击，该函数有大量的花指令，菜鸟阶段，不懂如何去除花指令，研究肯定不是一天两天的事。所以不能在一棵树上吊死，换个思维，为什么不能用程序本身的代码进行解密呢！！！

对，就让程序自身帮我们干活！说干就干，在步骤 `13` 的 `00EF723F` 处设断，反复运行程序，每执行一次查看 `ebp-2A0` 处的值，并 dump 出来，另存为 16 进制文件，这些文件就是解密过的文件，将其覆盖原来的加密文件。剩下要做的就是跳过上面代码中的解密模块（`\_hlk\_Crypt`）。

这个很简单，来到 `00EF8B8E` 处，即程序刚进行完 `\_hread` 之后，将该处的代码 `cmp eax,ss:\[ebp-2B0\]` 改为 `jmp 00EF8BB9` 然后汇编。由于 `\_hread` 函数和 `\_hlk\_Crypt` 函数用的是同一个缓冲区，所以 `\_hread` 函数执行完毕后，`ebp-2A0` 处的已解密数据就可顺利为后续程序使用了。

## 16. 启动成功

经过上面修改后，按 `F9`，程序顺利启动！！！！

【继续完善】

## 17. 消除定时检查

程序运行一段时间后，弹出对话框，通知确定狗的存在！！看来程序启动中还是要定时检查的。

*   **第一个念头**：查找 `SetTimer` 函数，果然找到，且只有一次调用，心想就是这个函数在作怪了，该函数的 `uElapsed` 值为 `1` 秒，我将其改为 `9A7EC800`，即 `30` 天。运行，可是仍然定时检查狗的存在。
*   **第二个念头**：查找其他有关时间的函数，但感觉有点费劲，进而转第三个念头。
*   **第三个念头**：以其他的函数设断点，如大家常用的 `CreateFile`、`DeviceIoControl` 等，但没用过，感觉肯定不是跟踪一两句代码就可搞定的事情，进而转第四个念头。
*   **第四个念头**：一劳永逸的想法，从源头解决问题。为什么不直接修改读狗函数的返回值？这样程序无论何时检查狗都无所谓。

重新跟进 `hlk\_LogIn`、`hlk\_Avail`、`hlk\_ReadDong` 三个函数，发现这三个函数返回前都有共同的代码：

```
00B5126C 837D FC 00 cmp dword ptr ss:\[ebp-4\],0 ;看 ebp-4 是否为 0
00B51270 0F94C0 sete al ;是，则 eax＝1，有狗
00B51273 8B4D F4 mov ecx,ss:\[ebp-C\]
00B51276 E8 C6640400 call HLK\_dll.00B97741 ;该 CALL 并不是真正函数，经验证不影响上面的 EAX 值
00B5127B 8BE5 mov esp,ebp
00B5127D 5D pop ebp
00B5127E C2 0C00 retn 0C
```

不插狗的情况下 `\[ebp-4\]` ＝ `9`，因此 `eax` ＝ `0`，无狗，因此只需将 `sete al` 改为 `setne al`，汇编，即可。

至此，打狗结束，程序非常正常。将修改后的文件另存到可执行文件，作为补丁，覆盖原文件，狗解除。

## 后记

由于这是本人正式调试的第一个大型程序，也是第一次写破文，中间走了不少弯路，将调试心得写出来，作为后记，总结经验。

1.  动态调试前要充分发挥 `Win32Dasm` 等静态反汇编工具的作用，挖掘蛛丝马迹。如字符串提示、对话框提示等。
2.  对于用狗加密的商业软件，上面的提示可能比较难于找到，但可以先浏览安装目录，看一看都有那些 `dll`，能从名字猜出含义更好，尤其关注 `HLK` 等与狗相关的字样（偶接触狗不多，所以也总结不出多少狗的名字呵呵，只是要有个意识）。
3.  有了第二步的信息之后，运用 `Win32Dasm` 等工具先查看可疑 `dll` 的输入输出表，看看之间的相互调用关系，尤其关注读狗 `dll` 是哪个模块调用的，这会大大节省搜索读狗函数的时间。



比如本文，如果前期我做了这个工作，就会知道 `user.dll` 调用了 `HLK.dll`，那么当程序领空一旦来到 `user.dll` 的时候（实际很快），再运用 `search for－>all interModular calls` 功能，就会立刻定位到 `hlk\_LogIn`、`hlk\_Avail`、`hlk\_ReadDong` 三个读狗函数，可能会节约一个上午的 `F8`

:)

4、该条建议尤其致心浮的又想学破解的朋友们（可能由于自己是老师，罗嗦了）：一定要拿起纸和笔，跟踪子程序，除非你是高手，有丰富经验，否则尽量画出堆栈存取示意图，这样你就不会纳闷为什么别人经常如此轻易的就判读某个地方的内容了。虽然你刚接触汇编、虽然你刚接触破解、虽然 `windows` 编程也不是那么棒，但只要有毅力、有耐心，终究会成功的。既然要学破解，就要有一座就是几天的毅力，当最后收获喜悦的时候，那份感觉给个神仙都不做：）

5、写程序是灵活的，同样破解也是灵活的，千万不要在一棵树上吊死，此路不同，另行他法，总有能绕过去的办法。重要的是掌握方法和思路，剩下的就是我们聪明的大脑，你要相信程序作者的水平未必比你高到哪里去。

6、要分析作者的思路，但也不要被作者的思路牵着鼻子走，否则可能会走弯路，就像本文解密那一块。

以上是本人的第一篇破文，来看雪学习快一个月了，很喜欢这里的气氛，和谐社会这里才有！！！感谢看雪，今天学生就以一篇文章呈上，

虽然我的职业是老师，但这儿的人却都是我的老师。感谢大家看完，欢迎指导。
