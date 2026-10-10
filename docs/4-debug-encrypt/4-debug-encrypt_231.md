

# CD MP3 Burner V2.15 - DES 算法简析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-221.htm`](https://www.pediy.com/kssd/pediy07/pediy7-221.htm)

【破文标题】：CD MP3 Burner V2.15 - DES 注算法简析  
【破文作者】：KuNgBiM\[DFCG\]  
【作者邮箱】：gb\_1227@163.com  
【软件简介】：CD MP3 Burner V2.15  
【产品地址】：[`www.mp3do.com/`](http://www.mp3do.com/)  
【保护方式】：功能限制+注册提示框+Keyfile  
【加密方式】：ASPack 2.12 -> Alexey Solodovnikov  
【编译语言】：Borland Delphi 6.0 - 7.0  
【调试环境】：WinXP、PEiD、Ollydbg  
【破解日期】：2005-05-15  
【破解目的】：研究算法分析  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
—————————————————————————————————  
【破解过程】：  
  
侦壳脱壳：用 PEiD 查壳，ASPack 2.12 -> Alexey Solodovnikov 所加的壳，OD 手动脱之后，程序为 Borland Delphi 6.0 - 7.0 编译。  
  
试探：运行主程序注册，输入 Name、Email、Key，确认！程序提示“Your serial number have not been accept.please try again!”  
  
开刀：拿出我们的法宝，OD 载入主程序，加载完毕后，搜索--->所有的参考文本字符串“Your serial number have not been accept.please try again!”，双击来到 004CD244，向上来到 004CD22D 处下断，F9 运行，输入试炼信息：  
  
\============ 试炼信息 ===========  
  
Name：KuNgBiM  
Code：9876543210  
  
\=================================  
  
确定后中断如下：  
  
004CD1FC    55              push ebp  
004CD1FD    8BEC            mov ebp,esp  
004CD1FF    6A 00           push 0  
004CD201    53              push ebx  
004CD202    8BD8            mov ebx,eax  
004CD204    33C0            xor eax,eax  
004CD206    55              push ebp  
004CD207    68 77D24C00     push Unpacked.004CD277  
004CD20C    64:FF30         push dword ptr fs:\[eax\]  
004CD20F    64:8920         mov dword ptr fs:\[eax\],esp  
004CD212    8D55 FC         lea edx,dword ptr ss:\[ebp\-4\]  
004CD215    8B83 F0020000   mov eax,dword ptr ds:\[ebx+2F0\]  
004CD21B    E8 ECB2F7FF     call Unpacked.0044850C  
004CD220    837D FC 00      cmp dword ptr ss:\[ebp\-4\],0  
004CD224    74 2A           je short Unpacked.004CD250  
004CD226    A1 F48B4F00     mov eax,dword ptr ds:\[4F8BF4\]  
004CD22B    8B00            mov eax,dword ptr ds:\[eax\]  
004CD22D    E8 BEDB0000     call Unpacked.004DADF0              // 中断在此! 算法 CALL，跟进!  
004CD232    84C0            test al,al  
004CD234    74 0E           je short Unpacked.004CD244  
004CD236    A1 F48B4F00     mov eax,dword ptr ds:\[4F8BF4\]  
004CD23B    8B00            mov eax,dword ptr ds:\[eax\]  
004CD23D    E8 CADC0000     call Unpacked.004DAF0C  
004CD242    EB 16           jmp short Unpacked.004CD25A  
004CD244    B8 8CD24C00     mov eax,Unpacked.004CD28C           // 注册码验证失败提示信息!  
004CD249    E8 4E44F7FF     call Unpacked.0044169C  
004CD24E    EB 0A           jmp short Unpacked.004CD25A  
004CD250    B8 D0D24C00     mov eax,Unpacked.004CD2D0                   ; ASCII "please input Your name!"  
004CD255    E8 4244F7FF     call Unpacked.0044169C  
004CD25A    8BC3            mov eax,ebx  
004CD25C    E8 FB7CF9FF     call Unpacked.00464F5C  
004CD261    33C0            xor eax,eax  
004CD263    5A              pop edx  
004CD264    59              pop ecx  
004CD265    59              pop ecx  
004CD266    64:8910         mov dword ptr fs:\[eax\],edx  
004CD269    68 7ED24C00     push Unpacked.004CD27E  
004CD26E    8D45 FC         lea eax,dword ptr ss:\[ebp\-4\]  
004CD271    E8 C676F3FF     call Unpacked.0040493C  
004CD276    C3              retn  
...........  
  
\==============  跟进 004CD22D    E8 BEDB0000     call Unpacked.004DADF0  ===============  
  
004DADF0    55              push ebp  
004DADF1    8BEC            mov ebp,esp  
004DADF3    33C9            xor ecx,ecx  
004DADF5    51              push ecx  
004DADF6    51              push ecx  
004DADF7    51              push ecx  
004DADF8    51              push ecx  
004DADF9    51              push ecx  
004DADFA    53              push ebx  
004DADFB    33C0            xor eax,eax  
004DADFD    55              push ebp  
004DADFE    68 CCAE4D00     push Unpacked.004DAECC  
004DAE03    64:FF30         push dword ptr fs:\[eax\]  
004DAE06    64:8920         mov dword ptr fs:\[eax\],esp  
004DAE09    8D55 FC         lea edx,dword ptr ss:\[ebp\-4\]  
004DAE0C    A1 D08A4F00     mov eax,dword ptr ds:\[4F8AD0\]  
004DAE11    8B00            mov eax,dword ptr ds:\[eax\]  
004DAE13    8B80 F0020000   mov eax,dword ptr ds:\[eax+2F0\]  
004DAE19    E8 EED6F6FF     call Unpacked.0044850C                 // 准备调用特殊字符串计算  
004DAE1E    8D4D F4         lea ecx,dword ptr ss:\[ebp\-C\]  
004DAE21    BA E4AE4D00     mov edx,Unpacked.004DAEE4              ; ASCII "burn2"  
004DAE26    8B45 FC         mov eax,dword ptr ss:\[ebp\-4\]           // 调用用户名  
004DAE29    E8 4A2FFFFF     call Unpacked.004CDD78                 // 算法 CALL，跟进!(DES 算法)  
004DAE2E    8B55 F4         mov edx,dword ptr ss:\[ebp\-C\]           // ASCII "88E019717896F614"  
004DAE31    B8 F8A24F00     mov eax,Unpacked.004FA2F8  
004DAE36    E8 559BF2FF     call Unpacked.00404990                 // 准备调用固定字符 A 计算  
004DAE3B    68 F4AE4D00     push Unpacked.004DAEF4                 ; ASCII "cmb21-"  
004DAE40    A1 F8A24F00     mov eax,dword ptr ds:\[4FA2F8\]  
004DAE45    E8 1EFFFFFF     call Unpacked.004DAD68                 // 调用 ASCII "88E019717896F614"运算！(DES 算法 sBox1),跟进!  
004DAE4A    8D55 F0         lea edx,dword ptr ss:\[ebp\-10\]  
004DAE4D    E8 C2E5F2FF     call Unpacked.00409414                 // 准备调用固定字符 B 计算，跟进！  
004DAE52    FF75 F0         push dword ptr ss:\[ebp\-10\]             // ASCII "93190"   注册码中段  
004DAE55    68 04AF4D00     push Unpacked.004DAF04                 ; ASCII "-2004"  
004DAE5A    8D45 F8         lea eax,dword ptr ss:\[ebp\-8\]  
004DAE5D    BA 03000000     mov edx,3  
004DAE62    E8 5D9EF2FF     call Unpacked.00404CC4  
004DAE67    8D55 EC         lea edx,dword ptr ss:\[ebp\-14\]          // 连接“固定字符 A+注册码中段+固定字符 B”  
004DAE6A    A1 D08A4F00     mov eax,dword ptr ds:\[4F8AD0\]  
004DAE6F    8B00            mov eax,dword ptr ds:\[eax\]  
004DAE71    8B80 F4020000   mov eax,dword ptr ds:\[eax+2F4\]  
004DAE77    E8 90D6F6FF     call Unpacked.0044850C  
004DAE7C    8B45 EC         mov eax,dword ptr ss:\[ebp\-14\]          // 试炼码"9876543210"  
004DAE7F    8B55 F8         mov edx,dword ptr ss:\[ebp\-8\]           // 注册码"cmb21-93190-2004"  
004DAE82    E8 C19EF2FF     call Unpacked.00404D48                 // 经典比对 CALL，跟进! 内存注册机  
004DAE87    75 1E           jnz short Unpacked.004DAEA7            // 爆破点  
004DAE89    B3 01           mov bl,1  
004DAE8B    B8 F0A24F00     mov eax,Unpacked.004FA2F0  
004DAE90    8B55 FC         mov edx,dword ptr ss:\[ebp\-4\]  
004DAE93    E8 F89AF2FF     call Unpacked.00404990  
004DAE98    B8 F4A24F00     mov eax,Unpacked.004FA2F4  
004DAE9D    8B55 F8         mov edx,dword ptr ss:\[ebp\-8\]  
004DAEA0    E8 EB9AF2FF     call Unpacked.00404990  
004DAEA5    EB 02           jmp short Unpacked.004DAEA9  
004DAEA7    33DB            xor ebx,ebx  
004DAEA9    33C0            xor eax,eax  
004DAEAB    5A              pop edx  
004DAEAC    59              pop ecx  
004DAEAD    59              pop ecx  
004DAEAE    64:8910         mov dword ptr fs:\[eax\],edx  
004DAEB1    68 D3AE4D00     push Unpacked.004DAED3  
004DAEB6    8D45 EC         lea eax,dword ptr ss:\[ebp\-14\]  
004DAEB9    E8 7E9AF2FF     call Unpacked.0040493C  
004DAEBE    8D45 F0         lea eax,dword ptr ss:\[ebp\-10\]  
004DAEC1    BA 04000000     mov edx,4  
004DAEC6    E8 959AF2FF     call Unpacked.00404960  
004DAECB    C3              retn  
  
\=============== 跟进 004DAE29    E8 4A2FFFFF     call Unpacked.004CDD78 \[DES 算法 sBox1 加密\]\==============  
  
004CDD78    55              push ebp  
004CDD79    8BEC            mov ebp,esp  
004CDD7B    83C4 E4         add esp,-1C  
004CDD7E    53              push ebx  
004CDD7F    56              push esi  
004CDD80    57              push edi  
004CDD81    33DB            xor ebx,ebx  
004CDD83    895D F4         mov dword ptr ss:\[ebp\-C\],ebx  
004CDD86    895D F0         mov dword ptr ss:\[ebp\-10\],ebx  
004CDD89    895D EC         mov dword ptr ss:\[ebp\-14\],ebx  
004CDD8C    8BF9            mov edi,ecx  
004CDD8E    8955 F8         mov dword ptr ss:\[ebp\-8\],edx  
004CDD91    8945 FC         mov dword ptr ss:\[ebp\-4\],eax  
004CDD94    8B45 FC         mov eax,dword ptr ss:\[ebp\-4\]  
004CDD97    E8 5070F3FF     call Unpacked.00404DEC  
004CDD9C    8B45 F8         mov eax,dword ptr ss:\[ebp\-8\]  
004CDD9F    E8 4870F3FF     call Unpacked.00404DEC  
004CDDA4    33C0            xor eax,eax  
004CDDA6    55              push ebp  
004CDDA7    68 4ADE4C00     push Unpacked.004CDE4A  
004CDDAC    64:FF30         push dword ptr fs:\[eax\]  
004CDDAF    64:8920         mov dword ptr fs:\[eax\],esp  
004CDDB2    8D4D F0         lea ecx,dword ptr ss:\[ebp\-10\]  
004CDDB5    8B55 F8         mov edx,dword ptr ss:\[ebp\-8\]  
004CDDB8    8B45 FC         mov eax,dword ptr ss:\[ebp\-4\]  
004CDDBB    E8 D4FDFFFF     call Unpacked.004CDB94   
004CDDC0    8D45 F4         lea eax,dword ptr ss:\[ebp\-C\]  
004CDDC3    E8 746BF3FF     call Unpacked.0040493C  
004CDDC8    8B45 F0         mov eax,dword ptr ss:\[ebp\-10\]  
004CDDCB    E8 346EF3FF     call Unpacked.00404C04  
004CDDD0    8BD8            mov ebx,eax  
004CDDD2    4B              dec ebx  
004CDDD3    85DB            test ebx,ebx  
004CDDD5    7C 4E           jl short Unpacked.004CDE25  
004CDDD7    43              inc ebx  
004CDDD8    33F6            xor esi,esi  
004CDDDA    8D45 EC         lea eax,dword ptr ss:\[ebp\-14\]  
004CDDDD    50              push eax  
004CDDDE    8B45 F0         mov eax,dword ptr ss:\[ebp\-10\]  
004CDDE1    0FB60430        movzx eax,byte ptr ds:\[eax+esi\]  
004CDDE5    8945 E4         mov dword ptr ss:\[ebp\-1C\],eax  
004CDDE8    C645 E8 00      mov byte ptr ss:\[ebp\-18\],0  
004CDDEC    8D55 E4         lea edx,dword ptr ss:\[ebp\-1C\]  
004CDDEF    33C9            xor ecx,ecx  
004CDDF1    B8 60DE4C00     mov eax,Unpacked.004CDE60              ; ASCII "%x"  
004CDDF6    E8 05C4F3FF     call Unpacked.0040A200  
004CDDFB    8B45 EC         mov eax,dword ptr ss:\[ebp\-14\]  
004CDDFE    E8 016EF3FF     call Unpacked.00404C04  
004CDE03    48              dec eax  
004CDE04    75 10           jnz short Unpacked.004CDE16  
004CDE06    8D45 EC         lea eax,dword ptr ss:\[ebp\-14\]  
004CDE09    8B4D EC         mov ecx,dword ptr ss:\[ebp\-14\]  
004CDE0C    BA 6CDE4C00     mov edx,Unpacked.004CDE6C  
004CDE11    E8 3A6EF3FF     call Unpacked.00404C50  
004CDE16    8D45 F4         lea eax,dword ptr ss:\[ebp\-C\]  
004CDE19    8B55 EC         mov edx,dword ptr ss:\[ebp\-14\]  
004CDE1C    E8 EB6DF3FF     call Unpacked.00404C0C  
004CDE21    46              inc esi  
004CDE22    4B              dec ebx  
004CDE23  ^ 75 B5           jnz short Unpacked.004CDDDA            // 向上作循环运算 16 次，得到加密后的代码"88E019717896F614"  
004CDE25    8BC7            mov eax,edi  
004CDE27    8B55 F4         mov edx,dword ptr ss:\[ebp\-C\]  
004CDE2A    E8 616BF3FF     call Unpacked.00404990  
004CDE2F    33C0            xor eax,eax  
004CDE31    5A              pop edx  
004CDE32    59              pop ecx  
004CDE33    59              pop ecx  
004CDE34    64:8910         mov dword ptr fs:\[eax\],edx  
004CDE37    68 51DE4C00     push Unpacked.004CDE51  
004CDE3C    8D45 EC         lea eax,dword ptr ss:\[ebp\-14\]  
004CDE3F    BA 05000000     mov edx,5  
004CDE44    E8 176BF3FF     call Unpacked.00404960  
004CDE49    C3              retn                                  // 运算完毕后返回  
...........  
  
\============== 跟进 004DAE45    E8 1EFFFFFF     call Unpacked.004DAD68 \[\[DES 算法 sBox1\]预处理密钥，准备解密\] ==============  
  
004DAD68    55              push ebp  
004DAD69    8BEC            mov ebp,esp  
004DAD6B    51              push ecx  
004DAD6C    53              push ebx  
004DAD6D    8945 FC         mov dword ptr ss:\[ebp\-4\],eax  
004DAD70    8B45 FC         mov eax,dword ptr ss:\[ebp\-4\]  
004DAD73    E8 74A0F2FF     call Unpacked.00404DEC  
004DAD78    33C0            xor eax,eax  
004DAD7A    55              push ebp  
004DAD7B    68 E2AD4D00     push Unpacked.004DADE2  
004DAD80    64:FF30         push dword ptr fs:\[eax\]  
004DAD83    64:8920         mov dword ptr fs:\[eax\],esp  
004DAD86    8B45 FC         mov eax,dword ptr ss:\[ebp\-4\]  
004DAD89    E8 769EF2FF     call Unpacked.00404C04  
004DAD8E    33C9            xor ecx,ecx  
004DAD90    8BD0            mov edx,eax  
004DAD92    85D2            test edx,edx  
004DAD94    76 29           jbe short Unpacked.004DADBF  
004DAD96    B8 01000000     mov eax,1  
004DAD9B    8D0C89          lea ecx,dword ptr ds:\[ecx+ecx\*4\]  
004DAD9E    8D0C89          lea ecx,dword ptr ds:\[ecx+ecx\*4\]  
004DADA1    8B5D FC         mov ebx,dword ptr ss:\[ebp\-4\]  
004DADA4    0FB65C03 FF     movzx ebx,byte ptr ds:\[ebx+eax\-1\]  
004DADA9    03CB            add ecx,ebx  
004DADAB    8B5D FC         mov ebx,dword ptr ss:\[ebp\-4\]  
004DADAE    0FB65C03 FF     movzx ebx,byte ptr ds:\[ebx+eax\-1\]  
004DADB3    6BDB 0D         imul ebx,ebx,0D  
004DADB6    C1E3 14         shl ebx,14  
004DADB9    33CB            xor ecx,ebx  
004DADBB    40              inc eax  
004DADBC    4A              dec edx  
004DADBD  ^ 75 DC           jnz short Unpacked.004DAD9B            // 向上作循环运算 16 次，得到解密后的代码  
004DADBF    8BC1            mov eax,ecx  
004DADC1    B9 A0860100     mov ecx,186A0  
004DADC6    33D2            xor edx,edx  
004DADC8    F7F1            div ecx  
004DADCA    8BDA            mov ebx,edx  
004DADCC    33C0            xor eax,eax  
004DADCE    5A              pop edx  
004DADCF    59              pop ecx  
004DADD0    59              pop ecx  
004DADD1    64:8910         mov dword ptr fs:\[eax\],edx  
004DADD4    68 E9AD4D00     push Unpacked.004DADE9  
004DADD9    8D45 FC         lea eax,dword ptr ss:\[ebp\-4\]  
004DADDC    E8 5B9BF2FF     call Unpacked.0040493C  
004DADE1    C3              retn  
...........  
  
\============== 跟进 004DAE4D    E8 C2E5F2FF     call Unpacked.00409414 \[\[DES 算法 sBox1\]开始解密\] ==============  
  
00409414    83C4 F8         add esp,-8                            // ASCII "R 甅"  
00409417    6A 00           push 0  
00409419    894424 04       mov dword ptr ss:\[esp+4\],eax  
0040941D    C64424 08 00    mov byte ptr ss:\[esp+8\],0             // 堆栈 ss:\[0012FC1C\]=4C ('L')  
00409422    8D4C24 04       lea ecx,dword ptr ss:\[esp+4\]          // ecx\=0  
00409426    8BC2            mov eax,edx  
00409428    BA 40944000     mov edx,Unpacked.00409440             ; ASCII "%d"  
0040942D    E8 E20D0000     call Unpacked.0040A214  
00409432    59              pop ecx  
00409433    5A              pop edx  
00409434    C3              retn                                  // 解密完毕，返回程序  
...........  
  
\============== 跟进 004DAE82    E8 C19EF2FF     call Unpacked.00404D48 \[比对 CALL\] ==============  
  
00404D48    53              push ebx  
00404D49    56              push esi  
00404D4A    57              push edi  
00404D4B    89C6            mov esi,eax                          // 试炼码入栈"9876543210"  
00404D4D    89D7            mov edi,edx                          // 注册码入栈"cmb21-93190-2004"  
00404D4F    39D0            cmp eax,edx                          // 经典对比，明码比较！  
00404D51    0F84 8F000000   je Unpacked.00404DE6  
00404D57    85F6            test esi,esi  
00404D59    74 68           je short Unpacked.00404DC3  
00404D5B    85FF            test edi,edi  
...........  
  
【算法总结】  
  
1、格式：注册码由 3 段构成，其中 2 段为固定字符串，格式为：  
  
        “固定字符 A”+“注册码中段”+“固定字符 B”  即：“cmb21-”+注册码中段+“-2004”  
  
2、加密运算：分别取用户名以及特殊字符串“burn2”的位数自加一，得到正确密钥后，做对应的 DES-sBox1 加密运算，得到新的密钥。  
  
3、解密运算：把新密钥再次通过 DES-sBox1 作解密运算，得到十进制代码，作为注册码的中段。  
  
4、注册码组合：以固定格式输出：“cmb21-”+注册码中段+“-2004”  
  
  
\=======================  
内存注册机：  
  
中断地址：004DAE82  
中断次数：1  
第一字节：E8  
指令长度：5  
  
内存方式--->EDX  
  
\=======================  
  
注册信息:  
  
Name：KuNgBiM  
Code：cmb21-93190-2004  
  
注册信息保存在安装目录下 burn.cfg 文件中（删除该文件后可重新注册）  
  
\======== burn.cfg 文件内容 =========  
  
\[reg\]  
Name=KuNgBiM  
Pass=cmb21-93190-2004  
check=88E019717896F614  
  
\====================================  
  
\--------------------------------------------------------------------------  
  
（本文完）  
  
版权所有（C）2005 KuNgBiM\[DFCG\]         Copyright (C) 2005 KuNgBiM\[DFCG\]  
  
  
\--------------------------------------------------------------------------  
          Cracked BY KuNgBiM\[DFCG\]  
  
                2005-05-15  
  
               03:24:00 AM 

 4dd466d0e1e0cc123afbe504da2462ca