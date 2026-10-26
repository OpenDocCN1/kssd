
在原程序中找一段可用的空间放IAT，取代跳转到壳中的地址，这样就可以Dump下来了。`jingulong`选择存放IAT的地方是从`40C000`开始的，我也选择这里吧。我们可以找在最下面找空白地址写入一段代码来恢复IAT

```
代码:
---

009571C8     55                  push ebp
009571C9     8BEC                mov ebp,esp
009571CB     81C4 50FDFFFF       add esp,-2B0
009571D1     53                  push ebx
009571D2     56                  push esi
009571D3     57                  push edi
009571D4     894D F8             mov dword ptr ss:[ebp-8],ecx
009571D7     8955 FC             mov dword ptr ss:[ebp-4],edx
009571DA     8BF8                mov edi,eax
009571DC     33C0                xor eax,eax//此时[EDI]=正确的函数地址 ★
009571DE     8D95 93FDFFFF       lea edx,dword ptr ss:[ebp-26D]
009571E4     8955 EC             mov dword ptr ss:[ebp-14],edx
009571E7     33F6                xor esi,esi//这里动手脚吧
009571E9     33DB                xor ebx,ebx
009571EB     8B17                mov edx,dword ptr ds:[edi]
009571ED     8955 E4             mov dword ptr ss:[ebp-1C],edx
009571F0     33D2                xor edx,edx
009571F2     8955 F0             mov dword ptr ss:[ebp-10],edx
009571F5     833F 00             cmp dword ptr ds:[edi],0
009571F8     0F84 01020000       je 009573FF

---
```

```
代码:
---

把009571E7   xor esi,esi 改为：JMP 00964000 

---
```

在`00964000`处汇编：

```
代码:
---

00964000     50                  push eax
00964001     56                  push esi
00964002     8B35 F03F9600       mov esi,dword ptr ds:[963FF0] ; UnPackMe.0040C000//初始地址
00964008     8B07                mov eax,dword ptr ds:[edi]    ; kernel32.GetCommandLineA
0096400A     8906                mov dword ptr ds:[esi],eax//正确的函数写入
0096400C     8933                mov dword ptr ds:[ebx],esi//取代原先入壳的跳转地址
0096400E     83C6 08             add esi,8//没办法
00964011     8935 F03F9600       mov dword ptr ds:[963FF0],esi//保存地址
00964017     58                  pop eax
00964018     5E                  pop esi//恢复堆栈
00964019     33F6                xor esi,esi//原先009571E7处代码挪到这里来运行
0096401B     33DB                xor ebx,ebx
0096401D     8B17                mov edx,dword ptr ds:[edi]
0096401F     E9 C931FFFF         jmp 009571ED//跳回去继续流程

---
```

别忘了在`[963FF0]`处写入初始地址：`0040C000`

我的`Code`能力几乎等于零，以上蹩脚的代码各位兄弟见笑了。

### AL=4的处理

```
代码:
---

把00957943   mov dword ptr ds:[ebx],esi//修改为：jmp 00964030  

---
```

在`00964030`处汇编：

```
代码:
---

00964030     56                  push esi
00964031     8B35 F03F9600       mov esi,dword ptr ds:[963FF0]
00964037     8906                mov dword ptr ds:[esi],eax; kernel32.ExitProcess
00964039     8933                mov dword ptr ds:[ebx],esi//取代原先入壳的跳转地址
0096403B     83C6 08             add esi,8
0096403E     8935 F03F9600       mov dword ptr ds:[963FF0],esi//保存地址
00964044     5E                  pop esi
00964045     33C0                xor eax,eax//原先00957945处代码挪到这里来运行
00964047     8A4424 1D           mov al,byte ptr ss:[esp+1D]
0096404B     E9 FB38FFFF         jmp 0095794B//跳回去继续流程吧

---
```

### 三、补上Stolen Code、修复输入表，完成脱壳

修改完以上代码后，F9运行，中断在`009575FC`处，IAT处理完毕。

```
代码:
---

009575FC     837C24 2C 00        cmp dword ptr ss:[esp+2C],0//此处下断，IAT处理完毕
00957601     74 0C               je short 0095760F

---
```

