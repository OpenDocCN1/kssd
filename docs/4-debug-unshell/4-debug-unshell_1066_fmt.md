
# AntiyPorts V1.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6993.htm`](https://www.pediy.com/kssd/pediy06/pediy6993.htm)

`tElock 0.98b1` 脱壳——`AntiyPorts V1.2`

下载地址：`http://www.antiy.com/service/freetools/antiyport.zip`

软件大小：`90 KB`

**【软件简介】：** `Antiy Port` 是一个在 `NT/2000/XP` 下察看，端口和进程对应关系的免费工具，其 `1.0` 版已经在去年发布，并作为功能模块嵌入到了专业级别安全工具 `Antiy Ghostbusters` 中，`1.02` 增加了对用户身份的判别，当用户以非管理员身份执行该软件时，会有相应错误提示。

**【作者声明】：** 初学 `Crack`，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】：** `WinXP`、`Ollydbg1.09`、`PEiD`、`LordPE`、`WinHex`、`PEditor`

—————————————————————————————————

**【脱壳过程】：**

## 一、部分脱壳，得到 IAT

```
00439BD6     E9 25E4FFFF           jmp antiypor.00438000
             ==== > 进入 OD 后断在这！F9 运行，程序会在异常处中断。

004380A7     F7F3                  div ebx
             ==== > 第 1 次异常

Shift+F9 通过异常，4 次程序运行。Try Again，按 4-2=2 次 Shift+F9，停下来。

00438AA1     66:F7F3               div bx
             ==== > 第 3 异常

这时 CTR+F 在“整个区段”查找命令：AND DWORD PTR [ESI+0C],00

00439210     8B95 62D34000         mov edx,dword ptr ss:[ebp+40D362]
00439216     8BB5 52D34000         mov esi,dword ptr ss:[ebp+40D352]
0043921C     85F6                  test esi,esi
             ==== > F2 此处下断！ESI=输入表的 RVA

0043921E     0F84 06040000         je antiypor.0043962A
00439224     03F2                  add esi,edx
00439226     83A5 52D44000 00      and dword ptr ss:[ebp+40D452],0
0043922D     8B46 0C               mov eax,dword ptr ds:[esi+C]
00439230     8366 0C 00            and dword ptr ds:[esi+C],0
             ==== > 找到这里！

00439234     85C0                  test eax,eax
00439236     0F84 EE030000         je antiypor.0043962A
0043923C     03C2                  add eax,edx
0043923E     8BD8                  mov ebx,eax
00439240     50                    push eax
00439241     FF95 D0D24000         call dword ptr ss:[ebp+40D2D0]
```

在 `0043921C` 下断，按 `Shift+F9` 断了下来，看看 `esi` 的值：`00006078`，这就是 `IAT` 的位置了，然后 `D 00406078`，看见 `IAT`，大小 `00406A8C-00406078=A14`。这时可以用 `LordPE` 部分脱壳。位置：`00406078`，大小：`A14`。存为：`部分 dumped.dmp`

—————————————————————————————————

## 二、另种方法找到 0043921C 处

`OK`，通过学习《加密与解密②版》`P421` 偶知道了还有一种方法可以快速定位此处，不用查找 `AND DWORD PTR [ESI+0C],00` 命令。呵呵，能多学点就多学点吧。

用 `Ollydbg` 载入程序后下 `BP GetModuleHandleA` `Shift+F9` 运行，3 次进入 `kernel32` 领空，停下：

```
77E59F93     837C24 04 00          cmp dword ptr ss:[esp+4],0
             ==== > 断在这里！

77E59F98     0F84 23060000         je kernel32.77E5A5C1
77E59F9E     FF7424 04             push dword ptr ss:[esp+4]
77E59FA2     E8 55080000           call kernel32.77E5A7FC
77E59FA7     85C0                  test eax,eax
77E59FA9     74 08                 je short kernel32.77E59FB3
77E59FAB     FF70 04               push dword ptr ds:[eax+4]
77E59FAE     E8 B0060000           call kernel32.GetModuleHandleW
77E59FB3     C2 0400               retn 4
             ==== > CTRL+F9 返回至 00439247

00439210     8B95 62D34000         mov edx,dword ptr ss:[ebp+40D362]
00439216     8BB5 52D34000         mov esi,dword ptr ss:[ebp+40D352]
0043921C     85F6                  test esi,esi
0043921E     0F84 06040000         je antiypor.0043962A
00439224     03F2                  add esi,edx
00439226     83A5 52D44000 00      and dword ptr ss:[ebp+40D452],0
0043922D     8B46 0C               mov eax,dword ptr ds:[esi+C]
00439230     8366 0C 00            and dword ptr ds:[esi+C],0
00439234     85C0                  test eax,eax
00439236     0F84 EE030000         je antiypor.0043962A
0043923C     03C2                  add eax,edx
0043923E     8BD8                  mov ebx,eax
00439240     50                    push eax
00439241     FF95 D0D24000         call dword ptr ss:[ebp+40D2D0]
             ==== > 原来此处就是 GetModuleHandleA

00439247     85C0                  test eax,eax
             ==== > 返回到这里。看看上面正是 0043921C
```

