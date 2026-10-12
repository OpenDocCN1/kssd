
# riijj 姐姐的 Riijj crackme 11 sp2 anti 分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-459.htm`](https://www.pediy.com/kssd/pediy09/pediy09-459.htm)

> 【文章标题】: `riijj` 姐姐的 `Riijj` `crackme` `11` `sp2` `anti` 分析  
> 【文章作者】: 绫濑遥  
> 【软件名称】: `riijj` 姐姐的 `Riijj` `crackme` `11` `sp2`  
> 【下载地址】: `http://bbs.pediy.com/showthread.php?threadid=38021`  
> 【加壳方式】: 无壳  
> 【使用工具】: `OllyICE`  
> 【操作平台】: `Windows` `2000`  
> 【软件介绍】: `riijj` 姐姐的 `Riijj` `crackme` `11` `sp2`  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> --------------------------------------------------------------------  
> 【前面的话】  
>   前几天写过一篇 `Riijjpack` (`version` `1.0`) `notepad` 脱壳分析的文章  
>   `http://bbs.pediy.com/showthread.php?threadid=37858`  
>   但是看雪老师没有给精华, `55555555555555555555555555555555555`  
>   `55555555555555555555555555555555555555555555555555555555555`  
>   这次卷土重来, 试练品对象仍然是 `riijj` 姐姐的作品  
>   原贴在 `http://bbs.pediy.com/showthread.php?threadid=38021`  
>   
>   【详细过程】  
>   
>   `riijj` 姐姐的 `Riijj` `crackme` `11` `sp2` 非常强悍, 用 `OllyICE` 跑不起来  
>   于是想到使用看雪老师的 `HideOD` 最新插件, 居然也跑不起来, `55555`  
>   `55555555555555555555555555555555555555555555555555555555555`  
>   这可怎么办呀? 只能自己分析了, 分析的目的是为了找出 `anti`, 而不  
>   是算法, 因为像我这样的还没有能力分析算法.  
>   
>   【`OllyICE` 载入】  
>   
> ```
> 004022AA >  55               push ebp                    我是 OEP
> 004022AB   8BEC             mov ebp,esp
> 004022AD   6A FF            push -1
> 004022AF   68 40A24000      push riijjcm1.0040A240
> 004022B4   68 E81F4000      push riijjcm1.00401FE8
> 004022B9   64:A1 00000000   mov eax,dword ptr fs:[0]
> 004022BF   50               push eax
> 004022C0   64:8925 0000000>mov dword ptr fs:[0],esp
> 004022C7   83EC 58          sub esp,58
> 004022CA   53               push ebx
> 004022CB   56               push esi
> 004022CC   57               push edi
> 004022CD   8965 E8          mov dword ptr ss:[ebp-18],esp
> 004022D0   FF15 98A04000    call dword ptr ds:[<&KERNEL32.GetVersion>]    
> ...
> 00402363   6A 0A            push 0A
> 00402365   58               pop eax                 
> 00402366   50               push eax
> 00402367   FF75 9C          push dword ptr ss:[ebp-64]
> 0040236A   56               push esi
> 0040236B   56               push esi
> 0040236C   FF15 94A04000    call dword ptr ss:[<&KERNEL32.GetModuleHandleA>]
> 00402372   50               push eax
> 00402373   E8 38F8FFFF      call riijjcm1.00401BB0          我是 WinMain
> ```
>   
>   【进 `WinMain`】  
>   
> ```
> 00401BB0   55               push ebp
> 00401BB1   8BEC             mov ebp,esp
> 00401BB3   6A FF            push -1
> 00401BB5   68 18A24000      push riijjcm1.0040A218
> 00401BBA   68 E81F4000      push riijjcm1.00401FE8
> 00401BBF   64:A1 00000000   mov eax,dword ptr fs:[0]
> 00401BC5   50               push eax
> 00401BC6   64:8925 0000000>mov dword ptr fs:[0],esp        装入 SEH
> 00401BCD   83EC 08          sub esp,8
> 00401BD0   53               push ebx
> 00401BD1   56               push esi
> 00401BD2   57               push edi
> 00401BD3   8965 E8          mov dword ptr ss:[ebp-18],esp
> 00401BD6   C745 FC 0000000>mov dword ptr ss:[ebp-4],0      __try{ mov [0], 0 }   //seh1
> 00401BDD   33C0             xor eax,eax
> 00401BDF   C600 00          mov byte ptr ds:[eax],0
> 00401BE2   C745 FC FFFFFFF>mov dword ptr ss:[ebp-4],-1
> 00401BE9   EB 4C            jmp short riijjcm1.00401C37
> 00401BEB   B8 01000000      mov eax,1                   __except(1)
> 00401BF0   C3               retn
> 00401BF1   8B65 E8          mov esp,dword ptr ss:[ebp-18]  {}
> 00401BF4   C745 FC 0100000>mov dword ptr ss:[ebp-4],1      __try{ 0/0 }          //seh2
> 00401BFB   33C0             xor eax,eax
> 00401BFD   33D2             xor edx,edx
> 00401BFF   F7F0             div eax
> 00401C01   83C8 FF          or eax,FFFFFFFF
> 00401C04   EB 2B            jmp short riijjcm1.00401C31
> 00401C06   B8 01000000      mov eax,1                   __except(1)
> 00401C0B   C3               retn
> 00401C0C   8B65 E8          mov esp,dword ptr ss:[ebp-18]  {}
> 00401C0F   C745 FC 0200000>mov dword ptr ss:[ebp-4],2      __try{ call 401770 }  //seh2
> 00401C16   E8 55FBFFFF      call riijjcm1.00401770
> 00401C1B   EB 0E            jmp short riijjcm1.00401C2B
> 00401C1D   B8 01000000      mov eax,1                   __except(1)
> 00401C22   C3               retn
> 00401C23   8B65 E8          mov esp,dword ptr ss:[ebp-18]  {}
> 00401C26   E8 75FDFFFF      call riijjcm1.004019A0         call 4019A0
> 00401C2B   83C8 FF          or eax,FFFFFFFF
> 00401C2E   8945 FC          mov dword ptr ss:[ebp-4],eax
> 00401C31   8945 FC          mov dword ptr ss:[ebp-4],eax
> 00401C34   8945 FC          mov dword ptr ss:[ebp-4],eax
> 00401C37   33C0             xor eax,eax
> 00401C39   8B4D F0          mov ecx,dword ptr ss:[ebp-10]
> 00401C3C   64:890D 0000000>mov dword ptr fs:[0],ecx
> 00401C43   5F               pop edi 
> 00401C44   5E               pop esi    
> 00401C45   5B               pop ebx   
> 00401C46   8BE5             mov esp,ebp
> 00401C48   5D               pop ebp    
> 00401C49   C2 1000          retn 10
> ```
>   
>   前两个 `SEH` 正常过, 第三个 `SEH` 看 `401770`
>   
> ```
> 00401770   6A 00            push 0
> 00401772   6A 00            push 0
> 00401774   6A 00            push 0
> 00401776   68 ED200000      push 20ED
> 0040177B   FF15 04A04000    call dword ptr ds:[<&KERNEL32.RaiseException>]
> 00401781   C3               retn
> ```
>   
>   `RaiseException(0x000020ED, 0, 0, 0);`  
>   `shift+F9` 可以正常过, 所以这个不算 `anti`  
>   第三个 `seh` 过了后到 `4019A0`
>   
> ```
> 004019A0   83EC 4C          sub esp,4C
> 004019A3   53               push ebx
> 004019A4   55               push ebp
> 004019A5   56               push esi
> 004019A6   33ED             xor ebp,ebp
> 004019A8   57               push edi
> 004019A9   55               push ebp
> 004019AA   FF15 94A04000    call dword ptr ds:[<&KERNEL32.GetModuleHandleA>]   
> 004019B0   8B3D E4A04000    mov edi,dword ptr ds:[<&USER32.LoadIconA>]         
> 004019B6   8BF0             mov esi,eax
> 004019B8   6A 65            push 65
> 004019BA   56               push esi
> 004019BB   C74424 34 30000>mov dword ptr ss:[esp+34],30
> 004019C3   896C24 38        mov dword ptr ss:[esp+38],ebp
> 004019C7   C74424 3C A0164>mov dword ptr ss:[esp+3C],riijjcm1.004016A0       消息循环入口 (五角星)
> 004019CF   896C24 40        mov dword ptr ss:[esp+40],ebp
> 004019D3   896C24 44        mov dword ptr ss:[esp+44],ebp
> 004019D7   897424 48        mov dword ptr ss:[esp+48],esi
> 004019DB   FFD7             call edi
> 004019DD   68 007F0000      push 7F00
> 004019E2   55               push ebp
> 004019E3   894424 4C        mov dword ptr ss:[esp+4C],eax
> 004019E7   FF15 E8A04000    call dword ptr ds:[<&USER32.LoadCursorA>]         
> 004019ED   6A 65            push 65
> 004019EF   56               push esi
> 004019F0   894424 50        mov dword ptr ss:[esp+50],eax
> 004019F4   C74424 54 10000>mov dword ptr ss:[esp+54],10
> 004019FC   896C24 58        mov dword ptr ss:[esp+58],ebp
> 00401A00   C74424 5C 08A24>mov dword ptr ss:[esp+5C],riijjcm1.0040A208       ; ASCII "myWindowClass"
> 00401A08   FFD7             call edi
> 00401A0A   894424 58        mov dword ptr ss:[esp+58],eax
> 00401A0E   8D4424 2C        lea eax,dword ptr ss:[esp+2C]
> 00401A12   50               push eax
> 00401A13   FF15 ECA04000    call dword ptr ds:[<&USER32.RegisterClassExA>]    ; USER32.RegisterClassExA
> ```
>   
>   所以看 `004016A0`
>   
> ```
> 004016A0   56               push esi
> 004016A1   8B7424 0C        mov esi,dword ptr ss:[esp+C]
> 004016A5   8BC6             mov eax,esi
> ```



