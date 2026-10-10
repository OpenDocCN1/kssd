

# some cheats in NP HS (代码在 10 楼)

> 原文：[`www.pediy.com/kssd/pediy10/59359.html`](https://www.pediy.com/kssd/pediy10/59359.html)

先  
祝各位  在鼠年喜气洋洋，满面阳光灿烂，爱情扬眉吐气，事业洋洋得意，晦气扬长而去，万事阳关大道！  
鼠属鼠，财源“鼠”不胜数  
人喜盛世;鼠兆丰年  
"鼠"不尽的快乐！  
"鼠"不尽的收获！  
"鼠"不尽的钞票！  
"鼠"不尽的笑容！  
"鼠"不尽的幸福！  
"鼠"不尽的朋友！  
"鼠"不尽的喜气！  
"鼠"不尽的财富！  
"鼠"不尽的甜蜜！  
"鼠"不尽的激情！  
"鼠"不尽的活力！  
"鼠"不尽的健康！  
"鼠"不尽的荣誉！   
祝贺词由 lin 版提供,大家不要忘记祝贺 lin 版阿:)   
  
   文章已经写过 2 篇了,也和大家学习了一些知识,先做 1 点点小小个人声明  
1.偶不是做游戏的,也不是做外挂的,研究 NP 没有什么商业目的,也许一些人不信  
不过,可以想想,我写这些东西出来只会使 NP 保护加难,不会有任何好处  
写文章其实是个学习的过程,我写的时候发现很多东西不懂,又得去查询资料  
写代码的时候是直接拿 windbg 出来的硬编码 hook,当然不能通用  
为了写文章可以学习到很多知识,这也是偶写出来的原因了![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")   
2.为啥要分析 NP?  这个,其实很简单,玩驱动总想拿点东西来分析 1 下  
正如玩壳的喜欢找点东西来拆了分析一样,为啥偶选择了 NP 呢,因为没有  
选择了阿,![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑") 如果偶把卡巴主动防御驱动部分一个一个拆开分析,岂不会  
得到个做病毒的称号?更有可能指责偶成了谋害广大网民了,所以,哈哈.  
分析 NP 最多就说偶是做挂了,不可能"谋害":lol 反正从技术本质上都是  
hook unhook checkhook 一类的东西  
3.NP 有一些小把戏,比如隐藏进程什么的,记住是断开链表就 ok 了  
我们去搜索 eprocess,自己恢复链表,具体代码很简单  
,偶从开始分析 NP 直到实现挂 od 飞 np,一共 5 天,比起 forgot 大侠 2 天  
来说是慢多了,不过还好,实现了目标,也就是所谓的,forgot 大侠说的  
加入 CUG 的必须会挂 od 飞 np:handshake   
4.其实 NP 很多大牛已经给了我们很多提示,比如阿莫西林同学,  
那张图片不知道大家看过没有,偶是仔细看了的,还算不错  
按照那个方法是可以的,相信你自己,偶一个菜鸟花了 5 天  
你应该比偶聪明,参考一些代码,3-4 天应该没问题了吧  
5.正因为偶不是做外挂的,所以偶不关心游戏本身,偶的文章尽力避开  
直接提到游戏,正如我们研究加壳保护尽力不提及目标软件一样,断开 np  
以后游戏是否会掉线?会异常?偶不关心,偶的目的就是 od 能够直接进入 np 进程了  
即可,  
6.听说最新更新 np 添加了 1 个什么主动防御打分机制?没有仔细看了,不过也不是什么  
难事情,关键还是一个理念问题,思路的灵巧,还有方法的多变,代码不是最重要的  
明白了理念,其实代码很好写  
7.总结 1 下那篇没有完成的小说吧,其实也不可能完成  
至于 NP 和 HS 谁厉害?仁者见仁智者见智吧,不过偶猜测 2 个一起运行没有问题  
不过偶没有做实验.因为偶的虚拟机跑不起;开 2 个游戏直接死机  
哪位大侠有兴趣可以试试.  
8.下面还要说什么?偶的 id?声明 1 下 不是外挂的意思  
大家记住 猪娃哥,就行了,意思就是喂猪的:lol   
9.貌似又灌水了很多,恩,最后还有 1 个东西,HS 的 int3 挂钩  
之前先提及 1 下一些 NP 的邪恶东西,还有一些关 kerneldebuger  
监控 0x60 64 之类的,不多说了,因为偶也没有仔细测试  
只说 0x60 64 发个特殊电位可以直接给你重启  
原因就是改变 reset 电位,键盘控制器可以直接改变 reset 电位,就像你按下  
机箱上的 reset 一样的效果,没有任何过程的直接重启  
  
先注意 1 下,下面东西又是很难的了..:loveliness:   
什么是中断?中断和异常(exception) 中断向量   
"中断被定义为当一个事件发生时，改变处理器的指令序列。这样的事件可由 CPU 芯片  
内部或者外部硬件产生电信号产生"  
Intel 参考手册上指出“同步中断”(在一个指令执行完成后，由 CPU 控制单元产生的)作为“异常”。  
异步中断（可能会在任意时刻由其他硬件产生的）才称为“中断”。中断被外部的 I/O 设备产生。  
但是异常是由编程错误或者是由反常情况（必须由内核来处理）触发的。在该文档中，  
术语“中断信号”既指异常又指中断。  
中断分为两种类型：可屏蔽中断--它在短时间片段里可被忽略；不可屏蔽中断--它必须被立即处理。  
不可屏蔽中断是由紧急事件产生例如硬件失败。著名的 IRQS（中断请求）失败划为可屏蔽中断。  
异常被分为不同的两类：处理器产生的异常(Faults, Traps, Aborts)和编程安排的  
异常（用汇编指令 int or int3 触发）。后一种就是我们经常说到的软中断  
  
后面那个就是 int3 int3 就是 CC,也就是我们的 bp 断点.  
  
什么是 IDT?  
  
IDT = Interrupt Descriptor Table 中断描述表  
  
IDT 是一个有 256 个入口的线形表，每个中断向量关联了一个中断处理过程。  
每个 IDT 的入口是个 8 字节的描述符，所以整个 IDT 表的大小为 256\*8=2048 bytes  
IDT 有三种不同的描述符或者说是入口：  
\- 任务门描述符 Task Gate Descriptor  
\- 陷阱门描述符  Trap Gate Descriptor  
\- DPL=Descriptor Privilege Level  
不详细说了,我们看看 r0cmd  
介绍下 r0cmd(不知道有没有类似的工具,有还请告知):  
r0cmd 就是 ring0 下读写内存的工具,主要是用来查看修改 IDT,SST 及相关只能在  
ring0 下才能访问的内存,例如 driver 的内存空间,这个很重要,破解需要修改 driver.  
使用前一定要把它的驱动文件放到 system32\\drivers 目录中,r0cmd 的主要命令:  
1. 读 IDT 表: r0cmd /viewidt:idt.txt 文本方式读出,用于查看;  
2. 读 IDT 表: r0cmd /dumpidt:idt.bin hex 方式读出,用于保存;  
3. 写 IDT 表: r0cmd /loadidt:idt.bin 用于恢复 IDT 表;  
4. 读 SST 表: r0cmd /dumpsst\_nt:sst.bin hex 方式读出,用于保存;  
5. 写 SST 表: r0cmd /loadsst\_nt:sst.bin 用于恢复;  
6. 读内存: r0cmd /dump raw.bin FABC000 1000 (文件/地址/长度)用于 dump 内存分析;  
7. 写内存: r0cmd /load raw.bin FABC000 1 (写内存一般不宜写太多,错误会造成 boot)  
  
  
MJ0011--HackShield 同冰刃存在冲突,问题是 KERNEL\_MODE\_EXCEPTION\_NOT\_HANDLED,异常号是 0x80000004（STATUS\_SINGLE\_STEP),引发错误的地址在 hal 的一个函数 READ\_PROT\_UCHAR 中,经过分析，出错当时 DR7 被 enable,开启了读写断点，DR2 的值是 0x60,即是 HackShield 会在 0x60 号端口上下 IO 硬件断点,而!idt -a 查看 IDT，int 1 & int3 号中断的处理例程都是在系统原始的 KiTrap01 & KiTrap03 上，所以当 READ\_PORT\_UCHAR 执行了 in al ,dx 后，就会发生异常下来，而 int 1 & 3 中处理例程没有被修改，因此直接发生异常，BSOD 了,进一步查看原因， Icesword 会不断重写 int 1 & int3 为系统原始的 KiTrap\*的地址，因此当 HackShield 修改了 int 3 的处理例程，然后设下硬件断点后，冰刃恢复了 int 3 的处理例程，一但再有键盘/鼠标的中断过来，触发了 i8042 中的处理例程调用 hal 中函数读写 0x60 号端口（键盘/鼠标的 input/output buffer 端口)，就会被断下来，而 int 3 已经被恢复，因此当掉  
  