—————————————————————————————————

## 三、Shift+F9 再来一次，手动寻找 OEP 啦！

```
004396F1     8DC0                  lea eax,eax
             ==== > 第 4 次异常 看看堆栈区的第二条地址是 004396FF

004396FF     8B6424 08             mov esp,dword ptr ss:[esp+8]
             ==== > 堆栈区的第二条地址 设断 Shift+F9 断在此处

00439703     33C0                  xor eax,eax
00439705     FF6424 08             jmp dword ptr ss:[esp+8]
00439714     64:8F00               pop dword ptr fs:[eax]
00439717     58                    pop eax
00439718     EB 02                 jmp short antiypor.0043971C
0043971C     58                    pop eax
0043971D     5D                    pop ebp
0043971E     F8                    clc
0043971F     73 02                 jnb short antiypor.00439723
00439723     2BC6                  sub eax,esi
00439725     40                    inc eax
00439726     E8 83000000           call antiypor.004397AE

004397AE     /EB 01                jmp short antiypor.004397B1
004397B1     33C5                  xor eax,ebp
004397B3     F9                    stc
004397B4     60                    pushad
004397B5     E8 06000000          call antiypor.004397C0

004397C0     33D2                  xor edx,edx
004397C2     64:FF32               push dword ptr fs:[edx]
004397C5     64:8922               mov dword ptr fs:[edx],esp
004397C8     F1                    int1
004397C9     FF02                  inc dword ptr ds:[edx]
             ==== > 异常！注意这里看看堆栈区的第二条地址！

004397BA     8B6424 08             mov esp,dword ptr ss:[esp+8]
             ==== > 堆栈区的第二条地址 设断 Shift+F9 断在此处

004397BE     EB 0D                 jmp short antiypor.004397CD
004397CD     0BE4                  or esp,esp
004397CF     75 01                 jnz short antiypor.004397D2
004397D2     83E8 94               sub eax,-6C
004397D5     2BDB                  sub ebx,ebx
004397D7     64:8F03               pop dword ptr fs:[ebx]
004397DA     5B                    pop ebx
004397DB     0BE4                  or esp,esp
004397DD     75 01                 jnz short antiypor.004397E0
004397E0     98                    cwde
004397E1     60                    pushad
004397E2     E8 06000000          call antiypor.004397ED

004397ED     64:67:FF36 0000       push dword ptr fs:[0]
004397F3     64:67:8926 0000       mov dword ptr fs:[0],esp
004397F9     9C                    pushfd
004397FA     810C24 00010000       or dword ptr ss:[esp],100
00439801     9D                    popfd
00439802     F8                    clc
             ==== > 异常！注意这里看看堆栈区的第二条地址！

004397E7     8B6424 08             mov esp,dword ptr ss:[esp+8]
             ==== > 堆栈区的第二条地址 设断 Shift+F9 断在此处

004397EB     EB 1A                 jmp short antiypor.00439807
00439807     64:67:8F06 0000       pop dword ptr fs:[0]
0043980D     58                    pop eax
0043980E     61                    popad
0043980F     EB 02                 jmp short antiypor.00439813

00439813     F5                    cmc
00439814     0BC1                  or eax,ecx
00439816     E8 00000000          call antiypor.0043981B

0043981B     0BE4                  or esp,esp
0043981D     75 01                 jnz short antiypor.00439820
00439820     40                    inc eax
00439821     2BC7                  sub eax,edi
00439823     8B3424                mov esi,dword ptr ss:[esp]
00439826     58                    pop eax
00439827     81EE 8A144100         sub esi,antiypor.0041148A
0043982D     EB 01                 jmp short antiypor.00439830

00439830     2BC5                  sub eax,ebp
00439832     F9                    stc
```



#### 00439833 E8 08000000 call antiypor.00439840

```
00439840     98                   cwde
00439841     33C0                 xor eax,eax
00439843     C3                   retn
```

> `====>` 返回到 `00439838`，其实是变形的 `JMP`