```
004016A7    83E8 02         sub eax,2  
004016AA    74 4D          je short riijjcm1.004016F9  
004016AC    83E8 0E        sub eax,0E  
004016AF    74 37          je short riijjcm1.004016E8  
004016B1    A1 E0DD4000    mov eax,dword ptr ds:[40DDE0]  
004016B6    33D2           xor edx,edx  
004016B8    40             inc eax  
004016B9    B9 05000000    mov ecx,5  
004016BE    A3 E0DD4000    mov dword ptr ds:[40DDE0],eax  
004016C3    F7F1           div ecx  
004016C5    85D2           test edx,edx  
004016C7    75 05          jnz short riijjcm1.004016CE  
004016C9    E8 C2FFFFFF    call riijjcm1.00401690          evil call (挂在这里)  
004016CE    8B5424 14      mov edx,dword ptr ss:[esp+14]  
004016D2    8B4424 10      mov eax,dword ptr ss:[esp+10]  
004016D6    8B4C24 08      mov ecx,dword ptr ss:[esp+8]  
004016DA    52             push edx  
004016DB    50             push eax  
004016DC    56             push esi  
004016DD    51             push ecx  
004016DE    FF15 20A14000  call dword ptr ds:[<&USER32.DefWindowProcA>]  
004016E4    5E             pop esi  
004016E5    C2 1000        retn 10  
```

