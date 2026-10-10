

# OllyDbg + ASProtect SKE 2.X +代码变形

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-655.htm`](https://www.pediy.com/kssd/pediy07/pediy7-655.htm)

【破解作者】 SYSCOM  
【使用工具】 Ollydbg1.10, LordPE,ImportREC1.65,PEid 0.93  
【破解平台】 Windows XP SP1  
【软件名称】 BetterJPEG 1, 4, 4, 0  
【下载地址】 [`www.betterjpeg.com/`](http://www.betterjpeg.com/)  
【软件简介】 Edit JPEG photos   
【加壳方式】 ASProtect SKE 2.X   
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
(1) 用 ollydbg 载入 BetterJPEG.exe ,设置忽略所有异常。  
    在命令栏输入  
    BP GetVersion ,按 F9,看 STACK 返回 431EE4 ,在按 ALT+F9  
    返回,主程式 ,往上看.....不远地方  
  
431EB8-PUSH  EBP          <======= OEP 进入点  
431EB9-MOV   EBP,ESP  
431EBB-PUSH   -1   
.....  
.....  
  
  
(2)修正 Import  Table and .CODE  写入补丁 PATCH(1)  
   先修正 9F4387-PUSH 9F43A4==>PUSH PATCH(1)+0x29 Offset Address     
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Start of Code in Object BinaryCode \*\*\*\*\*\*\*\*\*\*\*\*\*\*  
:00000000   mov edx, 00401000               ;主程式 Start address  
:00000005   cmp byte ptr \[edx\], E8          ;判断是否为 call 指令  
:00000008   jne 0000001E  
:0000000A   mov eax, dword ptr \[edx+01\]  
:0000000D   add eax, edx  
:0000000F   add eax, 00000005                                   
:00000014   cmp eax, 00CD0000               ;判断是否为 call API 加密,进入点  
:00000019   jne 0000001E  
:0000001B   pushad                          ;保存所有暂存器  
:0000001C   jmp edx  
:0000001E   inc edx  
:0000001F   cmp edx, 00461000               ;主程式 end address  
:00000025   jbe 00000005  
:00000027   jmp 00000027                    ;PATCH END  
:00000029   mov ecx, 00461000               ;IAT   start address (解码返回进入点)  
:0000002E   mov edx, dword ptr \[ebp-0C\]  
:00000031   cmp dword ptr \[ecx\], edx  
:00000033   jne 00000052  
:00000035   mov dword ptr \[00460800\], ecx    ;暂存 Import Address  
:0000003B   add esp, 0000023C  
:00000041   popad                            ;取回所有暂存器  
:00000042   mov ebx, dword ptr \[00460800\]    ;取出 Import Address  
:00000048   mov dword ptr \[edx+02\], ebx      ;修正 Import Address   
:0000004B   mov word ptr \[edx\], 15FF         ;修正 CALL   
:00000050   jmp 0000001E  
:00000052   add ecx, 00000004  
:00000055   cmp ecx, 004616AC                ;IAT   End address  
:0000005B   jl 00000031  
:0000005D   jmp 0000005D   
  
  
(3)写入补丁 PATCH(2)  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Start of Code in Object BinaryCode \*\*\*\*\*\*\*\*\*\*\*\*\*\*  
:00000000  mov edx, 00401000                ;主程式 Start address  
:00000005  cmp byte ptr \[edx\], E8           ;判断是否为 call 指令  
:00000008  jne 00000030  
:0000000A  mov eax, dword ptr \[edx+01\]  
:0000000D  add eax, edx  
:0000000F  add eax, 00000005  
:00000014  cmp eax, 00401000                ;判断是否为 > 401000  
:00000019  jb 00000030  
:0000001B  cmp eax, 00461000                ;判断是否为 < 461000  
:00000020  ja 00000030                                                   
:00000022  mov bx, word ptr \[eax\]  
:00000025  cmp bx, 15FF                      ;判断是否为 call 指令  
:0000002A  jne 00000030  
:0000002C  mov \[eax+01\], 25                  ;修正 CALL =>JMP  
:00000030  inc edx  
:00000031  cmp edx, 00461000                 ;主程式 End address  
:00000037  jne 00000005  
:00000039  jmp 00000039  
  
(4) PATCH 完后,Dump BetterJPEG\_dump.exe  
  
(5)run  BetterJPEG.exe 在用 ImportREC 抓出 Import  
    并修复 kernel32,中一个无效指针,改为 GetProcAddress  
    再修正,RestoreLastError ==>SetLastError  (98/XP/NT4/2003 相容)  
  
(6)使用 lordPE DUMP 变形程式,MEMORY 区块  
     并附加在 BetterJPEG\_dump.exe,的后面  
  
ADDRESS:        SIZE        SET  
\-----------------------------------  
9D0000           32000       RWE  
A64000           10000       RWE   
C80000            1000       RWE  
D10000            1000       RWE  
D30000            2000       RWE  
D40000            2000       RWE  
D50000            1000       RWE  
D60000            1000       RWE  
\-----------------------------------  
  
(7)修正 Route CHECK   
  
9F70CE-MOV  EAX,\[EAX+34\]    =>  9F70CE-NOP  
9F70D1-CALL EAX             =>  9F70CF-NOP  
9FD0D3-SUB  \[EBP+C\],EAX     =>  9F70D0-MOV   EAX,\[ESP+58\]    
9FD0D6-MOV  EAX,\[ENP+C\]     =>  9F70D4-SUB   EAX,5  
  
(9)OK->TEST->RUN  BetterJPEG\_dump.exe

> 引用:
> 
> * * *
> 
> *最初由 fly 发布*  
> **syscom 兄能否解释一下修正 Route CHECK **
> 
> * * *

  
  
  
  
 Route CHECK,算是壳的自我检查  
  
它是由 A,B 两数值,作互减运算  
  
A=GetCurrentThreadID   
B=CALL Route Address  
  
运算后 ...  
 B=B-A  
  
当你脱壳之后,B=会发生错误 ERROR 111  
所以我们只要,抓出正确的 CALL Route Address  
就可以通过  CHECK SUM ,也就是在 \[ESP+58\],的 STACK 位址   
所以我们使用 MOV   EAX,\[ESP+58\] ,来还原 B 值   
+5 后修正 B 值 Address  
  
9F70CE-MOV  EAX,\[EAX+34\]    
9F70D1-CALL EAX              =>  GetCurrentThreadID  
9FD0D3-SUB  \[EBP+C\],EAX      =>  B=B-A  
9FD0D6-MOV  EAX,\[ENP+C\]      
  

请问 patch1 和 patch2 是用什么工具(od 插件?)做？搞不懂这个地方。 

> 引用:
> 
> * * *
> 
> *最初由 heng9ml 发布*  
> **请问 patch1 和 patch2 是用什么工具(od 插件?)做？搞不懂这个地方。 **  
> 
> * * *

  
  
不用工具,找空白区写入,ASM 或 BIN CODE 例如:460700 (空白区)  
  写完后,COPY BIN BLOCK 保存在,文字档  
 下次,可以在贴上,不用重输入.~OK 

> 引用:
> 
> * * *
> 
> *最初由 cpshow 发布*  
> **看不懂哦  
> 能写个详细点的么  
> 我是 2ksp4 系统 为什么我载入后是停留在这里呢  
>   
> 00401000 >  68 01C04800           push 48C001  
> ........ **
> 
> * * *

  
  
OD 剛載入,是停這裡,沒錯  
 這是殼的入口,然後在命令行,下斷點  
BP GetVersion ,按 F9,ALT+F9 可找到  
 OEP 入口 ,看來,你和我一樣,剛學 OD  ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

我很奇怪里面没有 VM 的代码？不过 VM 代码不影响运行了。  
也奇怪代码居然 OEP 也没有抽取？  
  
我觉得至少还有两点需要解决：  
1，那么多节，能否想办法还原回去呢？如果不巧程序分配跨度达到了 2G 是否我们的 PE 文件的 ImageSize 也改这么大呢？  
2,VM 是否也该还原呢？VM 代码抽取也是一个难点，当然不是没有还原的可能，虽然 aspr 说随机抽取，但我用过，发现应该不是随机抽取，而是通过某种权衡以后抽取的，往往关键代码都会被抽取，而且，每次都是那几个，如果真随机，不可能每次都那么巧合。  
  
这两个问题不解决，分析还是很为难。 

> 引用:
> 
> * * *
> 
> *最初由 suddymail 发布*  
> **弱弱的问下。patch 里面直接是代码。如果是直接写入的话怎么运行？  
> 主程序代码中好象没调用这个啊。  
> 是不是在 patch 头建 eip。然后 patch 结果断？  
> 然后 run patch？还是怎么弄？  
> 没弄过 patch。希望老大赐教。 **
> 
> * * *

  
  
當你找到 OEP 時,找空白區,寫入 PATCH(1) ,寫完後  
  
令 EIP -->指向 PATCH 開頭位址,下斷點在 PATCH 結束位址  
 按下 F9 ,會終止在斷點處,就可以,再寫入 PATCH(2)  
  方法同 PATCH(1) ![](img/0cf410da109ac47387dd713fcee2cd79.jpg)

> 引用:
> 
> * * *
> 
> *最初由 kelven 发布*  
> **还是不太明白。  
> 先修正 9F4387-PUSH 9F43A4==>PUSH PATCH(1)+0x29 Offset Address     
> 如果是别的软件那要怎么找到对应 9F4387 的这个位置。 **
> 
> * * *

  
  
要请问楼主，是不是通过下 bp GetModuleHandleA,断两次返回之后：  
test eax, eax  
.  
.  
.  
popad   
jnz \*  
mov eax ,1  
retn  
push \*         <---------就是这里将＊改为“PATCH(1)+0x29”？  
retn ----->oep 

> 引用:
> 
> * * *
> 
> *最初由 kelven 发布*  
> **还是不太明白。  
> 先修正 9F4387-PUSH 9F43A4==>PUSH PATCH(1)+0x29 Offset Address     
> 如果是别的软件那要怎么找到对应 9F4387 的这个位置。 **
> 
> * * *

  
  
比對,搜尋,特徵碼....![](img/28f5e5b1c17d493bd22adb028ae8a71b.jpg)   
  
不過,如果,別的軟件,使用 ASPR 不同選項,的加密...  
  位置,可能就不同了,要在找一次,可是方法,都是一樣的...![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

> 引用:
> 
> * * *
> 
> *最初由 ledefa 发布*  
> **  
>   
> 要请问楼主，是不是通过下 bp GetModuleHandleA,断两次返回之后：  
> test eax, eax  
> .  
> ........ **
> 
> * * *

  
  
我是在找到 OEP 時,才修正的....![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)  
  
原來的 CODE...  
  
9F4387-PUSH 009F43A4  
  
假如,我們的 PATCH 開始位址是 5A3F00-  
這裡就,修改為 5A3F00+29=5A3F29  
  
9F4387-PUSH 005A3F29   
      -RETN  (返回 PATCH LOOP)  
  
  
  

 4dd466d0e1e0cc123afbe504da2462ca