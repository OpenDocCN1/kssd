
`004063F7`   `|.`  `33D2`              `|xor edx,edx`                      `\*\*\*\*\*\*EDX`清零

`004063F9`   `|.`  `F775 10`           `|div dword ptr ss:[ebp+10]`        `\*\*\*\*\*\*div   dword ptr ss:[ebp+10]=24  77 div 24之后，商EAX=3 余数EDX=B`

`004063FC`   `|.`  `8BC6`              `|mov eax,esi`                      `\*\*\*\*\*\*EAX=ESI=77`

`004063FE`   `|.`  `8BDA`              `|mov ebx,edx`                      `\*\*\*\*\*\*EBX=EDX=B`

`00406400`   `|.`  `33D2`              `|xor edx,edx`                      `\*\*\*\*\*\*EDX`依然清零

`00406402`   `|.`  `F775 10`           `|div dword ptr ss:[ebp+10]`        `\*\*\*\*\*再Div一次`

`00406405`   `|.`  `83FB 09`           `|cmp ebx,9`                        余数与9相比较

`00406408`   `|.`  `8BF0`              `|mov esi,eax`                      `\*\*\*\*\*商存于ESI＝3`

`0040640A`   `|.`  `76 05`             `|jbe short ToolBox.00406411`       余数少于等于9时，跳到`00406411`去，Add 30

`0040640C`   `|.`  `80C3 57`           `|add bl,57`                        否则Add 57

`0040640F`   `|.`  `EB 03`             `|jmp short ToolBox.00406414`

`00406411`   `|>`  `80C3 30`           `|add bl,30`

`00406414`   `|>`  `8819`              `|mov byte ptr ds:[ecx],bl`         保存，ECX保存内存地址：`0012EF3C`

`00406416`   `|.`  `41`                `|inc ecx`                          ECX自加1，开辟下一个保存地址！！

`00406417`   `|.`  `85F6`              `|test esi,esi`                     Div的商是0吗？看看是否要结束了！！

`00406419`   `|.^ 77 DA`             `\ja short ToolBox.004063F5`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

wofan

77 div 24=3 --------B

B>9    B+57=62（保存在0012EF3C

3<>0

3 div 24=0 --------3

3<9    3+30=33（保存在0012EF3D

0=0  结束循环！

××××××××××××××××××××××××××

```
0040641B   |.  8021 00           and byte ptr ds:[ecx],0       \*\*\*\*\*and一下，   清空下一个保存区
0040641E   |.  49                dec ecx                       \*\*\*\*\*Dec 一下，指针回移！
0040641F   |>  8A17              mov dl,byte ptr ds:[edi]      \*\*\*\*\*DL=62
00406421   |.  8A01              mov al,byte ptr ds:[ecx]      \*\*\*\*\*AL=33
00406423   |.  8811              mov byte ptr ds:[ecx],dl      \*\*\*\*\*以下将存储位置互换一下！
00406425   |.  8807              mov byte ptr ds:[edi],al
00406427   |.  49                dec ecx
00406428   |.  47                inc edi
00406429   |.  3BF9              cmp edi,ecx
0040642B   |.^ 72 F2             jb short ToolBox.0040641F
0040642D   |.  5F                pop edi
0040642E   |.  5E                pop esi
0040642F   |.  5B                pop ebx
00406430   |.  5D                pop ebp
00406431   \.  C3                retn
```

\*\*\*\*\*\*\*\*如果第1，2位注册码是正确的，就来到这里，这个Call也要看一下对字母小写转大写的Call：大写之后用来作为查表的依据\*\*\*\*\*\*\*\*

