
# 全球通高清晰数字网络电视的几种破解方法

> 原文：`[www.pediy.com/kssd/pediy07/pediy7-413.htm](https://www.pediy.com/kssd/pediy07/pediy7-413.htm)`

**【软件介绍】**：国产软件，简体中文

**【软件下载】**：`http://www.softreg.com.cn/shareware_view.asp?id=/271C44A8-F879-441E-9A56-B866FC86BDDF/`

**【保护方式】**：注册码 + 功能限制

**【破解工具】**：`Win2000`, `PEiD`, `Ollydbg`, `W32dasm`

**【破解目的】**：研究算法分析，没有其它目的！

**【作者申明】**：感谢 `skyege` 的帮助。

## 【破解过程】

用 `peid` 查看，是 `upx` 的壳，先将它脱掉

### 方法一： 暴破法

用 `W32` 查看字符串，有“注册不正确，无法注册”的提示和“感谢你注册全球通高清晰数字网络电视机！”的字样，点击“注册不正确”字符，来到：

```
:004B6548 53                     push ebx
:004B6549 8BD8                   mov ebx, eax
:004B654B 8BC3                   mov eax, ebx
:004B654D E87A1AFFFF             call 004A7FCC          关键 call
:004B6552 84C0                   test al, al
:004B6554 7409                   je 004B655F           '关键转跳
:004B6556 8BC3                   mov eax, ebx
:004B6558 E8BB17FFFF             call 004A7D18
:004B655D 5B                     pop ebx
:004B655E C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B6554(C)
|
:004B655F 6A20                   push 00000020
* Possible StringData Ref from Code Obj ->"信息提示"
                              |
:004B6561 B97C654B00             mov ecx, 004B657C
* Possible StringData Ref from Code Obj ->"注册码不正确,无法注册"   <＝＝这里
                              |
:004B6566 BA88654B00             mov edx, 004B6588
:004B656B A160014C00             mov eax, dword ptr [004C0160]
```

向上看到 `004B6554` 的转跳，猜这儿是关键转跳了！如果你还不能确认 `004B6554` 是不是关键的转跳，你可以跟进 `004B6558` 的 `call` 看看，会看到“感谢你注册 XXX”的字样，这下放心了，一定是关键转跳了，二话不说，将 `je` 改为 `jne` 便可以了！

### 方法二： 注册表跟踪

用 `w32` 看到输入函数里面有 `RegOpenKeyExA` 和 `RegQueryValueExA` 函数，用 `OD` 载入，将两个函数下断，将跟踪到要找开 `HKEY_USERS\.DEFAULT\Software\nettv\Demo` 时，如果自已没有，新建一个，`F9` 一下，这时 `RegQueryValueExA` 会去查没有没 `Name` 这个值，没有就建一个吧（其值的类型可以由 `RegQueryValueExA` 的参数来确定，这里随便什么类型都可以），再 `F9`，`RegQueryValueExA` 会去查值为 `Pass` 的值存不存在，没有关系，新建一个，类型不限，再 `F9`，发现自己已经注册了，这么简单就搞定了！！再点出上面的钻石版栏目激活，`VIP` 栏目激活，一看，所以灰色的菜单全都可以用了！

在方法三中我们可以知道这两上值到底是什么类型（分别为 `reg_sz` 和 `reg_dword`，两个都没有具体的值）

### 方法三： 跟踪算法（这个其实也很 Easy）

由第一得到 `004B654D` 的关键 `call`（这是个 `Borland Delphi 4.0 - 5.0` 写的，用 `dede` 也可以的） `F7` 跟进到此：

