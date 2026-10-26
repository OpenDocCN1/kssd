
# [原创]边锋游戏之《湖州红五记牌器》脱壳+算法分析一条龙

【破文标题】：边锋游戏之《湖州红五记牌器》脱壳+算法分析一条龙

【破文作者】：KuNgBiM[DFCG]

【作者邮箱】：gb\_1227@163.com

【软件名称】：边锋游戏之《湖州红五记牌器》

【软件大小】：377 KB

【软件类别】：国产软件/共享软件/游戏外挂

【整理时间】：2005-5-19

【下载地址】：[本地下载](https://bbs.kanxue.com/bbs.pediy.com/upload/2005/4/files/hzred5.rar)

【软件简介】：发第一张牌的时候你就知道可以亮什么主及每门花色会有几张主。

【保护方式】：注册码 + 使用次数限制

【加密保护】：ASPack 2.12 -> Alexey Solodovnikov

【编译语言】：Borland Delphi 6.0 - 7.0

【调试环境】：WinXP、PEiD、Ollydbg、LordPE、ImportREC、W32Dasm

【破解日期】：2005-06-02

【破解目的】：推广使用ESP定律脱壳，以及研究算法分析

【作者声明】：初学Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

―――――――――――――――――――――――――――――――――

## \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\ 【脱壳过程】 \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

侦壳脱壳：用`PEiD`查壳，`ASPack 2.12 -> Alexey Solodovnikov`加壳。

使用法宝：我们既然知道了是`ASPack`所加壳保护的，所以拿出`Ollydbg`结合文章题目手动脱之~~

――――――――――――――――――――

`Ollydbg`载入主程序：

```
0052C001 >  60              pushad                 //停在这里，F8一次
0052C002    E8 03000000     call hzred5.0052C00A   //来到这里，这时查看寄存器窗口
0052C007  - E9 EB045D45     jmp 45AFC4F7
0052C00C    55              push ebp
0052C00D    C3              retn
```

\\\\\\\\\\\\\\\寄存器\\\\\\\\\\\\\\\\

EAX `00000000`

ECX `00010101`

EDX `FFFFFFFF`

EBX `7FFDF000`

ESP `0012FFA4`       //`esp=0012ffa4`

EBP `0012FFF0`

ESI `00000000`

EDI `00000000`

\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

根据ESP定律规则，现在在命令栏中下 `hr 0012ffa4` 命令，回车，F9运行：

```
0052C3B0   /75 08           jnz short hzred5.0052C3BA    //这里断下，继续F8
0052C3B2   |B8 01000000     mov eax,1
0052C3B7   |C2 0C00         retn 0C
0052C3BA   \68 7C654C00     push hzred5.004C657C         //这里004C657C所指的就是OEP，F8继续走
0052C3BF    C3              retn                         //飞向光明之颠~~ F8继续走
```

```
004C657C    55              push ebp                     //飞向这里，程序入口，Dump！
004C657D    8BEC            mov ebp,esp
004C657F    83C4 F0         add esp,-10
004C6582    53              push ebx
004C6583    B8 E4624C00     mov eax,hzred5.004C62E4
004C6588    E8 8701F4FF     call hzred5.00406714
004C658D    8B1D 848F4C00   mov ebx,dword ptr ds:[4C8F84]               ; hzred5.004CAC30
004C6593    8B03            mov eax,dword ptr ds:[ebx]
004C6595    E8 02B6F9FF     call hzred5.00461B9C
004C659A    8B03            mov eax,dword ptr ds:[ebx]
004C659C    BA F4654C00     mov edx,hzred5.004C65F4
004C65A1    E8 1AB2F9FF     call hzred5.004617C0
004C65A6    8B0D A0904C00   mov ecx,dword ptr ds:[4C90A0]               ; hzred5.004CAD80
004C65AC    8B03            mov eax,dword ptr ds:[ebx]
004C65AE    8B15 34214C00   mov edx,dword ptr ds:[4C2134]               ; hzred5.004C2180
.......
```

脱壳修复：运行`LordPE`，Dump整个进程，然后打开`ImportREC`找到该程序对应进程，OEP填`000C657C`，“自动搜索IAT”，“获取输入表”，

删去一个无效指针，其余指针全部有效，“修复抓取文件”，OK，脱壳修复完成！运行成功！Fix Dump!!

用`PEiD`再次检测，程序为`Borland Delphi 6.0 - 7.0`所编译，优化一下，原始：377 KB -->脱壳优化：1.10 MB

―――――――――――――――――――――――――――――――――

## \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\ 【破解过程】 \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

试探：运行脱壳后的主程序注册，输入试炼码，确认！程序提示：" 注册码错误! "

初步下药：用`W32Dasm`进行静态反汇编，查找 " 注册码错误! " 字符串，找到`004C1F9C`处，确定注册环节应从`004C1E20`处开始。

对症下药：`Ollydbg`重新载入脱壳后的主程序，向上来到 `004C1E20` 处下断，F9运行，输入试炼信息：

\*\*\*\*\* 试炼信息 \*\*\*\*\*\*

机器码：`000AEB97B625`

试炼码：`9876543210`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

点击确定后OD断下:

```
004C1E20    6A 00           push 0
004C1E22    6A 00           push 0
004C1E24    49              dec ecx                               //ecx=5
004C1E25  ^ 75 F9           jnz short dumped\_.004C1E20            //程序向上检测5次
004C1E27    53              push ebx
004C1E28    56              push esi
004C1E29    8BD8            mov ebx,eax
004C1E2B    33C0            xor eax,eax
004C1E2D    55              push ebp
004C1E2E    68 16204C00     push dumped\_.004C2016
004C1E33    64:FF30         push dword ptr fs:[eax]
004C1E36    64:8920         mov dword ptr fs:[eax],esp
004C1E39    8D55 FC         lea edx,dword ptr ss:[ebp-4]
004C1E3C    8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]
004C1E42    E8 75F0F7FF     call dumped\_.00440EBC                 //取试炼码
004C1E47    837D FC 00      cmp dword ptr ss:[ebp-4],0            //比较序列号是否为0
004C1E4B    75 1E           jnz short dumped\_.004C1E6B            //为0则跳死！
004C1E4D    6A 30           push 30
004C1E4F    68 24204C00     push dumped\_.004C2024                 //提示“未输入注册码！”
004C1E54    68 2C204C00     push dumped\_.004C202C
004C1E59    8BC3            mov eax,ebx
004C1E5B    E8 0858F8FF     call dumped\_.00447668
004C1E60    50              push eax
004C1E61    E8 CE52F4FF     call dumped\_.00407134                 //调用通用对话框函数，MessageBoxA
004C1E66    E9 50010000     jmp dumped\_.004C1FBB
004C1E6B    8D55 F4         lea edx,dword ptr ss:[ebp-C]
004C1E6E    8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]
004C1E74    E8 43F0F7FF     call dumped\_.00440EBC                 //取试炼码
004C1E79    8B45 F4         mov eax,dword ptr ss:[ebp-C]          //ASCII "9876543210"
004C1E7C    8D55 F8         lea edx,dword ptr ss:[ebp-8]
004C1E7F    E8 EC68F4FF     call dumped\_.00408770
004C1E84    8B45 F8         mov eax,dword ptr ss:[ebp-8]
004C1E87    50              push eax                              //试炼码压栈，ASCII "9876543210"
004C1E88    8D55 E8         lea edx,dword ptr ss:[ebp-18]         //edx清零
004C1E8B    8B83 F4020000   mov eax,dword ptr ds:[ebx+2F4]
004C1E91    E8 26F0F7FF     call dumped\_.00440EBC                 //取机器码
004C1E96    8B45 E8         mov eax,dword ptr ss:[ebp-18]         //ASCII "000AEB97B625"
004C1E99    8D55 EC         lea edx,dword ptr ss:[ebp-14]
004C1E9C    E8 CF68F4FF     call dumped\_.00408770
004C1EA1    8B45 EC         mov eax,dword ptr ss:[ebp-14]         //机器码入栈，ASCII "000AEB97B625"
004C1EA4    8D55 F0         lea edx,dword ptr ss:[ebp-10]         //edx清零
004C1EA7    E8 74FCFFFF     call dumped\_.004C1B20                 //算法CALL，F7跟进！
004C1EAC    8B55 F0         mov edx,dword ptr ss:[ebp-10]         //真码出现，ASCII "aaarvsjvsccx"
004C1EAF    58              pop eax                               //eax清零
004C1EB0    E8 4B28F4FF     call dumped\_.00404700                 //经典比对CALL
004C1EB5    0F85 DA000000   jnz dumped\_.004C1F95                  //爆破点
004C1EBB    B2 01           mov dl,1
004C1EBD    A1 AC8B4800     mov eax,dword ptr ds:[488BAC]
004C1EC2    E8 E56DFCFF     call dumped\_.00488CAC
004C1EC7    8BF0            mov esi,eax
004C1EC9    BA 02000080     mov edx,80000002
004C1ECE    8BC6            mov eax,esi
004C1ED0    E8 776EFCFF     call dumped\_.00488D4C
004C1ED5    B1 01           mov cl,1
004C1ED7    BA 44204C00     mov edx,dumped\_.004C2044              //注册信息保存位置，ASCII "SOFTWARE\Microsoft\hzred5"
004C1EDC    8BC6            mov eax,esi
004C1EDE    E8 A96FFCFF     call dumped\_.00488E8C
004C1EE3    B9 01000000     mov ecx,1
004C1EE8    BA 68204C00     mov edx,dumped\_.004C2068              //注册表里保存机器码位置，ASCII "hzred5reg"
004C1EED    8BC6            mov eax,esi
004C1EEF    E8 3871FCFF     call dumped\_.0048902C
004C1EF4    8D55 E0         lea edx,dword ptr ss:[ebp-20]
004C1EF7    8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]
004C1EFD    E8 BAEFF7FF     call dumped\_.00440EBC
004C1F02    8B45 E0         mov eax,dword ptr ss:[ebp-20]
004C1F05    8D55 E4         lea edx,dword ptr ss:[ebp-1C]
004C1F08    E8 6368F4FF     call dumped\_.00408770
004C1F0D    8B4D E4         mov ecx,dword ptr ss:[ebp-1C]
004C1F10    BA 7C204C00     mov edx,dumped\_.004C207C              //注册表里保存注册码位置，ASCII "red5sn"
004C1F15    8BC6            mov eax,esi
004C1F17    E8 E470FCFF     call dumped\_.00489000
004C1F1C    8D55 D8         lea edx,dword ptr ss:[ebp-28]
004C1F1F    8B83 F4020000   mov eax,dword ptr ds:[ebx+2F4]
```
