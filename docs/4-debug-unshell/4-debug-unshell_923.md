

# 彩票快车白金版 v90

> 原文：[`www.pediy.com/kssd/pediy04/kanxue018.htm`](https://www.pediy.com/kssd/pediy04/kanxue018.htm)

发信人:柯南(我可是一位侦探呀!!!)  <<---这个白痴又来了!!   
题目:论彩票快车白金版 v9.0 的脱壳和爆破!   
目的:  1.)教新手们基本的手动脱壳技巧,如果熟练掌握,好处无穷!!   
        2.)本文没有太多的将注册码的破解.   
    (我破不了这么复杂的注册码,至于爆破吗我看就是 经验+运气+灵感).   
  
本文只是适合新手手动脱壳的入门和爆破入门,高手们不要笑小弟呀!   
(可以多多提意见,我是很谦虚的!).   
用到的工具:   
1.)softice 4.05   
2.)icedump  v6.0.1.6   
3.)filemon v245   
4.)peditor v1.7   
5.)ImportREC v 1.2beta2   
我没有用 trw2000,因为这个软件解密时检查 Trw2000,我看是因为这软件是中国人写的吧!!(比较了解国情).   
  
一)脱壳部分:   
1.)首先我们用 filemon 来检查文件的类型:   
  相信大家都会用吧:烤,怎么是 petite2.2 ,没搞过呀!!!.别急.   
  
2.)当然是检查程序的 pe 结构了:   
  打开 peditor,选中程序,用笔记录以下几个数据:   
  Entry Point:169042.   
  Image base :400000   
  size of image:16a000.   
3.)寻找入口点(OEP):先运行 icedump->再运行 softice loader-->   
  再 Ctrl+d   
  好了准备工作完成了,该选择断点了.一般大家脱壳时用   
  bpx loadlibrarya do "dd esp\->4"(经验,好好看看 Pe 结构并懂一些 Win32 编程即知)   
  再 Ctrl+d-->再 load program.   
  断了吧这是 loadlibrary 函数的入口,该函数是 Dll 中的不用理.   
  bc \*,按 F12 n 下,注意领空的变化呀!,到了主程序的领空后停下.   
  当前代码如下:   
0167:004E7679  JZ        004E7683                    
0167:004E767B  ADD      ESP,2A                      
0167:004E767E  JMP      00569015                    
0167:004E7683  MOV      ESI,000E5000                    
0167:004E7688  ADD      ESI,EBP                      
0167:004E768A  LEA      ECX,\[EBP+00000800\]            
0167:004E7690  MOV      EBX,EAX                        
0167:004E7692  CMP      DWORD PTR \[ESI\],00            
0167:004E7695  JZ        004E78A9              --->注意这是个捷径.      
0167:004E769B  PUSH      ECX                          
0167:004E769C  PUSH      ECX                      
0167:004E769D  CALL      \[EBP+00000790\]              
0167:004E76A3  TEST      EAX,EAX            ---->注意这是用 F12 跳到的第一条指令.          
0167:004E76A5  JZ        004E778A                      
0167:004E76AB  MOV      EDI,EAX                        
0167:004E76AD  ADD      EAX,\[EAX+3C\]                    
0167:004E76B0  MOV      EAX,\[EAX+78\]                      
0167:004E76B3  PUSH      DWORD PTR \[EDI+EAX+18\]            
0167:004E76B7  MOV      ECX,\[EDI+EAX+24\]                    
0167:004E76BB  ADD      ECX,EDI                              
0167:004E76BD  PUSH      ECX                                              
\------------------------CPKCBJB!.petite+2679----------------   
  程序第一次来到 004e76a3,正常的话用 F10 开道(用 F12 就出去了),可是我参考了精华 3 上的如下文章:   
  
      {标 题:脱壳----对用 Petite2.2 加壳的程序进行手动脱壳的一点分析 (5 千字)   
      发信人:ljttt   
      时 间:2000-7-27 19:57:10   
      详细信息:   
      脱壳----对用 Petite2.2 加壳的程序进行手动脱壳的一点分析 }   
  
  那上说在这时应向上找入口点的捷径.所以我直接用指令 g 004e78a9 来到:   
  (当然,你有耐心的话不用捷径用 F10 最后也可以来到这地方)   
  
0167:004E78A4  SUB      EBX,06                            
0167:004E78A7  JMP      004E786E                          
0167:004E78A9  POP      ECX                  --->g 指令来到的点.                
0167:004E78AA  POP      ESI                                    
0167:004E78AB  STD                                              
0167:004E78AC  XOR      EAX,EAX                            
0167:004E78AE  MOV      ECX,00000356                        
0167:004E78B3  CALL      00569039              --->F8 进入.                
0167:004E78B8  ADD      \[EAX\],AL                              
0167:004E78BA  ADD      \[EAX\],AL                              
0167:004E78BC  ADD      \[EAX\],AL                                  
0167:004E78BE  ADD      \[EAX\],AL                                
0167:004E78C0  ADD      \[EAX\],AL                                      
\------------------------CPKCBJB!.petite+288F----------------------   
  
  必须进入不然又掉了.来到:   
