

# 深圳腾讯\[第一阶段 第四题\]脱壳脚本

> 原文：[`www.pediy.com/kssd/pediy11/124796.html`](https://www.pediy.com/kssd/pediy11/124796.html)

![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")工作比较忙，利用业余时间写出来的脱壳脚本。脚本包括了修复该程序所需的所有功能，比如修复无效指令、修复 section 表等等 功能。不过这个脚本要运行比较久![](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心")  
使用了单链表~~  
附近是修复脚本，这两个脚本可以整成一个  到达 OEP 的脚本如下：  
  
bpcnd 0065350E, "ECX==0" // 4  
run  
bpcnd 00656D9D, "ECX==0" // 5  
run  
  
  
bp 00656DF4  
mov  eax, 0B7  
run  
  
  
bp 00656E3D        // F1              int1  
mov eip, 00656E5C  //  
  
  
  
  
bpcnd 00655D2E, "ECX==0" // 第二个程序 1  
run  
  
  
//pause  
  
  
  
bpcnd 00653A6B, "ECX==0" // 第二个程序 3  
run  
  
  
//pause  
  
  
bpcnd 00655F31, "ECX==0" // 第二个程序 4  
run  
  
  
bp 00655F92    // 一步到位  
esto  
bc 00655F92  
  
  
//pause  
//00655B2A    0958 20         or      dword ptr \[eax+20\], ebx          ; 这里抹掉了映像长度  
fill 00655B2A, 3, 90  
  
  
bpcnd 006567FA, "ECX==0" // 第二个程序 8  
run  
  
  
  
//pause    //4001E0  
fill 0065598B, 16, 90  
fill 006559B4, D, 90  
fill 006559C7, 9, 90  
  
  
bp 00655A05        // 下断 shift + F9  
esto  
bc 00655A05  
  
  
bpcnd 00656277, "ECX==0" // 第二个程序 11  
run  
  
  
bpcnd 0065463A, "ECX==0" // 第二个程序 12  
run  
  
  
bpcnd 0065775C, "ECX==0" // 第二个程序 13  下一条就是 F1  int1 了  
run  
  
  
bp 00657765        // F1              int1                                     ; 这里 EIP 重定向+2B = 657791  
mov eip, 657791  
bc 00657765      
  
  
  
  
//006548B5    8813              mov     byte ptr \[ebx\], dl                         ; DllName 清零 NOP  
fill 006548B5, 2, 90  
  
//00653F38    8327 00           and     dword ptr \[edi\], 0                         ; Clear HashCode NOP it  
fill 00653F38, 3, 90  
  
  
//0065413E    FE4F FF            dec     byte ptr \[edi-1\]  
//00654141    83C7 04            add     edi, 4  
//00654144    2BC7               sub     eax, edi  
//00654146    8947 FC            mov     dword ptr \[edi-4\], eax  
fill 0065413E, B, 90  
  
//0065418A    8907               mov     dword ptr \[edi\], eax      ;  这里填充 API 地址  
//            8902                  mov     dword ptr \[edx\], eax    ; Modify  
//006549B7   
//fill 0065418A, 2, 90  
fill 0065418B, 1, 02  
  
//00654A83    8366 10 00         and     dword ptr \[esi+10\], 0  
//00654A87   /74 01              je      short 00654A8A  
//这里需要再 NOP3 个字节  
fill 00654A83, 7, 90  
  
//00654C59    F3:A4              rep     movs byte ptr es:\[edi\], byte ptr \[esi\]                ; copy to 4001E0  NOP it  
fill 00654C59, 2, 90  
  
  
//006549B7    0FBA67 FF 07    bt      dword ptr \[edi-1\], 7             ; 这里做是否 Hook 的判断 可以改为 clc  F8  
fill 006549B7, 1, F8  
fill 006549B8, 4, 90  
  
//pause   
  
bp 00654CB5   
run  
  
  
mov  eip, 0041DD00  
mov ecx, 0041DD00  
mov \[ecx\], #558BEC6AFF6868DB5B0068182D420064A100000000506489250000000083C4A45356578965E8FF1521816500A3A4A75E00A1A4A75E00C1E80825FF000000A3B0A75E008B0DA4A75E0081E1FF000000#  
  
//0040013C    00040000    DD 00000400          ;  SizeOfHeaders = 400 (1024.)  
mov \[40013C\], #E0010000#  
mov \[400168\], #00E01E00#  
mov \[40016C\], #04010000#  
mov \[5EE0FC\], #00000000#  
mov \[5EE100\], #00000000#  
  
  
ret

上传的附件
| ![](img/30b3588c36e23afc1abe1f92c2cce6e9.jpg) || ProcessCodeFinal.txt  
 |