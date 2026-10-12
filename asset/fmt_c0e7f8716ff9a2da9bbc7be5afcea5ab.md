
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
