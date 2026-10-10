

# Themida 初步研究(ANTI 篇)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-651.htm`](https://www.pediy.com/kssd/pediy08/pediy8-651.htm)

Themida 初步研究(ANTI 篇)  
  
  
某些马甲靠 themida 来赚钱的,我很生气,搞技术就搞技术,骂人在论坛真恶心.刚好 200 篇水贴,凑一篇水贴作为纪念.  
  
  
    我说了 themida 在 ANTI 上很弱很弱,可以说,浪费了我的时间,我不想学它的 VM 方式  
用不上那么夸张,我写花指令垃圾可以写上 10000M,那么,我们看看它的 ANTI.  
  
    在 OD 中打开无效和特权指令异常.它仅仅 2 个 anti,都是解码后出现的,我现在明白了  
为什么它不解码就可以跳过的 BUG 问题出在什么地方了.它的调试字符串不值得提,老掉牙了  
      
    多话不说了,我们先将 themida 的 anti 选项都选择上,别看它花里胡哨的那么多,基本  
是花瓶,一碰就破的.  
  
    将虚拟机放到最小,其它选择都关了,我的是笔记本在跑,不想和它比机器占用率.  
    将 entry point 放到 0,这篇是玩它的 ANTI 的,这些后面的干扰手段我们先不用.  
    有空我们再一篇一篇的肢解它..  
      
      
    将 themida 给记事本加壳,轰轰烈烈的又是算 VM(基本的)又是拆 setion,最后加上了.  
    不知道为什么,我的 ODbgScript 1.47 有问题,下内存写断点变成了内存读断点,我晕  
      
      
      
    第一个异常:  
011ADA34    59              pop     ecx                              ; 0007FFE0  
011ADA35    40              inc     eax  
011ADA36    0F3F            ???                                      ; 未知命令  
011ADA38    07              pop     es  
  
    看 SEH 句柄过  
      
    第二个异常:  
011ADBA2    81EA E85F3400   sub     edx,345FE8  
011ADBA8    ED              in      eax,dx  
011ADBA9    81FB 68584D56   cmp     ebx,564D5868  
  
    同样看 SEH 句柄过      
      
    过 2 个异常后,下 VirtualAllocEx 断点三次(它将虚拟机解出来),  
      
011B3FE6    68 1541672C     push    2C674115  
011B3FEB  ^ E9 29BAFFFF     jmp     011AFA19  
011B3FF0    68 99C1004D     push    4D00C199        ; 从系统区返回这里  
011B3FF5  ^ E9 1FBAFFFF     jmp     011AFA19  
011B3FFA    68 C8AC9D17     push    179DACC8  
011B3FFF  ^ E9 15BAFFFF     jmp     011AFA19  
011B4004    68 60F14E17     push    174EF160  
011B4009  ^ E9 0BBAFFFF     jmp     011AFA19  
011B400E    68 91C85221     push    2152C891  
011B4013  ^ E9 01BAFFFF     jmp     011AFA19  
011B4018    68 A2EE450D     push    0D45EEA2  
      
    下 LoadLibraryA 断点,返回壳代码区  
  
011B9F08    FF95 25073509   call    near dword ptr \[ebp+9350725\]; kernel32.LoadLibraryA  
011B9F0E    66:8BF0         mov     si,ax                        ;返回这里   
011B9F11    8D9D EB774009   lea     ebx,dword ptr \[ebp+94077EB\]  
  
      
    直接在用户代码区下内存写断点.断在这里,取消内存写断点  
      
011EA3FD    8BFA            mov     edi,edx  
011EA3FF    8BF1            mov     esi,ecx  
011EA401    8BD1            mov     edx,ecx  
011EA403    8BC8            mov     ecx,eax      
011EA405    F3:A4           rep     movsb                     //断在这里  
011EA407    C685 55253509 5>mov     byte ptr \[ebp+9352555\],56  
  
这里就是将解码后的代码放回,过这里后,看 ESP,\[ESP+1C\]就是它要去的 OEP,这里下硬件写断点  
将停在这里.  
  
011F5FFC    8BC0            mov     eax,eax  
011F5FFE    61              popad  
011F5FFF    68 9D730000     push    739D                  ;看见了么,这个是什么  
011F6004    9C              pushfd  
011F6005    6A 00           push    0  
011F6007    50              push    eax  
  
  
F7 过一步,继续断这里,它肯定要加上段地址  
  