到此只要把 `evil call` 整个 `nop` 掉，`crackme` 就可以正常跑起来了。如果仅仅是这样，看雪不会给精华的。一定要去 `evil call` 里看个究竟。

```
00401690    E8 4BFFFFFF     call riijjcm1.004015E0       解密还原 ZwQueryInformationProcess ntdll.dll 字符串  
00401695    E8 86FFFFFF     call riijjcm1.00401620       anti1 call  
0040169A   ^ E9 61FBFFFF    jmp riijjcm1.00401200        毁灭 ZwQueryInformationProcess ntdll.dll 字符串  
```

答案就在 `anti1 call` 里。

```
00401620    55             push ebp  
00401621    8BEC           mov ebp,esp  
00401623    51             push ecx  
00401624    2BC0           sub eax,eax  
00401626    85C0           test eax,eax  
00401628    75 01          jnz short riijjcm1.0040162B  
0040162A    64:A1 18000000 mov eax,dword ptr fs:[18]  
00401630    8B40 30        mov eax,dword ptr ds:[eax+30]  
00401633    8945 FC        mov dword ptr ss:[ebp-4],eax  
00401636    8B45 FC        mov eax,dword ptr ss:[ebp-4]  
00401639    8A48 02        mov cl,byte ptr ds:[eax+2]  
0040163C    84C9           test cl,cl  
0040163E    74 05          je short riijjcm1.00401645  
00401640    E8 8BFBFFFF    call riijjcm1.004011D0       dead call (挂)  
00401645    E8 96FEFFFF    call riijjcm1.004014E0       anti2 call  
0040164A    8BE5           mov esp,ebp  
0040164C    5D             pop ebp  
0040164D    C3             retn  
```

