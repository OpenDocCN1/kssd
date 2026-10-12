
# ESP 定律

> 原文：[`www.pediy.com/kssd/pediy06/pediy6649.htm`](https://www.pediy.com/kssd/pediy06/pediy6649.htm)

定律 + 壹锅端——另类 `ACProtect` V1.20 壳的简便脱壳 And 修复方法

下载地址： `http://www.ultraprotect.com/acpr_pro.exe`

软件大小： 1.6M

软件语言： 英文

应用平台： `Win9x/NT/2000/XP`

【软件简介】：`ACProtect` is an application that allows you to protect Windows executable files against piracy,using public keys encryption algorithms (`RSA`) to create and verify the registration keys and unlock some `RSA` key locked code,it has embedded cryptor against dump and unpacker.it also has many anti debug tricks. And you can use it to create evaluation and trial application versions. with specialized `API` system, mutual communication between loader and application is also can be achieved.

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`flyODBG`、`PEiD`、`LordPE`、`ImportREC`

—————————————————————————————————

【脱壳过程】：

`ACProtect` 现在的最新版本是 `V1.20 [2004-02-08]` 专业版，升级很快。发觉 `ACProtect` 对 `Delphi` 的程序支持较好，以后 `Delphi` 的软件会有很多用 `ACProtect` 加壳的。这次的试炼品是 `ACProtect` 自带的 `ACP_Feedback.exe`， `Delphi` 的东东。

具体避开反跟踪请参考以前的相关笔记。或者用这个修改版就可以正常调试了。

`flyODBG` 1.10 汉化修改版

好几天没写东西了，容易的大家都写了，难的又脱不了，够无奈的。滚滚长江东逝水，浪花淘尽多少 `Cracker`？如果还不努力学习，我这菜鸟就更要被淘汰了。

—————————————————————————————————

## 一、ESP 定律：快速寻找 伪 OEP

老规矩：用 `IsDebug` 1.4 插件去掉 `Ollydbg` 的调试器标志。忽略除了“`int3` 异常”之外的所有其他异常选项。

何谓 伪 `OEP` ？这里的伪是假、伪装的意思，因为有 `Stolen Code` 作怪，所以从壳中返回程序的地方并不是真正的 `OEP`。

何谓 `ESP` 定律？这是胡诌的，也就是说这个词是我信口开河的，各位方家不必较真呀。 `:-)` 用这个 `ESP` 定律能快速找到大部分 `ACProtect` 1.20 加壳程序的伪 `OEP`！下个版本作者应该会想办法对付这个方法的，呵呵。`ACProtect` 会越来越强大的！

觉得有不少壳在壳入口和进入 `OEP` 时的 `ESP` 值是很有规律的。其实 `VolX` 老兄在很早的时候脱 `Videofixer` 3.23 英文版时就曾经妙用过；`exetool` 上也有人说过。因为单步或者 `TC` 跟踪 `ACProtect` 都很浪费时间，所以无聊的时候又看了几遍 `ACProtect`，找到了一个稍微快点的方法。这个方法比我以前的方法要好很多，对于其他的壳或许也有借鉴的价值，权衡几番还是发了出来。在此也希望有些兄弟能够谅解！

```
0040A000      60                     pushad//进入 OD 后停在这
0040A001      F9                     stc//ACProtect V1.20 以前的版本则利用入口 pushad 后的 ESP 值！
0040A002      D3FD                  sar ebp,cl
0040A004      0F80 01000000          jo Feedback.0040A00B
0040A00A      FC                     cld
0040A00B      FC                     cld
0040A00C      49                     dec ecx
0040A00D      C1DB FE                rcr ebx,0FE
0040A010      85CF                  test edi,ecx
0040A012      72 03                 jb short Feedback.0040A017
0040A014      73 01                 jnb short Feedback.0040A017
0040A016      7A 81                 jpe short Feedback.00409F99//ACProtect V1.20 典型入口代码
```

`F9` 运行，确定弹出的未注册提示。程序会中断在 `int3` 异常处。

```
00418A7A      CC                     int3//ACProtect V1.20 典型异常
00418A7B      90                     nop
00418A7C      64:67:8F06 0000        pop dword ptr fs:[0]
00418A82      83C4 04                add esp,4
00418A85      60                     pushad
00418A86      E8 00000000           call Feedback.00418A8B
00418A8B      5E                     pop esi
00418A8C      83EE 06                sub esi,6
00418A8F      B9 5B000000            mov ecx,5B
00418A94      29CE                  sub esi,ecx
00418A96      BA 5A630BF7            mov edx,F70B635A
00418A9B      C1E9 02                shr ecx,2
00418A9E      83E9 02                sub ecx,2
00418AA1      83F9 00                cmp ecx,0
00418AA4      7C 1A                  jl short Feedback.00418AC0
00418AA6      8B048E                 mov eax,dword ptr ds:[esi+ecx*4]
00418AA9      8B5C8E 04              mov ebx,dword ptr ds:[esi+ecx*4+4]
00418AAD      33C3                  xor eax,ebx
00418AAF      C1C8 1B                ror eax,1B
00418AB2      03C2                  add eax,edx
00418AB4      81EA 3403B581          sub edx,81B50334
00418ABA      89048E                 mov dword ptr ds:[esi+ecx*4],eax
00418ABD      49                     dec ecx
00418ABE      EB E1                  jmp short Feedback.00418AA1
00418AC0      61                     popad
00418AC1      61                     popad
00418AC2      C3                     retn//直接在此处下断，Shift+F9 断在这。F7 走 1 步
0041BF97      66:C1D9 5E            rcr cx,5E//返回这里。此时 ESP=0012FF84 ★
0041BF9B      66:8BD1              mov dx,cx
0041BF9E      81F7 6B108A77          xor edi,778A106B
0041BFA4      50                     push eax
0041BFA5      E8 01000000           call Feedback.0041BFAB
```

`OK`，现在 `ESP` 定律 上场！`8)` 去转存窗口，`Ctrl+G:0012FF84` 到达 `0012FF84` 内存处

`0012FF84` `00 00 00 00` ....

取消以前的断点。在 `0012FF84` 处的 4 个字节上下 硬件访问－>`Word` 断点 `F9` 运行,断下！

注意:`ACProtect V1.20[2004-02-08]`以前的版本，则有的需要利用在入口处 `pushad` 后的 `ESP` 值！

```
00427B4F      8F05 58AC4000         pop dword ptr ds:[40AC58]//中断第 1 次。可以取消断点了
00427B55      FF35 58AC4000          push dword ptr ds:[40AC58]
00427B5B      8915 40AC4000          mov dword ptr ds:[40AC40],edx
00427B61      FF35 40AC4000          push dword ptr ds:[40AC40]
00427B67      8F05 3CAC4000          pop dword ptr ds:[40AC3C]
00427B6D      8B35 3CAC4000          mov esi,dword ptr ds:[40AC3C]
00427B73      90                     nop
00427B74      90                     nop
00427B75      90                     nop
00427B76      90                     nop
00427B77      60                     pushad
00427B78      E8 5FF6FEFF            call Feedback.004171DC
00427B7D      8B85 6EEC4100          mov eax,dword ptr ss:[ebp+41EC6E]
00427B83      0385 26D04000          add eax,dword ptr ss:[ebp+40D026]
                                  //EAX=00002168 + 00400000=00402168 这就是伪 OEP 值 :-)
00427B89      8985 6EEC4100          mov dword ptr ss:[ebp+41EC6E],eax
                                  //[ebp+41EC6E]=[00427C6E] 这个就是 飞天一跃 的地方
00427B8F      61                     popad
00427B90      8935 38AC4000          mov dword ptr ds:[40AC38],esi
00427B96      FF35 38AC4000          push dword ptr ds:[40AC38]
00427B9C      8B0424                 mov eax,dword ptr ss:[esp]
00427B9F      8F05 34AC4000          pop dword ptr ds:[40AC34]
00427BA5      8F05 54AC4000          pop dword ptr ds:[40AC54]
00427BAB      FF35 54AC4000          push dword ptr ds:[40AC54]
00427BB1      5E                     pop esi
00427BB2      8F05 74AC4000          pop dword ptr ds:[40AC74]
00427BB8      60                     pushad
00427BB9      E8 1EF6FEFF            call Feedback.004171DC
00427BBE      C685 2BEC4100 E8        mov byte ptr ss:[ebp+41EC2B],0E8
00427BC5      61                     popad
00427BC6      8905 30AC4000          mov dword ptr ds:[40AC30],eax
00427BCC      FF35 30AC4000          push dword ptr ds:[40AC30]
00427BD2      68 74AC4000            push Feedback.0040AC74
00427BD7      58                     pop eax
00427BD8      8B10                 mov edx,dword ptr ds:[eax]
00427BDA      8F05 2CAC4000          pop dword ptr ds:[40AC2C]
00427BE0      8B05 2CAC4000          mov eax,dword ptr ds:[40AC2C] ; Feedback.0040211C
                                  //mov eax,0040211C 有点特别，伪 OEP 处的 EAX 就是这个值 :-)
00427BE6      90                     nop
00427BE7      90                     nop
00427BE8      90                     nop
00427BE9      90                     nop
00427BEA      90                     nop
00427BEB      90                     nop
00427BEC      90                     nop
00427BED      90                     nop
00427BEE      60                     pushad
00427BEF      E8 E8F5FEFF            call Feedback.004171DC
00427BF4      C785 2CEC4100FF25000  mov dword ptr ss:[ebp+41EC2C],25FF
00427BFE      8D85 6EEC4100          lea eax,dword ptr ss:[ebp+41EC6E]
00427C04      8985 2EEC4100          mov dword ptr ss:[ebp+41EC2E],eax
00427C0A      E8 CDF5FEFF            call Feedback.004171DC
00427C0F      8DBD 8CE94100          lea edi,dword ptr ss:[ebp+41E98C]
00427C15      8D8D 20EC4100          lea ecx,dword ptr ss:[ebp+41EC20]
00427C1B      2BCF                  sub ecx,edi
00427C1D      C1E9 02                shr ecx,2
```



00427C20 `E8 55F3FEFF` `call Feedback.00416F7A`

00427C25 `AB` `stos dword ptr es:[edi]` //清理刚才的战场

00427C26 `E2 F8` `loopd short Feedback.00427C20`

00427C28 `61` `popad` //F4 到这里跳出上面的 loopd

00427C29 `EB 01` `jmp short Feedback.00427C2C`

00427C2C `FF25 6E7C4200` `jmp dword ptr ds:[427C6E]` ;00402168 //GO！飞向光明之巅！:-)

00402168 `E8 43FCFFFF` `call Feedback.00401DB0` //伪 OEP 在这儿用 `LordPE` 完全 `DUMP` 进程

0040216D `68 D0214000` `push Feedback.004021D0;` `ASCII "ACProtect Feedback From"`

00402172 `6A 00` `push 0`

00402174 `6A 00` `push 0`

00402176 `E8 F9FCFFFF` `call Feedback.00401E74`

注意：在 `00402168` 伪 `OEP` 处记下寄存器和堆栈情况！ ★

—————————————————————————————————

## 二、修复输入表

既然 `ImportREC` 能够很好的对付 `ACProtect`，我就偷懒了。重新运行 `Feedback.exe`（否则会失去响应的），运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `00002168`，点 `IT AutoSearch`，点“`Get Import`”，用“追踪层次 3”全部修复无效函数（需要点时间）。`FixDump`！

—————————————————————————————————

## 三、仿真 `Stolen Code`，一锅端的方法修复变形代码

想从 `ACProtect 1.2` 中找到未被“改造”的 `Stolen Code` 有点麻烦，索性就根据伪 `OEP` 处的寄存器、堆栈情况以及上面的代码来手动仿真吧。谁有更好的方法请多指教。

在 `00402168` 伪 `OEP` 处 `Ollydbg` 中看到寄存器和堆栈的情况是：

`EAX=0040211C` //这就是在 `00427BE0` 处看到的值

`EBX=7FFDF000`

`EBP=0012FFC0`

`ESP=0012FFAC`

`0012FFAC` `7FFDF000`

`0012FFB0` `00000000`

`0012FFB4` `00000024`

`0012FFB8` `7FFDF000`

`0012FFBC` `7FFDF000`

`0012FFC0` `0012FFF0`

`0012FFC4` `77E5EB69` 返回到 `kernel32.77E5EB69`

————————————————————————

简单看了几种其他的 `Delphi 6.0-7.0` 程序的 `OEP` 处代码：

```
push ebp
mov  ebp,esp
add  esp,-XX
mov  eax,00XXXXXX
push ebp
mov  ebp,esp
add  esp,-XX
xor  eax,eax
mov  dword ptr ss:[ebp-10],eax
mov  eax,00XXXXXX
```

————————————————————————

当然，上面修复输入表后的程序还是无法运行的，`Stolen Code` 和变形代码还没有修复。`ACProtect 1.2` 的变形程度更严重了。这部分让人头疼，现在学习了 `heXer` 和 `jingulong` 两位脱神的一锅端方法居然奇兵突出轻松解决！感谢二位老师的教诲！当然，任何方法都不是万能的，灵活运用才是关键！用这个方法可以搞定一些 `ACProtect 1.2` 加壳的程序，还有某些程序是不行的，生搬硬套是不可取的。

用 `Ollydbg` 载入 `dumped\_.exe`，开始偶们的 一锅端 修复！`Ctrl+G`：`0040A000` 这是原先 `ACProtect` 加壳的入口。

猜、猜、猜…… 呵呵，结合寄存器、堆栈和壳中代码情况，仿真 `Stolen Code` 如下：

0040A000 `55` `push ebp`

0040A001 `8BEC` `mov ebp,esp`

0040A003 `83C4 F0` `add esp,-10`

0040A006 `53` `push ebx`

0040A007 `B8 1C214000` `mov eax,dumped\_.0040211C`

0040A00C `E9 5781FFFF` `jmp dumped\_.00402168` //跳向伪 `OEP`

把这些修改代码 复制入可执行文件，另存为 `修正-dumped\_.exe`。用 `LordPE` 修改其入口点＝`0000A000`

因为利用了原 `perplex` 区段，所以不可以删除这个区段；可以用 `File Scanner` 简单优化一下。

运行一下，如何？OK 了！ 8)

`BTW`:某些程序经过以上处理后可能还会存在无法跨系统平台运行的问题。

—————————————————————————————————

```
      ,     \_/
    /| \_.-~/      \_     ,        青春都一晌
   ( /~   /          ~-.\_ |
   \`\\  \_/             ~ )          忍把浮名
  _-~~~-.)  )\_\_/;;,.    _  //'
 /'\_,   --~    ~~~- ,;;\_\_\_ (.-~~~-.        换了破解轻狂
 \`~ \_( ,\_..-- (     ,;''/    ~--   /.\_\`
  /~~//'  /' \`~       ) /--..\_, )\_  \`~
  "  \`~"  "    \`"      /~'\`    \`\\~~
                          "   "   "~'  ""
```

      Cracked By 巢水工作坊——fly `[OCN][FCG][NUKE][DCM]`

                2004-02-19  03:00

`4dd466d0e1e0cc123afbe504da2462ca`
