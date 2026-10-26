
# [原创]一个小软件的算法分析（附爆破方法，内存注册机地址，算法注册机源码）

软件名称：幼儿园信息管理系统V3.0

下载地址：f53K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4W2F1P5r3u0K6i4K6u0W2j5$3!0E0i4K6u0r3h3f1g2Q4x3V1k6V1k6h3k6S2N6h3I4@1i4K6u0W2K9s2c8E0

软件编写：DELPHI

壳：没有

没有什么目的，只为学习。

此软件，爆破起来N简单的（有注明）。

作内存注册机也很简单（有注明）。。

作算法注册机也简单。

本文只适合于新手。菜鸟。高手可以飘过。。。。。。。。。。。。。。

不废话了，开始。

查下没壳，OK！继续。

试试运行软件，系统维护 -> 软件注册 发现软件默认是注册成功，可是名称是 满洲里市冬鸽少儿艺术学校 。我们要换其它的名字。

基础数据设置 -> 班级设置 -> 点击：那个提示为修改 的按钮，输入你要注册的名称：此处我输入：`Jason`

然后 -> 系统维护 -> 软件注册 发现提示要注册了，点一下 确定 按钮发现注册不成功了，这时随便输入一个数值。

然后用 `W32asm` 反汇编一下：查找字符：注册 ，很容易找到 注册码不正确......。

然后我们双击该字符串就到 `00699628` 这里，我们再往上看看。 这个 `006995D8` 地址很重要。

这个我打开 `OD` 加载 `main.exe` 程序。

然后我们就在工具栏点： `L` 按钮左边的那个按钮（转到汇编地址），然后输入 `006995D8` 回车。

转到此地址后，我们就按一下 `F2`（设置断点）

然后按`F9`运，，，一直到注册那个，，点击 确定，就被断在`006995D8` 这里了：

```
006995D8  /.  55            push    ebp
006995D9  |.  8BEC          mov     ebp, esp
006995DB  |.  33C9          xor     ecx, ecx
006995DD  |.  51            push    ecx
006995DE  |.  51            push    ecx
006995DF  |.  51            push    ecx
006995E0  |.  51            push    ecx
006995E1  |.  53            push    ebx
006995E2  |.  8BD8          mov     ebx, eax
006995E4  |.  33C0          xor     eax, eax
006995E6  |.  55            push    ebp
006995E7  |.  68 C7966900   push    006996C7
006995EC  |.  64:FF30       push    dword ptr fs:[eax]
006995EF  |.  64:8920       mov     dword ptr fs:[eax], esp
006995F2  |.  8D55 F8       lea     edx, dword ptr [ebp-8]
006995F5  |.  8B83 F4020000 mov     eax, dword ptr [ebx+2F4]
006995FB  |.  E8 848DDBFF   call    00452384                         ;  获取用户名
00699600  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
00699603  |.  8D55 FC       lea     edx, dword ptr [ebp-4]
00699606  |.  E8 B966E7FF   call    0050FCC4                         ;  计算得到真正的注册码,F7进去。
0069960B  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0069960E  |.  50            push    eax
0069960F  |.  8D55 F4       lea     edx, dword ptr [ebp-C]           ; 此处可以作内存注册机，只要读出 EAX的值就可以。
00699612  |.  8B83 FC020000 mov     eax, dword ptr [ebx+2FC]
00699618  |.  E8 678DDBFF   call    00452384
0069961D  |.  8B55 F4       mov     edx, dword ptr [ebp-C]
00699620  |.  58            pop     eax
00699621  |.  E8 7ABDD6FF   call    004053A0
00699626  |.  74 1A         je      short 00699642   ;关键跳，可以将JE改成jmp（74改成EB）达到爆破的目的。
```

此处我们不说爆破，我们要作的是得到注册算法：

刚才 `F7` 跟进后：