注意`009575FC`的`[esp+2C]`处保存的是自校验值！如果自校验失败的话在走到OEP之前会出错！所以跟踪未修改的壳程序，得知这里的校验值等于`0098FAF4`。把`[esp+2C]`处的值修改为`0098FAF4`，就可以通过ASProtect最后1次典型异常分析Stolen Code然后走至OEP啦。

其实在`009575FC`处所有的代码均已解开，可以直接Dump了。通过第一次的跟踪分析，我们在OEP处补上Stolen Code：

```
代码:
---

004010CC     55                  push ebp//在这儿用LordPE纠正ImageSize后完全DUMP这个进程
004010CD     8BEC                mov ebp,esp
004010CF     83EC 44             sub esp,44
004010D2     56                  push esi
004010D3     FF15 00C04000       call dword ptr ds:[40C000]; kernel32.GetCommandLineA
004010D9     8BF0                mov esi,eax
004010DB     8A00                mov al,byte ptr ds:[eax]
004010DD     3C 22               cmp al,22//上面是补好的Stolen Code
004010DF     75 1B               jnz short 004010FC
004010E1     56                  push esi
004010E2     FF15 08C04000       call dword ptr ds:[40C008]; user32.CharNextA
004010E8     8BF0                mov esi,eax
004010EA     8A00                mov al,byte ptr ds:[eax]
004010EC     84C0                test al,al
004010EE     74 04               je short 004010F4
004010F0     3C 22               cmp al,22
004010F2     75 ED               jnz short dumped_.004010E1
004010F4     803E 22             cmp byte ptr ds:[esi],22
004010F7     75 15               jnz short 0040110E
004010F9     46                  inc esi
004010FA     EB 12               jmp short 0040110E
004010FC     3C 20               cmp al,20
004010FE     7E 0E               jle short 0040110E
00401100     56                  push esi
00401101     FF15 10C04000       call dword ptr ds:[40C010]; user32.CharNextA

---
```

运行`ImportREC`，选择这个进程。把OEP改为`000010CC`，`RVA=0000C000`，大小＝`CB0`

点“Get Import”，函数排列的比较“难看”，但全部是有效的。FixDump，正常运行！

修改以下几个函数，脱壳后的程序就也可以在Win98上运行了：

```
代码:
---

0000C148  ntdll.dll   0255  RtlGetLastWin32Error//改为：kernel32.dll GetLastError
0000C3B8  ntdll.dll   02B3  RtlMoveMemory       //改为：kernel32.dll RtlMoveMemory
0000C450  ntdll.dll   0255  RtlGetLastWin32Error//改为：kernel32.dll GetLastError
0000C4F0  ntdll.dll   0255  RtlGetLastWin32Error//改为：kernel32.dll GetLastError
0000C970  ntdll.dll   0255  RtlGetLastWin32Error//改为：kernel32.dll GetLastError

---
```

另外发现脱壳后的记事本当进行“保存”和“另存为”操作时会非法操作。把以下2处修改一下就正常了。

```
代码:
---

00404FB0     FF15 80CC4000       call dword ptr ds:[40CC80];comdlg32.GetSaveFileNameA
//修改为：jmp dword ptr ds:[40CCA0]
00404FC8     FF15 A0CC4000       call dword ptr ds:[40CCA0];comdlg32.GetOpenFileNameA
//修改为：jmp dword ptr ds:[40CCA0]

---
```

```
代码:
---

                               

         ,     _/ 
        /| _.-~/            \_     ,        青春都一饷
       ( /~   /              \~-._ |\
       `\\  _/                \   ~\&
    _-~~~-.)  __/;;,.          \_  //'
   /'_,\   --~   \ ~~~-  ,;;\___(  (.-~~~-.        换了破解轻狂
  `~ _( ,_..--\ (     ,;'' /    ~--   /._`\ 
   /~~//'   /' `~\        
   "  `~"  "      `"      /~'`\    `\\~~\   
                          "     "   "~'  ""

---
```

    Cracked By 巢水工作坊――fly [OCN][FCG][NUKE][DCM]

            2004-05-09  00:00