0167:00569039  POP      EDI                                              
0167:0056903A  REPZ STOSB                                                  
0167:0056903C  POPAD                                                        
0167:0056903D  POPF                                                      
0167:0056903F  ADD      ESP,08                                            
0167:00569042  JMP      004C3E28      --->很关键的跳(OEP 的跳).   
0167:00569047  JMP      KERNEL32!GetCurrentThreadId                      
0167:0056904C  JMP      KERNEL32!VirtualFree                              
0167:00569051  JMP      KERNEL32!LocalFree                              
\------------------------CPKCBJB!+0039----------------------   
  
  接着来到 OEP.:   
0167:004C3E28  PUSH      EBP            --->程序的入口点        
0167:004C3E29  MOV      EBP,ESP                                      
0167:004C3E2B  ADD      ESP,-0C                                      
0167:004C3E2E  PUSH      EBX                                            
0167:004C3E2F  MOV      EAX,004C3B60                                  
0167:004C3E34  CALL      004061A8                                    
0167:004C3E39  MOV      EBX,\[004C6B34\]                                
  
\-----------------------CPKCBJB!+000C2E28------------------------   
  呵呵,OEP 找到了,成功一半了.   
  下指令/dump 400000 16da000 1.exe   
  退出 softice 和 icedump.   
  
4.)修补 dump 出来的文件:   
  打开 peditor,装入 1.exe.点击 section 按钮,右键选 dumpfixer,最后 exit.   
  
5.)修补 import 表:   
  运行 ImportREC1.2beta2，在 Attach to an Active Process 下拉框中选中 importrec.exe   
  (首先确保原来的程序在运行)，   
  然后在左下方 OEP 中输入 C3E28 就是(47e910-400000)，按 IAT AutoSearch。出现对话框 Found Something!=》   
  “Found address which may be in the Original IAT.Try 'Get Import'”   
  按 Get Import，再按 Show Invalids 按钮，在 Imported Function Found 框中会有很多蓝色的项目。   
  在其中点右键，选 Trace Level1(Disasm)，再按 Show Invalids 按钮，可以看到所有的 dll 都为 valid:YES   
  字样。输入表到此完成。   
  再选择 Add new section(缺省是选上的),按 Fix Dump，并选择刚才 Dump 出来的文件 1.exe。   
  ImpRECT 会在该文件所在目录生成一个新的文件：test\_.exe ,这就是修复后的文件。   
  至此"脱壳"工作终于大功告成，点击 1\_.exe 可以运行了。   
  
  呵呵:   
  关于为什么一开始我 mail 给 night007 的软件不好使,是因为我没做第五步,但是没修补 import 表却可以在本机   
  上正常运行,这是什么意思,请高手指点一二.   
  
  
二)爆破开始了(我虽然是个侦探,可是我可不愿算那可恶的注册码  <<----这个家伙又在吹,他不行).   
  打开软件,来到注册画面,添入注册码(假的,要和用户名<机器生成码>一样长呀,不然注册按钮不可用)   
  我添的是:   
          默认的注册名,和注册名一样常的 1111111....注册码!!   
  打开 softice,设断点 bpx hmemcpy--->Ctrl+n---->点击注册---->F12 来到程序领空.   
  再一路按 F10 来到:   
0167:004A1DAD  CALL      00403F84                                        
0167:004A1DB2  MOV      EDX,\[EBP\-24\]                                        
0167:004A1DB5  POP      EAX                                                      
0167:004A1DB6  CALL      00403E8C                                                
0167:004A1DBB  JZ        004A1DC1                                                
0167:004A1DBD  MOV      BYTE PTR \[EBP\-09\],00                                    
0167:004A1DC1  DEC      EBX                                                      
0167:004A1DC2  TEST      EBX,EBX                                                  
0167:004A1DC4  JNZ      004A1D7A                                                
0167:004A1DC6  CMP      BYTE PTR \[EBP\-09\],00                                    
0167:004A1DCA  JZ        004A1F64      --->关键跳.上面的算法太复杂.我不干了                
0167:004A1DD0  LEA      EDX,\[EBP\-28\]                                            
0167:004A1DD3  MOV      EAX,\[ESI+000002FC\]                                      
0167:004A1DD9  CALL      0042D8D4                                                
                                  
    这样爆破虽然不是很完美,但是软件已经可以用了.   
    剩下的我是不行了，谁行谁来吧!!   
  
    篇后语:   
我是一个 Crack 新手,在短短的二周不到的时间里,获得了不错的战绩,首先要感谢 teacher toye,是他的   
论坛和课本启发了我,再有我也应该感谢所有在看学论坛的前辈们,是你们的帮助和关怀使我们迅速成长   
起来,这是我的处女文章,本来我是一个侦探(<----这个逼又来了,嘿),所以文笔较差,但我不会辜负学长   
们的教导,我会继续努力提高 Crack 水平,写出更好的文章的!!   
                            柯男于 2001/9/14.   
看学前辈,这是我的处女文章呀!,你要是看好了请收入论坛精华 4,呵呵! 

* * *

> 标 题:重要声名,我的文章的爆破部分有问题,请高手帮助. (空)  
> 发信人:柯南(一名侦探)  
> 时 间:2001-9-14 22:32:03  
> 阅读次数:1  
> 详细信息:  

* * *

> 标 题:他的那些功能好像根本就没有，有可能是掏钱后再给你另一个真正的文件！ (空)  
> 发信人:洋白菜  
> 时 间:2001-9-15 10:10:27  
> 阅读次数:3  
> 详细信息:
> 
>  4dd466d0e1e0cc123afbe504da2462ca