```
0050FCC4  /$  55            push    ebp
0050FCC5  |.  8BEC          mov     ebp, esp
0050FCC7  |.  81C4 F8FEFFFF add     esp, -108
0050FCCD  |.  53            push    ebx
0050FCCE  |.  56            push    esi
0050FCCF  |.  57            push    edi
0050FCD0  |.  8955 F8       mov     dword ptr [ebp-8], edx
0050FCD3  |.  8945 FC       mov     dword ptr [ebp-4], eax
0050FCD6  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0050FCD9  |.  E8 6657EFFF   call    00405444
0050FCDE  |.  33C0          xor     eax, eax
0050FCE0  |.  55            push    ebp
0050FCE1  |.  68 4BFD5000   push    0050FD4B
0050FCE6  |.  64:FF30       push    dword ptr fs:[eax]
0050FCE9  |.  64:8920       mov     dword ptr fs:[eax], esp
0050FCEC  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0050FCEF  |.  E8 6057EFFF   call    00405454
0050FCF4  |.  8BD0          mov     edx, eax
0050FCF6  |.  8D85 F8FEFFFF lea     eax, dword ptr [ebp-108]
0050FCFC  |.  E8 43BBEFFF   call    0040B844
0050FD01  |.  33F6          xor     esi, esi
0050FD03  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0050FD06  |.  E8 5155EFFF   call    0040525C
0050FD0B  |.  8BD0          mov     edx, eax
0050FD0D  |.  85D2          test    edx, edx
0050FD0F  |.  7C 1A         jl      short 0050FD2B
0050FD11  |.  42            inc     edx
0050FD12  |.  33C9          xor     ecx, ecx
0050FD14  |.  8D85 F8FEFFFF lea     eax, dword ptr [ebp-108]
0050FD1A  |>  33DB          /xor     ebx, ebx
0050FD1C  |.  8A18          |mov     bl, byte ptr [eax]              ;  取字母送入BL中
0050FD1E  |.  8D79 01       |lea     edi, dword ptr [ecx+1]          ;  字符位置
0050FD21  |.  0FAFDF        |imul    ebx, edi                        ;  EBX = EBX * EDI
0050FD24  |.  03F3          |add     esi, ebx                        ;  ESI = ESI + EBX
0050FD26  |.  41            |inc     ecx
0050FD27  |.  40            |inc     eax
0050FD28  |.  4A            |dec     edx
0050FD29  |.^ 75 EF         \jnz     short 0050FD1A
0050FD2B  |>  8B55 F8       mov     edx, dword ptr [ebp-8]
0050FD2E  |.  8BC6          mov     eax, esi
0050FD30  |.  E8 37A9EFFF   call    0040A66C                         ;这里要 F7 跟进去
0050FD35  |.  33C0          xor     eax, eax
0050FD37  |.  5A            pop     edx
0050FD38  |.  59            pop     ecx
0050FD39  |.  59            pop     ecx
0050FD3A  |.  64:8910       mov     dword ptr fs:[eax], edx
0050FD3D  |.  68 52FD5000   push    0050FD52
0050FD42  |>  8D45 FC       lea     eax, dword ptr [ebp-4]
0050FD45  |.  E8 4A52EFFF   call    00404F94
0050FD4A  \.  C3            retn
```

以上从地址：`0050FD1A` 至地址：`0050FD29` 的算法(DELPHI)

```
var
  I,ebx,esi: Integer;
  s:string;
begin
  ebx:=0; esi:=0;
  s:= 'Jason';   //我刚才输入的注册名，还记得吧。
  for I := 1 to Length(s) do begin
    ebx:=Ord(s[i]);
    ebx :=ebx * i;
    esi:=esi + ebx;
  end;
  s :=IntTohex(esi,8);     //这里的结束是：00000647
end;
```

还没有完呢，上面的 `F7` 继续跟进:

```
0040A66C  /$  83C4 F8       add     esp, -8
0040A66F  |.  6A 00         push    0                                ; /Arg1 = 00000000
0040A671  |.  894424 04     mov     dword ptr [esp+4], eax           ; |
0040A675  |.  C64424 08 00  mov     byte ptr [esp+8], 0              ; |
0040A67A  |.  8D4C24 04     lea     ecx, dword ptr [esp+4]           ; |
0040A67E  |.  8BC2          mov     eax, edx                         ; |
0040A680  |.  BA 98A64000   mov     edx, 0040A698                    ; |ASCII "%d"
0040A685  |.  E8 F6190000   call    0040C080                         ; \MAIN.0040C080   ;继续F7
0040A68A  |.  59            pop     ecx
0040A68B  |.  5A            pop     edx
0040A68C  \.  C3            retn
```

看注释 继续F7:

```
0040C080  /$  55            push    ebp
0040C081  |.  8BEC          mov     ebp, esp
0040C083  |.  81C4 04F0FFFF add     esp, -0FFC
0040C089  |.  50            push    eax
0040C08A  |.  83C4 F8       add     esp, -8
0040C08D  |.  53            push    ebx
0040C08E  |.  56            push    esi
0040C08F  |.  894D F8       mov     dword ptr [ebp-8], ecx
0040C092  |.  8955 FC       mov     dword ptr [ebp-4], edx
0040C095  |.  8BF0          mov     esi, eax
0040C097  |.  BB 00100000   mov     ebx, 1000
0040C09C  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0040C09F  |.  E8 B891FFFF   call    0040525C
0040C0A4  |.  3D 000C0000   cmp     eax, 0C00
0040C0A9  |.  7D 26         jge     short 0040C0D1
0040C0AB  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0040C0AE  |.  E8 A991FFFF   call    0040525C
0040C0B3  |.  50            push    eax
0040C0B4  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
0040C0B7  |.  50            push    eax
0040C0B8  |.  8B45 08       mov     eax, dword ptr [ebp+8]
0040C0BB  |.  50            push    eax
0040C0BC  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
0040C0BF  |.  8D85 F8EFFFFF lea     eax, dword ptr [ebp-1008]
0040C0C5  |.  BA FF0F0000   mov     edx, 0FFF
0040C0CA  |.  E8 09FBFFFF   call    0040BBD8                       ;F7
0040C0CF  |.  EB 0C         jmp     short 0040C0DD
```

下面F7：

```
0040BBD8   $  55            push    ebp
0040BBD9   .  8BEC          mov     ebp, esp
0040BBDB   .  83C4 8C       add     esp, -74
0040BBDE   .  53            push    ebx
0040BBDF   .  33DB          xor     ebx, ebx
0040BBE1   .  895D EC       mov     dword ptr [ebp-14], ebx
0040BBE4   .  53            push    ebx
0040BBE5   .  56            push    esi
0040BBE6   .  57            push    edi
0040BBE7   .  89C7          mov     edi, eax
```
