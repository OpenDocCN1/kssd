
# 易语言 之 超想家电维修管理系统 v4.5 破解分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-401.htm`](https://www.pediy.com/kssd/pediy07/pediy7-401.htm)

## 基本信息

- **【破文标题】**: 易语言 之 超想家电维修管理系统 v4.5 破解分析
- **【破文作者】**: KuNgBiM\[DFCG\]
- **【作者邮箱】**: gb\_1227@163.com
- **【软件名称】**: 超想家电维修管理系统 v4.5
- **【软件大小】**: 1.68 MB
- **【软件类别】**: 国产软件/共享软件/行业管理
- **【整理时间】**: 2005-04-27
- **【下载地址】**: [`www.sharebank.com.cn/soft/soft\_view.php?id=12699`](http://www.sharebank.com.cn/soft/soft_view.php?id=12699)

### 软件简介

随着家电行业的不断发展服务质量的管理是一项不容忽视的重要工作，那么信息管理就是毕不可少了，还有随着家电的技术不断更新，资料的管理更是重中之重，本软件就是将信息管理与资料管理容为一体的综合管理软件。采用凭证打印式派工、各类结算明细式记帐、配件各类明细打印、及多功能报表式管理。

该系统通过对平时服务过的信息、配件的进出帐、员工的服务记录、的收集可以方便的了解动态状况，做出合理的管理制度与服务过程中的协调。

资料的收集做到多而不乱，方便快捷的资料查找让你分分秒秒成为金钱。将不同的资料用了不同的库，这样资料分的是一清二楚点滴不乱。通俗易懂的界面，让你使用更加方便，采用键盘快捷输入方式让你工作的更加轻松。

- **【保护方式】**: 注册码 + 试用次数限制
- **【加密保护】**: 无
- **【编译语言】**: 易语言 (伪装 `Microsoft Visual C++ 6.0 [Overlay]` )
- **【调试环境】**: `WinXP`、`PEiD`、`Ollydbg`、`Keymake2.0` 修改版
- **【破解日期】**: 2005-08-03
- **【破解目的】**: 研究学习易语言程序逆向分析
- **【作者声明】**: 初学 `Crack`，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

---

## 分析过程

1. 运行程序，东看看西看看，查找程序有那些功能限制和对我们有用的信息。
2. 用 `PEiD` 查壳，`Microsoft Visual C++ 6.0 [Overlay]`，`Ollydbg` 载入只找到一些无用的连接库信息，难道是伪装程序？再深一步分析(察看区段情况)：

| No | Name | VSize | VOffset | RSize | ROffset | Charact. |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 01 | `.text` | 00004D9C | 00001000 | 00005000 | 00001000 | 60000020 |
| 02 | `.rdata` | 00000A4A | 00006000 | 00001000 | 00006000 | 40000040 |
| 03 | `.data` | 00001F58 | 00007000 | 00002000 | 00007000 | C0000040 |
| 04 | `.ecode` | 00132000 | 00009000 | 00132000 | 00009000 | E0000040 | <-- 哈哈，就是这里暴露了目标（易语言程序标准区段名）
| 05 | `.rsrc` | 00000960 | 0013B000 | 00001000 | 0013B000 | 40000040 |

从区段名来看，我们发现第四区段的区段名为“`.ecode`”，这不是易语言的标准区段名么？哈哈，“狐狸尾巴”终于露出来了，易程序最主要是 `Dump` 易格式原体，到达代码段后才好我们下一步分析。

3. 用 `Ollydbg` 载入，`Dump` 易格式原体。

### Dump 易格式原体过程

这里使用的是我总结的【区段“内存断点”大法 `Dump` 易格式原体】：

`Ollydbg` 载入主程序：

```
00403831 >   55                    push ebp                            ; 载入程序后停在这里，这就是伪装 VC6 的 PE 头
00403832     8BEC                  mov ebp,esp
00403834     6A FF                 push -1
00403836     68 F0624000           push 超想家电.004062F0
0040383B     68 A44C4000           push 超想家电.00404CA4
00403840     64:A1 00000000        mov eax,dword ptr fs:[0]
00403846     50                    push eax
00403847     64:8925 00000000      mov dword ptr fs:[0],esp
0040384E     83EC 58               sub esp,58
00403851     53                    push ebx
00403852     56                    push esi
00403853     57                    push edi
00403854     8965 E8               mov dword ptr ss:[ebp-18],esp
00403857     FF15 48604000         call dword ptr ds:[<&KERNEL32.GetVersion>]    ; kernel32.GetVersion
0040385D     33D2                  xor edx,edx
0040385F     8AD4                  mov dl,ah
00403861     8915 6C8A4000         mov dword ptr ds:[408A6C],edx
00403867     8BC8                  mov ecx,eax
........
```

用 `Alt + M` 打开内存映射，找到“`.ecode`”区段，并设置内存访问断点，访问此区段，`F9` 运行：

```
100289DD     8B42 30               mov eax,dword ptr ds:[edx+30]           ; 这里断下，继续 F8 单步运行
100289E0     83E0 01               and eax,1
100289E3     85C0                  test eax,eax
100289E5     75 10                 jnz short krnln.100289F7
100289E7     8B4D 08               mov ecx,dword ptr ss:[ebp+8]            ; 易格式的验证字符串，ASCII "WTNE / MADE BY E COMPILER - WUTAO"
100289EA     51                    push ecx
100289EB     8B4D F8               mov ecx,dword ptr ss:[ebp-8]
100289EE     E8 FDEF0200           call krnln.100579F0                    ; 装载易格式核心库
100289F3     FFE0                  jmp eax                                 ; 易格式已经装载完成了，转交
100289F5     EB 0E                 jmp short krnln.10028A05
100289F7     8B55 08               mov edx,dword ptr ss:[ebp+8]
100289FA     52                    push edx
100289FB     8B4D F8               mov ecx,dword ptr ss:[ebp-8]
........
00530E69     FC                    cld                                     ; 易格式代码的起始位置
00530E6A     DBE3                  finit
00530E6C     E8 60FEFFFF           call 超想家电.00530CD1
00530E71     68 890D5300           push 超想家电.00530D89
00530E76     B8 03000000           mov eax,3
00530E7B     E8 31000000           call 超想家电.00530EB1
00530E80     83C4 04               add esp,4
00530E83     68 01000152           push 52010001
00530E88     E8 1E000000           call 超想家电.00530EAB
00530E8D     83C4 04               add esp,4
00530E90     6A 00                 push 0
00530E92     E8 0E000000           call 超想家电.00530EA5
00530E97     E8 03000000           call 超想家电.00530E9F
00530E9C     83C4 04               add esp,4
00530E9F  -  FF25 8E374C00         jmp dword ptr ds:[4C378E]               ; krnln.10029304
00530EA5  -  FF25 92374C00         jmp dword ptr ds:[4C3792]               ; krnln.1002926F
00530EAB  -  FF25 96374C00         jmp dword ptr ds:[4C3796]               ; krnln.10029299
00530EB1  -  FF25 9A374C00         jmp dword ptr ds:[4C379A]               ; krnln.10028AB5
00530EB7  -  FF25 8A374C00         jmp dword ptr ds:[4C378A]               ; krnln.100293CC
00530EBD  -  FF25 82374C00         jmp dword ptr ds:[4C3782]               ; krnln.100292C4
00530EC3  -  FF25 7E374C00         jmp dword ptr ds:[4C377E]               ; krnln.10029248
00530EC9  -  FF25 76374C00         jmp dword ptr ds:[4C3776]               ; krnln.10028B31
00530ECF  -  FF25 7A374C00         jmp dword ptr ds:[4C377A]               ; krnln.10029229
00530ED5  -  FF25 6A374C00         jmp dword ptr ds:[4C376A]               ; krnln.1002920B
00530EDB  -  FF25 72374C00         jmp dword ptr ds:[4C3772]               ; krnln.10028ADA
00530EE1  -  FF25 6E374C00         jmp dword ptr ds:[4C376E]               ; krnln.10028ACF
00530EE7     3800                  cmp byte ptr ds:[eax],al
00530EE9     0000                  add byte ptr ds:[eax],al
00530EEB     1F                    pop ds
........
```

好了，`Look`！我们是不是很容易就把易格式原体文件 `Dump` 出来了，所以下一步我们就能很方便的分析它了。

## 破解过程

接着上面的文章，继续在 `OD` 右键使用 `Ultra String Reference` 插件的 `Find ASCII` 功能项，查找我们需要的相关信息：

```
........
004F2985   push 超想家电.0040948A                SOFTWARE\
004F2A78   push 超想家电.00415968                注册成功，欢迎使用!!!             ★重要提示信息①★
004F2ACE   push 超想家电.0041597E                未注册成功，写入注册表失败!     ★重要提示信息★
004F2B03   push 超想家电.0041599A                注册码错误                       ★重要提示信息②★
004F2C86   push 超想家电.004099CC                当前没有用户资料，请查找到资料再修改吧！
........
```

在“★重要提示信息★①、②”处分别双击，然后在可疑的地址 `F2` 下断：

```
\=======================
注册试炼信息：
硬件码：1347726194
注册码：78787878
\=======================

004F2806     50                    push eax                            ; 此处下断,F9 运行，填写注册相关信息！ ^_^
004F2807     68 03000000           push 3
004F280C     BB 3C010000           mov ebx,13C
004F2811     E8 B3E60300           call 超想家电.00530EC9
004F2816     83C4 28               add esp,28
```
