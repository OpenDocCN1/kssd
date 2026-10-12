
# 注册表医生 V2.96

> 原文：[`www.pediy.com/kssd/pediy06/pediy6474.htm`](https://www.pediy.com/kssd/pediy06/pediy6474.htm)

浅谈 SVKP 1.3X 壳的输入表修复——注册表医生 V2.96 脱壳+破解

下载页面： http://www.skycn.com/soft/7156.html

软件大小： 1928 KB

软件语言： 英文

软件类别： 国产软件 / 共享版 / 卸载清除

应用平台： `Win9x/NT/2000/XP`

加入时间： 2004-02-20 17:12:14

下载次数： 40350

推荐等级： *****

### 软件简介
注册表医生(`Registry Medic`)。Windows 注册表是一个庞杂的数据库，它记载了 Windows 所必需要的硬件和软件信息，稍有闪失就足让 Windows 不能启动或出现这样那样的错误。在通常情况下，我们的计算机会经常性的安装和反安装一些程序，日积月累就会被无用的信息充满整个的注册表！Windows 不但会运行减慢，还会引起莫名的错误。如何解决呢？注册表医生就能帮助治疗您的注册表。注册表医生，能够帮您检查注册表中文件/目录的完整性和一致性。如果这些文件没有从原来的位置移到了另外一个地方，但是和注册表中的信息不一致，注册表医生会帮您寻找这些文件，并匹配到相应的注册表信息上。如果您的注册表中保存着磁盘上已经被删除的文件记录，那么注册表医生会找到它们，并建议您删除这些注册表信息。当然您也可以保留这些信息。

### 作者声明
只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

### 调试环境
`WinXP`、`Ollydbg`、`PEiD`、`LordPE`、`ImportREC`

————————————————————————————————— 

### 过程

SVK Protector 的 IAT 加密不是很变态，用 `ImportREC` 就能修复大部分函数，但依旧有一些函数是无法识别的，`SvkpIAT.dll` 插件有点好用，不过碰上 `ExitProcess` 就 OVER 了。用了几天时间看了几个 SVKP 1.3X 的壳，简单说一下避开 IAT 加密的方法，有不少是臆测，欢迎大家指正。

SVK Protector 1.32 的 IAT 加密我觉得可以分作三类：

1.  普通函数加密，这个简单
2.  特殊函数加密，`GetModuleHandleA`、`GetCommandLineA` 等
3.  SVK 检测函数，也算是特殊函数吧，主要功能是反脱壳或者检验注册

OK, Let's Go!

————————————————————————————————— 

## 一、Magic Jump ，避开 IAT 加密

设置 `Ollydbg` 忽略所有异常选项。老规矩：用 `IsDebug` 1.4 插件去掉 `Ollydbg` 的调试器标志。

```
00558000     60              pushad//进入 OD 后停在这
00558001     E8 00000000     call RegMedic.00558006
00558006     5D              pop ebp
00558007     81ED 06000000   sub ebp,6
0055800D     EB 05           jmp short RegMedic.00558014
```

F9 运行，程序在异常处中断

```
081F137F     6285 0E0B0000   bound eax,qword ptr ss:[ebp+B0E]//SVK 典型异常
```

下断 `BP GetModuleHandleA+5`，`Shift+F9` 通过 SVK 的典型异常。断下后取消断点，`Ctrl+F9` 返回

```
08213FC7     E8 02000000     call 08213FCE//返回这里
08213FCC     CD20 83042408   vxdcall 8240483
08213FD2     C3              retn
```

现在 `Ctrl+F` 在当前位置查找以下命令并下断：

```
sub dword ptr ds:[ebx],eax
mov dword ptr ds:[edi],eax
cmp dword ptr ds:[ebx],251097CC
cmp dword ptr ds:[ebx],2D66B1C5
cmp dword ptr ds:[ebx],D9B20494
cmp dword ptr ds:[ebx],D0861AA4
cmp dword ptr ds:[ebx],B1866570
cmp dword ptr ds:[ebx],9B76460E
cmp dword ptr ds:[ebx],E69307DB
cmp dword ptr ds:[ebx],A56C7B62
cmp dword ptr ds:[ebx],BB964E66
cmp dword ptr ds:[ebx],5BD70645
cmp dword ptr ds:[ebx],1DFCE00D
cmp dword ptr ds:[ebx],0FDD31
cmp dword ptr ds:[ebx],2651B795
cmp dword ptr ds:[ebx],4BF682B4
cmp dword ptr ds:[ebx],4CCF1A0F
cmp dword ptr ds:[ebx],DF87764A
cmp dword ptr ds:[ebx],FBB2B8B8
```

———————————————————————— 

### 1、普通函数加密 处理

```
08216AAF     8907            mov dword ptr ds:[edi],eax//写入重定位地址
08216AB1     7C 03           jl short 08216AB6
08216AB8     61              popad
08216AB9     E8 01000000     call 08216ABF
```

修改：

```
08216AAF     61              popad//呵呵，先 POPAD  ★
08216AB0     8907            mov dword ptr ds:[edi],eax//写入正确函数 ★
08216AB2     90              nop
08216AB3     EB 03           jmp short 08216AB8
08216AB8     90              nop//NOP 掉原先的 popad ★
08216AB9     E8 01000000     call 08216ABF
```

———————————————————————— 

### 2、特殊函数的处理

SVKP 的特殊函数（包括 SVK 检测函数）应该有 17 个

```
08214C3E     2903            sub dword ptr ds:[ebx],eax//计算 函数名的 HASH 值？
```

下面比较是否是特殊函数，是则跳转处理：

```
08214D9C     813B CC971025   cmp dword ptr ds:[ebx],251097CC
08214DA7     0F84 23310000   je 08217ED0//修改：jmp 082150B6  ★
08214E21     813B C5B1662D   cmp dword ptr ds:[ebx],2D66B1C5//GetCommandLineA
08214E2C     0F84 CB310000   je 08217FFD
08214E46     813B 9404B2D9   cmp dword ptr ds:[ebx],D9B20494
08214E51     0F84 FA350000   je 08218451
0821501D     813B A41A86D0   cmp dword ptr ds:[ebx],D0861AA4
08215028     0F84 DD380000   je 0821890B
0821503B     813B 706586B1   cmp dword ptr ds:[ebx],B1866570//GetModuleHandleA
08215046     0F84 343C0000   je 08218C80
08215055     813B 0E46769B   cmp dword ptr ds:[ebx],9B76460E
08215060     0F84 953F0000   je 08218FFB
0821506D     813B DB0793E6   cmp dword ptr ds:[ebx],E69307DB
08215078     0F84 C33F0000   je 08219041
08215087     813B 627B6CA5   cmp dword ptr ds:[ebx],A56C7B62
08215092     0F84 F33F0000   je 0821908B
082150A1     813B 664E96BB   cmp dword ptr ds:[ebx],BB964E66
082150AC     0F84 25400000   je 082190D7
082150B6     813B 4506D75B   cmp dword ptr ds:[ebx],5BD70645//SVK 反脱壳（注册）函数
082150C1     0F84 59400000   je 08219120
082150D0     813B 0DE0FC1D   cmp dword ptr ds:[ebx],1DFCE00D//SVK 反脱壳（注册）函数
082150DB     0F84 85400000   je 08219166
082150E8     813B 31DD0F00   cmp dword ptr ds:[ebx],0FDD31   //SVK 反脱壳（注册）函数
082150F2     0F84 B7400000   je 082191AF
082150FC     813B 95B75126   cmp dword ptr ds:[ebx],2651B795
08215107     0F84 F5400000   je 08219202//修改：jmp 08215164  ★
08215111     813B B482F64B   cmp dword ptr ds:[ebx],4BF682B4
0821511C     0F84 2B410000   je 0821924D
08215126     813B 0F1ACF4C   cmp dword ptr ds:[ebx],4CCF1A0F
08215130     0F84 62410000   je 08219298
0821513F     813B 4A7687DF   cmp dword ptr ds:[ebx],DF87764A
08215149     0F84 BA440000   je 08219609
08215158     813B B8B8B2FB   cmp dword ptr ds:[ebx],FBB2B8B8
0821515E     0F84 0C4B0000   je 08219C70
08215164     60              pushad
08215165     7C 03           jl short 0821516A
08215167     EB 03           jmp short 0821516C
```

可以直接把 `082150AC` 以上的几个 NOP 掉，这样原先 `ImportREC` 无法识别的 `GetModuleHandleA` 等特殊函数就变成普通函数来处理了。SVK 反脱壳（注册）函数现在不需要处理，后面的跳转也不能 NOP，否则会出错的。

为了“照顾”反脱壳（注册）函数，最后修改如下：

```
08214DA7     E9 0A030000     jmp 082150B6
08215107     EB 5B           jmp 08215164
```

修改之后这些特殊函数在 `08216AB0` 处就自动还原成正确函数写入地址了。

`RegMedical.exe` 的特殊函数如下：

```
001141A8  ?  0000  082733FC   //GetModuleHandleA
001141B8  ?  0000  08272A23   //GetCommandLineA
00114258  ?  0000  082733FC   //GetModuleHandleA
0011432C  ?  0000  0827B008   //GetVersionExA
00114330  ?  0000  0827AC83   //GetVersion
00114358  ?  0000  082733FC   //GetModuleHandleA
```

———————————————————————— 

### 3、SVK 反脱壳（注册）函数

这几个函数最后进入程序处理。

```
082191E1     83C4 04         add esp,4
082191E4     890F            mov dword ptr ds:[edi],ecx//SVK 反脱壳（注册）函数
082191E6     E8 02000000     call 082191ED
```

本程序共有三个 SVK 反脱壳（注册）函数：

```
FThunk: 001148B4  NbFunc: 00000003

0  001148B4  ?  0000  082785D2
0  001148B8  ?  0000  08278098
0  001148BC  ?  0000  08276FC4
```

————————————————————————————————— 

## 二、ESP 定律，走至伪 OEP。仿真 Stolen Code

输入表处理结束后程序当然要跳向 OEP 啦。我再次使用 ESP 定律，当然这个程序也可以在 CODE 段下内存断点走到伪 OEP。也可以直接 `Ctrl+F:jmp dword ptr ss:[esp-4]`，找到飞天一跃的地方。但推荐使用 ESP 定律的方法，这样可以看到部分变形的 Stolen Code

在★`0012FFB0`★处的 4 个字节上下 硬件访问－>Word 断点，F9 运行,断几次就会来到目的地啦！ :-)

