

# ExeCryptor v2.X.X 虚拟机不完全补完(处女帖)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-609.htm`](https://www.pediy.com/kssd/pediy08/pediy8-609.htm)

该文参考 softworm 大大文章及其各位研究 ExeCryptor VM 的部分总结完善而得  
  
  
序言  
ExeCryptor 的 VM 算不上完整的全代码 VM，因为他只 VM 少部分的常用 CODE，相对 THIMDA 的 VM 来比应该算少的多的 VM 了。ExeCryptor 将  
VM 代码，真实代码及其废代码混在一起，要你一个剥离还是比较累的。  
  
第一步  
解析 PCODE 的参数运算，和 softworm 分析的一样，他进行了参数的复杂运算，我这次的实列好象比以前 softworm 大大分析的解密代码是  
一致的如下：（dwData 为参数值）  
  
mov eax,dwData  
mov edx,dwData  
ror eax,0x10  
rol dx,0x3  
add edx,dwSeed1  
xor edx,dwSeed2  
xor ax,dx  
mov dwData,eax  
\------------------------------------------------  
这段代码循环 16 次，第四次的时候将参与下面的运算  
\------------------------------------------------  
第四轮后做一次检查  
007A7856 主       XOR AX,DX                                 ; FL=0, EAX=B172532C  
  
007A7859 主       PUSH EDI                                  ; ESP=0012F954---------------------->EFL  
007A785A 主       POP EDX                                   ; EDX=00200206, ESP=0012F958  
  
007B676F 主       TEST DH,1                                 ; FL=PZ  
007B6772 主       SETNE CL                                  ; ECX=0BC8E700  
007B6775 主       ROR EAX,CL  
007B6777 主       PUSH EAX                                  ; ESP=0012F954  
007B6778 主       PUSH E4A1CE3B                             ; ESP=0012F950  
007B677D 主       POP EAX                                   ; EAX=E4A1CE3B, ESP=0012F954  
007B677E 主       JMP lin.007A3638  
007A3638 主       AND EAX,9C6099A2                          ; FL=PS, EAX=84208822  
007A363E 主       ADD EAX,9DEA2DF4                          ; FL=CO, EAX=220AB616  
007A3644 主       JNS lin.007ABE59  
007ABE59 主       OR EAX,1ABB98AD                           ; FL=0, EAX=3ABBBEBF  
007ABE5F 主       XOR EAX,3ABBB67E                          ; EAX=000008C1  
007ABE65 主       AND EDX,EAX                               ; FL=PZ, EDX=00000000---------> dwEFlag & 0x000008c1  
007ABE67 主       POP EAX                                   ; EAX=B172532C, ESP=0012F958  
007ABE68 主       JMP lin.007A38AB  
007A38AB 主       JMP lin.007A522E  
007A522E 主       OR EDX,FFFFF73E                           ; FL=S, EDX=FFFFF73E  -------------> 前面运算的值   | 0xFFFFF73E  
007A5234 主       AND EAX,EDX                                -------> dwData & 前面运算的值  
  
换成 C 为：  
dwData &=(dwEFlag & 0x000008c1) | 0xFFFFF73E  
  
\------------------------------------  
上面提到的的 dwSeed1，dwSeed2 见下  
\---------------seed-----------------  
D5F32EE8,3BB9C21F  
4D51DFF,DA4AAB8A  
CCB64EEB,CA6D5E88  
CE1D5985,92FDE4E1  
B2928B43,E4291FEA  
2C9B2D4B,A987B1A4  
31F45613,180A72FB  
400C665B,5E22663D  
6B003FE8,C5DBB853  
9C3A8FE6,F91D2FB3  
3F187E27,E8F0242D  
70E9A6C8,C93FAB46  
B9BAE97A,531414AA  
DC7E74FF,6C993787  
F43F5FD6,929C0C05  
710F6FDC,372EF51B  
D75922C4,7D56C0C2  
618AC95F,63E27BB4  
DAA433DA,05092A86  
5E675CD8,19392E42  
E33B20AE,E68EF219  
70EA9678,E5BA7291  
DE827B5A,D38F78F6  
78232635,8A5DA9AF  
35D1C121,D1F503E7  
460C174B,E43D7B0A  
C25A9D9F,96495B57  
AD301179,4BE7E85F  
288E62D6,1251B610  
159FA2A1,6CBD33F9  
6791AA88,2D5D2DE1  
E3818646,0F70C0CE  
\------------------------------------  
这里的 SEED 和 softworm 取的不一样，难道每个单独的加密是各自不同的 SEED？没验证……  
  
  
第二步  
解析运算后的 OPCODE 指令含义  
  
