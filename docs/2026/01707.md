
# 破解 开机小精灵 2.11 (7千字)

破解 开机小精灵 2.11

软件名称：开机小精灵
整理日期：2001.11.10
最新版本：2.11
文件大小：897KB
软件授权：共享软件
使用平台：Win9x/Me/NT/2000
发布公司：http://vcghost.yeah.net/

软件简介：

　　开机小精灵是一个系统监控软件。主要用于监控你的开机情况，并作出一定的分析，可以作为你判断系统稳定的一个参考。对你不在情况下，别人对你机器的使用情况做下记录甚至可以得到别人在你机器上的一切输入信息，包括密码和个人资料。加强你对自己机器的监控和控制，真正成为自己机器的主人。末注册版可以用于记录开机状态分析和数据统计的功能。但不能使用键盘监控的高级功能。注册后将可以使用本软件的所有功能，所有的警告窗口将消失。并也免费享有以后升级版本一切功能的使用权力。

原版软件下载: http://newhua.ruyi.com/down/RebootEidolon.zip （华军，897k）

未注册版本限制: 功能限制(有提示)，启动弹出提示窗口

破解工具：w32dasm 黄金版中文版、File Info 2.45、UltraEdit、WinHex

不好意思，在下水平仍然很低很低，暂时还只能爆破，不过这个软件的注册码好像很好找的说...^\_^

## 破解过程

### 1. 暴力破解

先运行软件，随便输入注册码注册，得到错误提示“请不要投机取巧，如果你真的对本软件的高级功能感兴趣，请马上注册”，OK～用 Fileinfo 看看，没有加壳，w32dasm 反汇编，看“串式参考”，来到此提示所在区域：

```
\* Reference To: MSVCRT.\_mbscmp, Ord:0159h
            |
:00419081 FF157CE74100            Call dword ptr
[0041E77C]
:00419087 83C408
add esp, 00000008
:0041908A 85C0
  test eax, eax    < ==== 比较...
:0041908C 0F8591000000            jne 00419123
    < ==== 跳则注册码错误，改为 je 00419123 或干脆 nop 掉
:00419092 33F6
  xor esi, esi
:00419094 89742410                mov
dword ptr [esp+10], esi
:00419098 8D442414                lea
eax, dword ptr [esp+14]
:0041909C 8D4C240C                lea
ecx, dword ptr [esp+0C]
:004190A0 50
  push eax
:004190A1 51
  push ecx
:004190A2 56
  push esi
:004190A3 683F000F00              push 000F003F
:004190A8 56
  push esi
:004190A9 56
  push esi
:004190AA 56
  push esi
\* Possible StringData Ref from Data Obj ->"software\microsoft\opennow"
<====很可疑的主键啊！！等等我们看
            看这个地方有什么玄机^\_^
            |
:004190AB 68FC604200              push 004260FC
:004190B0 6802000080              push 80000002
:004190B5 C644244401              mov [esp+44],
01
:004190BA 89742430                mov
dword ptr [esp+30], esi
\* Reference To: ADVAPI32.RegCreateKeyExA, Ord:015Fh
            |
:004190BE FF1508E04100            Call dword ptr
[0041E008]
:004190C4 85C0
  test eax, eax
:004190C6 753B
  jne 00419103
:004190C8 8B442408                mov
eax, dword ptr [esp+08]
:004190CC 8B74240C                mov
esi, dword ptr [esp+0C]
:004190D0 53
  push ebx
:004190D1 50
  push eax
:004190D2 89742418                mov
dword ptr [esp+18], esi
:004190D6 8BD8
  mov ebx, eax
\* Reference To: KERNEL32.lstrlenA, Ord:0308h
            |
:004190D8 FF157CE14100            Call dword ptr
[0041E17C]
:004190DE 40
  inc eax
:004190DF 50
  push eax
:004190E0 53
  push ebx
:004190E1 6A01
  push 00000001
:004190E3 6A00
  push 00000000
\* Possible StringData Ref from Data Obj ->"OpenNow"
            |
:004190E5 68506B4200              push 00426B50
:004190EA 56
  push esi
\* Reference To: ADVAPI32.RegSetValueExA, Ord:0186h
            |
:004190EB FF150CE04100            Call dword ptr
[0041E00C]
:004190F1 85F6
  test esi, esi
:004190F3 5B
  pop ebx
:004190F4 740D
  je 00419103
:004190F6 56
  push esi
\* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh
            |
:004190F7 FF1500E04100            Call dword ptr
[0041E000]
:004190FD 33F6
  xor esi, esi
:004190FF 89742410                mov
dword ptr [esp+10], esi
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004190C6(C), :004190F4(C)
|
:00419103 6A00
  push 00000000
\* Possible StringData Ref from Data Obj ->"注册成功"  <====  注册成功信息
            |
:00419105 68506E4200              push 00426E50
\* Possible StringData Ref from Data Obj ->"你已经注册成功,请重新启动软件开启高级功能"
            |
:0041910A 68246E4200              push 00426E24
:0041910F 8BCF
  mov ecx, edi
\* Reference To: MFC42.Ordinal:1080, Ord:1080h
            |
:00419111 E82C130000              Call 0041A442
:00419116 85F6
  test esi, esi
:00419118 741C
  je 00419136
:0041911A 56
  push esi
\* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh
            |
:0041911B FF1500E04100            Call dword ptr
[0041E000]
:00419121 EB13
  jmp 00419136
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041908C(C)
    < ==== 从这里慢慢往上看是哪里出错的
|
:00419123 6A00
  push 00000000
\* Possible StringData Ref from Data Obj ->"注册失败"
            |
:00419125 68186E4200              push 00426E18
\* Possible StringData Ref from Data Obj ->"请不要投机取巧，如果你真的对本软件的高级功能感"
                  ->"兴趣，请马上注册"
    <====  在这里错误，往上找是哪里跳过来的
            |
:0041912A 68D86D4200              push 00426DD8
:0041912F 8BCF
  mov ecx, edi
```

