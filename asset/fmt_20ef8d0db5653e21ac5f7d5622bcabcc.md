
想了一下子，即然游戏 `COPY` 到另一个目录下就会有人物不可全选的现象，那应该记录的地方不是在注册表里面。不是在注册表里面，那就很有可能是在文件里面，试试下 `CreateFileA` 看看有没有什么东西可用的：

经过 1 个中断，断在一个很可疑的地方，此时的堆栈是：

```
0558FB9C   0047B3B3  /CALL to CreateFileA from SSS.0047B3AD
0558FBA0   0559B5D0  |FileName = "I:\ToHeart2 XRATED\sys.sav"
0558FBA4   80000000  |Access = GENERIC_READ
0558FBA8   00000001  |ShareMode = FILE_SHARE_READ
0558FBAC   00000000  |pSecurity = NULL
0558FBB0   00000003  |Mode = OPEN_EXISTING
0558FBB4   00000080  |Attributes = NORMAL
0558FBB8   00000000  \hTemplateFile = NULL
```

我的游戏目录明明是在 `i:\ToHeart2 XRATED\sss\` 下面的，它干嘛要去上一层目录去找 `SAV` 文件啊？

按 `Alt+F9` 返回用户代码

```
0047B39C  |> \53            push ebx                              ; /hTemplateFile
0047B39D  |.  68 80000000  push 80                                 ; |Attributes = NORMAL
0047B3A2  |.  6A 03         push 3                                  ; |Mode = OPEN_EXISTING
0047B3A4  |.  53            push ebx                                ; |pSecurity
0047B3A5  |.  6A 01         push 1                                  ; |ShareMode = FILE_SHARE_READ
0047B3A7  |.  68 00000080   push 80000000                           ; |Access = GENERIC_READ
0047B3AC  |.  50            push eax                                ; |FileName
0047B3AD  |.  FF15 44F05F00 call dword ptr ds:[<&KERNEL32.CreateFile>; \CreateFileA
0047B3B3  |.  8BF8          mov edi,eax
0047B3B5  |.  83FF FF      cmp edi,-1
0047B3B8  |.  0F85 01030000 jnz SSS.0047B6BF
0047B3BE  |.  50            push eax
0047B3BF  |.  53            push ebx
0047B3C0  |.  8D4C24 3C     lea ecx,dword ptr ss:[esp+3C]
```

之后走出几个 `CALL` 后：

```
00454348  |.  8D4C24 58     lea ecx,dword ptr ss:[esp+58]
0045434C  |.  E8 6F740200   call SSS.0047B7C0
00454351  |.  3BC7          cmp eax,edi
00454353  |.  74 33         je short SSS.00454388
00454355  |.  8D4C24 3C     lea ecx,dword ptr ss:[esp+3C]
```

往下走几步后，发现这个比较可疑的：

```
004543B3  |.  8BF3          mov esi,ebx
004543B5  |>  8B44B4 14     /mov eax,dword ptr ss:[esp+esi*4+14]
004543B9  |.  05 6E180000   |add eax,186E
004543BE  |.  3BC7          |cmp eax,edi
004543C0  |.  73 25         |jnb short SSS.004543E7
004543C2  |.  395C85 00     |cmp dword ptr ss:[ebp+eax*4],ebx
004543C6  |.  75 1F         |jnz short SSS.004543E7
004543C8  |.  85F6          |test esi,esi
004543CA  |.  7C 1B         |jl short SSS.004543E7
004543CC  |.  83FE 0A       |cmp esi,0A
004543CF  |.  7D 16         |jge short SSS.004543E7
004543D1  |.  B9 A0DE7200   |mov ecx,SSS.0072DEA0
004543D6  |.  E8 A5F9FAFF   |call SSS.00403D80
004543DB  |.  A1 44736A00   |mov eax,dword ptr ds:[6A7344]
004543E0  |.  889C06 210D00>|mov byte ptr ds:[esi+eax+D21],bl
004543E7  |>  46            |inc esi
004543E8  |.  83FE 0A       |cmp esi,0A
004543EB  |.^ 7C C8         \jl short SSS.004543B5
004543ED  |.  8D4C24 3C     lea ecx,dword ptr ss:[esp+3C]
```

这一段有一行代码是很有可疑的

```
004543E8  |.  83FE 0A       |cmp esi,0A
```

因为人物是 10 个，它又刚好的循环 `0xA`(10)次

```
004543C2  |.  395C85 00     |cmp dword ptr ss:[ebp+eax*4],ebx
```

走到这一句时，`EBX` 则为 0，下面又有一个 `JNZ` 的条件跳转

```
004543DB  |.  A1 44736A00   |mov eax,dword ptr ds:[6A7344]
004543E0  |.  889C06 210D00>|mov byte ptr ds:[esi+eax+D21],bl
004543E7  |>  46            |inc esi                              ;  JNZ 跳到这里
004543E0  |.  889C06 210D00>|mov byte ptr ds:[esi+eax+D21],bl
```

走到这一句时，`BL` 为 1，貌似是把 `[esi+eax+D21]` 置为 1，很可疑，`EAX` 的值是个定数（上一行代码可知），`ESI` 就是一个人物个数的计数器，第 1 个人物时，`ESI == 1`，第 2 个人物时，`ESI == 2`；

把 `JNZ` 改为 `NOP`，使它不跳，继续运行

发现这样以后人物都可以选择了，看来关键点就在这里；

可是很快又发现一个问题了，我写的是一个外挂，不是一个 `LOADER`，所以在运行这个外挂的时候这一句代码早就运行过了，不可能再次跳到这里运行了，

所以还得想另一种方法。

既然 `ESI+EAX+D21` 是一个 `static` 指针，那地址就应该是相对稳定的，

刚才还记录了第 1 个地址是 `0xCFCA82`，直接在该地址上面下 `byte` 的断点

再次运行程序，在选择人物时程序中断在一个新的点上：

```
0040D8D3  |.  E8 A864FFFF   call SSS.00403D80
0040D8D8  |.  A1 44736A00   mov eax,dword ptr ds:[6A7344]
0040D8DD  |.  8A8430 210D00>mov al,byte ptr ds:[eax+esi+D21]         ;  断在这里
0040D8E4  |.  5E            pop esi
0040D8E5  |.  C3            retn
0040D8E6  |>  32C0          xor al,al
0040D8E8  |.  5E            pop esi
0040D8E9  \.  C3            retn
```

`retn` 后就是这里了：

```
0040FD80   .  8B8E BC000000 mov ecx,dword ptr ds:[esi+BC]
0040FD86   .  51            push ecx
0040FD87   .  E8 34DBFFFF   call SSS.0040D8C0
0040FD8C   .  83C4 04       add esp,4
0040FD8F   .  84C0          test al,al
0040FD91   .  75 07         jnz short SSS.0040FD9A
0040FD93   .  C746 24 25000>mov dword ptr ds:[esi+24],25
0040FD9A   >  8B46 24       mov eax,dword ptr ds:[esi+24]
0040FD9D   .  C1E0 04       shl eax,4
```

嗯，在 `40D8DD` 里面改就可以了；

一句老话，还要跳到空白代码上：

```
0040D8DD   /E9 1E0D1F00     jmp SSS.005FE600
0040D8E2   |90              nop
0040D8E3   |90              nop
0040D8E4   |5E              pop esi
0040D8E5   |C3              retn
```

其二进制代码是 `E9 1E 0D 1F 00 90 90`

```
005FE5FF     00            db 00
005FE600     B0 01         mov al,1                              ;  AL 置 1
005FE602     888430 210D00>mov byte ptr ds:[eax+esi+D21],al      ;  使人物变为可选
005FE609     56            push esi                                ;  保存 ESI 指针
005FE60A     50            push eax                                ;  保存 EAX 数据
005FE60B     A1 44736A00   mov eax,dword ptr ds:[6A7344]           ;  取得 Static 变量的地址
005FE610     83C6 01       add esi,1                               ;  人物指针+1
005FE613     C68430 210D00>mov byte ptr ds:[eax+esi+D21],1         ;  激活人物
005FE61B     58            pop eax                                 ;  恢复 EAX 数据
005FE61C     5E            pop esi                                 ;  恢复 ESI 指针
005FE61D     5E            pop esi                                 ;  原代码
005FE61E     C3            retn                                    ;  原代码
005FE61F     00            db 00
```

其二进制代码是 `B0 01 88 84 30 21 0D 00 00 56 50 A1 44 73 6A 00 83 C6 01 C6 84 30 21 0D 00 00 01 58 5E 5E C3`

不过这也有个缺点：在选择人物前就要更改代码了，不然的话还是没用，

不是 `loader` 真是受限制啊 :(`

`VC` 中的代码：

```cpp
void CSSSEditorDlg::OnButtonactive()

{

  BYTE mem[7] =  { 0xE9, 0x1E, 0x0D, 0x1F, 0x00, 0x90, 0x90 };

  BYTE mem2[31] = { 0xB0, 0x01, 0x88, 0x84, 0x30, 0x21, 0x0D, 0x00, 0x00, 0x56,

            0x50, 0xA1, 0x44, 0x73, 0x6A, 0x00, 0x83, 0xC6, 0x01, 0xC6,

            0x84, 0x30, 0x21, 0x0D, 0x00, 0x00, 0x01, 0x58, 0x5E, 0x5E,

            0xC3

          };

  DWORD addr = 0x40D8DD, addr2 = 0x5FE600, DMemWrite;

  MessageBox( "在选择人物前就要激活", "SSSEDitor", MB_OK|MB_ICONINFORMATION );

  WriteProcessMemory( hPrc, (void*)addr, &mem, 7, &DMemWrite );

  WriteProcessMemory( hPrc, (void*)addr2, &mem2, 31, &DMemWrite );

}
```

[KFC]fish (Pr0Zel)

编译好的修改器可以到这里下载：[`www.keyfc.net/bbs/disp.asp?titleid=13592`](http://www.keyfc.net/bbs/disp.asp?titleid=13592)

游戏可以去 `MO` 上面下载 `www.mofile.com`

提取码是 `4164151347219943`

大小是 `19MB`

`BTW`:有一个朋友对我说“21 世纪有技术的年代，所以作弊也要选一种有技术的作弊方法”，我倒....... ![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