011F6849    5D              pop     ebp  
011F684A    F5              cmc  
011F684B    014424 08       add     dword ptr \[esp+8\],eax   又断在这里,eax==01000000  
011F684F    6A 00           push    0  
011F6851    52              push    edx                              ;  
  
(它的最后去 OEP 是这样的)  
  
011F689D    9D              popfd  
011F689E    C3              retn                ;刚才将 OEP 算出来了压在堆栈,飞去 OEP 了  
  
  
  
好了 OEP 出来了,就这样简单.直接下断到 OEP.  
  
飞到光明地方,DUMP 出来,IAT 修复,砍掉壳区,88K!!  
  
再次同样选项测试 regedit.exe  一模一样,再次增减各个选项测试,一模一样,收工...  
  
  
这就是它第一项花里胡哨的东西,我写脚本和文章时间超过我跟踪它这些选项的时间.切........  
最近老有活,不然有空把它虚拟机的东西也写了,它的 API 复制就不用写了,完全没有技术含量,复制后插花罢了.  
代码重放其实也不很复杂,绕点弯弯..  
  
      
      
      
      
    ////////////////////////////////////////////////////   
// XXXXX OEP 查找脚本   
// Written by hnhuqiong   
// Email:hnhuqiong@126.com  
// OS : Win XP SP1,OllyICE 1.10,ODbgScript v1.47  
// Data: 2006-01-30   
// NOTE:   
//  晕死了,为什么我的脚本下内存写断点断不上.  
//   
// Thanks!:  
//  OLLYICE (OLLYDBG patch V1.10 and HideOD plugin V0.13 by PEDIY )  
//  ODbgScript plugin by E3  
//  bbs.pediy.com(看雪学院)  
////////////////////////////////////////////////////   
  
  
initialization:                           //初始化  
dbh  
var cbase  
var csize  
var tmp  
var oep  
  
gmi eip,CODEBASE  
mov cbase,$RESULT  
gmi eip,CODESIZE  
mov csize,$RESULT  
  
  
pretreatment:                            //预处理  
   cmp $VERSION, "1.47"  
   jb version\_above\_147  
   msgyn "请设置忽略所有的中断,除了无效或者特权指令,继续?"  
   cmp $RESULT,1  
   je Start  
   jmp end  
  
Start:  
  run    
  mov tmp,\[esp+4\]  
  bp tmp  
  esto tmp                             //过第一个 SEH  
  bc tmp  
    
  run  
  mov tmp,\[esp+4\]  
  bp tmp                               //过第二个 SEH  
  esto tmp   
  bc tmp  
   
  gpa "VirtualAllocEx","Kernel32.dll"  
  mov tmp,$RESULT  
  bp tmp  
  run  
  run  
  run                                //过三次  
  bc tmp  
  rtu   
    
  gpa "LoadLibraryA","Kernel32.dll"  
  mov tmp,$RESULT  
  bp tmp  
//  run  
  run  
  bc tmp  
  rtu      
 ----------对不起大家了,后面的我 ODbgScript 有问题,内存写断点变性....   
  bpwm cbase,csize                    
  run  
  bpmc  
    
   
  mov tmp,eip  
  add tmp,2  
  go tmp  
    
  mov tmp,esp  
  add tmp,1C  
    
  bphws tmp,"w"  
  esto                           //求出 OEP 的 AV  
  bphwc  
    
  sti  
    
  bphws tmp,"w"  
  esto                           //加上段地址,得出 RAV     
  bphwc  
  
  mov tmp,esp  
  add tmp,8  
  mov tmp,\[tmp\]  
  bp tmp  
  esto  
  bc tmp  
     
end:  
ret            // 结束脚本   
  
  
/////////////////////////  
//一些判断和提示信息  
////////////////////////  
  version\_above\_147:  
   msg "请下载最新的 ODbgScript 1.47 以上版本,Script by hnhuqiong,Thank you for using my script!"  
   jmp end  
  
  error\_all:  
       msg "错误,请联系脚本作者 hnhuqiong@126.com"  
       jmp end  
  error\_bpint:  
       msg "断点错误,请联系脚本作者 hnhuqiong@126.com"  
       jmp end  
  error\_function:  
       msg "查找不到函数地址,请联系脚本作者 hnhuqiong@126.com!"  
       jmp end  
  errror\_bpint:  
       msg "查找不到断点,请联系脚本作者 hnhuqiong@126.com"  
       jmp end  
     
//////////////////结束//////////////////////////////   

 4dd466d0e1e0cc123afbe504da2462ca