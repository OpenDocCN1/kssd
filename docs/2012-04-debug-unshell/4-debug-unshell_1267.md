
# crackme 大赛第十回之 crackme 脱壳

> 原文：[`www.pediy.com/kssd/pediy11/98036.html`](https://www.pediy.com/kssd/pediy11/98036.html)

crackme 大赛第十回之 crackme 脱壳

在 `unpack` 上看见海风发了个这个 `unpackme`，其实是 crackme 大赛第十回之 crackme，遂下来研究了一下，与大家分享一下心得。

程序 `ep`：

```
0047A034  <>  60                 pushad
0047A035     9C                 pushfd
0047A036     E8 8E000000        call CrackMe.0047A0C9               ; 来到这里 esp 定律 命令行下 hr esp
0047A03B     C3                 retn
```

`F9` 运行来到这里：

```
0037087A     E8 980D0000        call 00371617                        ; f7 走一下
0037087F     51                 push ecx
00370880     E9 600C0000        jmp 003714E5
00371617     8D6424 04          lea esp,dword ptr ss:[esp+4]          ; 这里
0037161B     61                 popad
0037161C   ^ E9 E7F9FFFF        jmp 00371008                          ; 这里继续 F7
00371008   - FFE0               jmp eax                               ; 跳向 oep
```

`OEP`:

```
0047148B     55                 push ebp
0047148C     8BEC               mov ebp,esp
0047148E     6A FF              push -1
00471490     68 28514700        push CrackMe.00475128
00471495     68 58214700        push CrackMe.00472158
0047149A     64:A1 00000000     mov eax,dword ptr fs:[0]
004714A0     50                 push eax
004714A1     64:8925 00000000   mov dword ptr fs:[0],esp
004714A8     83EC 58            sub esp,58
004714AB     53                 push ebx
004714AC     56                 push esi
004714AD     57                 push edi
004714AE     8965 E8            mov dword ptr ss:[ebp-18],esp
004714B1     FF15 80504700      call dword ptr ds:[475080]
004714B7     33D2               xor edx,edx
004714B9     8AD4               mov dl,ah
004714BB     8915 C4854700      mov dword ptr ds:[4785C4],edx
004714C1     8BC8               mov ecx,eax
004714C3     81E1 FF000000      and ecx,0FF
004714C9     890D C0854700      mov dword ptr ds:[4785C0],ecx
004714CF     C1E1 08            shl ecx,8
004714D2     03CA               add ecx,edx
004714D4     890D BC854700      mov dword ptr ds:[4785BC],ecx
004714DA     C1E8 10            shr eax,10
004714DD     A3 B8854700        mov dword ptr ds:[4785B8],eax
004714E2     33F6               xor esi,esi
004714E4     56                 push esi
004714E5     E8 160B0000        call CrackMe.00472000
004714EA     59                 pop ecx
004714EB     85C0               test eax,eax
004714ED     75 08              jnz short CrackMe.004714F7
```

可以看到 `IAT` 加密了，数据窗口 `ctrl+g` 来到 `00475080`：

```
00475000  00552264
00475004  005521A0
00475008  00552137  ASCII "h
"
0047500C  00552328
。。。。。。。。
00475074  00554BF5  
00475078  00554C1D
0047507C  00556690
00475080  00555039 ********
00475084  00554607
00475088  00554986
0047508C  00552ECB
00475090  00552369
```

到 `552264` 去新建 `eip`，看看它怎么解密的。多次跟踪发现：

```
005522DF     8B3F               mov edi,dword ptr ds:[edi]           ; GDI32.SetTextColor
```

当上面指令执行完之后 `edi` 就是函数地址。

我写了一段脚本来完成正确 `iat` 的填充工作，脚本水平很烂，呵呵.

`esp` 定律到达 `oep` 之后就可以运行脚本了

```
var thunk  
var oep  
var address  
var temp  

mov oep,eip  

mov thunk,00475000 ;iat 起始地址  

label1:  

cmp [thunk],0    ;处理下一个 dll  

jne next  

add thunk,4  

cmp [thunk],0    ;判断是否处理结束  

je stop  

next:  

mov address,[thunk]  

mov eip,address    ;新建 eip  

find eip,#8b3f#          ;查找关键位置  

bp $RESULT          

esto  

bc $RESULT  

sti                  ;走一下     

mov [thunk],edi        ;正确地址写入  

add thunk,4  

jmp label1  

stop:  

mov eip,oep  

ret  
```

脚本运行完成之后就可以转存，修复了。
