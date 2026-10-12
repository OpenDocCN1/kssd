
## `WARNING:` `Stack` `unwind` `information` `not` `available.` `Following` `frames` `may` `be` `wrong.`

```
0012c5c8 00454059 00120106 0012c5e0 004bb410 USER32!MessageBoxA  

0012cbe4 0045dce7 004bc1ea 063f0050 005bd670 image00400000!Error+0x3d  

0012d2d0 0045f7eb 063f0000 76bc1f1c 76bc2075 image00400000!Createlistwindow+0x2af7  

00000000 00000000 00000000 00000000 00000000 image00400000!Decoderange+0x180b  
```

在反汇编窗口输入 `00454059`

```
0045401c 55                push    ebp  
0045401d 8bec              mov     ebp,esp  
0045401f 81c4fcf9ffff      add     esp,0FFFFF9FCh  
00454025 8d450c            lea     eax,[ebp+0Ch]  
00454028 50                push    eax  
00454029 8b5508            mov     edx,dword ptr [ebp+8]  
0045402c 52                push    edx  
0045402d 8d8dfcf9ffff      lea     ecx,[ebp-604h]  
00454033 51                push    ecx  
00454034 e81b2c0500        call    image00400000!Createpatchwindow+0x79f4 (004a6c54)  
00454039 83c40c            add     esp,0Ch  
0045403c 8d85fcf9ffff      lea     eax,[ebp-604h]  
00454042 8b157c3b4d00      mov     edx,dword ptr [image00400000!__CPPdebugHook+0x23a54 (004d3b7c)]  
00454048 6810200000        push    2010h  
0045404d 6810b44b00        push    offset image00400000!__CPPdebugHook+0xb2e8 (004bb410)  
00454052 50                push    eax  
00454053 52                push    edx  
00454054 e8bdb40500        call    image00400000!Createpatchwindow+0x102b6 (004af516)  
00454059 6a00              push    0   ；MESSAGEBOX 的返回地址  
0045405b e878dafdff        call    image00400000!Animate (00431ad8)  
00454060 59                pop     ecx  
00454061 8be5              mov     esp,ebp  
00454063 5d                pop     ebp  
00454064 c3                ret  
```

在 `00454064` 下断，返回 `OD` 按确定。

```
0045dcd1 eb30              jmp     image00400000!Createlistwindow+0x2b13 (0045dd03)  
0045dcd3 837df402          cmp     dword ptr [ebp-0Ch],2  
0045dcd7 7513              jne     image00400000!Createlistwindow+0x2afc (0045dcec)  
0045dcd9 8d4e50            lea     ecx,[esi+50h]  
0045dcdc 51                push    ecx  
0045dcdd 68eac14b00        push    offset image00400000!__CPPdebugHook+0xc0c2 (004bc1ea)  
0045dce2 e83563ffff        call    image00400000!Error (0045401c)  
0045dce7 83c408            add     esp,8   ；返回到这里  
0045dcea eb17              jmp     image00400000!Createlistwindow+0x2b13 (0045dd03)  
```

这里比较 `EBP-C` 是否为 `2`，是则弹出错误窗口。

```
0:000> ? ebp-0c  
Evaluate expression: 1233604 = 0012d2c4  
```

重新载入 `OD`，下条件断点

```
0:000> ba w4 0012d2c4 "r @$t0=poi(0012d2c4);j(@$t0!=2)'g'"  
```

`OD` 载入 `MSLRHv0.32a`。

```
0045c651 7527              jne     image00400000!Createlistwindow+0x148a (0045c67a)  
0045c653 0fb79534faffff    movzx   edx,word ptr [ebp-5CCh]  
0045c65a 81c280000000      add     edx,80h  
0045c660 8b8da0f9ffff      mov     ecx,dword ptr [ebp-660h]  
0045c666 c1e103            shl     ecx,3  
0045c669 81c1e0000000      add     ecx,0E0h  
0045c66f 3bd1              cmp     edx,ecx  
0045c671 7407              je      image00400000!Createlistwindow+0x148a (0045c67a)  
0045c673 c745f402000000    mov     dword ptr [ebp-0Ch],2  
0045c67a 837df400          cmp     dword ptr [ebp-0Ch],0 ss:0023:0012d2c4=00000002  ；断在这里  
0045c67e 7524              jne     image00400000!Createlistwindow+0x14b4 (0045c6a4)  
```

这里对比 `ebp-5CC` 和 `ebp-660`。一定条件 `ebp-0Ch=2`。

```
0:000> db ebp-660  
0012cc70  ff 00 00 00 00 00 00 00-00 00 00 00 f8 19 05 00  ................  
```

重新载入 `OD`，下条件断点

```
0:000> ba w4 0012cc70 "r @$t0=poi(0012cc70);j(@$t0!=ff)'g'"  
```

`OD` 载入 `MSLRHv0.32a`。

```
eax=0012cc14 ebx=004caf4c ecx=00000020 edx=000000e0 esi=01b028a8 edi=0012cc74  
eip=004a3547 esp=0012cb90 ebp=0012cb98 iopl=0         nv up ei pl nz ac po nc  
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00200212  

image00400000!Createpatchwindow+0x42e7:  
004a3547 f3a5              rep movs dword ptr es:[edi],dword ptr [esi] es:0023:0012cc74=00160106 ds:0023:01b028a8=00000000  
```

在内存窗口中输入 `ESI`

