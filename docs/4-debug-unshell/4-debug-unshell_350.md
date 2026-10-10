

# FTPrint

> 原文：[`www.pediy.com/kssd/pediy03/forum569.htm`](https://www.pediy.com/kssd/pediy03/forum569.htm)

FTPrint 的脱壳(asprotect)   
  
小弟只是菜鸟一个，看了冰毒的 AZPR3.0 脱壳的文章，还有 hying 对于 HTMLZip 脱壳写的文章，受益匪浅，先拿 Advanced Registry Tracer 做了实验（Ken 写过脱壳的方法），然后拿来了 FTPrint 这个软件，独立战斗，结果成功。(软件在哪里下载的我不记得了，估计在华军主页上吧)   
  
不过，仍有不明白的地方。就是 hying 兄使用的 imp\_list.exe 在这个软件中似乎 dump 不出正确的.idata，如果哪位兄台成功了，请告知详细方法。   
  
软件介绍：   
\_\_\_\_\_\_\_\_\_   
  
    一般打印都是在图形模式下，而 FTPrint 可以在文本模式下打印，这样打印更加快速，更加省墨，特别是对于喷墨打印而言。   
  
加壳：   
\_\_\_\_\_   
  
    asprotect 1.00 (FI 检测)   
  
使用工具：   
\_\_\_\_\_\_\_\_\_   
  
TRW2000   
PE-Editor   
冲击波 2000   
HEX WORKSHOP   
  
过程：   
\_\_\_\_\_   
  
1\. 使用冲击波 2000 探测入口，在 46B8AC。   
2\. PE-Editor 查看 Section，.idata 的 Virtual Size=3000，Virtual Offset=77000，ImageBase=400000。   
3\. 用 TRW2000 载入 FTPrint.exe，   
（1）g 46B8AC，   
（2）pedump h.exe   
4\. 运行 FTPrint，退出，然后再 Load FTPrint。   
  \[以下借用冰毒在其文 AZPR3.0 的脱壳中的方法，文见看雪兄的论坛精华 II\]   
  下 bpm 477000 (Virtual Offset + ImageBase)，在 3 次 g 后用 F10 追，直到   
016F:004E8369 50              PUSH    EAX   
016F:004E836A B894764E00      MOV      EAX,004E7694   
016F:004E836F 50              PUSH    EAX   
016F:004E8370 B8A8764E00      MOV      EAX,004E76A8   
016F:004E8375 50              PUSH    EAX   
016F:004E8376 6A00            PUSH    BYTE +00   
016F:004E8378 B8B0764E00      MOV      EAX,004E76B0   
016F:004E837D 50              PUSH    EAX   
016F:004E837E B830A54D00      MOV      EAX,004DA530   
016F:004E8383 50              PUSH    EAX   
016F:004E8384 8B4508          MOV      EAX,\[EBP+08\]   
016F:004E8387 8B4020          MOV      EAX,\[EAX+20\]   
016F:004E838A 50              PUSH    EAX   
016F:004E838B 8B4508          MOV      EAX,\[EBP+08\]   
016F:004E838E 8D4824          LEA      ECX,\[EAX+24\]   
016F:004E8391 8B4508          MOV      EAX,\[EBP+08\]   
016F:004E8394 8B500C          MOV      EDX,\[EAX+0C\]   
016F:004E8397 8B4508          MOV      EAX,\[EBP+08\]   
016F:004E839A 8B4008          MOV      EAX,\[EAX+08\]   
016F:004E839D E8FEF4FFFF      CALL    004E78A0   
016F:004E83A2 33C0            XOR      EAX,EAX  <-------- RET 在这里，dump .idata   
    TRW2000 为 w 477000 l 3000 idata.bin(文件名字自己定的)   
5\. 运行 FTPrint，退出，关闭 TRW2000。   
  
以下为修复 PE:   
6\. PE-Editor 打开 h.exe，Directory 按钮，改 Import Table 的 RVA 为 77000，Size 为 3000。退出 PE-Editor。   
7\. Hex Workshop 打开 h.exe，go 到 77000，选定 Block 为 3000，然后再打开 idata.bin（干净的 import table），选定 Block 为 3000，拷贝，粘贴到打开的 h.exe 中，存盘。   
  
  这样应该就可以了。   
  
  由于初学脱壳，文中有不妥之处还请多指教。  4dd466d0e1e0cc123afbe504da2462ca