
# [原创]时间提醒助手20040425 算法分析

时间提醒助手20040425 算法分析

日期：2005年3月9日　　　破解人：Baby2008

---

【软件名称】：时间提醒助手20040425

【软件大小】：2.26M

【下载地址】：c48K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2&6y4e0x3H3i4K6u0W2j5$3!0E0

## 【软件简介】：

“时间提醒助手”是一款个性化的事务提醒及定时运行程序软件。有以下功能：

1. 农历日历，方便查看农历各个节日；
2. 指定时间、每天、每逢周一、周二、周三、周四、周五、周六、周日、周一至周五提醒；
3. 整点报时，声音提醒，多种好听提醒声音；
4. 界面方案，提供多个性化界面选择。
5. 定时运行程序，选择外部程序指定时间运行它；
6. 定时关机，可选择关机方案、指定时间关机。支持Win9x/2000/ME/NT/XP操作系统。
7. 启动方式，最小化或隐藏，让您使用更容易、更快捷；
8. 在线升级，完整的在线服务功能，只点击升级，您的软件自动升级，不需要下载安装程序来升级。
9. 提交信息，把您的信息直接提交到internet上，我们更好的为您服务，及时了解最新信息，如：有新版本、新软件通知。
10. 临时备忘录，记录您的备忘录信息，代替手写备忘录。

在办公室经常要做的事情，将安排好的备忘录写便条上或日历格子上有时却忘记去看了？上Internet网要上N小时、什么时候下班、什么时间约会 、什么时候打电话给哪位男生/女生、什么时间让电脑自动关机……

这一切，“时间提醒助手”会帮您解决这所有的问题，她拥有强大的各种事务提醒功能，以提高效率并节省时间和费用。

【软件限制】：未注册30天试用限制

【破解声明】：初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：OLLYDBG 1.1 ，PEID0.92，DeDe,Quick Unpack 0.6

---

## 【破解过程】：

先用PEID 0.93汉化增强版查壳，ASPack 2.12 -> Alexey Solodovnikov，用Quick Unpack 0.6轻松脱去，默认另存为`TimeHelper\_\_.exe`

脱壳后再用PEID 0.93汉化增强版查壳，Borland Delphi 6.0 - 7.0

delphi程序，先用Dede处理，输出map文件

用od打开目标程序：`TimeHelper\_\_.exe` ，用mapconv插件引入Dede输出的map文件，替换标签及插件。然后查找字符，可以找到“注册注册成功，谢谢你的支持”字样，双击来到cpu窗口，来到0054AC46 向上翻看可以找到这里：以下就是具体算法