```
01b02908 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ........................  
01b02920 00 00 00 00 00 00 00 00 2e 65 64 61 74 61 00 00 00 10 04 00 00 10 00 00  .edata..........  
01b02938 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 00 00 e0  ........................  
01b02950 2e 65 64 61 74 61 00 00 00 d0 00 00 00 20 04 00 00 c8 00 00 00 04 00 00  .edata....... ..........  
01b02968 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 c0 00 00 00 00 03 00 00 00  ..............@...........  
01b02980 00 30 00 00 00 f0 04 00 00 2c 00 00 00 cc 00 00 00 00 00 00 00 00 00 00  .0.......,..............  
01b02998 00 00 00 00 40 00 00 c0 00 00 00 00 00 00 00 00 00 50 00 00 00 20 05 00  ....@............P... ..  
01b029b0 00 50 00 00 00 f8 00 00 00 00 00 00 00 00 00 00 00 00 00 00 20 00 00 e0  .P.................. ...  
01b029c8 2e 74 65 78 74 00 00 00 00 10 00 00 00 70 05 00 00 00 00 00 00 48 01 00  .text........p.......H..  
```

象 `PE` 头。向上移看看，果然

```
01b02830 50 45 00 00 4c 01 05 00 fe df 9c 42 00 00 00 00 00 00 00 00 e0 00 0f 01  PE..L......B............  
01b02848 0b 01 05 0c 00 30 01 00 00 90 00 00 00 10 04 00 00 20 05 00 00 20 05 00  .....0........... ... ..  
01b02860 00 f0 04 00 00 00 40 00 00 10 00 00 00 02 00 00 04 00 00 00 04 00 00 00  ......@.................  
01b02878 04 00 00 00 00 00 00 00 00 80 05 00 00 10 00 00 00 00 00 00 02 00 00 00  ........................  
01b02890 00 00 10 00 00 10 00 00 00 00 10 00 00 10 00 00 00 00 00 00 ff 00 00 00  ........................  
01b028a8 00 00 00 00 00 00 00 00 f8 19 05 00 88 01 00 00 00 f0 04 00 f8 29 00 00  .....................)..  
```

既然是 `PE`，用 `LORDPE` 打开 `MSLRHv0.32a`。

可选头部大小 `00e0`

`RVA` 数及大小 `FF`

点 `RVA` 数及大小旁边的减号

可选头部大小 `0850`

`RVA` 数及大小 `FE`

再点加

可选头部大小 `0858`

`RVA` 数及大小 `FF`

这两项是相关联的，可见 `OD` 中 `EBP-660` 和 `EBP-5CC` 就是这两项了，校验不正确就报错。程序在系统中能正确运行。`RVA` 数及大小错，只影响 `OD` 加载。

```
0045c671 7407              je      image00400000!Createlistwindow+0x148a (0045c67a)  
```

为了以后对该 `ANTI` 免疫，将上面的 `JE` 改成 `JMP`。

现在可以用 `OD` 分析了。该壳跟上一版差不多，又是 `RDTSC` 海，不过限制宽了，从 `FF` 变成 `FFFF`，最多只能防单步，有等于没有。同样是插入宏的方式加花，方向向下，看到有意义的地方直接下断 `F9` 过去就好了。相同的地方就不说了，说说不同的地方，不过不代表不需要处理 `CreateFile`。

```
00453118    64:FF35 0000000>push dword ptr fs:[0]  
0045311F    64:8925 0000000>mov dword ptr fs:[0],esp  
00453126    33C0              xor eax,eax  
00453128    CD 01             int 1  
0045312A    40                inc eax  
0045312B    40                inc eax  
```

此处在 `INT 1` 后下断就可以了。然后是用 `CREATEMUTEX` 建立互斥对象，存在则继续，不存在则新建一个进程，自己退出。修改 `EIP` 直接跳到后面就可以了。再来是 `FindWindow`，也查 `IMPORTRECT` 窗口，最好也自己动手把窗口标题改了。

`SetUnhandledExceptionFilter` 用得有点诡异。

```
00455216    8B46 38          mov eax,dword ptr ds:[esi+38]  ；[esi+38]=SetUnhandledExceptionFilter  
00455219    8B40 0B          mov eax,dword ptr ds:[eax+B]  
0045521C    8908             mov dword ptr ds:[eax],ecx      ；ECX 是程序自定义的 SEH  
```

我们来看看为什么可以这么用

```
77E678A7 >/$ 8B4C24 04     mov ecx,dword ptr ss:[esp+4]  
77E678AB  .|. A1 4C04EC77   mov eax,dword ptr ds:[77EC044C]  
77E678B0  .|. 890D 4C04EC77 mov dword ptr ds:[77EC044C],ecx  
77E678B6  \\. C2 0400       retn 4  
```

`SetUnhandledExceptionFilter` 入口点+B 的地方刚好是 `4C04EC77`。当然也可以+5。

```
0045549A    33DB              xor ebx,ebx  
0045549C    F7F3              div ebx  
```

除 0 错误，进入 `SEH` 例程,之后便来到最后的自校验了。

该版本跟前一个版本一样，将一些 `DEBUG` 的校验放到最后,校验值的存放位置 `45700c` 到 `45700F`，处理 `CREATEMUTEX` 和 `CreateFile` 后，直接在该处下内存断点就能到最后的自校验了，仍然是 `UPX` 内核。

`4dd466d0e1e0cc123afbe504da2462ca`