总结 `anti1 call`：

```
mov eax, fs:[18h]
mov eax, [eax+30h]
mov cl, [eax+2]
```

如果 `cl` 为 `1`，则进 `dead call`；如果 `cl` 为 `0`，则进 `anti2 call`。根据笨笨雄的整理，得知，这个 `anti1` 其实就是 `IsDebuggerPresent()`。

接着 `anti2`：

```
004014E0     55             push ebp  
004014E1     8BEC           mov ebp,esp  
004014E3     83EC 18        sub esp,18  
004014E6     8D45 F8        lea eax,dword ptr ss:[ebp-8]  
004014E9     8D4D FC        lea ecx,dword ptr ss:[ebp-4]  
004014EC     8945 E8        mov dword ptr ss:[ebp-18],eax  
004014EF     894D EC        mov dword ptr ss:[ebp-14],ecx  
004014F2     2BC0           sub eax,eax  
004014F4     85C0           test eax,eax  
004014F6     75 08          jnz short riijjcm1.00401500  
004014F8     2BC0           sub eax,eax  
004014FA     85C0           test eax,eax  
004014FC     75 02          jnz short riijjcm1.00401500  
004014FE     68 A4DD4000    push riijjcm1.0040DDA4          ; ASCII "ntdll.dll"  
00401503     E8 48010000    call riijjcm1.00401650          LoadLibrary("ntdll.dll")  
00401508     8BC8           mov ecx,eax  
0040150A     2BC0           sub eax,eax  
0040150C     85C0           test eax,eax  
0040150E     75 09          jnz short riijjcm1.00401519  
00401510     2BC0           sub eax,eax  
00401512     85C0           test eax,eax  
00401514     75 01          jnz short riijjcm1.00401517  
00401516     68 78DD4000    push riijjcm1.0040DD78          ; ASCII "ZwQueryInformationProcess"  
0040151B     51             push ecx  
0040151C     E8 4F010000    call riijjcm1.00401670          GetProcAddress  
00401521     83C4 0C        add esp,0C  
00401524     8945 F4        mov dword ptr ss:[ebp-C],eax  
00401527     FF15 0CA04000  call dword ptr ds:[<&KERNEL32.GetCurrentProcess>]  
0040152D     8945 F0        mov dword ptr ss:[ebp-10],eax  
00401530     C745 FC 0000000 mov dword ptr ss:[ebp-4],0  
00401537     FF75 E8        push dword ptr ss:[ebp-18]  
0040153A     6A 04          push 4  
0040153C     FF75 EC        push dword ptr ss:[ebp-14]  
0040153F     6A 07          push 7  
00401541     FF75 F0        push dword ptr ss:[ebp-10]  
00401544     FF55 F4        call dword ptr ss:[ebp-C]       call ZwQueryInformationProcess  
00401547     8B45 FC        mov eax,dword ptr ss:[ebp-4]  
0040154A     85C0           test eax,eax  
0040154C     74 05          je short riijjcm1.00401553  
0040154E     E8 7DFCFFFF    call riijjcm1.004011D0          dead call(挂)  
00401553     E8 08FFFFFF    call riijjcm1.00401460          anti3 call  
00401558     8BE5           mov esp,ebp  
0040155A     5D             pop ebp  
0040155B     C3             retn  
```

这个 `anti2` 原来是：

```
ZwQueryInformationProcess(GetCurrentProcess(), ProcessDebugPort, &information, 4, &returnlength);
```

这个也有史料记载的，如果返回的 `information` 中为 `0` 就可以了。

接着 `anti3`：

