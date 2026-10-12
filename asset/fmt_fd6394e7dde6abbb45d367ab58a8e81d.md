
### 00481A5E - 00481B33 代码段

```
00481A5E  58              pop eax
00481A5F  EB 01           jmp short 00481A62
00481A62  40              inc eax
00481A63  EB 01           jmp short 00481A66
00481A66  FFE0            jmp eax
00481A5C  EB 0B           jmp short 00481A69
00481A69  61              popad
00481A6A  8BA5 4C394200   mov esp,dword ptr ss:[ebp+42394C]
00481A70  9C              pushfd
00481A71  EB 01           jmp short 00481A74
00481A74  9D              popfd
00481A75  EB 01           jmp short 00481A78
00481A78  50              push eax
00481A79  60              pushad
00481A7A  E8 03000000     call 00481A82
00481A82  EB 01           jmp short 00481A85
00481A85  58              pop eax
00481A86  EB 01           jmp short 00481A89
00481A89  40              inc eax
00481A8A  EB 01           jmp short 00481A8D
00481A8D  FFE0            jmp eax
00481A80  EB 0E           jmp short 00481A90
00481A90  61              popad
00481A91  8B85 30394200   mov eax,dword ptr ss:[ebp+423930]
00481A97  9C              pushfd
00481A98  EB 01           jmp short 00481A9B
00481A9B  9D              popfd
00481A9C  EB 01           jmp short 00481A9F
00481A9F  8BB5 40394200   mov esi,dword ptr ss:[ebp+423940]
00481AA5  60              pushad
00481AA6  E8 03000000     call 00481AAE
00481AAE  EB 01           jmp short 00481AB1
00481AB1  58              pop eax
00481AB2  EB 01           jmp short 00481AB5
00481AB5  40              inc eax
00481AB6  EB 01           jmp short 00481AB9
00481AB9  FFE0            jmp eax
00481AAC  EB 0E           jmp short 00481ABC
00481ABC  61              popad
00481ABD  8BBD 44394200   mov edi,dword ptr ss:[ebp+423944]
00481AC3  9C              pushfd
00481AC4  EB 01           jmp short 00481AC7
00481AC7  9D              popfd
00481AC8  EB 01           jmp short 00481ACB
00481ACB  8B9D 34394200   mov ebx,dword ptr ss:[ebp+423934]
00481AD1  60              pushad
00481AD2  E8 03000000     call 00481ADA
00481ADA  EB 01           jmp short 00481ADD
00481ADD  58              pop eax
00481ADE  EB 01           jmp short 00481AE1
00481AE1  40              inc eax
00481AE2  EB 01           jmp short 00481AE5
00481AE5  FFE0            jmp eax
00481AD8  EB 0E           jmp short 00481AE8
00481AE8  61              popad
00481AE9  8B8D 38394200   mov ecx,dword ptr ss:[ebp+423938]
00481AEF  EB 01           jmp short 00481AF2
00481AF2  60              pushad
00481AF3  E8 03000000     call 00481AFB
00481AFB  58              pop eax
00481AFC  EB 01           jmp short 00481AFF
00481AFF  40              inc eax
00481B00  EB 01           jmp short 00481B03
00481B03  FFE0            jmp eax
00481AF9  EB 0B           jmp short 00481B06
00481B06  61              popad
00481B07  8B95 3C394200   mov edx,dword ptr ss:[ebp+42393C]
00481B0D  9C              pushfd
00481B0E  EB 01           jmp short 00481B11
00481B11  9D              popfd
00481B12  EB 01           jmp short 00481B15
00481B15  8BAD 48394200   mov ebp,dword ptr ss:[ebp+423948]
00481B1B  60              pushad
00481B1C  E8 03000000     call 00481B24
00481B24  EB 01           jmp short 00481B27
00481B27  58              pop eax
00481B28  EB 01           jmp short 00481B2B
00481B2B  40              inc eax
00481B2C  EB 01           jmp short 00481B2F
00481B2F  FFE0            jmp eax
00481B22  EB 0E           jmp short 00481B32
00481B32  61              popad
00481B33  C3              retn
```

> `====>飞向光明之巅！ 返回至 00425C21`

---

### 00425C21 代码段

```
00425C21  55              push ebp
```

> `====>在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程`

```
00425C22  8BEC            mov ebp,esp
00425C24  6A FF           push -1
00425C26  68 A8CF4200     push 42CFA8
00425C2B  68 289B4200     push 429B28
00425C30  64:A1 00000000  mov eax,dword ptr fs:[0]
00425C36  50              push eax
00425C37  64:8925 00000000 mov dword ptr fs:[0],esp
00425C3E  83EC 58         sub esp,58
00425C41  53              push ebx
00425C42  56              push esi
00425C43  57              push edi
00425C44  8965 E8         mov dword ptr ss:[ebp-18],esp
00425C47  FF15 08C14200   call dword ptr ds:[42C108] ; kernel32.GetVersion
```

---

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `00025C21`，点 `IT AutoSearch`，点“`Get Import`”，有 2 个函数无效。奇怪，不能用“追踪层次 2、3”修复，否则 `PC Guard` 自动退出！如果 `CUT` 掉则修复的程序可以运行，但是退出时会非法操作。呵呵，试着将其改为：`kernel32.dll ExitProcess`。`FixDump`，正常运行！`VC++ 6.0` 编写，`576K`->`612K` 重建 `PE` 后是 `564K`

---

### 简化脱壳步骤

设置 `Ollydbg` 忽略除了“单步中断”之外的所有异常。

```
00483000  FC              cld
```

> `====>进入 OD 后断在这！F9 运行，断下！`

```
004836D8  EB 01           jmp short PCGWIN32.004836DB
```

> `====>第 1 个单步异常`

`SHIFT+F9` 通过异常，不用记下次数，呵呵，直到程序弹出一系列的“入口点预警”就停下！

把那些对话框一一确定之。呵呵，看看偶现在停在什么地方？ :-)

```
0048198F  EB 01           jmp short 00481992
```

> `====>停在这个单步异常的地方`

看看堆栈：

```
0012FF24  0012FFE0   指针到下一个 SEH 记录
0012FF28  0048194B   SE 句柄 //呵呵，这是什么？
```

`0048194B` 就是 `0048198E int1` 异常的堆栈区的第二条地址！！:-) 又快了点，接下来就可以找 `OEP` 了！

---

### 备注

以上的脱壳过程是针对 `PC Guard for Win32 V5.0 DEMO` 版的 `PCGWIN32.EXE` 主程序，而不是那种没有 `KEY` 就无法运行的保护类型的加壳程序。偶也简单测试了一下，以上寻找 `OEP` 的过程同样适用于用这个版本加壳的其他同类程序！当然有些地方是不可能完全一样的，有兴趣的朋友可以自己测试。

这个帖子是偶在 看雪论坛 的第 `1000` 个帖子，谨以此篇简单的脱壳笔记留个简单的纪念。:-)

---

```
        ,     \_/

      /| \_.-~/          \_    ,       青春都一饷
     ( /~   /             ~-.\_ |
     \`\\  \_/                 ~ )             忍把浮名

 _-~~~-.)  )\_//;;,.          \_  //'
   /\_,   --~     ~~~-  ,;;\_\_(  (.-~~~-.         换了破解轻狂
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`
   /~~//'   /~\`~          ) /--..\_, )\_  \`~

   "  \`~"  "      \`"       /~'\`    \`\\~~

                "    "   "~'  ""
```

`Cracked By 巢水工作坊——fly [OCN][FCG]`

`2003-10-26 00:00`

`4dd466d0e1e0cc123afbe504da2462ca`
