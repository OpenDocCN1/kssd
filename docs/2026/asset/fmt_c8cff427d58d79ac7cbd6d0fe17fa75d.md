
# CopyMinder有Key脱壳——Hard Boot Life Gold V21

↑

下载地址： `b83K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6W2L8#2)9J5k6s2c8Z5k6g2)9J5k6r3!0F1k6g2)9J5k6s2u0W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7$3q4H3M7W2)9J5c8V1S2S2M7X3c8Q4y4h3k6n7L8$3!0@1i4K6g2X3e0r3W2X3k6g2)9#2k6V1N6G2L8r3c8Q4y4h3k6$3x3U0q4Q4x3X3g2\*7K9i4l9`.

软件大小： 3.77M

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：WinXP、OllyDbg、PEiD、LordPE、ImportREC

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## 【脱壳过程】：

![](https://bbs.kanxue.com/upload/attach/200707/4_8a712048099b1108.gif)

CopyMinder is a copy protection system that operates differently from all other systems and achieves what has, up to now, been thought impossible: hassle free, secure, flexible copy protection.CopyMinder requires occasional Internet access to achieve this flexibility but those without Internet access are also accommodated.

此壳用者稀少，壳的知名度也不高。

```
[CopyMinder -> Microcosm.Ltd   * Sign.By.fly]

signature = 83 25 ?? ?? ?? ?? EF 6A 00 E8 ?? ?? ?? ?? E8 ?? ?? ?? ?? CC FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25 ?? ?? ?? ?? FF 25

ep_only = true
```

据csjwaman说可以无key脱壳，解密密钥就在本地，我没有研究下去。

下面演示的是有key可运行的CopyMinder保护程序的脱壳，尽量精简了脱壳流程。

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## 一.关键Api脱壳法

众所周知，一般脱壳中用于壳要处理输入表，`LoadLibraryA`/`GetModuleHandleA`/`GetProcAddress`这三个函数在脱壳中被当作断点使用的很多，但是在某些壳中使用某些Api可能脱起壳来更迅速更方便，这些Api姑且统称为脱壳关键Api。

有很多人问我，你怎么知道这个壳的关键Api是这个函数？

事非所历而不知。大抵前面所说的三个Api可以看作大多压缩壳的脱壳关键Api，但是某些壳的独特的关键Api是我跟踪调试分析得知的。

设置OllyDbg忽略所有异常选项，用IsDebugPresent插件Hide，清除以前的所有断点

```
00517940    8325 14805100 EF    and dword ptr ds:[518014],FFFFFFEF

//进入OllyDbg后暂停在这

00517947    6A 00               push 0

00517949    E8 E5F6FFFF         call 00517033

0051794E    E8 4F000000         call 005179A2 ; <jmp.&KERNEL32.ExitProcess>

00517953    CC                  int3
```

BP `VirtualProtect`

Shift+F9，CopyMinder会联网校验Key，校验通过后中断，取消断点

```
0013FA58   00A134DC   /CALL 到 VirtualProtect 来自 00A134D6

0013FA5C   00401000   |Address = Hard_Boo.00401000

0013FA60   0003EA00   |Size = 3EA00 (256512.)

0013FA64   00000040   |NewProtect = PAGE_EXECUTE_READWRITE

0013FA68   0013FED1   \pOldProtect = 0013FED1
```

BP `VirtualAlloc`

Shift+F9，中断后取消断点，返回调用处

```
0013FA50   00516571   /CALL 到 VirtualAlloc 来自 Hard_Boo.0051656F

0013FA54   00000000   |Address = NULL

0013FA58   00000C38   |Size = C38 (3128.)

0013FA5C   00001000   |AllocationType = MEM_COMMIT

0013FA60   00000040   \Protect = PAGE_EXECUTE_READWRITE
```

```
0051656F    FFD0                call eax

00516571    5A                  pop edx

//返回这里

00516572    8BF8                mov edi,eax

00516574    50                  push eax

00516575    52                  push edx

00516576    8B33                mov esi,dword ptr ds:[ebx]

00516578    8B43 20             mov eax,dword ptr ds:[ebx+20]

0051657B    03C2                add eax,edx

0051657D    8B08                mov ecx,dword ptr ds:[eax]

0051657F    894B 20             mov dword ptr ds:[ebx+20],ecx

00516582    8B43 1C             mov eax,dword ptr ds:[ebx+1C]

00516585    03C2                add eax,edx

00516587    8B08                mov ecx,dword ptr ds:[eax]

00516589    894B 1C             mov dword ptr ds:[ebx+1C],ecx

0051658C    03F2                add esi,edx

0051658E    8B4B 0C             mov ecx,dword ptr ds:[ebx+C]

00516591    03CA                add ecx,edx

00516593    8D43 1C             lea eax,dword ptr ds:[ebx+1C]

00516596    50                  push eax

00516597    57                  push edi

00516598    56                  push esi

00516599    FFD1                call ecx

0051659B    5A                  pop edx

0051659C    58                  pop eax

0051659D    0343 08             add eax,dword ptr ds:[ebx+8]

005165A0    8BF8                mov edi,eax

005165A2    52                  push edx

005165A3    8BF0                mov esi,eax

005165A5    8B46 FC             mov eax,dword ptr ds:[esi-4]

005165A8    83C0 04             add eax,4

005165AB    2BF0                sub esi,eax

005165AD    8956 08             mov dword ptr ds:[esi+8],edx

005165B0    8B4B 10             mov ecx,dword ptr ds:[ebx+10]

005165B3    894E 24             mov dword ptr ds:[esi+24],ecx

005165B6    8B4B 14             mov ecx,dword ptr ds:[ebx+14]

005165B9    51                  push ecx

005165BA    894E 28             mov dword ptr ds:[esi+28],ecx

005165BD    8B4B 0C             mov ecx,dword ptr ds:[ebx+C]

005165C0    894E 14             mov dword ptr ds:[esi+14],ecx

005165C3    FFD7                call edi
//解密程序
```

进入`005165C3` `call edi`，着重看输入表修复

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## 二.完美修复输入表

Ctrl+S搜索命令序列：

```
  mov ecx,dword ptr ds:[esi+34]

  test ecx,ecx
```

找到在`00AB02E1`处，F2设断，F9运行中断