```
004A800D      |.  BE 01000000      mov esi,1
004A8012      |>  8D45 F0         /lea eax,dword ptr ss:[ebp-10]
004A8015      |.  50             |push eax
004A8016      |.  B9 01000000    |mov ecx,1
004A801B      |.  8BD6           |mov edx,esi
004A801D      |.  8B45 F8        |mov eax,dword ptr ss:[ebp-8]
004A8020      |.  E8 DBC0F5FF    |call unpackun.00404100
004A8025      |.  8B45 F0        |mov eax,dword ptr ss:[ebp-10]
004A8028      |.  E8 8FC0F5FF    call unpackun.004040BC
004A802D      |.  8A00           |mov al,byte ptr ds:[eax]      '开始运算机器码
004A802F      |.  25 FF000000    and eax,0FF                    取 al
004A8034      |.  0145 F4        |add dword ptr ss:[ebp-C],eax  将 al 的 ascii 码相加
004A8037      |.  46                       |inc esi
004A8038      |.  4B                       |dec ebx
004A8039      |.^ 75 D7                 \jnz short unpackun.004A8012
004A803B      |>  8D55 EC               lea edx,dword ptr ss:[ebp-14]
004A803E      |.  8B45 FC               mov eax,dword ptr ss:[ebp-4]
004A8041      |.  8B80 D8090000         mov eax,dword ptr ds:[eax+9D8]
004A8047      |.  E8 4C18F8FF           call unpackun.00429898
004A804C      |.  8B45 EC               mov eax,dword ptr ss:[ebp-14]
004A804F      |.  E8 F80EF6FF           call unpackun.00408F4C
 ｛
00408F4C      /$  53                    push ebx
00408F4D      |.  56                    push esi
00408F4E      |.  83C4 F4               add esp,-0C
00408F51      |.  8BD8                  mov ebx,eax
00408F53      |.  8BD4                  mov edx,esp
00408F55      |.  8BC3                  mov eax,ebx
00408F57      |.  E8 4C9DFFFF           call unpackun.00402CA8
              ｛
00402CA8      /$  53                    push ebx
00402CA9      |.  56                    push esi
00402CAA      |.  57                    push edi
00402CAB      |.  89C6                  mov esi,eax
00402CAD      |.  50                    push eax
00402CAE      |.  85C0                  test eax,eax
00402CB0      |.  74 73                 je short unpackun.00402D25
00402CB2      |.  31C0                  xor eax,eax
00402CB4      |.  31DB                  xor ebx,ebx
00402CB6      |.  BF CCCCCC0C          mov edi,0CCCCCCC
00402CBB      |> /8A1E                 /mov bl,byte ptr ds:[esi]
00402CBD      |. |46                   |inc esi
00402CBE      |. |80FB 20              |cmp bl,20      ;空格
00402CC1      |.^\\74 F8                \\je short unpackun.00402CBB
00402CC3      |.  B5 00                 mov ch,0        ch=0
00402CC5      |.  80FB 2D               cmp bl,2D        ; ASCII"-"表示负数
00402CC8      |.  74 69                 je short unpackun.00402D33
00402CCA      |.  80FB 2B               cmp bl,2B          " +"  表正数
00402CCD      |.  74 66                 je short unpackun.00402D35
00402CCF      |.  80FB 24               cmp bl,24          "$" 输入的为十六进制
00402CD2      |.  74 66                 je short unpackun.00402D3A
00402CD4      |.  80FB 78               cmp bl,78          "x"  输入的为十六进制
00402CD7      |.  74 61                 je short unpackun.00402D3A
00402CD9      |.  80FB 58               cmp bl,58       "X"  输入的为十六进制
00402CDC      |.  74 5C                 je short unpackun.00402D3A
00402CDE      |.  80FB 30               cmp bl,30          "0"  首字为 0 就忽略
00402CE1      |.  75 13                 jnz short unpackun.00402CF6
00402CE3      |.  8A1E                  mov bl,byte ptr ds:[esi]
00402CE5      |.  46                   inc esi
00402CE6      |.  80FB 78               cmp bl,78
00402CE9      |.  74 4F                 je short unpackun.00402D3A
00402CEB      |.  80FB 58               cmp bl,58
00402CEE      |.  74 4A                 je short unpackun.00402D3A
00402CF0      |.  84DB                  test bl,bl
00402CF2      |.  74 20                 je short unpackun.00402D14
00402CF4      |.  EB 04                 jmp short unpackun.00402CFA
00402CF6      |>  84DB                  test bl,bl
00402CF8      |.  74 34                 je short unpackun.00402D2E
00402CFA      |>  80EB 30               /sub bl,30   输入的字符为 0~9 之间即为十进制
00402CFD      |.  80FB 09               |cmp bl,9
00402D00      |.  77 2C                 |ja short unpackun.00402D2E  这个是将输入的十进制转为十六进制
00402D02      |.  39F8                  |cmp eax,edi
00402D04      |.  77 28                 |ja short unpackun.00402D2E
00402D06      |.  8D0480                |lea eax,dword ptr ds:[eax+eax*4]
00402D09      |.  01C0                  |add eax,eax
00402D0B      |.  01D8                  |add eax,ebx
00402D0D      |.  8A1E                  |mov bl,byte ptr ds:[esi]
00402D0F      |.  46                    |inc esi
00402D10      |.  84DB                  |test bl,bl
00402D12      |.^ 75 E6                 \\jnz short unpackun.00402CFA
00402D14      |>  FECD                  dec ch          ch=ch-1
```
