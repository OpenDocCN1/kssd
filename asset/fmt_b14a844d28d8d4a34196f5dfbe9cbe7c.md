
# 逆向分析与破解记录

## 1. 定位 OEP (Original Entry Point)

跟踪执行流，计算跳转目标地址：

```
00AA3C74     894424 1C            mov dword ptr ss:[esp+1C],eax
00AA3C78     61                   popad
00AA3C79     FFE0                 jmp eax
```

`EAX` 的值计算如下：`EAX = 000066A0 + 00400000 = 004066A0`。
因此，OEP 值为 `004066A0`。

跳转至新地址：

```
004066A0     /E9 F32D0100         jmp TqCrackM.00419498
```

这里使用了动态跳转技术。

## 2. 动态解码过程

执行至 `00419498`：

```
00419498     9C                   pushfd
00419499     60                   pushad
0041949A     E8 00000000          call TqCrackM.0041949F
0041949F     5D                   pop ebp
004194A0     8DB5 32010000        lea esi,dword ptr ss:[ebp+132]
004194A6     8B5C24 24            mov ebx,dword ptr ss:[esp+24]
004194AA     81E3 0000E0FF        and ebx,FFE00000
004194B0     8DBD 2D020000        lea edi,dword ptr ss:[ebp+22D]
004194B6     E8 D6000000          call TqCrackM.00419591
004194BB     8D4D 2B              lea ecx,dword ptr ss:[ebp+2B]
004194BE     51                   push ecx
004194BF     8D5D 4E              lea ebx,dword ptr ss:[ebp+4E]
004194C2     89DE                 mov esi,ebx
004194C4     E8 C8000000          call TqCrackM.00419591
004194C9     C3                   ret
```

`ret` 指令返回至 `004066A0`。这是典型的动态解码过程，此时内存中数据已正确还原。

## 3. DUMP 与修复

在 `004066A0` 处：

```
004066A0     55                   push ebp
```

在此处使用 `LordPE` 完全 DUMP 当前进程。

**修复步骤：**

1. 重新运行被保护程序。
2. 运行 `ImportREC`，选择当前进程。
3. 将 OEP 改为 `000066A0`。
4. 点击 `IT AutoSearch`。
5. 修改 RAV (Rebase Address Value) 为 `0001E000`。
6. 修改大小 (Size) 为 `00001000`。
7. 点击 `Get Import`。
8. 使用插件手动修复无效函数，删除多余项。
9. 点击 `FixDump`。
10. 使用 `LordPE` 重建修复后的程序。

文件大小变化：从 `104K` DUMP 后变为 `208K`，重建修复后变为 `188K`，可正常运行。

## 4. 注册码验证逻辑

作者描述了高难度的保护机制：“杀手级：断点设置，简单加密，复杂算法，耐心灵感”。涉及数千次的循环。

关键验证位置（逐位比较）：

```
00404DA5     DC5D D8              fcomp qword ptr ss:[ebp-28] //比较 ①
00404EB2     DC5D D8              fcomp qword ptr ss:[ebp-28] //比较 ②
004050CC     DC5D D8              fcomp qword ptr ss:[ebp-28] //比较 ③
00404FBF     DC5D D8              fcomp qword ptr ss:[ebp-28] //比较 ④
```

**有效注册信息：**

- 用户名：`fly[FCG]`
- 注册码：`fhlr-defj-bacb-a998`

---

## 5. 结语

```
         ,     \_/
       /| \_.-~/          \_     ,        青春都一饷
      ( /~   /             ~-.\_ |
      \`\\  \_/                   ~ )          忍把浮名
   _-~~~-.)  )\_\_/;;,.          \_  //'
  /\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂
 \`~ \_( ,\_..-- (      ,;'' /    ~--   /.\_\`
  /~~//'   / \`~          ) /--..\_, )\_  \`~
  "  \`~"  "       \`"       /~'\`    \`\\~~
                     "     "   "~'  ""

            Cracked By 巢水工作坊——fly [OCN][FCG]

                  2003-09-27  21:50
```

`4dd466d0e1e0cc123afbe504da2462ca`