为何要选择在 `0012FFB0` 处？呵呵，只是我的猜测+试验得出的，你当然可以想其他办法啦。

```
08271D45     60              pushad//中断。F9 数十次会来到以下地方
0827D1CE     58              pop eax
0827D1CF     68 EE91DFBB     push BBDF91EE
0827D1D4     87E4            xchg esp,esp
0827D1D6     50              push eax ; RegMedic.004E2268//这个值可以在伪 OEP 处看到
0827D1D7     B8 7D946E44     mov eax,446E947D
0827D1DC     014424 04       add dword ptr ss:[esp+4],eax
0827D1E0     58              pop eax
0827D1E1     89DB            mov ebx,ebx
0827D1E3     83C4 04         add esp,4
0827D1E6     87FF            xchg edi,edi
0827D1E8     FF6424 FC       jmp dword ptr ss:[esp-4]; 004E266B//GO！飞向光明之巅！:-)
```



在伪 `OEP` 处看看相关寄存器和堆栈数据：`EBP`=0012FFB4 `ESP`=0012FFC0 `EAX`=004E2268

`SVKP` 的 Stolen Code 一般不多，根据以上数据和壳中代码处理部分，补上 `OEP` 处的 Stolen Code

```
004E2660     55                   push ebp//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程
004E2661     8BEC                 mov ebp,esp
004E2663     83C4 F4              add esp,-0C
004E2666     B8 68224E00          mov eax,004E2268//补上的 Stolen Code   :-)
004E266B     E8 A443F2FF          call RegMedic.00406A14//伪 OEP
004E2670     A1 D4B94F00          mov eax,dword ptr ds:[4FB9D4]
004E2675     8B00                 mov eax,dword ptr ds:[eax]
004E2677     E8 8CC2F6FF          call RegMedic.0044E908
004E267C     A1 D4B94F00          mov eax,dword ptr ds:[4FB9D4]
004E2681     8B00                 mov eax,dword ptr ds:[eax]
004E2683     BA AC264E00          mov edx,RegMedic.004E26AC; ASCII "Registry Medic"
```

