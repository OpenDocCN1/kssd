
# Code Injection 破解 Armadillo V4.20 加壳程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-595.htm`](https://www.pediy.com/kssd/pediy07/pediy7-595.htm)

Code Injection 破解 Armadillo V4.20 单进程加壳程序——`FTPRush` Unicode V1.0.RC6.build.568

下载页面： http://www.ftprush.com

软件大小： 3342 KB

软件语言： 简体中文

软件类别： 国外软件 / 共享版 / FTP 工具

应用平台： Win9x/NT/2000/XP

加入时间： 2005-07-12 11:56:33

下载次数： 55097

推荐等级： *****

软件介绍： 一个高性能的 FTP 客户端, 通过它你可以轻松地上传下载和 FXP(Ftp 服务器之间的文件传输), 完全多线程的构造, 让你自由自在地存取 FTP 服务器的资源。主要的特性：连接到多个 ftp 服务器、支持 Ftp 代理、脚本支持、多语言支持、容易使用、自动 CRC 校验、支持 mIRC 宣告。

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：WinXP、OllyDBD、PEiD、LordPE、IDA

—————————————————————————————————

【破解过程】：

Code Injection 用来 Patch 的思路如下：

1、在壳中寻找适当的切入点

2、Hook 特定 Api，取得对壳代码的某些控制权

3、搞定壳的检验

4、干掉壳所设置的限制，一般为时间限制

5、程序代码解开后 Patch 程序中的限制代码

下面主要是 stephenteh 的成果，我只是整理和演示一下。

—————————————————————————————————

## 一、在壳中寻找适当的切入点

用 PE 工具观察 Armadillo 加壳后的 `FTPRush.exe` 区段情况

No.   名称        VSize        VOffset      RSize        ROffset      Charact    

01    CODE        0045AEAC     00001000     00000000     00000000     60000020     

02    DATA        00016B80     0045C000     00000000     00000000     C0000040     

03    BSS         00037C45     00473000     00000000     00000000     C0000000     

04    .idata      000041D4     004AB000     00000000     00000000     C0000040     

05    .tls        00000024     004B0000     00000000     00000000     C0000000     

06    .rdata      00000018     004B1000     00001000     00001000     50000040     

07    .reloc      00045160     004B2000     00000000     00000000     50000040     

08    .text       00050000     004F8000     00041000     00002000     E0000020     

09    .adata      00010000     00548000     0000D000     00043000     E0000020     

0A    .data       00020000     00558000     0000A000     00050000     C0000040     

0B    .pdata      00250000     00578000     00250000     0005A000     C0000040     

0C    .rsrc       00043000     007C8000     00043000     002AA000     50000040     

可以看到壳区段 `.data` 和 `.pdata` 的区段属性都是可读可写的，并且 Armadillo 没有对这 2 个段进行校验。

因此可以把 Patch 代码放在这 2 个段中的空白处。

我选择 `.data` 段 RVA＝`00561990`（ROffset＝`00059990`）作为放置 Patch 代码的开始处。

再察看其输入表，选择 `KERNEL32.dll` 的 `GetProcAddress` 作为第一个桥梁

```
0091F0D3    FF15 70809500   call dword ptr ds:[958070]; kernel32.GetProcAddress
00920E87    FF15 B0809500   call dword ptr ds:[9580B0]; kernel32.ReadFile
0091EE1C    FF15 68809500   call dword ptr ds:[958068]; kernel32.VirtualProtect
```

`GetProcAddress`、`ReadFile`、`VirtualProtect` 是我们将要使用到的三个函数。

—————————————————————————————————

## 二、第一个接口+第二个接口 Hook `GetProcAddress`

使用修改版 OllyDBD，否则你要手动处理 `OutputDebugStringA`

设置 OllyDBD 忽略所有异常选项。用 IsDebug 插件去掉 OllyDBD 的调试器标志。

复制 `FTPRush.exe`，另存为 `1.eXe`，可以开 2 个 OllyDBD 分别载入这 2 个程序。

