
# [原创]也谈多功能时钟日历的爆破及部分算法分析

【破文标题】：也谈多功能时钟日历的爆破及部分算法分析

【破解目标】：多功能时钟日历2.23

【破解作者】：jney2

【破解日期】：2005年3月18日

【作者声明】：破解，我是菜鸟，更要学习，练习！！！一点心得，大家分享。

【破解平台】：XP＋OD＋Dede

【简要说明】：看了gestapo8写的文章《爆破多功能时钟日历V2.5》，他的第一次，第一次能达这样的水平，已经相当不错了。特别是他的方法在某些地方值得我们借鉴。刚好，我手头的有2.23版，也不知什么时候下的，觉得他的思路有些复杂，于是我也玩了一下，便有了下文。虽然没有完全找出算法，但大家共同提高吧！

## 破解过程

一、Peid查壳：ASPack 2.12，用Stripper2.07脱之，Peid再查：Borland Delphi 4.0 - 5.0；

二、用Dede反编译，通过查看“程序”、“窗体”两项，可知`TFormTest2`即为注册窗口，`Button1click`即为注册按钮，双击可看到如下有用的提示信息：

```
0054D294   55                     push    ebp
0054D295   8BEC                   mov     ebp, esp
........略........
* Possible String Reference to: 'SOFTWARE\MGC\RL'
|
0054D327   BA1CD45400             mov     edx, $0054D41C
0054D32C   8B45FC                 mov     eax, [ebp-$04]
........略........
* Possible String Reference to: 'NAME'
|
0054D350   BA34D45400             mov     edx, $0054D434
0054D355   8B45FC                 mov     eax, [ebp-$04]
........略........
* Possible String Reference to: 'CODE'
|
0054D379   BA44D45400             mov     edx, $0054D444
0054D37E   8B45FC                 mov     eax, [ebp-$04]
0054D39E   E8C95DEBFF             call    0040316C
0054D3A3   C3                     ret
```

