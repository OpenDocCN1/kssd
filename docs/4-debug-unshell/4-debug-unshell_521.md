

# ASProtect SKE 2.11 变形代码浅浅谈

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-544.htm`](https://www.pediy.com/kssd/pediy08/pediy8-544.htm)

这段时间学习了一下 ASProtect SKE 2.11 的壳，感谢 shoooo 给与技巧指点，让我少走了不少弯路。shoooo 一文[**nspack3.5 主程序脱壳分析(Aspr SKE 2.X)** ](http://bbs.pediy.com/showthread.php?threadid=19313 ) 给人启发不少，看懂这篇文章，再展开一下，ASProtect SKE 的壳基本能拿下。  
ASProtect 脱壳其实也可以不修复 Stolen Code，直接将壳的这段代码借用过来。但如果要完美脱壳的话，就必须了解点 ASProtect 中的一些变形技巧了，将代码一句句还原（是个体力活）。  
  
ASProtect SKE 难点就在代码变形，用了一段代码来模拟如下指令（可能是作者自己写的变形引擎）：  
cmp x,y  
jxx n  
  
call xxxxx  
jmp  xxxxx  
  
jxx  n    
  
等等…  
  
例如：  
下面这段变形代码的作用是模拟 cmp 指令。  
cmp x,y  
jxx n  
  
00A8896A    8BCE            mov     ecx, esi                       
00A8896C    8B55 0C         mov     edx, \[ebp+C\]  
00A8896F    8B45 F8         mov     eax, \[ebp-8\]  
00A88972    E8 D5FDFFFF     call    00A8874C                           
{  
      00A8877E    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
      00A88782    8BC7            mov     eax, edi  
      00A88784    FFD2            call    edx           ;此 CALL 返回决定是何寄存器 (x)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
      ……  
      00A887BA    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
      00A887BE    8BC7            mov     eax, edi  
      00A887C0    FFD2            call    edx           ; (y)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
      ……  
}  
  
00A88977    8945 10         mov     \[ebp+10\], eax  
00A8897A    EB 01           jmp     short 00A8897D  
00A8897C  - E9 33C08A43     jmp     443349B4  
00A88981    04 8B           add     al, 8B  
00A88983    55              push    ebp  
00A88984    F8              clc  
00A88985    8B5482 40       mov     edx, \[edx+eax\*4+40\]  
00A88989    8BC6            mov     eax, esi  
00A8898B    FFD2            call    edx                              ; 此 CALL 返回值决定跳转类型 3=jnb,2=jb,……  
00A8898D    8BD8            mov     ebx, eax  
00A8898F    8B4D 10         mov     ecx, \[ebp+10\]  
00A88992    8BD3            mov     edx, ebx  
00A88994    8B45 F8         mov     eax, \[ebp-8\]  
00A88997    E8 74FBFFFF     call    00A88510                         ; 里面比较(cmp x,y)  
00A8899C    84C0            test    al, al                             
00A8899E    74 17           je      short 00A889B7  
  
  
  
ASProtect SKE 最新版也在这方面加强了，这部分我还没搞明白，别问我。;(  
  
  
对于一般的指令 ASProtect 也变形，这部分识别起来简单多了，自己用 ASProtect SKE 压一个软件，对比跟踪一下，很快能找到规律。下面列出的是 ASProtect 常用的一些简单变形指令。原理是利用 lea 来指令来计算加,减,乘法，或利用堆栈来传递一些数据。  
  
1.  
原代码：  
add     esi, 8  
变形后的代码：  
lea     esi, \[esi+8\] //esi=esi+8  
  
2  
原代码：  
add     edi, 4  
变形后的代码：  
lea     edi, \[edi+ecx+4\] //edi=edi+ecx+4  
sub     edi, ecx         //edi=edi-ecx=edi+ecx+4-ecx=edi+4  
  
3.  
原代码：  
 mov     ecx, eax  
变形后的代码：  
  lea     ecx, \[eax+C\]  //ecx=eax+c  
  lea     ecx, \[ecx-C\]  //ecx=ecx-c=eax+c-c=eax  
  
4.  
原代码：mov     esi, eax  
变形后的代码：  
  or      esi, BD20817E  
  push    eax //有效指令   
  rol     esi, 9D  
  xor     esi, \[esp+8\]  
  pop     esi//有效指令   
  
5.  
mov     esi, eax  
变形后的代码：  
  sub     esi, ecx  
  lea     esi, \[eax+2B\]  
  lea     esi, \[esi+ebp-2B\]  
  sub     esi, ebp  
  
  
6.  
mov     edi, eax  
变形后的代码：  
  lea     edi, \[edx+edi\*2+50\]   
  adc     edi, F299E634  
  lea     edi, \[eax+56\]     //edi =eax+56  
  lea     edi, \[edi+ecx-56\] //edi=eax+56+ecx-56=eax+ecx  
  sub     edi, ecx          //edi=edi-ecx=eax+ecx-ecx=eax  
  
7  
mov     edi, eax  
变形后的代码：  
  lea     edi, \[ebx+ecx+48612E\]  
  sub     edi, ecx  
  xor     edi, 1ABFD486  
  lea     edi, \[eax+ecx+39\] //edi=eax+ecx+39  
  sub     edi, ecx          //edi=edi-ecx=eax+ecx+39-ecx=eax+39  
  lea     edi, \[edi-39\]     //edi=edi-39=eax+39-39=eax  
   
  
8.  
mov     edi, eax  
变形后的代码：  
  lea     edi, \[ebx+ecx+48612E\]  
  sub     edi, ecx  
  xor     edi, 1ABFD486  
  lea     edi, \[eax+ecx+39\]  
  sub     edi, ecx  
  lea     edi, \[edi-39\]  
  
  
9.  
原代码：  
  mov     eax, 800  
变形后的代码：  
  push    ABCAA2A8  
  pushfw  
  push    ebx  
  xor     ebx, C567C100  
  and     ebx, edi  
  lea     ebx, \[esp+69\]  
  sub     ebx, 69  
  prefix repne:  
  jmp L010  
  prefix rep:  
L010:  
  add     ebx, 6  
  mov     dword ptr \[ebx\], 800 //有效指令  
  pop     ebx  
  popfw  
  add     eax, A38396E6  
  pop     eax                 //有效指令  
  
10  
原代码：  
mov     ecx, edi  
变形后的代码：  
  mov     ecx, 42338A  
  mov     ecx, 42A4FA  
  xor     ecx, \[esp+8\]  
  mov     ecx, 4780E2  
  rcr     ecx, 0ED  
  lea     ecx, \[edi+eax+5C\]  //ecx=edi+eax+5C  
  sub     ecx, eax           //ecx=ecx-eax=edi+eax+5C-eax=edi+5C  
  lea     ecx, \[ecx-5C\]      //ecx=ecx-5c=edi+5C-5C=edi  
  sar     eax, 5  
  and     ecx, 1F  
  
11  
原代码：  
mov     edi, 405904  
变形后的代码：  
  and     edi, ebx  
  mov     edi, 43FB92  
  ror     edi, 0C5  
  lea     edi, \[ebp+esi\*2+53\]  
  lea     edi, \[eax+ecx+405904\] //edi=eax+ecx+405904  
  sub     edi, ecx              //edi=edi-ecx=eax+ecx+405904-ecx=eax+405904  
  sub     edi, eax              //edi=edi-eax=eax+405904-eax=405904  
  
12.  
原代码：  
mov   ebp, esp  
变形后的代码：  
lea     ebp, \[esp+ecx+5C\]  //ebp=esp+ecx+5c  
sub     ebp, ecx           //ebp=ebp-ecx=esp+ecx+5c-ecx=esp+5c  
lea     ebp, \[ebp+ebx-5C\]  //ebp=ebp+ebx-5C=esp+5c+ebx-5C=esp+ebx  
sub     ebp, ebx           //ebp=ebp-ebx=esp+ebx-ebx=esp  
  
  
13  
 mov     eax, \[405A10\]  
变形后的代码：  
  lea     eax, \[ecx+405A10\]  
  sub     eax, ecx  
  push    dword ptr \[eax\]  
  xor     eax, \[esp+28\]  
  xor     eax, \[esp+8\]  
  pop     eax  
  
14.    
原代码：mov     ecx, \[eax\]  
变形后的代码：  
  xor     ecx, ebx  
  push    dword ptr \[eax\]  //\[eax\]入栈  
  rcr     ecx, 0C9  
  mov     ecx, 43378A  
  pop     ecx              //栈入数据放到 ecx  
  
  
15.  
原代码：    
  mov     eax, \[405900\]  
变形后的代码：  
  lea     eax, \[ebp+esi+424388\]    
  sub     eax, esi                 
  lea     eax, \[ecx+405900\]      //eax=ecx+405900  
  sub     eax, ecx               //eax=eax-ecx=ecx+405900-ecx=405900  
  mov     eax, \[eax\]               
  
  
16.  
原代码：  
  mov     esi, \[eax\]  
  
变形后的代码：  
  or      esi, edi  
  push    dword ptr \[eax\]  
  sub     esi, -45  
  rol     esi, 97  
  pop     esi  
  
17  
mov     eax, \[405900\]  
变形后的代码：  
  lea     eax, \[edi+42CA6C\]  
  xor     eax, 710E8EFA  
  mov     eax, 427EBE  
  add     eax, \[esp+18\]  
  lea     eax, \[ebp+esi+405900\]    
  sub     eax, esi  
  sub     eax, ebp  
  mov     eax, \[eax\]  
  
  
18.  
  
mov     eax, \[405664\]  
变形后的代码：  
  
  lea     eax, \[ecx+edx-7F\]  
  lea     eax, \[edi+405664\] //eax=edi+405664  
  sub     eax, edi          //eax=eax-edi=edi+405664-edi=405664  
  prefix repne:  
  jmp L006  
  ???  
L006:  
  push    dword ptr \[eax\]   //\[405664\]入栈  
  add     eax, \[esp+18\]  
  mov     eax, 4487CE  
  pop     eax              //\[405664\]出栈  
  
  
  
19.  
原代码：  
mov     ebp, esp  
变形后的代码：  
 add     ebp, ebx  
 mov     ebp, 45A51E  
 mov     ebp, \[esp+10\]                  
 mov     ebp, 4A54FE  
 mov     ebp, 463636  
 lea     ebp, \[esp+esi+E\]  //ebp=esp+esi+E  
 push    eax  
 push    edx  
 push    ecx  
 mov     edx, 45F39E  
 mov     edx, 447AD6   
 mov     edx, 7DF8A705     //edx=7DF8A705  
 mov     ecx, 4A86CA     
 mov     ecx, 488942    
 mov     ecx, edx          //ecx=edx=7DF8A705  
 sub     ecx, B783FC03     //ecx=ecx-B783FC03=7DF8A705-B783FC03=C674AB02  
 neg     ecx               //ecx=398B54FE  
 push    ecx               //398B54FE 入栈  
 xor     edx, \[esp+8\]  
 rcl     edx, 0EB  
 pop     edx               //398B54FE 出栈入 edx 中  
 add     edx, 63BE880B     //edx=edx+63BE880B=398B54FE+63BE880B=9D49DD09  
 xor     eax, \[esp+28\]  
 rcl     eax, 49  
 push    edx               //值 9D49DD09 入栈  
 sub     eax, ebx  
 pop     eax               //值 9D49DD09 出栈进 eax  
 xor     eax, 62B622F9     //9D49DD09 XOR 62B622F9=FFFFFFF0  
 neg     eax               //eax=10  
 lea     edx, \[ebp+esi+6C\] //edx=ebp+esi+6C  
 push    eax               //10 入栈  
 push    edi  
 mov     eax, 93D3D3A9  
 xchg    eax, edi  
 xor     edi, 4B566F99  
 neg     edi  
 sub     edi, 6A63E261  
 neg     edi  
 xchg    edi, eax  
 sub     eax, 42E99E29  
 dec     eax  
 lea     edx, \[edx+eax\*2+6A\]   
 lea     edx, \[edx+ecx-6A\]    
 sub     edx, ecx          
 sub     edx, eax          
 pop     edi  
 pop     eax   
 sub     edx, esi     
 push    eax   
 sbb     edx, -17  
 ror     edx, 7B  
 pop     edx                 //10 出栈放 edx，edx＝10  
 lea     ebp, \[ebp+edx+6F\]   //ebp=esp+esi+E+10+6F  
 lea     ebp, \[ebp+ecx-6F\]   //ebp=esp+esi+E+edx+6F+ecx-6F=esp+esi+E+10+ecx  
 sub     ebp, ecx            //ebp=ebp-ecx=esp+esi+E+edx+ecx-ecx=esp+esi+E+10  
 lea     ecx, \[ecx+edx\*2+72\]  
 lea     ecx, \[ecx+ebx-72\]  
 sub     ecx, ebx  
 pop     ecx  
 xor     edx, EF5EAB78  
 pop     edx  
 add     eax, \[esp+38\]  
 sbb     eax, 2B  
 pop     eax  
 sub     ebp, esi            //ebp=ebp-esi=esp+esi+E+edx-esi=esp+E+10＝esp+1E  
 lea     ebp, \[ebp-1E\]       //ebp=ebp-1E=esp+1E-1E=esp  
  
  
20.  
原代码：push    004040D0  
变形后的代码：  
  push    -6  
  pushfw  
  push    ebp  
  sub     ebp, 61388612  
  mov     ebp, esp  
  lea     ebp, \[ebp+ebx+6\]  
  sub     ebp, ebx  
  push    4040D0  
  pop     dword ptr \[ebp\]  
  pop     ebp  
  popfw  
  
  
21.  
原代码：  push    00401ED4  
变形后的代码：  
  
  push    dword ptr \[esp+14\]  
  pushfw  
  push    ecx  
  xor     ecx, ebp  
  mov     ecx, esp         //ecx=esp  
  lea     ecx, \[ecx+edi+6\] //ecx=ecx+edi+6=esp+edi+6  
  sub     ecx, edi         //ecx=ecx-edi=esp+edi+6-edi=esp+6  
  push    edi  
  sub     edi, -59  
  sub     edi, -3D  
  lea     edi, \[eax+401ED4\] //edi=eax+401ED4  
  sub     edi, eax          //edi=edi-eax=401ED4  
  mov     \[ecx\], edi  
  pop     edi  
  pop     ecx              //相当于 mov \[esp+6\],401ED4  
  popfw  
  
  
22.  
原代码：push 100  
变形后的代码：  
  
  push    dword ptr fs:\[0\]  
  pushfw  
  push    ecx  
  xor     ecx, 428F6676  
  mov     ecx, esp            
  lea     ecx, \[ecx+edi+6\]    
  sub     ecx, edi            
  push    edi  
  sub     edi, -57  
  mov     edi, 46894A  
  adc     edi, 1AA636D6  
  lea     edi, \[eax+ecx+100\] //edi=eax+ecx+100  
  sub     edi, ecx           //edi=edi-ecx=eax+ecx+100-ecx=eax+100  
  sub     edi, eax           //edi=edi-eax=eax+100-eax=100  
  push    edi  
  pop     dword ptr \[ecx\]  
  pop     edi  
  pop     ecx  
  popfw  
  
  
www.pediy.com   
2006.1.28. 

 4dd466d0e1e0cc123afbe504da2462ca