这里要说下，解密出来的 OPCODE 包含了指令和运算的寄存器及其是否读下个常数等  
这里可以参看 softworm 大大的文章  
<ExeCryptor v2.2.6 虚拟机不完全分析>  
http://bbs.pediy.com/showthread.php?s=&threadid=17763&highlight=%B2%BB%CD%EA%C8%AB%B7%D6%CE%F6  
  
  
  
我这里简单补充下其他 softworm 大大没讲到的：(简单说明，请大家不要见笑)  
  
switch (dwPlain & 0x000000FF)  
  {  
  case 0x3:  //mov \[reg32\],reg32  
  第一个寄存器：(dwPlain >> 0xb) & 0x7  
  第二个寄存器：(dwPlain >> 0x8) & 0x7  
    
    
  case 0x4: //mov reg32,\[reg32\]  
  第一个寄存器：(dwPlain >> 0xb) & 0x7  
  第二个寄存器：(dwPlain >> 0x8) & 0x7  
    
    
  case 0x07: //not reg32  
  第一个寄存器：(dwPlain >> 0x8) & 0x7  
    
    
  case 0x10://pushfd  
    
    
  case 0x11://popfd  
    
    
  case 0x12://TEST reg32,reg32  
  第一个寄存器：(dwPlain >> 0x8) & 0x7  
  第二个寄存器：(dwPlain >> 0xb) & 0x7  
    
    
  case 0x16:// ROR reg32,reg8  
  第一个寄存器：(dwPlain >> 0x8) & 0x7  
  第二个寄存器：(dwPlain >> 0xb)   
    
  case 0x20://TEST reg8,reg8  
  第一个寄存器：(dwPlain >> 0x8) & 0x7  
  第二个寄存器：(dwPlain >> 0xb) & 0x7  
  }  
  
if(0x8 == (dwPlain & 0x000000F8))    
    {  
      switch(dwType)    
      {  
  case 0x01://or reg32,reg32  
  第一个寄存器：(dwPlain >> 0xb) & 0x7  
  第二个寄存器：(dwPlain >> 0x8) & 0x7  
    
    
  case 0x04://AND reg32,reg32  
  第一个寄存器：(dwPlain >> 0xb) & 0x7  
  第二个寄存器：(dwPlain >> 0x8) & 0x7  
    
  case 0x05://SUB reg32,reg32  
  第一个寄存器：(dwPlain >> 0xb) & 0x7  
  第二个寄存器：(dwPlain >> 0x8) & 0x7  
    
  case 0x06://XOR reg32,reg32  
  第一个寄存器：(dwPlain >> 0xb) & 0x7  
  第二个寄存器：(dwPlain >> 0x8) & 0x7    
  }  
}  
  
  
第三步  
去废代码  
  
这个时候我们应该得到很完整的去 VM 的代码了，当然 ExeCryptor 不会让你那么轻松。他有很多的废代码等你去看，呵呵  
  
列如这样的代码等于 NOP：  
  
PUSH XXXXXX  
RETN  
  
  
CALL XXXXXXX  
XCHG DWORD PTR SS:\[ESP\],EDI  
POP EDI  
  
CALL XXXXXXX  
XCHG \[ESP\],EDI  
XCHG DWORD PTR SS:\[ESP\],ESI  
MOV EDI,ESI  
POP ESI  
  
……  
  
这些代码等于混乱正常代码用的，自己写个程序去一下花指令就又清爽和减肥了！  
  
  
恩，还有其他的参与运算的也会这样变换：  
比如 ADD EAX,100000  
  
他处理成这样  
PUSH ECX  
PUSH ED214  
POP ECX  
ADD ECX, 12DEC  
ADD EAX,ECX  
POP ECX  
  
当然你不管也可以，追求完美的可以自行修复！这样的变换很多，基本雷同，没什么技术含量了！  
  
注：本文没技术含量，都是借鉴各位大大自己学习总结而得，如有冒犯请联系斑竹删除！  
  
  
参考文献：  
ExeCryptor v2.2.6 虚拟机不完全分析                softworm  
EXECryptor2.3.9 主程序脱壳【分享】                DarkBull 

 4dd466d0e1e0cc123afbe504da2462ca