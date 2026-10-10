

# xprotector 之中斷處理 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6819.htm`](https://www.pediy.com/kssd/pediy06/pediy6819.htm)

對於一個 ring0 級加殼程序來說，中斷處理是很重要的。在它的幫助中這樣描述:  
Wizard: Adding Interrupt 1-3 Support   
  
In this step you can add debugging interrupts support in your program to make your program fully compatible with Xtreme-Protector in case that you use the interrupt 1 or 3 in your program or dlls.   
  
Details   
  
Xtreme-Protector has a total control of the debugging interrupts to protect your program against debuggers. If your program uses the INT3 or INT1 instruction to detect debuggers or you insert debugging exceptions inside your code to debug your OWN process, you must add support for it with Xtreme-Protector.   
  
If you don't use or don't know if you use INT1-3 instructions in your program, do NOT add support for it to avoid incompatibilities with your program.   
從上面可以看出，xprotector 對於中斷 1\\3 的處理是很強勁的。讓我們實地勘測吧！運行 xprotector，出現主窗口後。呼出?#123;試器，idt 查看中斷。顯示如下:  
IDTbase=8003F400  Limit=07FF  
0000  0  P   IntG32    00000008:8052F0B0  
0001  3  P   IntG32    00000008:FFFFFFFF  中斷 1  
0002  0  P   TaskG     00000058  
0003  3  P   IntG32    00000008:FFFFFFFF  中斷 3  
0004  3  P   IntG32    00000008:8052F694  
0005  0  P   IntG32    00000008:8052F7DC  
0006  0  P   IntG32    00000008:8052F93C  
0007  0  P   IntG32    00000008:8052FF14  
0008  0  P   TaskG     00000050  
0009  0  P   IntG32    00000008:80530334  
000A  0  P   IntG32    00000008:8053043C  
000B  0  P   IntG32    00000008:80530568  
000C  0  P   IntG32    00000008:80530734  
000D  0  P   IntG32    00000008:805309FC  
000E  0  P   IntG32    00000008:F8368961  中斷 0E  
000F  0  P   IntG32    00000008:80531434  
  