至此，运行软件，输入任意注册码注册即可成功，关闭程序，再打开，也没有弹出提示注册的窗口了，原来不能使用的高级选项也ok了...(不会吧？这么简单？？？我都不太敢相信了啊~~~)。如果怕不保险的话，可以顺手把未注册用户不能使用的提示信息 `nop` 掉，有好几个地方。(最近发现，9090 真的很好用~~)

刚才在上面的过程中看到了一个很可疑的主键`software\microsoft\opennow`，我们不妨看看，到底里面是什么东东~~

用 Regmon 监视了一下软件启动到关闭的注册表访问,得到这么一句:

```
16.25505329    OpenNow.exe:1068    QueryValue
  HKLM\software\microsoft\opennow\OpenNow    SUCCESS
  "303735363036303436353231"
```

哇！这个 "303735363036303436353231" 非常的可疑啊~~

把这个主键删除，然后启动软件，又变成未注册版本了，用上面得到的数字注册，居然成功了！！我@#$%^&...这个软件在我爆破后，随便输入注册码注册，居然会自动将正确的注册码写入注册表？？不是吧？？我咧……居然有这种软件……

### 2. Winhex 找出注册码

这个不需要多说吧，很简单的说~~

启动开机小精灵，随便输入注册码，Name：炎之川，S/N:1234567890 ，并点击"注册",出现注册码错误提示时不要点确定，启动 WinHex ->工具 -> Ram编辑器，选择`OpenNow.exe`-> 主要内存 -> 搜索 1234567890，在搜索到的第二处往上看一些，就是正确的注册码：“303735363036303436353231”。

总结一下，我的注册码: Name：炎之川，S/N:303735363036303436353231,OK~~收工~~

话说回来，感觉似乎太简单了一些，不知道里面还有没有什么陷阱！哪位大哥感兴趣，不妨帮我看看有没有什么问题！我在我的 Windows 2000 及 Windows ME OS下，测试有效，确实可用，所有的功能均无问题。

另，我的水平很低很低，所以写的破文也很一般(不要问我算法，不要问我动态调试……暂时我还没学会~~嘿嘿~~)，如果有人还想看这类的文章(我个人以为还是很适合什么都不会的初学者看的^\_^)，我就继续多写几篇！

炎之川 于 2001.11.12

mali: skipli@sina.com