```asm
0054AB78 <>/.  55                      push ebp                             ;  <-TFrmRegistry@SpeedButton4Click
0054AB79   |.  8BEC                    mov ebp,esp
0054AB7B   |.  33C9                    xor ecx,ecx
0054AB7D   |.  51                      push ecx
0054AB7E   |.  51                      push ecx
0054AB7F   |.  51                      push ecx
0054AB80   |.  51                      push ecx
0054AB81   |.  51                      push ecx
0054AB82   |.  51                      push ecx
0054AB83   |.  51                      push ecx
0054AB84   |.  53                      push ebx
0054AB85   |.  56                      push esi
0054AB86   |.  8BD8                    mov ebx,eax
0054AB88   |.  33C0                    xor eax,eax
0054AB8A   |.  55                      push ebp
0054AB8B   |.  68 55AD5400             push <TimeHelp.->System.@HandleFinal>
0054AB90   |.  64:FF30                 push dword ptr fs:[eax]
0054AB93   |.  64:8920                 mov dword ptr fs:[eax],esp
0054AB96   |.  8D55 FC                 lea edx,dword ptr ss:[ebp-4]
0054AB99 <>|.  8B83 2C030000           mov eax,dword ptr ds:[ebx+32C]       ;  \*TFrmRegistry.Edit3:TEdit
0054AB9F <>|.  E8 38EBEFFF             call TimeHelp.004496DC               ;  取试炼码
->Controls.TControl.GetText(TControl):TCaption;
0054ABA4   |.  837D FC 00              cmp dword ptr ss:[ebp-4],0           ;  试炼码不能为空
0054ABA8   |.  0F84 3C010000           je TimeHelp.0054ACEA
0054ABAE   |.  8D55 F8                 lea edx,dword ptr ss:[ebp-8]
0054ABB1 <>|.  8B83 2C030000           mov eax,dword ptr ds:[ebx+32C]       ;  \*TFrmRegistry.Edit3:TEdit
0054ABB7 <>|.  E8 20EBEFFF             call TimeHelp.004496DC               ;
->Controls.TControl.GetText(TControl):TCaption;
0054ABBC   |.  8B45 F8                 mov eax,dword ptr ss:[ebp-8]         ;  取试炼码
0054ABBF   |.  50                      push eax
0054ABC0   |.  68 6CAD5400             push TimeHelp.0054AD6C               ;  ASCII "aB"
0054ABC5   |.  8D55 EC                 lea edx,dword ptr ss:[ebp-14]
0054ABC8 <>|.  8B83 30030000           mov eax,dword ptr ds:[ebx+330]       ;  \*TFrmRegistry.Edit2:TEdit
0054ABCE <>|.  E8 09EBEFFF             call TimeHelp.004496DC               ;
->Controls.TControl.GetText(TControl):TCaption;
0054ABD3   |.  8B45 EC                 mov eax,dword ptr ss:[ebp-14]        ;  机器码
0054ABD6 <>|.  E8 4DEEEBFF             call TimeHelp.00409A28               ;  将机器码转换成16进制字符串
0054ABDB   |.  8D4D F0                 lea ecx,dword ptr ss:[ebp-10]
0054ABDE   |.  BA 08000000             mov edx,8
0054ABE3 <>|.  E8 C4EDEBFF             call TimeHelp.004099AC               ;  ->Unit\_004085A8.Proc\_004099AC
0054ABE8   |.  FF75 F0                 push dword ptr ss:[ebp-10]           ;  机器码16进制字符串
0054ABEB   |.  68 78AD5400             push TimeHelp.0054AD78               ;  ASCII "Cd"
0054ABF0   |.  8D45 F4                 lea eax,dword ptr ss:[ebp-C]
0054ABF3   |.  BA 03000000             mov edx,3
0054ABF8 <>|.  E8 7FA3EBFF             call TimeHelp.00404F7C               ;  ->System.Proc\_00404F7C
0054ABFD   |.  8B55 F4                 mov edx,dword ptr ss:[ebp-C]         ;  'aB'＋机器码16进制字符串＋'Cd'
0054AC00   |.  A1 98FA5500             mov eax,dword ptr ds:[55FA98]
0054AC05   |.  8B00                    mov eax,dword ptr ds:[eax]
0054AC07 <>|.  8B80 58030000           mov eax,dword ptr ds:[eax+358]       ;  \*TFrmTimeHelper.Reg:TRegwareII
0054AC0D   |.  B9 84AD5400             mov ecx,TimeHelp.0054AD84            ;  ASCII "my530.com"
0054AC12 <>|.  E8 919CF4FF             call TimeHelp.004948A8               ; 关键，跟进
0054AC17   |.  84C0                    test al,al
0054AC19   |.  0F84 B1000000           je TimeHelp.0054ACD0
0054AC1F   |.  BA 98AD5400             mov edx,TimeHelp.0054AD98
0054AC24   |.  A1 B8D75600             mov eax,dword ptr ds:[56D7B8]
0054AC29 <>|.  E8 DEEAEFFF             call TimeHelp.0044970C               ;
->Controls.TControl.SetText(TControl;TCaption);
0054AC2E   |.  A1 98FA5500             mov eax,dword ptr ds:[55FA98]
0054AC33   |.  8B00                    mov eax,dword ptr ds:[eax]
0054AC35   |.  BA B0AD5400             mov edx,TimeHelp.0054ADB0
0054AC3A <>|.  E8 CDEAEFFF             call TimeHelp.0044970C               ;
->Controls.TControl.SetText(TControl;TCaption);
0054AC3F   |.  6A 40                   push 40
0054AC41   |.  B9 C0AD5400             mov ecx,TimeHelp.0054ADC0
0054AC46   |.  BA C8AD5400             mov edx,TimeHelp.0054ADC8
call TimeHelp.004948A8
```

---

```asm
004948A8   /$  55                      push ebp
004948A9   |.  8BEC                    mov ebp,esp
004948AB   |.  83C4 F8                 add esp,-8
004948AE   |.  53                      push ebx
004948AF   |.  894D F8                 mov dword ptr ss:[ebp-8],ecx         ;  my530.com
004948B2   |.  8955 FC                 mov dword ptr ss:[ebp-4],edx
004948B5   |.  8BD8                    mov ebx,eax
004948B7   |.  8B45 FC                 mov eax,dword ptr ss:[ebp-4]
004948BA   |.  E8 E507F7FF             call TimeHelp.004050A4
004948BF   |.  8B45 F8                 mov eax,dword ptr ss:[ebp-8]
004948C2   |.  E8 DD07F7FF             call TimeHelp.004050A4
004948C7   |.  8B45 08                 mov eax,dword ptr ss:[ebp+8]         ;  试炼码
004948CA   |.  E8 D507F7FF             call TimeHelp.004050A4
004948CF   |.  33C0                    xor eax,eax
004948D1   |.  55                      push ebp
004948D2   |.  68 15494900             push TimeHelp.00494915
004948D7   |.  64:FF30                 push dword ptr fs:[eax]
004948DA   |.  64:8920                 mov dword ptr fs:[eax],esp
004948DD   |.  8B45 08                 mov eax,dword ptr ss:[ebp+8]
004948E0   |.  50                      push eax
004948E1   |.  6A 00                   push 0
004948E3   |.  8B4D F8                 mov ecx,dword ptr ss:[ebp-8]         ;  my530.com
004948E6   |.  8B55 FC                 mov edx,dword ptr ss:[ebp-4]         ;  'aB'＋机器码16进制字符串＋'Cd'
004948E9   |.  8BC3                    mov eax,ebx
004948EB   |.  E8 38000000             call TimeHelp.00494928               ;关键，跟进
004948F0   |.  8BD8                    mov ebx,eax
004948F2   |.  33C0                    xor eax,eax
004948F4   |.  5A                      pop edx
004948F5   |.  59                      pop ecx
004948F6   |.  59                      pop ecx
004948F7   |.  64:8910                 mov dword ptr fs:[eax],edx
004948FA   |.  68 1C494900             push TimeHelp.0049491C
004948FF   |>  8D45 F8                 lea eax,dword ptr ss:[ebp-8]
```