從上面可以看出，程式運行後，改變了中斷 1 和中斷 3 的處理入口\_ 慒 FFFFFFFH,改變中斷 0e 的處理入口\_ 慒 8368961H。  
我們再看看 F8368961 處的代碼,U F8368961,如下：  
0xFC682961: 9C             pushfd                                 
0xFC682962: 60             pushad                                 
0xFC682963: E800000000     call     0xfc682968                    
0xFC682968: 5D             pop      ebp                          <==0xFC682963(\*-0x5)  
0xFC682969: 81EDFB45FF01   sub      ebp,0x1ff45fb                 
0xFC68296F: 8B442428       mov      eax,dword \[esp+0x28\]   出錯地址    
0xFC682973: 83F8FF         cmp      eax,0xff                      
0xFC682976: 7413           jz       0xfc68298b                   (\*+0x15)   
0xFC682978: 8B442424       mov      eax,dword \[esp+0x24\]      
0xFC68297C: 83F8FF         cmp      eax,0xff                      
0xFC68297F: 0F85CD000000   jnz      0xfc682a52                   (\*+0xD3)   
0xFC682985: 61             popad                                  
0xFC682986: 9D             popfd                                  
0xFC682987: 83C40C         add      esp,0xc                       
0xFC68298A: CF             iretd                                  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
0xFC68298B: B801000000     mov      eax,0x1                      <==0xFC682976(\*-0x15)  
0xFC682990: 0BC0           or       eax,eax                       
0xFC682992: 753E           jnz      0xfc6829d2                   (\*+0x40)   
0xFC682994: B878563412     mov      eax,0x12345678                
0xFC682999: 8B00           mov      eax,dword \[eax\]           
0xFC68299B: 83780800       cmp      dword \[eax+0x8\],0x0       
0xFC68299F: 7473           jz       0xfc682a14                   (\*+0x75)   
0xFC6829A1: 83780802       cmp      dword \[eax+0x8\],0x2       
0xFC6829A5: 776D           ja       0xfc682a14                   (\*+0x6F)   
0xFC6829A7: 83780801       cmp      dword \[eax+0x8\],0x1       
0xFC6829AB: 741A           jz       0xfc6829c7                   (\*+0x1C)   
0xFC6829AD: 8BDC           mov      ebx,esp                      <==0xFC682A05(\*+0x58)  
0xFC6829AF: 83C334         add      ebx,0x34                      
0xFC6829B2: 8B03           mov      eax,dword \[ebx\]           
0xFC6829B4: 8078FFCC       cmp      byte \[eax\-0x1\],0xcc       
0xFC6829B8: 7507           jnz      0xfc6829c1                   (\*+0x9)   
0xFC6829BA: C640FFC5       mov      byte \[eax\-0x1\],0xc5       
0xFC6829BE: 40             inc      eax                           
0xFC6829BF: EB77           jmp      0xfc682a38                   (\*+0x79)   
0xFC6829C1: C640FEC5       mov      byte \[eax\-0x2\],0xc5      <==0xFC6829B8(\*-0x9)  
0xFC6829C5: EB71           jmp      0xfc682a38                   (\*+0x73)   
0xFC6829C7: 8D85E746FF01   lea      eax,dword \[ebp+0x1ff46e7\]; <==0xFC6829AB(\*-0x1C)  
0xFC6829CD: 895001         mov      dword \[eax+0x1\],edx       
0xFC6829D0: FFE0           jmp      eax                           
0xFC6829D2: 0FA0           push     fs                           <==0xFC682992(\*-0x40)  
0xFC6829D4: B830000000     mov      eax,0x30                      
0xFC6829D9: 668EE0         mov      fs,al                         
0xFC6829DC: 64A124010000   mov      eax,fs:\[0x124\]               eax<---pETHREAD  
0xFC6829E2: 8B4044         mov      eax,dword \[eax+0x44\]    +044 struct \_KPROCESS  
0xFC6829E5: 0FA1           pop      fs                              
0xFC6829E7: 8BD8           mov      ebx,eax                       
0xFC6829E9: 81E3FFFFFF7F   and      ebx,0x7fffffff                
0xFC6829EF: BE492F68FC     mov      esi,0xfc682f49               DATA: daG  
0xFC6829F4: 83C604         add      esi,0x4                      <==0xFC682A07(\*+0x13)  
0xFC6829F7: 813E20646147   cmp      dword \[esi\],0x47616420    
0xFC6829FD: 7415           jz       0xfc682a14                   (\*+0x17)   
0xFC6829FF: 3906           cmp      dword \[esi\],eax           
0xFC682A01: 7406           jz       0xfc682a09                   (\*+0x8)   
0xFC682A03: 391E           cmp      dword \[esi\],ebx           
0xFC682A05: 74A6           jz       0xfc6829ad                   (\*-0x58)   
0xFC682A07: EBEB           jmp      0xfc6829f4                   (\*-0x13)   
0xFC682A09: 8D85E746FF01   lea      eax,dword \[ebp+0x1ff46e7\]; <==0xFC682A01(\*-0x8)  
0xFC682A0F: 895001         mov      dword \[eax+0x1\],edx       
0xFC682A12: FFE0           jmp      eax                           
0xFC682A14: 8BDC           mov      ebx,esp                      <==0xFC6829FD(\*-0x17), 0xFC6829A5(\*-0x6F), 0xFC68299F(\*-0x75)  
0xFC682A16: 83C334         add      ebx,0x34                      
0xFC682A19: 8B03           mov      eax,dword \[ebx\]           
0xFC682A1B: 8078FFCC       cmp      byte \[eax\-0x1\],0xcc       
0xFC682A1F: 742A           jz       0xfc682a4b                   (\*+0x2C)   
0xFC682A21: 668178FECD03   cmp      word \[eax\-0x2\],0x3cd      
0xFC682A27: 7422           jz       0xfc682a4b                   (\*+0x24)   
0xFC682A29: 668178FECD01   cmp      word \[eax\-0x2\],0x1cd      
0xFC682A2F: 7407           jz       0xfc682a38                   (\*+0x9)   
0xFC682A31: 61             popad                                  
0xFC682A32: 9D             popfd                                  
0xFC682A33: 83C410         add      esp,0x10                      
0xFC682A36: EB26           jmp      0xfc682a5e                   (\*+0x28)   
0xFC682A38: 83E802         sub      eax,0x2                      <==0xFC682A2F(\*-0x9), 0xFC6829C5(\*-0x73), 0xFC6829BF(\*-0x79)  
0xFC682A3B: 8903           mov      dword \[ebx\],eax           
0xFC682A3D: 61             popad                                  
0xFC682A3E: 9D             popfd                                  
0xFC682A3F: 83C40C         add      esp,0xc                       
0xFC682A42: C704242A000000 mov      dword \[esp\],0x2a          
0xFC682A49: EB1F           jmp      0xfc682a6a                   (\*+0x21)   
0xFC682A4B: 61             popad                                 <==0xFC682A27(\*-0x24), 0xFC682A1F(\*-0x2C)  
0xFC682A4C: 9D             popfd                                  
0xFC682A4D: 83C410         add      esp,0x10                      
0xFC682A50: EB12           jmp      0xfc682a64                   (\*+0x14)   
0xFC682A52: 61             popad                                 <==0xFC68297F(\*-0xD3)  
0xFC682A53: 9D             popfd                                  
0xFC682A54: 6888105380     push     0x80531088                    
0xFC682A59: C3             ret                                    
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
0xFC682A5E: 6804F25280     push     0x8052f204                   <==0xFC682A36(\*-0x28)  
0xFC682A63: C3             ret                                    
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
0xFC682A64: 682CF55280     push     0x8052f52c                   <==0xFC682A50(\*-0x14)  
0xFC682A69: C3             ret                                    
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
0xFC682A6A: 68FC095380     push     0x805309fc                   <==0xFC682A49(\*-0x21)  
0xFC682A6F: C3             ret                                    
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
從上面的代碼可以看出，當用戶級的程式産生 int 3 異常後，將會跳到到 ffffffffh，但 ffffffffh 是不訪問的，所以就會産生一個頁異常，對應的中斷就是 int 0e，在跳到 int 0e 的處理程式中，它會用 peb 來判斷産生異常的程式是不是 xprotector 自身程式。如果不是，那麽會判斷産生異常的 eip 處的指令是不是 CC,CD 等，也就是看是不是 int 3,int 1，如果是的話，就跳到原來正確的 int 1 和 int 3 處理入口繼續運行，這樣就完成了用戶級程式的 INT 1,INT 3 處理了。  
寫得不好，請指教!  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;此處對應 0xFC6829EF 處的代碼,用於判斷當前程式的 peb:  
0xFC682F49  20 64 61 47 D0 7C 8F FF  20 64 61 47 00 00 00 00   daG.|.. daG....  
0xFC682F59  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................  
  
