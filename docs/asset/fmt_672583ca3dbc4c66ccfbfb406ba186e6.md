
```
  004026A0  - E9 9FDBFFFF         JMP UnPackMe.00400244
  004026A5    90                 NOP
  004026A6    90                 NOP
  004026A7    8DC0               LEA EAX,EAX                 ; Illegal use of register
   
  00400244    833D 20AD4000 01   CMP DWORD PTR DS:[40AD20],1
  0040024B  - E9 57240000        JMP UnPackMe.004026A7
   
```

我们记下 `00400244` 处的代码，然后再补回去就好了，但是这里的工作量比较大，所以我就用网上的一个脚本来修复了。修复完毕之后，我们用 `LordPE` 抓取镜像（★记得纠正镜像大小），打开 `ImportRec`，填入 OEP：`2598`，在空白处右键，选择高级命令->获取 API 调用，然后剪切掉无效的指针就可以得到一份完整的 API 了，直接修复 dump。下面就要修复 nanomites 了，这个我们可以直接用工具进行修复。修复完毕之后双击脱壳文件，没有反应，⊙⊙b 我们重新载入脱壳文件，单步跟踪发现问题出现在这里：

代码:

```
  0040264F  |.  E8 2D030000     CALL tuowen\_.00402981
   
  00402981   $- E9 AAD9FFFF     JMP tuowen\_.00400330
  00402986      90              NOP
  00402987      90              NOP
   
  00400330    0000             ADD BYTE PTR DS:[EAX],AL    ★这里看来是脚本没有修复好，我们手工修复一下好了
```

这里是我修复的方案，大家可以参考一下：

代码:

```
  00400330    833D C8B34000 0>  CMP DWORD PTR DS:[40B3C8],0
  00400337  - E9 18230000       JMP dumped_1.00402654
```

这里修复完毕之后保存，双击已经可以看到界面了，但是点击 OK 按钮之后会崩溃，下面我们就来修复 SDK 保护吧。O(∩\_∩)O~

用 `OD` 载入，`F9` 跑起来，然后点击 OK 按钮之后，发现造成崩溃的地方在这里。

代码:

```
  00401160   $  55              PUSH EBP    ★我们要修复这里的代码
  00401161   .  8BEC            MOV EBP,ESP
  00401163   .  81EC 60020000   SUB ESP,260
  00401169   .  53              PUSH EBX
  0040116A   .  56              PUSH ESI
  0040116B   .  57              PUSH EDI
  0040116C   .  FF15 8F124200   CALL DWORD PTR DS:[42128F]
  00401172   .  B2 1E           MOV DL,1E
  00401174   .  57              PUSH EDI
  00401175   .  6F              OUTS DX,DWORD PTR ES:[EDI]             ;  I/O command
  00401176   .  DBBF 81D54987   FSTP TBYTE PTR DS:[EDI+8749D581]
  0040117C      C5              DB C5
  0040117D   .  C8 08118B       ENTER 1108,8B
  00401181   .  C3              RETN
```

我们带壳调试源文件，然后看看对应的代码应该是什么样子，然后再补上去就可以恢复那个按钮的功能了。由于这里只有一处，所以比较简单：

代码:

```
  00401160    55               PUSH EBP
  00401161    8BEC             MOV EBP,ESP
  00401163    81EC 60020000    SUB ESP,260
  00401169    53               PUSH EBX
  0040116A    56               PUSH ESI
  0040116B    57               PUSH EDI
  0040116C    FF15 8F124200    CALL DWORD PTR DS:[42128F]
```

此时 `DS:[0042128F]`=`00D30000`，看来源文件是跑到壳临时申请的空间里面去直接代码解码了，我们跟进去之后，把代码记下来，然后找个空白的地方把代码补上即可，下面是我自己补的代码，贴出来仅供大家参考下 O(∩\_∩)O~：

代码:

```
  00401160   $  55             PUSH EBP
  00401161   .  8BEC          MOV EBP,ESP
  00401163   .  81EC 60020000  SUB ESP,260
  00401169   .  53             PUSH EBX
  0040116A   .  56             PUSH ESI
  0040116B   .  57             PUSH EDI
  0040116C   .  90             NOP
  0040116D   .  E8 CC650000    CALL dumped_2.0040773E    ★到空白的地方去解码
   
   
  0040773E    9C               PUSHFD
  0040773F    60               PUSHAD
  00407740    8B4424 24        MOV EAX,DWORD PTR SS:[ESP+24]
  00407744    8B08             MOV ECX,DWORD PTR DS:[EAX]
  00407746    8D78 04          LEA EDI,DWORD PTR DS:[EAX+4]
  00407749    897C24 24        MOV DWORD PTR SS:[ESP+24],EDI
  0040774D    81E9 BC1D576F   SUB ECX,6F571DBC
  00407753    FC               CLD
  00407754    8A07             MOV AL,BYTE PTR DS:[EDI]
  00407756    2C B2            SUB AL,0B2
  00407758    F9               STC
  00407759    32C1             XOR AL,CL
  0040775B    F8               CLC
  0040775C    34 98            XOR AL,98
  0040775E    C0C0 CE          ROL AL,0CE                       ; Shift constant out of range 1..31
  00407761    34 41            XOR AL,41
  00407763    C0C8 06          ROR AL,6
  00407766    2AC1             SUB AL,CL
  00407768    34 EF            XOR AL,0EF
  0040776A    C0C8 9E          ROR AL,9E                        ; Shift constant out of range 1..31
  0040776D    32C1             XOR AL,CL
  0040776F    C0C8 8B          ROR AL,8B                        ; Shift constant out of range 1..31
  00407772    02C1             ADD AL,CL
  00407774    04 F6            ADD AL,0F6
  00407776    04 6B            ADD AL,6B
  00407778    AA               STOS BYTE PTR ES:[EDI]
  00407779    49               DEC ECX
  0040777A  ^ 75 D8            JNZ SHORT dumped_2.00407754
  0040777C    61               POPAD
  0040777D    9D               POPFD
  0040777E    C3               RETN
```

补上这里的代码之后保存，再次打开发现 OK 按钮的功能也正常了，`PEID` 再扫描下，显示“Microsoft Visual C++ 6.0”看来我们已经成功脱壳了( ⊙o⊙ )

--------------------------------------------------------------------------------

【经验总结】

脱这个壳的方法总得来说还是比较传统的，如果能跟踪一下壳的流程，相信能学到不少东西。这篇文章是匆忙写成的，有错误的地方希望大家能指正。另外我这里修复 SDK 的方法还不能够跨平台，如果有哪位大侠有更好的方法的话，希望能够给出给我们学习学习。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 07 月 08 日 11:55:00

`Unpackme.rar`

脱壳文件我在 `Windows XP SP2` 上测试通过，其他平台好像都不可以。![尴尬](img/906f120cfe711ccbc8463fd6eb86eee9.jpg)
