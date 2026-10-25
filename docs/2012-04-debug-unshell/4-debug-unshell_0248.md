
# Mr.Captor V2.8

> 原文：[`www.pediy.com/kssd/pediy06/pediy6447.htm`](https://www.pediy.com/kssd/pediy06/pediy6447.htm)

`Armadillo` 壳时间问题的解决 And 脱壳——`Mr.Captor V2.8`

下载页面： `http://www.skycn.com/soft/11437.html`

软件大小： `1718 KB`

软件语言： 英文

软件类别： 国外软件 / 共享版 / 图像捕捉

应用平台： `Win9x/NT/2000/XP`

加入时间： `2004-02-15 11:01:12`

下载次数： `2628`

推荐等级： `*****`

【软件简介】：`Mr.Captor` 是一个非常好用的屏幕截图程序!他可以帮助你把看到的任何图片忠实的保留下来!并可以保存为 `BMP`, `PCX`, `GIF`, `JPEG`, `PNG`, `TIFF`, `TGA`, `CUR`, `ICO`, `AVI` 等格式!支持自定义热键!非常容易使用呀!

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg`、`PEiD`、`LordPE`、`ImportREC`

—————————————————————————————————

【脱壳过程】：

我的系统时间是“变幻莫测”的，呵呵，为了测试软件的缘故经常更改系统时间。`Mr.Captor V2.8` 运行后提示：“`Your system clock appears to have been set back, possibly in an attempt to defeat the security system on this program. Please correct your system clock before trying to run this program again. If your clock is correct, please contact the author of this program for instructions on correcting this error (report code 0,1995,1903)`”。 然后程序退出拒绝运行了。

今天就来解决这个“`System Clock Set Back`”的问题。

`Mr.Captor V2.8` 是 `Armadillo` 标准壳。设置 `Ollydbg` 忽略所有的异常选项。

—————————————————————————————————

## 一、解决“System Clock Set Back”的问题

`BP CompareFileTime` 断下后取消断点，`Ctrl+F9` 返回程序

```
77E51702     8B4424 04           mov eax,dword ptr ss:[esp+4]//断在这,Ctrl+F9 返回    
77E51706     8B4C24 08           mov ecx,dword ptr ss:[esp+8]   

77E5170A     8B10                mov edx,dword ptr ds:[eax]   

77E5170C     8B40 04             mov eax,dword ptr ds:[eax+4]   

77E5170F     56                  push esi   

77E51710     8B31                mov esi,dword ptr ds:[ecx]   

77E51712     8B49 04             mov ecx,dword ptr ds:[ecx+4]   

77E51715     3BC1                cmp eax,ecx   

77E51717     0F87 D4E5FFFF       ja kernel32.77E4FCF1   

77E5171D     73 07               jnb short kernel32.77E51726   

77E5171F     83C8 FF             or eax,FFFFFFFF   

77E51722     5E                  pop esi   

77E51723     C2 0800             retn 8//返回 00D55088    

00D55088     85C0                test eax,eax//Ctrl+F9 返回    

   ………… 省 略 …………   

00D550C7     C3                  retn//返回 00D5854B    

00D58546     E8 64CAFFFF         call 00D54FAF   

00D5854B     84C0                test al,al//返回这里    

00D5854D     59                  pop ecx   

00D5854E     74 43               je short 00D58593   

00D58593     3BFB                cmp edi,ebx   

00D58595     74 08               je short 00D5859F   

00D5859F     8B45 F0             mov eax,dword ptr ss:[ebp-10]   

00D585A2     3BC3                cmp eax,ebx   

00D585A4     74 56               je short 00D585FC//在这里改变标志位 Z=1 使其跳转！★    

00D585FC     A1 3CCAD700         mov eax,dword ptr ds:[D7CA3C]   

00D58601     3918                cmp dword ptr ds:[eax],ebx   

00D58603     75 67               jnz short 00D5866C//在这里改变标志位 Z=0 使其跳转！★    

00D5866C     60                  pushad//到这里就 OK 啦  :-)    

00D5866D     33C0                xor eax,eax   

00D5866F     75 02               jnz short 00D58673   
```

在 `00D585A4` 和 `00D58603` 处改变标志位，使这 2 个地方跳转就避开“`System Clock Set Back`”的问题了。注意：不要直接修改代码，程序会出错的。也可以跟踪修改，使程序“`FixClock`”，接受现在的系统时间为合法时间。至于“`Expired`”的问题也在下面的代码段处理，修改跳转就行了。

—————————————————————————————————

## 二、Magic Jump，避开 IAT 加密

取消以前断点，现在下断：`BP GetModuleHandleA+5` 断下后取消断点

```
77E59F93     837C24 04 00        cmp dword ptr ss:[esp+4],0   

77E59F98     0F84 23060000       je kernel32.77E5A5C1//断在这    

77E59F9E     FF7424 04           push dword ptr ss:[esp+4]   

77E59FA2     E8 55080000         call kernel32.77E5A7FC   

77E59FA7     85C0                test eax,eax   

77E59FA9     74 08               je short kernel32.77E59FB3   

77E59FAB     FF70 04             push dword ptr ds:[eax+4]   

77E59FAE     E8 B0060000         call kernel32.GetModuleHandleW   

77E59FB3     C2 0400             retn 4//返回 00D26A5C    
```

`☆` `注意看` `BP GetModuleHandleA+5` `时的堆栈：` `☆`

```
0012BE5C     00D26A5C  返回到 00D26A5C 来自 kernel32.GetModuleHandleA   

0012BE60     0012BF98  ASCII "kernel32.dll"   
```

这次修改 `Magic Jump` 采用 `pyzpyz` 兄弟提出的方法，直接在这个时机返回程序

```
00D26A56     FF15 C4D0D400       call dword ptr ds:[D4D0C4];  kernel32.GetModuleHandleA   

00D26A5C     8B0D B885D500       mov ecx,dword ptr ds:[D585B8]   

00D26A62     89040E              mov dword ptr ds:[esi+ecx],eax   

00D26A65     A1 B885D500         mov eax,dword ptr ds:[D585B8]   

00D26A6A     393C06              cmp dword ptr ds:[esi+eax],edi   

00D26A6D     75 16               jnz short 00D26A85   

00D26A6F     8D85 B4FEFFFF       lea eax,dword ptr ss:[ebp-14C]   

00D26A75     50                  push eax   

00D26A76     FF15 CCD0D400       call dword ptr ds:[D4D0CC];  kernel32.LoadLibraryA   

00D26A7C     8B0D B885D500       mov ecx,dword ptr ds:[D585B8]   

00D26A82     89040E              mov dword ptr ds:[esi+ecx],eax   

00D26A85     A1 B885D500         mov eax,dword ptr ds:[D585B8]   

00D26A8A     393C06              cmp dword ptr ds:[esi+eax],edi   

00D26A8D     0F84 AD000000       je 00D26B40//改为 JMP  ★    

00D26A93     33C9                xor ecx,ecx   

00D26A95     8B03                mov eax,dword ptr ds:[ebx]   

00D26A97     3938                cmp dword ptr ds:[eax],edi   

00D26A99     74 06               je short 00D26AA1   

00D26A9B     41                  inc ecx   

00D26A9C     83C0 0C             add eax,0C   

00D26A9F     EB F6               jmp short 00D26A97   
```

看到上面 `GetModuleHandleA` 和 `LoadLibraryA` 了吗？直接改 `00D26A8D jmp 00D26B40`，此时修改代码很方便，不需要在 `IAT` 处理结束后再还原代码了。 `8)`

—————————————————————————————————

## 三、401000 段 内存断点 大法，直达 OEP

`Alt+M` 查看内存，在 `401000` 开始的段上下 内存访问断点，`F9` 运行，直接中断在 `OEP` 处

```
0043B0FF     55                  push ebp//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程    
0043B100     8BEC                mov ebp,esp   

