
```
00485C08       |.  59              pop ecx
00485C09       \.  C3              retn
```

在`00485BAB`可以确定序列号第8位为`0x39`（9），从`00485BAD`到`retn`其实要求“序列号后8位的第4位”与“序列号中间4位的第3位”构成的数据等于`0x13*3`即`0x39`，根据`序列号检查函数4-1`算法，可以确定“序列号后8位的第4位”与“序列号中间4位的第3位”为`5`、`B`。

这样注册码修改为`GP3EHA79-1AB4-12B55N7A`。

下面接着分析：

```
00514130       |.  59              pop ecx
00514131       |.  84C0            test al,al
00514133       |.  0F84 B0000000   je oC4PE.005141E9
00514139       |.  8B83 E0020000   mov eax,dword ptr ds:[ebx+2E0]
0051413F       |.  E8 0003F7FF     call oC4PE.00484444
00514144       |.  84C0            test al,al
00514146       |.  74 34           je short oC4PE.0051417C
00514148       |.  B9 DF010000     mov ecx,1DF
0051414D       |.  BA B8010000     mov edx,1B8
00514152       |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]
00514155       |.  E8 3686F6FF     call oC4PE.0047C790             ;  版本检查函数
0051415A       |.  84C0            test al,al
0051415C       |.  75 4F           jnz short oC4PE.005141AD
0051415E       |.  8D55 D4         lea edx,dword ptr ss:[ebp-2C]
00514161       |.  A1 98D05D00     mov eax,dword ptr ds:[5DD098]
00514166       |.  E8 9117EFFF     call oC4PE.004058FC
0051416B       |.  8B45 D4         mov eax,dword ptr ss:[ebp-2C]
0051416E       |.  33C9            xor ecx,ecx
00514170       |.  33D2            xor edx,edx
00514172       |.  E8 4DF00700     call oC4PE.005931C4
00514177       |.  E9 A1000000     jmp oC4PE.0051421D
0051417C       |>  B9 B7010000     mov ecx,1B7
00514181       |.  BA 90010000     mov edx,190
00514186       |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]
00514189       |.  E8 0286F6FF     call oC4PE.0047C790
0051418E       |.  84C0            test al,al
00514190       |.  74 1B           je short oC4PE.005141AD
00514192       |.  8D55 D0         lea edx,dword ptr ss:[ebp-30]
00514195       |.  A1 98D05D00     mov eax,dword ptr ds:[5DD098]
0051419A       |.  E8 5D17EFFF     call oC4PE.004058FC
0051419F       |.  8B45 D0         mov eax,dword ptr ss:[ebp-30]
005141A2       |.  33C9            xor ecx,ecx
005141A4       |.  33D2            xor edx,edx
005141A6       |.  E8 19F00700     call oC4PE.005931C4
005141AB       |.  EB 70           jmp short oC4PE.0051421D
005141AD       |>  8B83 E0020000   mov eax,dword ptr ds:[ebx+2E0]
005141B3       |.  83F8 0F         cmp eax,0F
005141B6       |.  74 25           je short oC4PE.005141DD
005141B8       |.  83F8 10         cmp eax,10
005141BB       |.  74 20           je short oC4PE.005141DD
005141BD       |.  83F8 13         cmp eax,13
005141C0       |.  74 1B           je short oC4PE.005141DD
005141C2       |.  8D55 CC         lea edx,dword ptr ss:[ebp-34]
005141C5       |.  A1 64D25D00     mov eax,dword ptr ds:[5DD264]
005141CA       |.  E8 2D17EFFF     call oC4PE.004058FC
005141CF       |.  8B45 CC         mov eax,dword ptr ss:[ebp-34]
005141D2       |.  33C9            xor ecx,ecx
005141D4       |.  33D2            xor edx,edx
005141D6       |.  E8 E9EF0700     call oC4PE.005931C4
005141DB       |.  EB 40           jmp short oC4PE.0051421D
005141DD       |>  C783 34020000 0>mov dword ptr ds:[ebx+234],1
005141E7       |.  EB 34           jmp short oC4PE.0051421D
005141E9       |>  FF83 E4020000   inc dword ptr ds:[ebx+2E4]
005141EF       |.  8D55 C8         lea edx,dword ptr ss:[ebp-38]
005141F2       |.  A1 10D55D00     mov eax,dword ptr ds:[5DD510]
005141F7       |.  E8 0017EFFF     call oC4PE.004058FC
005141FC       |.  8B45 C8         mov eax,dword ptr ss:[ebp-38]
005141FF       |.  33C9            xor ecx,ecx
00514201       |.  33D2            xor edx,edx
00514203       |.  E8 BCEF0700     call oC4PE.005931C4
00514208       |.  8BC3            mov eax,ebx
0051420A       |.  E8 69F8FFFF     call oC4PE.00513A78
0051420F       |.  84C0            test al,al
00514211       |.  74 0A           je short oC4PE.0051421D
00514213       |.  C783 34020000 0>mov dword ptr ds:[ebx+234],2
0051421D       |>  33C0            xor eax,eax
0051421F       |.  5A              pop edx
00514220       |.  59              pop ecx
00514221       |.  59              pop ecx
00514222       |.  64:8910         mov dword ptr fs:[eax],edx
00514225       |.  68 4C425100     push oC4PE.0051424C
0051422A       |>  8D45 C8         lea eax,dword ptr ss:[ebp-38]
0051422D       |.  BA 05000000     mov edx,5
00514232       |.  E8 F5F9EEFF     call oC4PE.00403C2C
00514237       |.  8D45 DC         lea eax,dword ptr ss:[ebp-24]
0051423A       |.  BA 02000000     mov edx,2
0051423F       |.  E8 E8F9EEFF     call oC4PE.00403C2C
00514244       \.  C3              retn
```

