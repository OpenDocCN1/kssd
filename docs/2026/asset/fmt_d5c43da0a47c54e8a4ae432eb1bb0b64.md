
# [原创]脱Armadillo 1.xx - 2.xx的壳教程

【破解作者】 落魄浪子

【作者邮箱】 zxy223_szb@21cn.net

【使用工具】 `FlyOD1.1` `PeiD` `IRC1.6`

【破解平台】 Windows XP SP2

【软件名称】 `PixHunter`

【下载地址】 [ae0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2F1L8%4k6S2i4K6u0V1M7$3!0X3N6q4)9J5k6h3y4G2L8b7`.`.](https://bbs.kanxue.com/elink@f1bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2F1L8%4k6S2i4K6u0V1M7$3!0X3N6q4)9J5k6h3y4G2L8b7`.`.)

【软件简介】 《图片猎人》（Pix Hunter)是一个专门用于从Internet上搜索和下载、管理图片的工具软件。它保持了“网际快车”、“网络蚂蚁”等下载软件操作简便、下载迅速等优点，并扩展很多功能：如智能搜索、图片浏览、图片管理等等。这是一款集成了众多同类软件的优点，并增加了更多功能的图片下载专用工具软件。

简而言之,《图片猎人》就象是下载工具ACDSee和下载工具FlashGet的完美结合。

【加壳方式】 Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解内容

这个壳把我搞的头破血流，以前总是用工具脱，但总用工具也不是办法，人总要长大吧，只好学用手工脱，一次次的失败，其间看了好多大侠的文章，也没能解决，一有时间就慢慢分析，和参考大侠的文章，现在终于把她搞定了，兴奋啊！！！！

还是来正题吧，用`PEID`查是Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks的壳，但用`FI`查报是Armadillo 2.5，唉，不管是哪个版本加的，知道是Armadillo就行了。这个程序运行之后有两个进程，应该是Armadillo的双进程。不管了，对着大侠的文章用`OD`载入，设置`Ollydbg`忽略所有的异常选项。用`IsDebug 1.4`插件隐藏`OD`。

载入后`OD`停在这里：

```
00629379 PixHunte.<ModuleEntryPoint>   /$  55                   push ebp
0062937A                               |.  8BEC                 mov ebp,esp
0062937C                               |.  6A FF                push -1
0062937E                               |.  68 78E26400          push PixHunte.0064E278
00629383                               |.  68 608D6200          push PixHunte.00628D60                        ;  SE handler installation
00629388                               |.  64:A1 00000000       mov eax,dword ptr fs:[0]
0062938E                               |.  50                   push eax
0062938F                               |.  64:8925 00000000     mov dword ptr fs:[0],esp
00629396                               |.  83EC 58              sub esp,58
00629399                               |.  53                   push ebx
0062939A                               |.  56                   push esi
0062939B                               |.  57                   push edi
0062939C                               |.  8965 E8              mov dword ptr ss:[ebp-18],esp
0062939F                               |.  FF15 4CB16400        call dword ptr ds:[<&KERNEL32.GetVersion>]    ;  kernel32.GetVersion
```

下断点：`BP OpenMutexA`，然后F9运行

```
7C80EC1B kernel32.OpenMutexA             8BFF                   mov edi,edi      OD中断在这里
7C80EC1D                                 55                     push ebp
7C80EC1E                                 8BEC                   mov ebp,esp
7C80EC20                                 51                     push ecx
7C80EC21                                 51                     push ecx
7C80EC22                                 837D 10 00             cmp dword ptr ss:[ebp+10],0
7C80EC26                                 56                     push esi
7C80EC27                                 0F84 7A500300          je kernel32.7C843CA7
7C80EC2D                                 64:A1 18000000         mov eax,dword ptr fs:[18]
7C80EC33                                 FF75 10                push dword ptr ss:[ebp+10]
7C80EC36                                 8DB0 F80B0000          lea esi,dword ptr ds:[eax+BF8]
7C80EC3C                                 8D45 F8                lea eax,dword ptr ss:[ebp-8]
7C80EC3F                                 50                     push eax
7C80EC40                                 FF15 8C10807C          call dword ptr ds:[<&ntdll.RtlInitAnsiString>]; ntdll.RtlInitAnsiString
```