三、由此可知，用户的注册名和注册码保存在注册表中，至于注册按钮是灰色不可用，大可不必管它，我们伪造一份注册信息注入注册表中（XP系统）：

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\MGC\RL]
"NAME"="jney2"
"CODE"="8888888888888888888888"
```

四、在OD中，搜索：字符串参考，我们再通过字符串`'SOFTWARE\MGC\RL'`找到程序读取注册表中`'NAME'`和`'CODE'`的地方（也可用W32Dasm反汇编再搜索，很容易找到，）：

```
0055F3E1   .  BA 90F55500   MOV EDX,_CCC.0055F590                    ;  ASCII "SOFTWARE\MGC\RL"
0055F3E6   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0055F3E9   .  E8 2AA5F4FF   CALL _CCC.004A9918
0055F3EE   .  BA A8F55500   MOV EDX,_CCC.0055F5A8                    ;  ASCII "NAME"
0055F3F3   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0055F3F6   .  E8 9DA8F4FF   CALL _CCC.004A9C98
0055F3FB   .  84C0          TEST AL,AL
0055F3FD   .  75 07         JNZ SHORT _CCC.0055F406                  //读取用户名成功则继续
0055F3FF   .  E8 A045EAFF   CALL _CCC.004039A4
0055F404   .  EB 77         JMP SHORT _CCC.0055F47D                  //跳走，OVER
0055F406   >  BA B8F55500   MOV EDX,_CCC.0055F5B8                    ;  ASCII "CODE"
0055F40B   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0055F40E   .  E8 85A8F4FF   CALL _CCC.004A9C98
0055F413   .  84C0          TEST AL,AL
0055F415   .  75 07         JNZ SHORT _CCC.0055F41E                  //读取注册码成功则继续
0055F417   .  E8 8845EAFF   CALL _CCC.004039A4
0055F41C   .  EB 5F         JMP SHORT _CCC.0055F47D                  //跳走，OVER
0055F41E   >  8D4D E0       LEA ECX,DWORD PTR SS:[EBP-20]
0055F421   .  BA A8F55500   MOV EDX,_CCC.0055F5A8                    ;  ASCII "NAME"
0055F426   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0055F429   .  E8 D2A6F4FF   CALL _CCC.004A9B00
0055F42E   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]
0055F431   .  A1 6CC25600   MOV EAX,DWORD PTR DS:[56C26C]          //将用户名指针保存在56C26C处
0055F436   .  E8 694AEAFF   CALL _CCC.00403EA4
0055F43B   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]
0055F43E   .  BA B8F55500   MOV EDX,_CCC.0055F5B8                    ;  ASCII "CODE"
0055F443   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0055F446   .  E8 B5A6F4FF   CALL _CCC.004A9B00
0055F44B   .  8B55 DC       MOV EDX,DWORD PTR SS:[EBP-24]
0055F44E   .  A1 9CBE5600   MOV EAX,DWORD PTR DS:[56BE9C]           //将注册码指针保存在56BE9C处
0055F453   .  E8 4C4AEAFF   CALL _CCC.00403EA4
0055F458   .  33C0          XOR EAX,EAX
0055F45A   .  5A            POP EDX
0055F45B   .  59            POP ECX
0055F45C   .  59            POP ECX
```

五、程序只是把用户名和注册码从注册表中读出，简单地变换一下保存起来，并没有急于判断注册码的真假。不管怎么样，程序总是要判断真假的对吧，那好我们在数据转存中对`56C26C`和`56BE9C`下`DWORD`硬件访问断点（下次运行时，断点还在）。F9，运行。

六、经过一次异常后，程序断在`00550AAF`：

```
00550A93   .  C705 C4ED5600>MOV DWORD PTR DS:[56EDC4],7
00550A9D   .  C705 C0ED5600>MOV DWORD PTR DS:[56EDC0],7
00550AA7   .  8B15 9CBE5600 MOV EDX,DWORD PTR DS:[56BE9C]            ;  _CCC.0056D920
00550AAD   .  8B12          MOV EDX,DWORD PTR DS:[EDX]
00550AAF   .  A1 6CC25600   MOV EAX,DWORD PTR DS:[56C26C]            //断在这里，寄存器 EDX 00F0DEF0 ASCII "8888888888888888888888"
00550AB4   .  8B00          MOV EAX,DWORD PTR DS:[EAX]
00550AB6   .  E8 EDB5F5FF   CALL _CCC.004AC0A8                      //关键CALL，F7跟进
00550ABB   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00550ABE   .  E8 D9DCFFFF   CALL _CCC.0054E79C
00550AC3   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00550AC6   .  E8 91DAFFFF   CALL _CCC.0054E55C
00550ACB   .  33C0          XOR EAX,EAX
```

七：下面通过OD的动态调试可得到：

```
004AC0A8   $  55            PUSH EBP
004AC0A9   .  8BEC          MOV EBP,ESP
004AC0AB   .  B9 0C000000   MOV ECX,0C
004AC0B0   >  6A 00         PUSH 0
004AC0B2   .  6A 00         PUSH 0
004AC0B4   .  49            DEC ECX
004AC0B5   .^ 75 F9         JNZ SHORT _CCC.004AC0B0
004AC0B7   .  53            PUSH EBX
004AC0B8   .  56            PUSH ESI
004AC0B9   .  57            PUSH EDI
004AC0BA   .  8955 F8       MOV DWORD PTR SS:[EBP-8],EDX
004AC0BD   .  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
004AC0C0   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004AC0C3   .  E8 CC81F5FF   CALL _CCC.00404294
004AC0C8   .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004AC0CB   .  E8 C481F5FF   CALL _CCC.00404294
004AC0D0   .  33C0          XOR EAX,EAX
004AC0D2   .  55            PUSH EBP
004AC0D3   .  68 52C44A00   PUSH _CCC.004AC452
004AC0D8   .  64:FF30       PUSH DWORD PTR FS:[EAX]
004AC0DB   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
004AC0DE   .  C745 F4 D4070>MOV DWORD PTR SS:[EBP-C],7D4
004AC0E5   .  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004AC0E8   .  BA 6CC44A00   MOV EDX,_CCC.004AC46C                    ;  ASCII "        "
004AC0ED   .  E8 F67DF5FF   CALL _CCC.00403EE8
004AC0F2   .  837D FC 00    CMP DWORD PTR SS:[EBP-4],0               //判断用户名是否为空
004AC0F6   .  0F84 26030000 JE _CCC.004AC422                         //跳走则OVER
004AC0FC   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004AC0FF   .  E8 DC7FF5FF   CALL _CCC.004040E0                       //取用户名长度
004AC104   .  83F8 08       CMP EAX,8
004AC107   .  7D 3B         JGE SHORT _CCC.004AC144                  //大于等于8则跳走
004AC109   .  8D45 D4       LEA EAX,DWORD PTR SS:[EBP-2C]
004AC10C   .  50            PUSH EAX
004AC10D   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004AC110   .  E8 CB7FF5FF   CALL _CCC.004040E0
004AC115   .  B9 08000000   MOV ECX,8
004AC11A   .  2BC8          SUB ECX,EAX
004AC11C   .  BA 01000000   MOV EDX,1
004AC121   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
004AC124   .  E8 BF81F5FF   CALL _CCC.004042E8
004AC129   .  8B45 D4       MOV EAX,DWORD PTR SS:[EBP-2C]
004AC12C   .  50            PUSH EAX
004AC12D   .  8D55 D0       LEA EDX,DWORD PTR SS:[EBP-30]
004AC130   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004AC133   .  E8 C8DAF5FF   CALL _CCC.00409C00
004AC138   .  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]
004AC13B   .  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004AC13E   .  59            POP ECX
004AC13F   .  E8 E87FF5FF   CALL _CCC.0040412C                      //用户名长度小于8，则在用户名后面补空格到8位
004AC144   >  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]           //都来这里
004AC147   .  50            PUSH EAX
004AC148   .  B9 01000000   MOV ECX,1
004AC14D   .  BA 01000000   MOV EDX,1
004AC152   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004AC155   .  E8 8E81F5FF   CALL _CCC.004042E8
004AC15A   .  8B45 CC       MOV EAX,DWORD PTR SS:[EBP-34]
004AC15D   .  E8 76F9FFFF   CALL _CCC.004ABAD8
004AC162   .  8BF0          MOV ESI,EAX
004AC164   .  81E6 FF000000 AND ESI,0FF                             //保存用户名第1位（16进制，只取后7位）
004AC16A   .  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]
004AC16D   .  50            PUSH EAX
004AC16E   .  B9 01000000   MOV ECX,1
004AC173   .  BA 02000000   MOV EDX,2
004AC178   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
```