```
009336D3    55              push ebp
//进入 OllyDBD 后暂停在这
009336D4    8BEC            mov ebp,esp
009336D6    6A FF           push -1
009336D8    68 F8DE9500     push 95DEF8
009336DD    68 10349300     push 933410
009336E2    64:A1 00000000  mov eax,dword ptr fs:[0]
009336E8    50              push eax
009336E9    64:8925 000000> mov dword ptr fs:[0],esp
009336F0    83EC 58         sub esp,58
009336F3    53              push ebx
009336F4    56              push esi
009336F5    57              push edi
009336F6    8965 E8         mov dword ptr ss:[ebp-18],esp
009336F9    FF15 88819500   call dword ptr ds:[958188] ; kernel32.GetVersion
009336FF    33D2            xor edx,edx
00933701    8AD4            mov dl,ah
00933703    8915 84F59500   mov dword ptr ds:[95F584],edx
00933709    8BC8            mov ecx,eax
0093370B    81E1 FF000000   and ecx,0FF
00933711    890D 80F59500   mov dword ptr ds:[95F580],ecx
00933717    C1E1 08         shl ecx,8
0093371A    03CA            add ecx,edx
0093371C    890D 7CF59500   mov dword ptr ds:[95F57C],ecx
00933722    C1E8 10         shr eax,10
00933725    A3 78F59500     mov dword ptr ds:[95F578],eax
0093372A    33F6            xor esi,esi
0093372C    56              push esi
0093372D    E8 78160000     call 00934DAA
00933732    59              pop ecx
00933733    85C0            test eax,eax
00933735    75 08           jnz short 0093373F
00933737    6A 1C           push 1C
00933739    E8 B0000000     call 009337EE
0093373E    59              pop ecx
0093373F    8975 FC         mov dword ptr ss:[ebp-4],esi
00933742    E8 43130000     call 00934A8A
00933747    FF15 8C809500   call dword ptr ds:[95808C] ; kernel32.GetCommandLineA
0093374D    A3 840B9600     mov dword ptr ds:[960B84],eax
00933752    E8 01120000     call 00934958
00933757    A3 D8F59500     mov dword ptr ds:[95F5D8],eax
0093375C    E8 AA0F0000     call 0093470B
00933761    E8 EC0E0000     call 00934652
00933766    E8 2DFAFFFF     call 00933198
0093376B    8975 D0         mov dword ptr ss:[ebp-30],esi
0093376E    8D45 A4         lea eax,dword ptr ss:[ebp-5C]
00933771    50              push eax
00933772    FF15 90809500   call dword ptr ds:[958090] ; kernel32.GetStartupInfoA
00933778    E8 7D0E0000     call 009345FA
0093377D    8945 9C         mov dword ptr ss:[ebp-64],eax
00933780    F645 D0 01      test byte ptr ss:[ebp-30],1
00933784    74 06           je short 0093378C
00933786    0FB745 D4       movzx eax,word ptr ss:[ebp-2C]
0093378A    EB 03           jmp short 0093378F
0093378C    6A 0A           push 0A
0093378E    58              pop eax
0093378F    50              push eax
00933790    FF75 9C         push dword ptr ss:[ebp-64]
00933793    56              push esi
00933794    56              push esi
00933795    FF15 4C809500   call dword ptr ds:[95804C] ; kernel32.GetModuleHandleA
0093379B    50              push eax
0093379C    E8 CFC7FEFF     call 0091FF70
//过了此 CALL 壳就全部操作完成，程序运行，因此选择在这里面做 Patch ★
009337A1    8945 A0         mov dword ptr ss:[ebp-60],eax
009337A4    50              push eax
009337A5    E8 1BFAFFFF     call 009331C5
009337AA    8B45 EC         mov eax,dword ptr ss:[ebp-14]
009337AD    8B08            mov ecx,dword ptr ds:[eax]
009337AF    8B09            mov ecx,dword ptr ds:[ecx]
009337B1    894D 98         mov dword ptr ss:[ebp-68],ecx
009337B4    50              push eax
009337B5    51              push ecx
009337B6    E8 BB0C0000     call 00934476
009337BB    59              pop ecx
009337BC    59              pop ecx
009337BD    C3              retn
```

在 OllyDBD 里面直接修改 `1.eXe`，然后保存。

```
0091FF70    55              push ebp
//修改为：jmp 00961990  ★   第 1 个接口
0091FF71    8BEC            mov ebp,esp
0091FF73    B8 0C200000     mov eax,200C
0091FF78    E8 932D0100     call 00932D10
```

在 `00961990` 处键入以下代码：

```
00961990    60              pushad
00961991    A1 70809500     mov eax,dword ptr ds:[958070]
//[958070]中保存的是 GetProcAddress 的系统地址
00961996    A3 801A9600     mov dword ptr ds:[961A80],eax
//保存 GetProcAddress 系统地址到[961A80]处
0096199B    C705 70809500 C> mov dword ptr ds:[958070],9619C6
//修改原来的[958070]处为 9619C6
//这样壳所有的 GetProcAddress 调用都会到 9619C6 处理，我们取得了控制 ★
009619A5    A1 B0809500     mov eax,dword ptr ds:[9580B0]
//[9580B0]中保存的是 ReadFile 的系统地址
009619AA    A3 841A9600     mov dword ptr ds:[961A84],eax
//保存 ReadFile 系统地址到[961A84]处
009619AF    C705 70FF9100 5> mov dword ptr ds:[91FF70],B8EC8B55
//还原 0091FF70 处代码
009619B9    C605 74FF9100 0> mov byte ptr ds:[91FF74],0C
//还原 0091FF74 处代码
009619C0    61              popad
009619C1    E9 AAE5FBFF     jmp 0091FF70
//跳回 0091FF70 处继续原来的流程
```

上面是我们在壳身上做了第 1 个接口，运行我们的代码，再做第 2 个接口 Hook 壳的 `GetProcAddress` 调用。然后还原壳身上第 1 个接口处修改的代码，了无痕迹，再继续壳的运行流程。

—————————————————————————————————

## 三、跟踪获取原版 `FTPRush.exe` 的壳检验值