运行 `ImportREC`，设置“启用调试权限”的选项，选择这个进程。把 `OEP` 改为 `000E2660`、`RVA`=00114150、`Size`=000007B4，点“Get Import”，呵呵，除了 3 个 `SVK` 反脱壳（注册）函数其他的全部是有效的！`FixDump`。

---

### 三、修正 SVK 反脱壳（注册）函数

程序现在还是无法正常运行的，别忘了还有 3 个 SVK 函数没有修复。

```
0  001148B4  ?  0000  082785D2
0  001148B8  ?  0000  08278098
0  001148BC  ?  0000  08276FC4
```

在这几个地方下 内存 断点，很容易就找到调用的地方：

```
0044FB78     FF25 BC485100       jmp dword ptr ds:[5148BC]
0044FB80     FF25 B8485100       jmp dword ptr ds:[5148B8]
0044FB88     FF25 B4485100       jmp dword ptr ds:[5148B4]
```

把这几个修改为：

```
0044FB78     C3                   retn
0044FB80     C3                   retn
0044FB88     C3                   retn
```

`OK`，程序已经能够运行了。比较幸运的是这个程序没有使用分段解密。

---

### 四、破解

虽然程序能够运行了，但提示 `Trail Expired`，弹出大大的注册框让偶注册。呵呵，看来注册部分没用 SVK 的。

