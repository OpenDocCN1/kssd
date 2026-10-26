
```asm
0049725E      8A4D FA            mov cl,byte ptr ss:[ebp-6]            ; 第2字节，7F
00497261      C1E9 04            shr ecx,4
00497264      0AD1               or dl,cl
00497266      80E2 3F            and dl,3F
00497269      8855 F6            mov byte ptr ss:[ebp-A],dl            ; 27
0049726C      25 FF000000        and eax,0FF
00497271      C1E8 02            shr eax,2
00497274      24 3F              and al,3F
00497276      8845 F5            mov byte ptr ss:[ebp-B],al            ; 12
00497279      8D45 FC            lea eax,dword ptr ss:[ebp-4]
```

取参数的前3字节，计算得到4个字节，逆序排列：

```text
0012FD3D  12 27 3D 34 4A 7F 74                             '=4Jt
```

然后对这4个字节查表：`0012FD3D  12 27 3D 34`

```asm
00497290      8D45 F0            lea eax,dword ptr ss:[ebp-10]
00497293      33D2               xor edx,edx
00497295      8A13               mov dl,byte ptr ds:[ebx]              ; 取得字节
00497297      8A92 2DFD4900      mov dl,byte ptr ds:[edx+49FD2D]       ; 查表
0049729D      E8 BED0F6FF        call pcwxzs.00404360
004972A2      8B55 F0            mov edx,dword ptr ss:[ebp-10]
004972A5      8BC7               mov eax,edi
004972A7      E8 94D1F6FF        call pcwxzs.00404440
004972AC      43                 inc ebx
004972AD      4E                 dec esi
004972AE    ^ 75 E0              jnz short pcwxzs.00497290
```

表：

```text
0049FD2D  49 59 41 47 50 58 44 4A 51 57 4D 48 56 43 4E 46  IYAGPXDJQWMHVCNF
0049FD3D  55 5A 52 42 4B 45 53 4F 4C 54 74 66 6B 79 73 62  UZRBKESOLTtfkysb
0049FD4D  6F 68 6C 75 6A 77 65 63 70 6D 69 61 71 6E 64 78  ohlujwecpmiaqndx
0049FD5D  7A 76 67 72 34 36 2B 30 32 35 37 33 2F 38 31 3D  zvgr46+02573/81=
0049FD6D  39 8B C0 30 31 32 33 34 35 36 37 38 39 61 62 63  9?0123456789abc
0049FD7D  64 65 66 01 23 45 67 89 AB CD EF FE DC BA 98 76  def#Eg?惋??v
```

得到4个字符：`Rc84`。

因为参数总共是9个字节，所以得到12个字符：ASCII `"Rc84QljdQOEa"`

然后返回：

```asm
00497405      E8 9AFDFFFF      call pcwxzs.004971A4            ; 计算得到注册码
0049740A      8B55 E0          mov edx,dword ptr ss:[ebp-20]   ; 返回处
```

```text
堆栈 ss:[0012FD68]=00E05BC0, (ASCII "Rc84QljdQOEa")
```

虽然注册码的计算没有涉及到MD5，我想可能是机器码的计算涉及到MD5，要不然对算法的定位点下断是到不了这里的。

然后返回n层，调出注册的NAG，进行比对：

```asm
004996B3      FF92 EC000000    call dword ptr ds:[edx+EC]   ; 调出注册
```

没有进行跟踪，因为输入得到的注册码：ASCII `"Rc84QljdQOEa"`后就成功了，呵呵。

写入注册表：

```text
HKCU\Software\eBook Edit Pro\Login\5318724F\LoginUser        SUCCESS        "4379315163"
HKCU\Software\eBook Edit Pro\Login\5318724F\LoginPassword        SUCCESS        "Rc84QljdQOEa"
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
