
```
00401016     50                   push eax

00401017     E8 20020000          call 试炼ACP.0040123C
```

――――――――――――――――――――――――

重新运行`试炼ACP.exe`，运行`ImportREC`，选择这个进程。把`OEP`改为`00001000`，点`IT AutoSearch`，点“Get Import”，函数无效，用“追踪层次3”全部修复。`FixDump`，正常运行！ `114K ->136K`  用`FileScan`优化后是`115K`。

―――――――――――――――――――――――――――――――――

### 三、关于输入表及其他问题

因为偶用`ImportREC`能够修复输入表，没有出现无法修复的问题，所以只是看了一下`123112`所说的需要`NOP`的地方。

```
0040E319     FF95 68C24100        call dword ptr ss:[ebp+41C268]

0040E31F     3B9D 28404000        cmp ebx,dword ptr ss:[ebp+404028]

0040E325     7C 0F                jl short 试炼ACP.0040E336

0040E327     90                   nop

0040E328     90                   nop

0040E329     90                   nop

0040E32A     90                   nop

0040E32B     60                   pushad

0040E32C     2BC0                 sub eax,eax

0040E32E     8803                 mov byte ptr ds:[ebx],al

0040E330     43                   inc ebx

0040E331     3803                 cmp byte ptr ds:[ebx],al

0040E333     75 F9                jnz short 试炼ACP.0040E32E

0040E335     61                   popad

0040E336     0BC0                 or eax,eax

0040E338     0F84 2EFFFFFF        je 试炼ACP.0040E26C

0040E33E     3B85 78C24100        cmp eax,dword ptr ss:[ebp+41C278]

0040E344     75 0A                jnz short 试炼ACP.0040E350
```

――――――――――――――――

```
0040E296     FF95 74C24100        call dword ptr ss:[ebp+41C274]

0040E29C     60                   pushad

0040E29D     2BC0                 sub eax, eax

0040E29F     8803                 mov byte ptr ds:[ebx], al

0040E2A1     43                   inc ebx

0040E2A2     3803                 cmp byte ptr ds:[ebx], al

0040E2A4     75 F9                jnz short 试炼ACP.0040E29F

0040E2A6     61                   popad

0040E2A7     8985 20404000        mov dword ptr ss:[ebp+404020], eax

0040E2AD     C785 24404000 000000>mov dword ptr ss:[ebp+404024], 0

0040E2B7     8B95 28404000        mov edx, dword ptr ss:[ebp+404028]

0040E2BD     8B06                 mov eax, dword ptr ds:[esi]

0040E2BF     0BC0                 or eax, eax

0040E2C1     75 07                jnz short 试炼ACP.0040E2CA
```

注：这2个地方的地址差值：`0040E32C-0040E29D＝8F` 好像是固定的 :-)

其他问题：偶发现用`ACProtect`相同选项加壳的某些程序入口处的6个字节被替换了，某些程序却又正常。以上只是偶的一点肤浅之见，欢迎朋友们讨论、指正！

―――――――――――――――――――――――――――――――――

         ,     \_/

        /| \_.-~/            \_     ,        青春都一饷

       ( /~   /              ~-.\_ |

       `\  \_/                   ~ )          忍把浮名

   \_-~~~-.)  )\_\_/;;,.          \_  //'

  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂

 `~ \_( ,\_..-- (     ,;'' /    ~--   /.\_`

  /~~//'   /' `~         ) /--..\_, )\_  `~

  "  `~"  "      `"      /~'`    `\~~

                         "     "   "~'  ""

            Cracked By 巢水工作坊――fly [OCN][FCG]

                    2003-11-01 01:10
