

# ACDSEE3.1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue391.htm`](https://www.pediy.com/kssd/pediy04/kanxue391.htm)

ACDSEE3.1 的 90 天与 9 分钟   
\----ACDSEE3.1Trial Version build921 时间限制破解手记   
《电脑报》2000 年合订本光盘上所带 ACDSEE3.1 有 90 天时间限制，最近应朋友之托只有为其破解（到处都能找到注册版，干嘛还用它？）！在此将破解过程简单说一下，希望有人能喜欢！   
  
使用工具：冲击波（BW2000），TRW2000，W32Dasm8.93 黄金版，Hiew6.40。   
  
因为 ACDSEE 加了壳，所以先脱壳。   
1.运行 BW2000，按下 Track 按钮，然后运行 ACDSEE,BW2000 显示 Entry point 为 004AD2FE。   
2.运行 TRW2000，下 BPX 4AD2FE 命令设断点，F5 返回 Windows。运行 ACDSEE，程序会断在 004AD2FE 一行，这就是程序入口点了，下命令 PEDUMP C:\\1.exe，这样便在 c 盘根目录下生成一可执行文件 1.exe，这就是脱壳后的程序。将其拷贝到 ACDSEE 安装目录备用。   
3.关闭 ACDSEE。   
  
因为每次启动都会提示使用天数还剩几天，所以反汇编看有没有什么希望。   
1.W32Dasm 反汇编 1.exe，在其中查找文本 days（因为每次启动都会提示使用天数还剩几天就是 xx days）找到好多处，其中有一处比较可疑（我是怎么知道是这里?凭感觉了。即使感觉错误也可以再试其他的嘛，哈哈^\_^），就是下面   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404558(C)   
|   
\* Reference To: InTouchClient.?TrialPeriod\_GetDaysLeft@InTouchClient@@QAEHXZ, Ord:005Dh   
|                                              |-----（此处找到这个 days）   
:00404596 FF1564824D00            Call dword ptr \[004D8264\]   
:0040459C 85C0                    test eax, eax--------------这里保存剩余天数   
:0040459E 7F16                    jg 004045B6----------------这里只要跳就可以运行程序   
2.Ctrl-n，下命令 BPX 40459C，F5 返回 Windows,再次运行 ACDSEE，TRW 会拦在 0040459C 处，下 D EAX 命令，看到一个 5A（5A 是什么，就是十进制的 90，就是还可以使用的天数了）。F5 让程序运行完，关闭 ACDSEE。   
3.为了验证前面的猜想，将系统时间调过一年（只要大于 90 天就行），再运行 ACDSEE。   
4.依然拦在 0040459C 处，下 D EAX 命令，看到一个 0，F5 让程序继续运行，此时不能进入程序主界面。   
5.修改程序，用 Hiew 打开 11.exe，F4 选择 Decode 模式，F5 填入 459E 就到了关键处，F3 修改为如下代码：   
:0040459E E913000000              jmp 004045B6   
6.将 1.exe 改名为 ACDSEE.exe（如果觉得脱壳程序太大可用 UPX 之类的压缩程序压缩一下），并复制到 ACDSEE 安装目录覆盖原文件，OK，永不过期！   
  
计时：脱壳 1 分钟，反汇编 2 分钟（受机器配置限制，否则可更快），查找关键点 1 分钟，验证关键点 3 分钟（受打字速度限制，否则可更快），修改程序并完成工作 2 分钟，累计用时 9 分钟。   
  
总结发言：大家好，俺是菜鸟一个，Crack 的水平嘛，一般般了！破了这个程序，可是让我那位“菜菜鸟”朋友佩服得五体投地，俺也赚足了面子，一个字：挺不错的！   
  
esoft2001.51.net   
2002 年 2 月 7 日  4dd466d0e1e0cc123afbe504da2462ca