```
00439838     E9 0C000000          jmp antiypor.00439849
00439849     1BC2                 sbb eax,edx
0043984B     B9 F67BB059          mov ecx,59B07BF6
00439850     81F1 6C68F159        xor ecx,59F1686C
00439856     F9                   stc
00439857     72 01                jb short antiypor.0043985A
0043985A     83F0 A0              xor eax,FFFFFFA0
0043985D     03CE                 add ecx,esi
0043985F     33D2                 xor edx,edx
00439861     81F2 6BC3FC21        xor edx,21FCC36B
00439867     81C2 B23C03DE        add edx,DE033CB2
0043986D     EB 02                jmp short antiypor.00439871
00439871     F9                   stc
00439872     0BC2                 or eax,edx
00439874     33FF                 xor edi,edi
00439876     81F7 023B36C2        xor edi,C2363B02
0043987C     F8                   clc
0043987D     73 02                jnb short antiypor.00439881
00439881     F8                   clc
00439882     33C4                 xor eax,esp
00439884     6BFF 3B              imul edi,edi,3B
00439887     3139                 xor dword ptr ds:[ecx],edi
00439889     D1C7                 rol edi,1
0043988B     F9                   stc
0043988C     83D7 4F              adc edi,4F
0043988F     81C7 D4D0F3E0        add edi,E0F3D0D4
00439895     0BE4                 or esp,esp
00439897     75 01                jnz short antiypor.0043989A
0043989A     98                   cwde
0043989B     8BC6                 mov eax,esi
0043989D     4A                   dec edx
0043989E     0BE4                 or esp,esp
004398A0     75 01                jnz short antiypor.004398A3
004398A3     1D 96E4C132          sbb eax,32C1E496
004398A8     B8 04000000          mov eax,4
004398AD     03C8                 add ecx,eax
004398AF     EB 01                jmp short antiypor.004398B2
004398B2     1BC5                 sbb eax,ebp
004398B4     FC                   cld
004398B5     F8                   clc
004398B6     73 02                jnb short antiypor.004398BA
004398BA     2BC0                 sub eax,eax
004398BC     48                   dec eax
004398BD     03C2                 add eax,edx
004398BF   ^ 79 C3                jns short antiypor.00439884
```

> `====>` `F4` 下去跳出循环！

```
004398C1    /EB 01                jmp short antiypor.004398C4
004398C4     13C5                 adc eax,ebp
004398C6     61                   popad
004398C7     F9                   stc
004398C8     72 01                jb short antiypor.004398CB
004398CB     F5                   cmc
004398CC     C3                   retn
```

> `====>` 返回到 `0043972B`，呵呵，离胜利不远啦

```
0043972B     8B9D 82D34000        mov ebx,dword ptr ss:[ebp+40D382]
00439731     33F6                 xor esi,esi
00439733     F7D3                 not ebx
00439735     0BF3                 or esi,ebx
00439737     75 08                jnz short antiypor.00439741
00439741     039D 62D34000        add ebx,dword ptr ss:[ebp+40D362]
```

> `====>` `EBX=00003C16 + 00400000=00403C16`，这就是 `OEP` 值

![icon_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)

```
00439747     895C24 F0            mov dword ptr ss:[esp-10],ebx
0043974B     8DBD 84D24000        lea edi,dword ptr ss:[ebp+40D284]
00439751     33C0                 xor eax,eax
00439753     B9 9E030000          mov ecx,39E
00439758     F3:AA                rep stos byte ptr es:[edi]
0043975A     8DBD A2B64000        lea edi,dword ptr ss:[ebp+40B6A2]
00439760     B9 58170000          mov ecx,1758
00439765     F3:AA                rep stos byte ptr es:[edi]
00439767     66:AB                stos word ptr es:[edi]
00439769     8DBD A2B64000        lea edi,dword ptr ss:[ebp+40B6A2]
0043976F     85F6                 test esi,esi
00439771     75 08                jnz short antiypor.0043977B
0043977B     C607 E9              mov byte ptr ds:[edi],0E9
0043977E     47                   inc edi
0043977F     2BDF                 sub ebx,edi
00439781     83EB 04              sub ebx,4
00439784     891F                 mov dword ptr ds:[edi],ebx
00439786     8DBD FACD4000        lea edi,dword ptr ss:[ebp+40CDFA]
0043978C     B9 2C000000          mov ecx,2C
00439791     F3:AA                rep stos byte ptr es:[edi]
00439793     66:AB                stos word ptr es:[edi]
00439795     EB 02                jmp short antiypor.00439799
00439799     61                   popad
0043979A     FF6424 D0            jmp dword ptr ss:[esp-30]
```

> `====>` 飞向光明之巅！ 跳至 `00403C16`

---

```
00403C16       55                   db 55
```

> `====>` 在这儿用 `LordPE` 完全 `DUMP` 这个进程

```
00403C17       8B                   db 8B
00403C18       EC                   db EC
00403C19       6A                   db 6A
```

---

### 四、手动修复

1. 用 `WinHex` 把部分 `dumped.dmp` 的代码复制、写入到 `dumped.exe` 的相应位置保存。
2. 再用 `PEditor` 打开 `dumped.exe`，修改入口点为 `00003C16`；用 `dumpfixer` 修正区块。
3. 用 `LordPE` 修正输入表地址为：`00006078`。最后重建 `PE`。`OK`，正常运行！`98.5K->221K`

程序是用 `VC++ 6.0` 编译的，脱壳后可以跨系统平台运行！

---

```
           ,     \_/
         /| \_.-~/          \_     ,        青春都一饷
        ( /~   /             ~-.\_ |
        \`\\   \_/                ~ )          忍把浮名
   _-~~~-.)  )\_\_/;;,.          \_  //'
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂
  `~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`
   /~~//'   /' \`~         ) /--..\_, )\_  \`~
  "  \`~"  "      \`"      /~'\`    \`\\~~
                "      "   "~'  ""
              Cracked By 巢水工作坊——fly [OCN][FCG]
                 2003-10-15  13:50
```

`4dd466d0e1e0cc123afbe504da2462ca`
