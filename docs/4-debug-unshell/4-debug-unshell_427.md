

# Advanced PDF Password Recovery Pro 1.70

> 原文：[`www.pediy.com/kssd/pediy05/pediy50110.htm`](https://www.pediy.com/kssd/pediy05/pediy50110.htm)

Asprotect 1.2x 加壳的 Advanced PDF Password Recovery Pro 1.70 脱壳   
crack by fwnl   
软件说明:可以帮你恢复 PDF 的加密文件。密码恢复的速度相当快，使用上也相当简单。可以帮你解开多达 32 位的密码。   
  
软件下载:http://extend.hk.hi.cn/~czy/dl/apprp10.zip   
破解工具:loader,trw2000, superbpm,Import REConstructor v1.4.2+ winhex   
  
首先要谢谢电神大哥的指点(电神真是大好人,呵呵),没有他的指点,我现在还在拿着这个软件晕 :)   
  
这个软件比较怪,用 loader 找到的入口是 jmp xxxxxxxx,开始我还以为找错了,后来脱出后用 peid 看是 Watcom C/C++编写的软件,可能与 vc++不一样吧.   
  
一.找入口点   
  感谢 fs0 给我们带来 loader 这么个一个超 cool 的工具,用 loader 载入 apdfprp.exe 轻易找到入口   
41a964.   
  
二.用 trw2000 初步脱壳   
  
  下面要在入口处脱壳，打开 SuperBPM，点 erase，用 trw 载入 apdfprp.exe，下 g 41a964. 接着下 pedump 脱出程序.   
  
三.修复 import table   
  我脱这个程序的难点就在这里,因为用 Import REConstructor v1.4.2+,在 oep 那里填 0001a964,点 IAI AutoSearch,结果说无效 :(  后来问电神大哥说 oep 那里要填 00001000(至于为什么我就不清楚了,那位知   
道的能不能告诉我一下,是不是 Watcom C/C++的原因??)   
  
  打开原加壳程序,在 Import REConstructor v1.4.2+ 的 Attach to an Active Process 窗口中选取 apdfprp.exe 的进程,然后在下方的 oep 处填入 00001000,点 IAI AutoSearch,再点 Get Imports 然后点 Show Invalid,在 Imported Functions Found 窗口里的无效地址上点鼠标右键，选 Trace Leve11(disasm),再点 show invaids，发现还有几个没有修复，再次在那几个没有修复的地址上点鼠标右键.   
  选中 Plugin Tracer(Asprotect 1.2X Emul),再点 show invaids 应发现所有的 dll 显示 valid:Yes 了。   
因为真正的入口是 41a964,所以这时在 oep 那里填入 0001a964,再点 Fix Dump,选中你用 trw2000 pedump 出的文件修复.   
  
四.程序除错   
运行脱后的程序报 API not found, please contect technical support   
用 trw2000 载入下断点 bpx messageboxexa ,F5 返回,程序被中断,按 F10 可看出 出错对话框地址是   
00409518   
  
0167:004094FB A3F0084500      MOV      \[004508F0\],EAX   
0167:00409500 85C0            TEST    EAX,EAX   
0167:00409502 7526            JNZ      0040952A    //只要这里改成 jmp  409552(7526 改 EB4E)就可以了   
0167:00409504 6A10            PUSH    BYTE +10   
0167:00409506 68F9424400      PUSH    DWORD 004442F9   
0167:0040950B 6804434400      PUSH    DWORD 00444304   
0167:00409510 2EFF15FC254400  CALL    NEAR \[CS:004425FC\]   
0167:00409517 50              PUSH    EAX   
0167:00409518 2EFF1554264400  CALL    NEAR \[CS:00442654\] //这里报 API not found, please contect technical support    
  
0167:0040951F 6A00            PUSH    BYTE +00   
0167:00409521 2EFF15F8264400  CALL    NEAR \[CS:004426F8\]   
0167:00409528 EB28            JMP      SHORT 00409552   
0167:0040952A 6A10            PUSH    BYTE +10   
0167:0040952C 57              PUSH    EDI   
0167:0040952D FF15F0084500    CALL    NEAR \[004508F0\]  //这里弹出警告   
0167:00409533 85C0            TEST    EAX,EAX   
0167:00409535 751B            JNZ      00409552        //关键   
0167:00409537 6A10            PUSH    BYTE +10   
0167:00409539 6835434400      PUSH    DWORD 00444335   
0167:0040953E 683A434400      PUSH    DWORD 0044433A   
0167:00409543 2EFF15FC254400  CALL    NEAR \[CS:004425FC\]   
0167:0040954A 50              PUSH    EAX   
0167:0040954B 2EFF1554264400  CALL    NEAR \[CS:00442654\]   
  
最好还是要谢谢电神大哥,没有他的指点,我是怎么也脱不了这个壳的, :)   
                            
                                                                            fwnl   
                                                                          2002.5.1   
                                                                            长沙  4dd466d0e1e0cc123afbe504da2462ca