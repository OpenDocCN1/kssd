

# 浅谈 Unpacked Armdillo 主程序后的暗桩

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-498.htm`](https://www.pediy.com/kssd/pediy09/pediy09-498.htm)

> 浅谈 Unpacked Armdillo 主程序后的暗桩  
>   
>     Armdillo 主程序脱壳后存在暗桩，要找出它确实要花一定的时间和跟踪力量！正所谓功欲善其事，必先利其器，有了道具的保证，跟踪才显得没有那么繁琐。脱壳过程我选择了 DillDie1.6，它是个不错的 Armdillo 脱壳工具，不过我对它调试日志没得选择还是有点敏感，所以用它脱壳会慢些，不过对于我这等懒人来说没什么能挑剔了...，除了 Nanomites 外（不能跨平台和方便跟踪），其它的项目都修复得不错！ Nanomites 方面可用修复器进行修复处理，效果也不错，有了工具的保证，脱壳就是这么容易，修复器还对跟踪提供了方便的保证！  
>   
>     每逢春节倍思春，思觉失调，寂寞难耐，好想再遇见她，把她在怀里（组织郎情妾意）！言归正，这里已经假设你脱壳没有麻烦了，主程序脱壳后，运行它，没报什么错误，也不退出程序，看来，程序还是个安分的家伙，找个试炼品加壳后，也没报警，挺好的吧，再运行加壳后的程序，哟，不行，有错，看来程序是有暗桩的了！  
>   
>     设想了好多常用的反脱壳反跟踪的可能性，比如运行时 CreateFileA 自校验（打开自己，但之后什么也没动作）、读取 PE 头（只读取加壳标志位）、命令行参数、时间反跟踪、环境字符变量等等，都摸不着头绪，而且，每次加壳同一个目标，出来的产品除了固定的部分，永远都不一样的！看来，还是要跟踪原版和脱版的加壳过程的差别（长征）！  
>   
>     初步，使用 OD、脚本和修复器的配合方便地来到子进程运行入口（与脱版差别就是 IAT 调用），跟踪了子进程加壳的 Call，F8 步过也显得挺长的，没有收获！郁闷了些日子，听闻网上有脱壳版，找回来的调试下，哇哇，加 Asprotect 的，虽然 IAT 没加密，但代码很多都被强奸了，要恢复一个它的原始贞操，不是正常人能办到的，我可是正常的，没工具恢复太难缠了，没参考价值，丢到垃圾回收站，网上问了一下人，没给答复，登陆资料又丢了，只好有空慢慢来。  
>   
>     于是，换了个想法，同样来到入口环境，将脱壳的代码节和 IAT 部分换到子进程对应空间，运行后加壳，发现情况一样！看来，我们的暗桩范围可以缩小了，就是说它不检测调试器和文件检验，那么，它肯定和壳有一腿，什么呢？  
>   
>     没错，就是 IAT，跟踪之下（调试父进程，加壳过程它除了处理 Nanomites 和满足条件就加密来解码去子进程外，就无所事事地等待、等待、再等待）我们知道，壳除了 IAT 之外就没有什么可以实现暗桩的了！好了，范围进一步缩小了吧！天呀，这么多 IAT 被加密过的，跟踪那个才是好呀！再者，我们先看看，处理后的 API 有什么模样的, 有存放 Call Real API 结果的，有解密来加密去的，有计算还原 API 位置的，有检 CC 的（很小很无聊），有直接 API 的，看来秘密就存在“解密来加密去的”的了，废话... 别扔鸡蛋，不然要完蛋的！  
>   
>     这时，不禁想起了以前的 CreateFileA，为何打开自己，但之后什么也没动作？于是，翻查了它，发现它保留打开文件的句柄，而且还参与加壳过程的，有可疑，真是举头望明月，低头思“故”香。连忙跟踪子进程加壳过程，发现句柄不一样，是固定的“一级棒”，哈哈，第一个暗桩找到了！为何是暗桩，没和壳一腿，就让你没门，自己跟一趟就知道了！  
>   
>     修正后，再加壳，还是一样！啃定还有暗桩，看来不是一帆风顺的！继续征途，这时只能跟踪加壳过程了，发现壳使用随机数参与加壳，使得每次加壳后都不一样的，也令跟踪显得不知所措，惘然了好些日子，后来想了想，能不能反客为主，如果一直跟着子进程加壳的原始流程走，跟个 10 年 8 年也没东西可找到的！于是，在使用随机数的 Call 上修改代码，使其返回一个固定的值，果然，这条路好走多了，很多产生加密数值的地方原本是千变万化的都固定起来了，记录吧，路程有垃圾代码，好在暗桩的参与都安排在加壳的大路上，虽然长路慢慢，终究是发现鼠窃狗偷的地方，目的同出一切，修正后加壳运行后 OK 啦！至于，其它的暗桩是什么，如果你愿意长征，肯定发现的，是跟 SDK 有关的吧，有的低版无，有的版本有 1 个，有的 2 个，我在这就不点明了，只玩过 2、3 个壳，没法说准！可能新版本或 Custom 版本会更多，不得而知！  
>   
>     欢迎你一直看到这里，如果你觉得上面没有真实代码举例，如果你觉得无聊 -- I am Sorry! 引用某人 Cracked 版的话 -- Game 来的！Game 不是你自已玩就失出了 Game 的意义！希望想玩 Game 的朋友都能自己打爆机！总之，就像渔翁撒网，不断收缩范围，只要有 fish 就会在收网的时候有斩获！  
>   
>   
> 给个脚本 For Debug  
>   
> //////////////////////////////////////////////////////////  
> // FileName    :  WaitForDebugEvent.osc  
> // Comment     :  Armadillo V4.X Nanomites WaitForDebugEvent  
> // Environment :  WinXP SP2,OllyDbg V1.10,OllyScript V0.92  
> // Author      :  fly  
> // WebSite     :  http://www.unpack.cn  
> // Date        :  2005-11-04 16:40  
> //////////////////////////////////////////////////////////  
> // Config Above Environment  
> // Set OllyDbg on Just Time Debugger  
> // Perhaps apply on Arm3.78..4.X  
> // Close Current OllyDbg and Enter Child Process OEP If System Call you Click Debug Button  
> // This Script Traces Trick Only for Debug   
> // Modified by NewHand  
> //////////////////////////////////////////////////////////  
> #log  
> dbh  
>   
> var FixIP  
> var EP  
> var Tmp  
> var TmpAddr  
> var PathAddr  
> var WaitForDebugEvent  
> var DAProcess  
> var ChildID  
> var HoldEP  
> var StopDebugIP  
> var WritePM  
>   
> gpa "OutputDebugStringA", "KERNEL32.dll"  
> mov \[$RESULT\], #C20400#  
>   
> gpa "DebugActiveProcess", "KERNEL32.dll"  
> cmp $RESULT, 0  
> je Error  
>   
> mov DAProcess, $RESULT  
> eob LB\_DAProcess  
> bp DAProcess  
> esto  
> Skip:  
> esto  
> LB\_DAProcess:  
> cmp eip,DAProcess  
> jne Skip  
> bc DAProcess  
> mov ChildID,esp  
> add ChildID,4  
> mov ChildID,\[ChildID\]  
> eval "Child ProcessID = {ChildID}"  
> log $RESULT  
> cob  
>   
> gpa "WaitForDebugEvent", "KERNEL32.dll"  
> find $RESULT,#C20800#  
> mov WaitForDebugEvent,$RESULT  
> bp WaitForDebugEvent  
> esto  
> bc WaitForDebugEvent  
> rtu  
> log eip  
> //#inc "arm\_dust\_clear.txt"  
> find eip,#7607#  
> cmp $RESULT, 0  
> je Error  
> mov \[$RESULT\], #EB#  
>   
> find eip,#83BD??F5FFFF00#  
> cmp $RESULT, 0  
> je Error  
> bp $RESULT  
> run  
> bc eip  
> mov PathAddr, eip  
> add PathAddr, 2  
> mov Tmp, \[PathAddr\]  
> add PathAddr, 13  
> mov PathAddr, \[PathAddr\]  
> sub eip, 43  
> mov \[eip\], #52EB195959EBFE680010000068FE3AE97758FFD08B85BCF5FFFFEBFE8B0DB4464E005159493B0C2474EA83F9007CD4898DC0F5FFFF519090#  
> mov TmpAddr, eip  
> sub TmpAddr, 51  
> mov TmpAddr, \[TmpAddr\]  
> and TmpAddr, 0FF  
> log TmpAddr  
> sub TmpAddr, C1  
> add \[eip\], TmpAddr  
>   
> mov TmpAddr, eip  
> add TmpAddr, 16  
> mov \[TmpAddr\], Tmp  
> sub \[TmpAddr\], 4  
> add TmpAddr, 8  
> log PathAddr  
> mov \[TmpAddr\], PathAddr  
> add TmpAddr, 13  
> mov \[TmpAddr\], Tmp   
> mov Tmp, eip  
> find eip, #0F84D5010000#  
> cmp $RESULT, 0  
> je Error  
> mov FixIP, $RESULT  
> add Tmp, 23  
> eval "jne {Tmp}"  
> asm FixIP,$RESULT  
> find eip, #0F8ED9010000#  
> cmp $RESULT, 0  
> je Error  
> mov \[$RESULT\], #90E9#  
> mov Tmp, eip  
> add Tmp, 8  
> mov \[Tmp\], ChildID  
> add Tmp, 4  
> asm Tmp, "call Kernel32.DebugActiveProcessStop"  
> mov StopDebugIP, eip  
> add StopDebugIP, 5  
> bp StopDebugIP  
> log StopDebugIP  
> mov HoldEP, eip  
> add HoldEP, 1A  
> log HoldEP  
> bp HoldEP  
> run  
> cmp eip, HoldEP  
> jne Error  
> bc eip  
>   
> log eip  
> add eip,15  
> log eip  
> gpa "WriteProcessMemory", "KERNEL32.dll"  
> mov WritePM, $RESULT  
> bp WritePM  
> log WritePM  
> mov EP, eax  
> Log EP  
> and EP, 0FFF  
> log EP  
>   
> run  
> cmp eip, WritePM  
> jne Error  
> bc eip  
> mov Tmp, esp  
> add Tmp, 0C  
> mov Tmp, \[Tmp\]  
> add Tmp, EP  
> mov \[Tmp\], #EBFE#  
> run  
> cmp eip, StopDebugIP  
> jne Error  
> bc eip  
> add eip, 2  
> log eip  
> add StopDebugIP, 0C  
> log StopDebugIP  
> bp StopDebugIP  
> run  
> bc eip  
>   
> ret  
>   
> OK:                          
> MSG " Plz Continue Fix Nanomites !  Game Over.     "    
> ret  
>   
> Error:  
> msg "Error!"  
> ret 

 4dd466d0e1e0cc123afbe504da2462ca