D0 7C 8F FF 也就是 FF8F7CD0 處的數據\_ 懀?br />   
0xFF8F7CD0  03 00 1B 00 01 00 00 00  D8 7C 8F FF D8 7C 8F FF  .........|...|..  
0xFF8F7CE0  E0 7C 8F FF E0 7C 8F FF  00 B0 70 06 00 C0 D8 04  .|...|....p.....  
0xFF8F7CF0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................  
0xFF8F7D00  88 00 00 00 00 00 00 00  4A 00 00 00 34 00 00 00  ........J...4...  
0xFF8F7D10  10 7D 8F FF 10 7D 8F FF  00 00 00 00 00 00 00 00  .}...}..........  
0xFF8F7D20  20 7D 8F FF 20 7D 8F FF  00 00 00 00 01 00 00 00   }.. }..........  
0xFF8F7D30  00 00 0D 24 00 00 00 00  00 00 00 00 00 00 00 00  ...$............  
0xFF8F7D40  A0 DE C0 8B 4F D1 C3 01  20 63 63 C4 50 D1 C3 01  ....O... cc.P...  
0xFF8F7D50  01 00 00 00 90 0F 00 00  B0 1A 1E 81 60 A8 93 FD  ............\`...  
0xFF8F7D60  00 00 00 00 00 00 00 00  00 00 00 00 B8 29 00 00  .............)..  
0xFF8F7D70  28 A9 00 00 70 07 00 00  00 00 00 00 00 80 30 05  (...p.........0.  
0xFF8F7D80  00 E0 A5 04 5C 6E 48 FD  8C A8 93 FD 00 00 00 00  ....\\nH.........  
0xFF8F7D90  40 73 6A E1 00 00 00 00  72 04 26 E1 01 00 00 00  @sj.....r.&.....  
0xFF8F7DA0  30 CC 05 F8 00 00 00 00  01 00 04 00 00 00 00 00  0...............  
0xFF8F7DB0  B0 7D 8F FF B0 7D 8F FF  00 00 00 00 8E 2B 00 00  .}...}.......+..  
0xFF8F7DC0  01 00 00 00 00 00 00 00  00 00 00 00 01 00 04 00  ................  
0xFF8F7DD0  00 00 00 00 D4 7D 8F FF  D4 7D 8F FF 00 00 00 00  .....}...}......  
0xFF8F7DE0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................  
0xFF8F7DF0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................  
0xFF8F7E00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 40 00  ..............@.  
0xFF8F7E10  98 19 10 81 00 00 00 00  2C 00 00 00 B4 04 00 00  ........,.......  
0xFF8F7E20  00 00 00 00 00 00 00 00  00 00 00 00 80 93 CF E1  ................  
0xFF8F7E30  30 7E 8F FF 30 7E 8F FF  00 00 00 00 00 00 00 00  0~..0~..........  
0xFF8F7E40  00 E0 9D FC 58 70 72 6F  74 65 63 74 6F 72 2E 65  ....Xprotector.e  
0xFF8F7E50  78 65 00 00 00 00 00 00  00 00 00 00 00 00 00 00  xe..............  
0xFF8F7E60  60 7E 8F FF 60 7E 8F FF  00 00 00 00 00 00 00 00  \`~..\`~..........  
0xFF8F7E70  00 00 00 00 FF 0F 1F 00  01 00 00 00 DE 00 00 00  ................  
0xFF8F7E80  00 F0 FD 7F 00 00 00 00  04 00 00 00 00 00 00 00  ................  
0xFF8F7E90  01 00 00 00 00 00 00 00  C0 00 00 00 00 00 00 00  ................  
0xFF8F7EA0  44 96 22 00 00 00 00 00  48 00 00 00 00 00 00 00  D.".....H.......  
0xFF8F7EB0  04 0A 00 00 00 00 00 00  00 00 00 00 70 07 00 00  ............p...  
從上面的數據可以看出，這是一段典型的 peb 結構,在我的機子上，偏移 17ch 處指向 imagename:xprotector  
  
在 0xFC68296F 處下斷點，動態觀看處理過程，運行一個再有 int 3 指令的程式，自己寫的，運行後，中斷在 0xFC68296F，我們看看堆棧。  
d esp+24，如下：  
0xFCb89454  00 00 00 00 FF FF FF FF  08 00 00 00 46 00 01 00   ....    ...F...  
0xFCb89464  BE 11 40 00 1B 00 00 00  46 03 00 00 98 F7 12 00  ..@.....F.......  
可以看到，的確如上面的分析，程式在 int 3 處理入口産生了頁異常再跳到 int 0e 的處理入口.說明：004011BE 處的指令就是一個 INT 3。  
希望對 xprotector 有過研究的人多提意見。共同探討.  
  

 4dd466d0e1e0cc123afbe504da2462ca