`OD`断下后，看看堆栈：

```
0012F5B0   0061FDA6  /CALL 到 OpenMutexA 来自 PixHunte.0061FDA0
0012F5B4   001F0001  |Access = 1F0001
0012F5B8   00000000  |Inheritable = FALSE
0012F5BC   0012FBF0  \MutexName = "644::DA8D229493"      \*重要等下有用
```

Ctrl+G定位到`00401000`处：

```
00401000                                 0000                   add byte ptr ds:[eax],al
00401002                                 0000                   add byte ptr ds:[eax],al
00401004                                 0000                   add byte ptr ds:[eax],al
00401006                                 0000                   add byte ptr ds:[eax],al
00401008                                 0000                   add byte ptr ds:[eax],al
0040100A                                 0000                   add byte ptr ds:[eax],al
0040100C                                 0000                   add byte ptr ds:[eax],al
0040100E                                 0000                   add byte ptr ds:[eax],al
```

以上全是空码，在`00401000`处Ctrl+E贴上以下代码，这是从大侠那里偷来的，呵呵

```
00401000                                 60                     pushad
00401001                                 9C                     pushfd
00401002                                 68 F0FB1200            push 12FBF0  \*这是堆栈中看到的
00401007                                 33C0                   xor eax,eax
00401009                                 50                     push eax
0040100A                                 50                     push eax
0040100B                                 E8 2FDB407C            call kernel32.CreateMutexA
00401010                                 9D                     popfd
00401011                                 61                     popad
00401012                               - E9 04DC407C            jmp kernel32.OpenMutexA
```

然后在`00401000`处单击右键，在弹出的菜单上选“在此处新建`EIP`”，确定。然后F9运行

```
7C80EC1B kernel32.OpenMutexA             8BFF                   mov edi,edi  运行之后又中断在这里，F2取消断点
7C80EC1D                                 55                     push ebp
7C80EC1E                                 8BEC                   mov ebp,esp
7C80EC20                                 51                     push ecx
7C80EC21                                 51                     push ecx
7C80EC22                                 837D 10 00             cmp dword ptr ss:[ebp+10],0
7C80EC26                                 56                     push esi
7C80EC27                                 0F84 7A500300          je kernel32.7C843CA7
```

Ctrl+G定位到`00401000`处，取消先前的修改。之后下断点：`BP GetModuleHandleA`，F9运行

```
7C80B529 kernel32.GetModuleHandleA       8BFF                   mov edi,edi     OD中断在这里，F2取消断点。
7C80B52B                                 55                     push ebp
7C80B52C                                 8BEC                   mov ebp,esp
7C80B52E                                 837D 08 00             cmp dword ptr ss:[ebp+8],0
7C80B532                                 74 18                  je short kernel32.7C80B54C    在这里F2重新下断点
7C80B534                                 FF75 08                push dword ptr ss:[ebp+8]
7C80B537                                 E8 682D0000            call kernel32.7C80E2A4
7C80B53C                                 85C0                   test eax,eax
7C80B53E                                 74 08                  je short kernel32.7C80B548
7C80B540                                 FF70 04                push dword ptr ds:[eax+4]
7C80B543                                 E8 F4300000            call kernel32.GetModuleHandleW
7C80B548                                 5D                     pop ebp
7C80B549                                 C2 0400                retn 4
```

F9运行，中断几次后，看看堆栈中的数据

```
0012ED48  |77F45BB0  返回到 SHLWAPI.77F45BB0 来自 kernel32.GetModuleHandleA
0012ED4C  |77F44FF4  ASCII "KERNEL32.DLL"
0012ED50  |00000001
0012ED54  |77F40000  SHLWAPI.77F40000
-----------------------------------------------------------------------
0012EC88  |5D175394  返回到 COMCTL32.5D175394 来自 kernel32.GetModuleHandleA
0012EC8C  |5D1753E0  ASCII "kernel32.dll"
0012EC90  |5D1E2B38  COMCTL32.5D1E2B38
```