用户名：`@ fly @`

试炼码：`13572-46890-12135－20040－03080－24689－123`

根据提示信息，很容易找到注册核心的地方。

```
004E1294     8B45 F4              mov eax,dword ptr ss:[ebp-C]//[ebp-C]=@ fly @
004E1297     E8 8C2CF2FF          call dumped_.00403F28
004E129C     83F8 06              cmp eax,6//用户名至少 6 位
004E129F     0F8C F0000000        jl dumped_.004E1395
004E12A5     8B45 F8              mov eax,dword ptr ss:[ebp-8]//[ebp-8]=试炼码
004E12A8     E8 7B2CF2FF          call dumped_.00403F28
004E12AD     83F8 21              cmp eax,21//试炼码至少 33 位
004E12B0     7C 1D                jl short dumped_.004E12CF
004E12B2     8B45 FC              mov eax,dword ptr ss:[ebp-4]
004E12B5     E8 CE0A0000          call dumped_.004E1D88//黑名单检测
```

黑名单：`Cindy J. Leonheardt`、`[DJ]neo`  呵呵，希望以后不要加上 `fly`   :-)

```
004E12BA     84C0                 test al,al
004E12BC     75 11                jnz short dumped_.004E12CF//是黑名单上的就直接 OVER  :oops:
004E12BE     8B55 F4              mov edx,dword ptr ss:[ebp-C]
004E12C1     B8 F4134E00          mov eax,dumped_.004E13F4
004E12C6     E8 492FF2FF          call dumped_.00404214//检测用户名中有无 @ 这个字符
004E12CB     85C0                 test eax,eax
004E12CD     75 45                jnz short dumped_.004E1314//没有@就 OVER  :oops:
004E12CF     8D45 EC              lea eax,dword ptr ss:[ebp-14]
004E12D2     BA 00144E00          mov edx,dumped_.004E1400
               //ASCII "Registration failed. Please try again!"       BAD BOY！  :-(
004E130A     E8 8960F2FF          call <jmp.&user32.MessageBoxA>//呵呵，Please try again!
004E130F     E9 B0000000          jmp dumped_.004E13C4
004E1314     8B55 F8              mov edx,dword ptr ss:[ebp-8]
004E1317     8B45 F4              mov eax,dword ptr ss:[ebp-C]
004E131A     E8 91A9FAFF          call dumped_.0048BCB0
004E131F     E8 C4B4FAFF          call dumped_.0048C7E8//运算、比较注册码
004E1324     A1 D4B74F00          mov eax,dword ptr ds:[4FB7D4]
004E1329     8038 00              cmp byte ptr ds:[eax],0//判断[4FB7D4]处标志
004E132C     74 25                je short dumped_.004E1353
004E132E     A1 58B84F00          mov eax,dword ptr ds:[4FB858]
004E1333     8338 00              cmp dword ptr ds:[eax],0//判断[4FB858]处标志
004E1336     74 0C                je short dumped_.004E1344
004E1338     A1 58B84F00          mov eax,dword ptr ds:[4FB858]
004E133D     8B00                 mov eax,dword ptr ds:[eax]
004E133F     E8 68D1FFFF          call dumped_.004DE4AC
004E1344     8D45 EC              lea eax,dword ptr ss:[ebp-14]
004E1347     BA 30144E00          mov edx,dumped_.004E1430
               //ASCII "Thank you for registering Registry Medic."  呵呵，胜利女神！:-)
004E134C     E8 EF29F2FF          call dumped_.00403D40
004E1351     EB 0D                jmp short dumped_.004E1360
004E1353     8D45 EC              lea eax,dword ptr ss:[ebp-14]
004E1356     BA 00144E00          mov edx,dumped_.004E1400
               //ASCII "Registration failed. Please try again!"       BAD BOY！  :-(
```