0043B102     6A FF               push -1   

0043B104     68 E0BC4700          push MrCaptor.0047BCE0   

0043B109     68 A4A34300         push MrCaptor.0043A3A4   

0043B10E     64:A1 00000000      mov eax,dword ptr fs:[0]   

0043B114     50                  push eax   

0043B115     64:8925 00000000     mov dword ptr fs:[0],esp   

0043B11C     83EC 58             sub esp,58   

0043B11F     53                  push ebx   

0043B120     56                  push esi   

0043B121     57                  push edi   

0043B122     8965 E8             mov dword ptr ss:[ebp-18],esp   

0043B125     FF15 EC034700       call dword ptr ds:[4703EC];  kernel32.GetVersion   
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `0003B0FF`，点 `IT AutoSearch`，`CUT` 掉无效函数。

我是在 `XP` 下脱壳的，若要脱壳后的程序能够在 `98` 下正常运行，需修改以下 2 个函数：

1. `000703B0 kernel32.dll 02AC RestoreLastError` //改为：`SetLastError`
2. `000709A0 winspool.drv 008F DEVICECAPABILITIES` //`CUT` 掉！

`FixDump`，正常运行！可以简单优化一下脱壳后的程序，用 `LordPE` 删除 `text1` 和其下的 `adata`、`data1`、`pdata` 共 4 个区段，注意保留 `rsrc` 区段！然后再用 `LordPE` 重建 `PE`，提示最小到 `50％`，`1.62M->835K`。呵呵，精简了许多。

—————————————————————————————————

## 四、破解

```
00428213     FFD7                call edi;  kernel32.GetEnvironmentVariableA   

00428215     85C0                test eax,eax   

00428217     74 0C               je short dumped_.00428225//NOP 掉就 OK 了  :-)    

00428219     C783 B8020000 E7030>mov dword ptr ds:[ebx+2B8],3E7   

00428223     EB 2D               jmp short dumped_.00428252   
```

这样修改之后 `About` 里面的 `Unregistered` 和时间提示就没用了，已然注册。

—————————————————————————————————

, \_/
/| \_.-~/    \_ ,
( /~ /  ~.\_ |
\`\\ \_/    ~ )  忍把浮名
\_--~~~-.) )\_\_/;;,. \_ //'
/'\_, --~ ~~- ,;;\_\_\_( (.-~~~-.  换了破解轻狂
\`~ \_( ,\_..-- ( ,;''/ ~-- /.\_\`
/~~//' /' \`~ ) /--..\_, )\_ \`~
" \`~" " \`" /~'\` \`\\~~
" " "~' ""

青春都一晌




Cracked By 巢水工作坊——fly `[OCN]` `[FCG]` `[NUKE]` `[DCM]`

2004-03-18 19:30

`4dd466d0e1e0cc123afbe504da2462ca`