到此分析完成了一部分，这时如果你利用得到的序列号`GP3EHA79-1AB4-12B55N7A`输入，程序会提示“感谢注册，重启......”，但是一旦重启就会显示仍然是试用版。很明显有重启验证了。

下面来对付重启验证了。

首先做点准备工作，利用对注册表操作函数下断点可以发现程序将序列号存储在：`HKEY_CLASSES_ROOT\CLSID\{D81EDBF9-D167-4011-B77D-211DF920EB80}\Version`下的`id`及`idx`中。

简单说来，重启验证后有功能限制，那么我们就利用功能限制（呵呵，我本来想利用程序初始化话阶段的处理函数，但失败了，因为程序是读取了`id`和`idx`后，利用`SendMessage`来判断的，我没有找到下手地方，菜鸟嘛可以理解了）了，非注册版不能保存文件，这样利用“文件保存”按钮对应的函数可以定位到下面关键代码：

```
00530CC2        > \8D45 E8         lea eax,dword ptr ss:[ebp-18]
00530CC5        .  50              push eax
00530CC6        .  8D45 EC         lea eax,dword ptr ss:[ebp-14]
00530CC9        .  50              push eax
00530CCA        .  8D45 F0         lea eax,dword ptr ss:[ebp-10]
00530CCD        .  50              push eax
00530CCE        .  56              push esi
00530CCF        .  E8 554CF5FF     call oC4PE.00485929             ;  验证函数1
00530CD4        .  83C4 10         add esp,10
00530CD7        .  83E0 7F         and eax,7F
00530CDA        .  03F8            add edi,eax
00530CDC        .  8D45 DA         lea eax,dword ptr ss:[ebp-26]
00530CDF        .  50              push eax
00530CE0        .  8D45 DB         lea eax,dword ptr ss:[ebp-25]
00530CE3        .  50              push eax
00530CE4        .  56              push esi
00530CE5        .  E8 874DF5FF     call oC4PE.00485A71             ;  验证函数2
00530CEA        .  83C4 0C         add esp,0C
00530CED        .  83E0 7F         and eax,7F
00530CF0        .  03F8            add edi,eax
00530CF2        .  56              push esi
00530CF3        .  E8 6B4EF5FF     call oC4PE.00485B63             ;  验证函数3
00530CF8        .  59              pop ecx
00530CF9        .  83E0 7F         and eax,7F
00530CFC        .  03F8            add edi,eax
00530CFE        .  B9 1E000000     mov ecx,1E
00530D03        .  33D2            xor edx,edx
00530D05        .  8B45 E0         mov eax,dword ptr ss:[ebp-20]
00530D08        .  E8 83BAF4FF     call oC4PE.0047C790             ;  验证函4
00530D0D        .  83E0 7F         and eax,7F
00530D10        .  03D8            add ebx,eax
00530D12        .  8B4D E8         mov ecx,dword ptr ss:[ebp-18]
00530D15        .  8B55 EC         mov edx,dword ptr ss:[ebp-14]
00530D18        .  8B45 F0         mov eax,dword ptr ss:[ebp-10]
00530D1B        .  E8 CC37F5FF     call oC4PE.004844EC             ;  验证函数5
00530D20        .  B9 1E000000     mov ecx,1E
00530D25        .  33D2            xor edx,edx
00530D27        .  E8 64BAF4FF     call oC4PE.0047C790             ;  验证函数6
00530D2C        .  83E0 7F         and eax,7F
00530D2F        .  03D8            add ebx,eax
00530D31        .  837D DC 02      cmp dword ptr ss:[ebp-24],2
00530D35        .  74 18           je short oC4PE.00530D4F         ;
00530D37        .  B9 9C0E5300     mov ecx,oC4PE.00530E9C
00530D3C        .  B2 01           mov dl,1
00530D3E        .  A1 7C414800     mov eax,dword ptr ds:[48417C]
00530D43        .  E8 8093EDFF     call oC4PE.0040A0C8
00530D48        .  E8 8B29EDFF     call oC4PE.004036D8
00530D4D        .  EB 38           jmp short oC4PE.00530D87
00530D4F        >  83FF 05         cmp edi,5
00530D52        .  74 05           je short oC4PE.00530D59         ;
00530D54        .  83FB 02         cmp ebx,2
00530D57        .  75 15           jnz short oC4PE.00530D6E
00530D59        >  33C9            xor ecx,ecx
```