直接看注册码的比较部分吧： :-)

```
0048B300     8B45 D8              mov eax,dword ptr ss:[ebp-28]//EAX=50e8c2d4bc92
0048B303     8B55 CC              mov edx,dword ptr ss:[ebp-34]//EDX=135724689012
0048B306     E8 2D8DF7FF          call dumped_.00404038//比较注册码前 12 位
0048B30B     75 11                jnz short dumped_.0048B31E
0048B30D     8B45 D4              mov eax,dword ptr ss:[ebp-2C]//[ebp-2C]=123
0048B310     8B55 D0              mov edx,dword ptr ss:[ebp-30]//[ebp-30]=2db
0048B313     E8 208DF7FF          call dumped_.00404038//比较注册码最后 3 位，根据前 30 位得出
0048B318     75 04                jnz short dumped_.0048B31E
0048B31A     C645 F7 01           mov byte ptr ss:[ebp-9],1//置成功标志位
0048B31E     33C0                 xor eax,eax
0048B320     5A                   pop edx
0048B321     59                   pop ecx
0048B322     59                   pop ecx
0048B323     64:8910              mov dword ptr fs:[eax],edx
0048B326     68 38B34800          push dumped_.0048B338
0048B32B     E8 5048FCFF          call dumped_.0044FB80
0048B330     C3                   retn
```

---

**【注册信息保存】：**

注册信息和时间信息保存在注册表中的 2 个地方：

`[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\{B087CE89-A8EE-4890-91E7-696B3F749500}]`

`"AppIdK"="CA000DFE903977C2EB7C-5CD3A51B5BED33B772DB-316841B94D9A0C-50E8C2D4BC921352004003080246892DB"`

`[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\{0C985599-3A7C-4D1D-9D73-B6BBC812D950}]`

`"AppIdK"="CA000DFE903977C2EB7C-5CD3A51B5BED33B772DB-316841B94D9A0C-50E8C2D4BC921352004003080246892DB"`

---

**【整理】：**

用户名：`@ fly @`

注册码：`50E8C－2D4BC－92135－20040－03080－24689－2DB`

---

```
        ,      _/      
      /| _.-~/           _     ,       
     ( /~   /            ~-.\ |
     \`\\  _/                  ~ )          忍把浮名  
  _-~~~-.)  )\_/;;,.           _  //'    
 /'\_,   --~    ~~~-  ,;;\_\_(  (.-~~~-.        换了破解轻狂 
 \`~ _( ,_..-- (     ,;'' /    ~--   /.\_\`  
  /~~//'   /`~          ) /--..\_, )\_  `~    
  "  `~"  "      `"      /~'`    \`\\~~   
                       "     "   "~'  ""
```

Cracked By 巢水工作坊——`fly` `[OCN]` `[FCG]` `[NUKE]` `[DCM]`

2004-03-12 14:00

`4dd466d0e1e0cc123afbe504da2462ca`