```
00401460     55             push ebp  
00401461     8BEC           mov ebp,esp  
00401463     83EC 10        sub esp,10  
00401466     8925 A0DD4000  mov dword ptr ds:[40DDA0],esp  
0040146C     892D 9CDD4000  mov dword ptr ds:[40DD9C],ebp  
00401472     2BC9           sub ecx,ecx  
00401474     85C9           test ecx,ecx  
00401476     75 12          jnz short riijjcm1.0040148A  
00401478     2BC0           sub eax,eax  
0040147A     85C0           test eax,eax  
0040147C     75 06          jnz short riijjcm1.00401484  
0040147E     68 A4DD4000    push riijjcm1.0040DDA4          ; ASCII "ntdll.dll"  
00401483     E8 C8010000    call riijjcm1.00401650  
00401488     8BC8           mov ecx,eax  
0040148A     2BC0           sub eax,eax  
0040148C     85C0           test eax,eax  
0040148E     75 09          jnz short riijjcm1.00401499  
00401490     2BC0           sub eax,eax  
00401492     85C0           test eax,eax  
00401494     75 01          jnz short riijjcm1.00401497  
00401496     68 78DD4000    push riijjcm1.0040DD78          ; ASCII "ZwQueryInformationProcess"  
0040149B     51             push ecx  
0040149C     E8 CF010000    call riijjcm1.00401670  
004014A1     83C4 0C        add esp,0C  
004014A4     8945 F8        mov dword ptr ss:[ebp-8],eax  
004014A7     FF15 08A04000  call dword ptr ds:[<&KERNEL32.GetCurrentThread>]   
004014AD     8945 F4        mov dword ptr ss:[ebp-C],eax  
004014B0     8D45 FC        lea eax,dword ptr ss:[ebp-4]  
004014B3     C745 FC 0000000 mov dword ptr ss:[ebp-4],0  
004014BA     8945 F0        mov dword ptr ss:[ebp-10],eax  
004014BD     6A 04          push 4  
004014BF     FF75 F0        push dword ptr ss:[ebp-10]  
004014C2     6A 11          push 11  
004014C4     FF75 F4        push dword ptr ss:[ebp-C]      



## 汇编代码分析

```
004014C7     FF55 F8         call dword ptr ss:[ebp-8]                     call ZwQueryInformationProcess  
004014CA     E8 A1FDFFFF    call riijjcm1.00401270                        anti4 call  
004014CF     8B25 A0DD4000  mov esp,dword ptr ds:[40DDA0]  
004014D5     8B2D 9CDD4000  mov ebp,dword ptr ds:[40DD9C]  
004014DB     8BE5           mov esp,ebp  
004014DD     5D             pop ebp  
004014DE     C3             retn  
```

这个 `anti3` 是这样的：

```
ZwQueryInformationProcess(GetCurrentThread(), ProcessEnableAlignmentFaultFixup, &tmp, 4, &tmp);
```

这个地方我怀疑 `riijj` 姐姐用错了。

是不是应该：

```
ZwSetInformationThread(GetCurrentThread(), ThreadHideFromDebugger, 0, 0);
```

呢?

先不管它了, 看最后一个 `anti4`。

## anti4 反汇编

```
00401270     55             push ebp  
00401271     8BEC           mov ebp,esp  
00401273     6A FF          push -1  
00401275     68 50A14000    push riijjcm1.0040A150  
0040127A     68 E81F4000    push riijjcm1.00401FE8  
0040127F     64:A1 00000000 mov eax,dword ptr fs:[0]  
00401285     50             push eax  
00401286     64:8925 0000000> mov dword ptr fs:[0],esp  
0040128D     83EC 08        sub esp,8  
00401290     53             push ebx  
00401291     56             push esi  
00401292     57             push edi  
00401293     8965 E8        mov dword ptr ss:[ebp-18],esp  
00401296     C745 FC 0000000> mov dword ptr ss:[ebp-4],0           __try{RaiseException(0x40010006, 0, 0, 0)}  
0040129D     2BC9           sub ecx,ecx  
0040129F     85C9           test ecx,ecx  
004012A1     75 13          jnz short riijjcm1.004012B6  
004012A3     85C9           test ecx,ecx  
004012A5     75 08          jnz short riijjcm1.004012AF  
004012A7     6A 00          push 0  
004012A9     6A 00          push 0  
004012AB     6A 00          push 0  
004012AD     68 06000140    push 40010006  
004012B2     FF15 04A04000  call dword ptr ds:[<&KERNEL32.RaiseException>]  
004012B8     E8 13FFFFFF    call riijjcm1.004011D0                   dead call (挂)  
004012BD     EB 09          jmp short riijjcm1.004012C8  
004012BF     B8 01000000    mov eax,1                                  __except(1)  
004012C4     C3             retn  
004012C5     8B65 E8        mov esp,dword ptr ss:[ebp-18]            {}  
004012C8     C745 FC FFFFFFF> mov dword ptr ss:[ebp-4],-1  
004012CF     8B4D F0        mov ecx,dword ptr ss:[ebp-10]  
004012D2     64:890D 0000000> mov dword ptr fs:[0],ecx  
004012D9     5F             pop edi  
004012DA     5E             pop esi  
004012DB     5B             pop ebx  
004012DC     8BE5           mov esp,ebp  
004012DE     5D             pop ebp  
004012DF     C3             retn  
```

到了 `anti4` 已明白, 4 个 `anti` 中, 前 3 个用看雪的 `HideOD` 插件都可以过。

现在跑不起来,显然是 `anti4` 在作怪, 所以, 只要把 `anti4` 写清楚, 就有精华了.

`abc123`:

```
__try
{
    RaiseException(0x40010006, 0, 0, 0);
    dead();
}
__except (1)
{
}
```

这个到底怎么了呢? 我们来整理一下

a. 没有被 `OllyICE` 调试的时候, 先 `RaiseException(0x40010006, 0, 0, 0);` 然后发生异常, 接着跑到 `__except(1)`中, 没有吃到 `dead();` 没挂
b. 被 `OllyICE` 调试的时候, 先 `RaiseException(0x40010006, 0, 0, 0);` 然后异常被 `OllyICE` 处理了, 接着到了 `dead()`, 所以挂了.
(不明白的话 `a`,`b` 两种情况多读几遍)

为了增加悬念, 我们再回头看 `WinMain` 中的第三个 `SEH`

```
void 401770()
{
    RaiseException(0x000020ED, 0, 0, 0);
}
__try
{
    call 401770
}
__except (1)
{
}
```

经过实验, 这一处 `SEH` 中, 无论是否被 `OllyICE` 调试, 运行的流程都是 `RaiseException` 后进入`__except (1)`中, 这就非常神奇了

也就是说在被 `OllyICE` 调试的时候
如果我们 `RaiseException(0x40010006, 0, 0, 0);` 不会进入自身的异常处理
如果我们 `RaiseException(0x000020ED, 0, 0, 0);` 会进入自身的异常处理

为虾米呢?

难道 `0x40010006` 和 `0x000020ED` 有一个是特殊的么?

所以我们需要 `google`
先查 `40010006`
第一条结果是
谷氨酰胺价格-谷氨酰胺图片商品编号：40010006. 【市场价】￥185 元【爱我价】￥155 元

所以我们要查 `0x40010006`
第一条结果是
绿盟科技--www.nsfocus.com--绿盟月刊 `OutputDebugStringA` 函数(`kernel32.dll`)
实际上使用 `RaiseException` 函数引发了一个异常号为 `0x40010006` 的软件异常

谜底解开了

当被调试时
`RaiseException(0x40010006, 0, 0, 0);` 实质是 `OutputDebugStringA`
也就是说

当 `OllyICE` 收到了 `0x40010006` 的异常号, 处理了异常, 并告诉被调试程序这个异常我已经处理了, 因此程序本身收到的信号是异常已被处理, 没有进入程序本身的异常处理

当 `OllyICE` 收到了 `0x000020ED` 的异常号, 没有处理异常, 并告诉被调试程序这个异常我没有处理, 你自己搞定, 于是程序本身会进入自身的异常处理

问题到里已经明白了, 但为了精华贴更有价值, 再多说一些

现在的问题是
`RaiseException(0x40010006, 0, 0, 0);`
和
`OutputDebugStringA` 有区别么?

我们来看看 `2000` 下的 `OutputDebugStringA`

```
77E6F7AD K>   55             push ebp  
77E6F7AE       8BEC           mov ebp,esp  
77E6F7B0       6A FF          push -1  
77E6F7B2       68 581CE677    push KERNEL32.77E61C58  
77E6F7B7       68 6C21EB77    push KERNEL32.77EB216C  
77E6F7BC       64:A1 00000000  mov eax,dword ptr fs:[0]  
77E6F7C2       50             push eax  
77E6F7C3       64:8925 0000000> mov dword ptr fs:[0],esp             装入 SEH  
77E6F7CA       51             push ecx  
77E6F7CB       51             push ecx  
77E6F7CC       81EC 28020000  sub esp,228  
77E6F7D2       53             push ebx  
77E6F7D3       56             push esi  
77E6F7D4       57             push edi  
77E6F7D5       8965 E8        mov dword ptr ss:[ebp-18],esp  
77E6F7D8       8365 FC 00     and dword ptr ss:[ebp-4],0             打开 SEH  
77E6F7DC       8B55 08        mov edx,dword ptr ss:[ebp+8]         
77E6F7DF       8BFA           mov edi,edx        
77E6F7E1       83C9 FF        or ecx,FFFFFFFF  
77E6F7E4       33C0           xor eax,eax  
77E6F7E6       F2:AE          repne scas byte ptr es:[edi]  
77E6F7E8       F7D1           not ecx  
77E6F7EA       894D E0        mov dword ptr ss:[ebp-20],ecx  
77E6F7ED       8955 E4        mov dword ptr ss:[ebp-1C],edx  
77E6F7F0       8D45 E0        lea eax,dword ptr ss:[ebp-20]  
77E6F7F3       50             push eax  
77E6F7F4       6A 02          push 2  
77E6F7F6       6A 00          push 0  
77E6F7F8       68 06000140    push 40010006  
77E6F7FD       E8 29C40100    call KERNEL32.RaiseException          RaiseException(0x40010006, 0, ...);  
77E6F802       E9 1A020000    jmp KERNEL32.77E6FA21                 如果被调试, 从这行出来  
77E6F807       6A 01          push 1                                 __except(1) 如果没有被调试, 走自身的异常处理到这里  
77E6F809       58             pop eax        
77E6F80A       C3             retn  
77E6F80B       8B65 E8        mov esp,dword ptr ss:[ebp-18]         {  
```

区别是明显的, 就是 `OutputDebugStringA` 多包了一层异常处理
正因为有了这一层异常处理, 使得在没有被调试的情况下, 执行 `OutputDebugStringA` 函数不会挂掉, 为了验证这一点
我写了个小程序验证, `2000` 下的硬编码, `xp` 的要改一下地址

```c
#include <windows.h>

DWORD add = 0x77E6F80B;  //xp 下相应地址改一下

void test()
{ 
  MessageBox(0, "1", "1", 0);
}

void __declspec(naked) hook()
{ 
  __asm
  {
    pushad
    call test
    popad
    mov esp,dword ptr ss:[ebp-0x18]   
    xor ebx, ebx                   //xp 下这行改 xor edi ,edi
    push dword ptr [add]
    add dword ptr [esp], 5
    retn
  }
}

void main()
{ 
  DWORD tmp;

  VirtualProtect((LPVOID)add, 5, PAGE_EXECUTE_READWRITE, &tmp);
  *(LPBYTE)add = 0xE9;
  *(LPDWORD)(add+1) = (DWORD)hook - add - 5;
  OutputDebugString("test");
}
```



> 这个程序, 在`OllyICE`下, 不会弹`MessageBox`, 直接运行 , 会弹`MessageBox`
> 
> 原理就是前面说的有`OllyICE`时,`OutputDebugStringA`不用处理自身异常
> 
> 没有`OllyICE`时, 需要处理自身异常
> 
> 现在`riijj`姐姐直接调`RaiseException(0x40010006, 0, 0, 0);`就相当于直接
> 
> 调用了没有包异常处理的`OutputDebugStringA`, 所以想要让它走的正确路, 只
> 
> 有`OllyICE`不处理这个异常, 直接还给程序本身.
> 
> 如果这里看不明白`goto abc123;`
> 
> 下面来说说最重要的部分: 解决方案
> 1. 找`kanxue`让他升级`HideOD`插件

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 01 月 17 日 16:30:0

`4dd466d0e1e0cc123afbe504da2462ca`