幻影,不知道还有人记得吗?幻影最大的特点可谓是其反跟踪了，幻影的 Anti 能力之强在 Xtreme-Protector 未出之前是首屈一指的。正如 CoDe\_inJect 大侠所言：“这个壳在国内如日中天，奥妙在于哪里？就在于 anti 做得比较完善，它是我见到的最早使用驱动的壳，虽然驱动的运用仅仅是为了在 NT 下切到 Ring0,但他开创了壳使用驱动的先例。对 idt 进行修改，改变了 int 1 and int 3 的地址。”  
  
理论部分完毕,我们知道 int3 会发生 1 个异常,异常处理流程很复杂,..但是会进入中断.  
IDT hook 是这样做的  
  NTSTATUS   AddInterrupt()    
  {    
          PIdtEntry\_t                         IdtEntry;    
     
          /\*   Get   the   Base   and   Limit   of   IDTR   Register   \*/    
          \_asm   sidt   buffer    
          IdtEntry=(PIdtEntry\_t)Idtr->Base;    
     
          /\*   Save   away   the   old   IDT   entry   \*/    
          memcpy(&OldIdtEntry,   &IdtEntry\[ADDINT\],   sizeof(OldIdtEntry));    
       
     
          \_asm   cli    
          /\*   Initialize   the   IDT   entry   according   to   the   interrupt   gate   requirement   \*/    
          IdtEntry\[ADDINT\].OffsetLow=(unsigned   short)InterruptHandler;    
          IdtEntry\[ADDINT\].Selector=8;    
          IdtEntry\[ADDINT\].Reserved=0;    
          IdtEntry\[ADDINT\].Type=0xE;    
          IdtEntry\[ADDINT\].Always0=0;    
          IdtEntry\[ADDINT\].Dpl=3;    
          IdtEntry\[ADDINT\].Present=1;    
          IdtEntry\[ADDINT\].OffsetHigh=(unsigned   short)((unsigned   int)InterruptHandler>>16);    
          \_asm   sti    
     
          return   STATUS\_SUCCESS;    
  }    
     
     
  void   RemoveInterrupt()    
  {    
          PIdtEntry\_t                         IdtEntry;    
     
          /\*   Reach   to   IDT   \*/    
          IdtEntry=(PIdtEntry\_t)Idtr->Base;    
     
          \_asm   cli    
          /\*   Restore   the   old   IdtEntry   \*/    
          memcpy(&IdtEntry\[ADDINT\],   &OldIdtEntry,   sizeof(OldIdtEntry));    
          \_asm   sti    
  }    
     
     
  void   \_\_declspec   (naked)   InterruptHandler()          
  {    
  \_asm    
  {    
  PUSHAD    
  PUSHFD    
  PUSH   FS    
     
  MOV           EBX,00000030h    
  MOV           FS,BX    
  SUB           ESP,   50h                    
  MOV           EBP,ESP    
     
  ;Setup   the   exception   frame   to   NULL    
  MOV           EBX,DWORD   PTR   CS:\[0FFDFF000h\]    
  MOV           DWORD   PTR   DS:\[0FFDFF000h\],   0FFFFFFFFh    
  MOV           DWORD   PTR   \[EBP\],EBX    
     
  ;Save   away   the   existing   KSS   EBP    
  MOV           ESI,   DWORD   PTR   CS:\[0FFDFF124h\]    
  MOV           EBX,DWORD   PTR   \[ESI+00000128h\]    
  MOV           DWORD   PTR   \[EBP+4h\],EBX    
  MOV           DWORD   PTR   \[ESI+00000128h\],EBP    
     
  ;Save   away     the   thread   mode   (kernel/user)    
  MOV           EDI,DWORD   PTR   \[ESI+00000134h\]    
  MOV           DWORD   PTR   \[EBP+8h\],EDI    
     
  ;Set   the   thread   mode   (kernel/user)   based   on   the   code   selector    
  MOV           EBX,DWORD   PTR   \[EBP+7Ch\]    
  AND           EBX,01    
  MOV           BYTE   PTR   \[ESI+00000134h\],BL    
     
  STI    
  }    
     
  DbgPrint("%S:Test   Call\\n",DRIVER\_DEVICE\_NAME);    
     
  \_asm    
  {    
  MOV           ESI,DWORD   PTR   CS:\[0FFDFF124h\]    
  MOV           EBX,DWORD   PTR   \[EBP+4\]    
  MOV           DWORD   PTR   \[ESI+00000128h\],EBX    
     
  ;Restore   the   exception   frame      
  MOV           EBX,DWORD   PTR   \[EBP\]    
  MOV           DWORD   PTR   FS:\[00000000\],EBX    
     
  ;Restore   the   thread   mode    
  MOV           EBX,DWORD   PTR   \[EBP+8h\]    
  MOV           ESI,DWORD   PTR   FS:\[00000124h\]    
  MOV           BYTE   PTR   \[ESI+00000134h\],BL    
  ADD           ESP,   50h    
  POP           FS    
  POPFD    
  POPAD    
  }    
     
  \_asm   iretd    
     
  }     
  
看起来很复杂,其实没有什么,就是替换 替换 替换阿  
把你自己的异常处理函数换掉系统的异常处理函数.恢复起来很简单,无非就是先保存系统的函数而已  
但是不要忘记 HS 的监视,我们需要 1 些特殊手段做掉 HS  
当然也可以做掉 NP 的监视线程  
我们可以直接结束线程,但是更好的手段是断开线程调度链表  
线程调度链表--系统 CPU 时间分片给每个线程,线程要执行就必须要调度,我们安静的把邪恶线程  
给去掉,邪恶线程虽然还在,但是永远没有机会运行.哈哈,我们可以做自己想干的事情了:smoke   
一些代码偶也没有完成,就没办法提供了  
参考 PHRACK 一些资料,个人感觉 PHRACK 虽然是 hack 期刊,听说最近重新开了  
但是有很多内核知识很不错  
  
感谢各位看到这里,再次祝福新年快乐 

IDT 的 hook 与 add 代码 相当完善  
不是偶写的..哈   
偶的代码是硬编码的,不贴了.![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || idtextsrc.rar  
 |