……
```
00405E35   |.  8BEC              mov ebp,esp
00405E37   |.  51                push ecx
00405E38   |.  A1 24FD4000       mov eax,dword ptr ds:[40FD24]
00405E3D   |.  53                push ebx
00405E3E   |.  33DB              xor ebx,ebx
00405E40   |.  3BC3              cmp eax,ebx
00405E42   |.  895D FC           mov dword ptr ss:[ebp-4],ebx
00405E45   |.  75 21             jnz short ToolBox.00405E68
00405E47   |.  8B45 08           mov eax,dptr ss:[ebp+8]
00405E4A   |.  8BD0              mov edx,eax
00405E4C   |.  3818              cmp byte ptr ds:[eax],bl
00405E4E   |.  74 7F             je short ToolBox.00405ECF
00405E50   |>  8A0A              /mov cl,byte ptr ds:[edx]
00405E52   |.  80F9 61           |cmp cl,61
00405E55   |.  7C 0A             |jl short ToolBox.00405E61
00405E57   |.  80F9 7A           |cmp cl,7A
00405E5A   |.  7F 05             |jg short ToolBox.00405E61
00405E5C   |.  80E9 20           |sub cl,20                       \*\*\*\*\*\*\*\*\*62(b)是字母，就来到这里，62-20=42（B）变成大写的过程。
00405E5F   |.  880A              |mov byte ptr ds:[edx],cl       \*\*\*\*\*EDX=0012EF3D,这可接着上面来的存储区噢！存起来吧！
00405E61   |>  42                |inc edx                       \*\*\*\*inc EDX  开辟下一个存储区
00405E62   |.  381A              |cmp byte ptr ds:[edx],bl
00405E64   |.^ 75 EA             \jnz short ToolBox.00405E50
00405ED1   \.  C3                retn
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*查表找到后面的注册码或中间数据！！！！！\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
004051A9   /$  55                push ebp
004051AA   |.  8BEC              mov ebp,esp
004051AC   |.  81EC 20010000     sub esp,120
004051B2   |.  53                push ebx
004051B3   |.  56                push esi
004051B4   |.  57                push edi
004051B5   |.  6A 34             push 34
004051B7   |.  5B                pop ebx
004051B8   |.  C785 E0FEFFFF 300>mov dword ptr ss:[ebp-120],30        \*\*\*\*\*从这里一开始，复制了一张表
004051C2   |.  6A 35             push 35
004051C4   |.  C785 E4FEFFFF 310>mov dword ptr ss:[ebp-11C],31
004051CE   |.  5F                pop edi
004051CF   |.  C785 E8FEFFFF 320>mov dword ptr ss:[ebp-118],32
004051D9   |.  6A 36             push 36
004051DB   |.  C785 ECFEFFFF 330>mov dword ptr ss:[ebp-114],33
004051E5   |.  5E                pop esi
004051E6   |.  899D F0FEFFFF     mov dword ptr ss:[ebp-110],ebx
004051EC   |.  6A 37             push 37
004051EE   |.  89BD F4FEFFFF     mov dword ptr ss:[ebp-10C],edi
004051F4   |.  5A                pop edx
004051F5   |.  89B5 F8FEFFFF     mov dword ptr ss:[ebp-108],esi
004051FB   |.  6A 38             push 38
004051FD   |.  8995 FCFEFFFF     mov dword ptr ss:[ebp-104],edx
00405203   |.  59                pop ecx
00405204   |.  C785 08FFFFFF 610>mov dword ptr ss:[ebp-F8],61
0040520E   |.  6A 39             push 39
00405210   |.  898D 00FFFFFF     mov dword ptr ss:[ebp-100],ecx
00405216   |.  58                pop eax
00405217   |.  C785 0CFFFFFF 620>mov dword ptr ss:[ebp-F4],62
00405221   |.  8985 04FFFFFF     mov dword ptr ss:[ebp-FC],eax
00405227   |.  C785 10FFFFFF 630>mov dword ptr ss:[ebp-F0],63
00405231   |.  C785 14FFFFFF 640>mov dword ptr ss:[ebp-EC],64
0040523B   |.  C785 18FFFFFF 650>mov dword ptr ss:[ebp-E8],65
00405245   |.  C785 1CFFFFFF 660>mov dword ptr ss:[ebp-E4],66
0040524F   |.  C785 20FFFFFF 670>mov dword ptr ss:[ebp-E0],67
00405259   |.  C785 24FFFFFF 680>mov dword ptr ss:[ebp-DC],68
00405263   |.  C785 28FFFFFF 690>mov dword ptr ss:[ebp-D8],69
0040526D   |.  C785 2CFFFFFF 6A0>mov dword ptr ss:[ebp-D4],6A
00405277   |.  C785 30FFFFFF 6B0>mov dword ptr ss:[ebp-D0],6B
00405281   |.  C785 34FFFFFF 6C0>mov dword ptr ss:[ebp-CC],6C
0040528B   |.  C785 38FFFFFF 6D0>mov dword ptr ss:[ebp-C8],6D
00405295   |.  C785 3CFFFFFF 6E0>mov dword ptr ss:[ebp-C4],6E
0040529F   |.  C785 40FFFFFF 6F0>mov dword ptr ss:[ebp-C0],6F
004052A9   |.  C785 44FFFFFF 700>mov dword ptr ss:[ebp-BC],70
004052B3   |.  C785 48FFFFFF 710>mov dword ptr ss:[ebp-B8],71
004052BD   |.  C785 4CFFFFFF 720>mov dword ptr ss:[ebp-B4],72
004052C7   |.  C785 50FFFFFF 730>mov dword ptr ss:[ebp-B0],73
004052D1   |.  C785 54FFFFFF 740>mov dword ptr ss:[ebp-AC],74
004052DB   |.  C785 58FFFFFF 750>mov dword ptr ss:[ebp-A8],75
004052E5   |.  C785 5CFFFFFF 760>mov dword ptr ss:[ebp-A4],76
004052EF   |.  C785 60FFFFFF 770>mov dword ptr ss:[ebp-A0],77
004052F9   |.  C785 64FFFFFF 780>mov dword ptr ss:[ebp-9C],78
00405303   |.  C785 68FFFFFF 790>mov dword ptr ss:[ebp-98],79
0040530D   |.  C785 6CFFFFFF 7A0>mov dword ptr ss:[ebp-94],7A
00405317   |.  C785 70FFFFFF 300>mov dword ptr ss:[ebp-90],30
00405321   |.  C785 74FFFFFF 310>mov dword ptr ss:[ebp-8C],31
0040532B   |.  C785 78FFFFFF 320>mov dword ptr ss:[ebp-88],32
00405335   |.  C785 7CFFFFFF 330>mov dword ptr ss:[ebp-84],33
0040533F   |.  895D 80           mov dword ptr ss:[ebp-80],ebx
00405342   |.  897D 84           mov dword ptr ss:[ebp-7C],edi
00405345   |.  8975 88           mov dword ptr ss:[ebp-78],esi
00405348   |.  8955 8C           mov dword ptr ss:[ebp-74],edx
0040534B   |.  894D 90           mov dword ptr ss:[ebp-70],ecx
0040534E   |.  8945 94           mov dword ptr ss:[ebp-6C],eax
00405351   |.  C745 98 41000000  mov dword ptr ss:[ebp-68],41
00405358   |.  C745 9C 42000000  mov dword ptr ss:[ebp-64],42
```
