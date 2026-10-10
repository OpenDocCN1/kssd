

# Speed Video Converter 3.0.16.8

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-198.htm`](https://www.pediy.com/kssd/pediy08/pediy8-198.htm)

【破解软件】Speed Video Converter 3.0.16.8   
【下载地址】http://www.onlinedown.net/soft/46810.htm  
【运行环境】Win9x/Me/NT/2000/XP/2003  
【软件类别】国外软件/共享版/视频工具  
【保护方式】用户名、注册码   
【作者声明】初学 Crack，只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教。  
【调试环境】Winxp、OllyDBD、PEiD  
【软件信息】是一款小型视频转换工具。快速和易用是其卖点。支持各种视频格式，例如 AVI(Divx,xDiv), MPEG-4, mpeg               
  
           (vcd,svcd,dvd 兼容), wmv, asf, QuickTime, VOB, DAT。它支持批量文件转换可以一次点击转换多个文件。  
【破解过程】太菜开始不知道是 RSA 算法,跟踪分析了半天，一个 Call 接着一个 Call,转得晕乎乎的,KANAL 分析什么也没发现。在验证注  
  
册码与用户名关键处又比较简单，改一下跳转可以爆破，多试几遍也能找到一组可用的注册码与用户名，当然用户名的样子肯定是怪怪  
  
的。因此怀疑是一种现成的加密算法。对照《加密与解密》第六章，找书上介绍的各种加密算法特征及计算方法，最终证实是 RSA。  
　　　  
一、算法跟踪  
  
 PEiD 分析：Microsoft Visual C++ 6.0　  
 OD 载入程序查找字串参考，找到：“invalid username or registration code”双击来到:004032C3 处，向上在 004031C0 处下断，F9  
  
运行程序。在注册框里填用户名：wzwgp　注册码：12345678-22345678-32345678-42345678-52345678-62345678-72345678-82345678　  
  
点“OK”  
   
004031C0   .  6A FF         PUSH -1                                  ;  断下  
004031C2   .  68 E83C4100   PUSH Speed\_Vi.00413CE8                   ;  SE 处理程序安装  
004031C7   .  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]  
  
       ---------------------中间省略--------------------------  
  
004031FD   .  E8 A2FD0000   CALL <JMP.&MFC42.#1669>  
00403202   .  8B46 60       MOV EAX,DWORD PTR DS:\[ESI+60\]            ;  假码入 EAX  
00403205   .  8B4E 64       MOV ECX,DWORD PTR DS:\[ESI+64\]            ;  用户名入 ECX  
00403208   .  50            PUSH EAX  
00403209   .  51            PUSH ECX  
0040320A   .  C64424 1C 01  MOV BYTE PTR SS:\[ESP+1C\],1  
0040320F   .  E8 CCFBFFFF   CALL Speed\_Vi.00402DE0                   ;  判断注册是否成功 F7 进入   
00403214   .  83C4 08       ADD ESP,8  
00403217   .  85C0          TEST EAX,EAX                             ;  成功返回 1，失败返回 0  
00403219   .  0F95C0        SETNE AL  
0040321C   .  84C0          TEST AL,AL  
0040321E   .  A2 2CCF4100   MOV BYTE PTR DS:\[41CF2C\],AL  
00403223      0F84 93000000 JE Speed\_Vi.004032BC                     ;  注册失败跳  
00403229   .  8B46 64       MOV EAX,DWORD PTR DS:\[ESI+64\]  
0040322C   .  8D4C24 04     LEA ECX,DWORD PTR SS:\[ESP+4\]  
00403230   .  50            PUSH EAX  
00403231   .  68 F0C14100   PUSH Speed\_Vi.0041C1F0                   ;  license to:%s  
00403236   .  51            PUSH ECX  
00403237   .  E8 62FD0000   CALL <JMP.&MFC42.#2818>  
0040323C   .  8B5424 10     MOV EDX,DWORD PTR SS:\[ESP+10\]  
00403240   .  83C4 0C       ADD ESP,0C  
00403243   .  8BCE          MOV ECX,ESI  
00403245   .  6A 40         PUSH 40  
00403247   .  68 E4C14100   PUSH Speed\_Vi.0041C1E4                   ;  thank you  
0040324C   .  52            PUSH EDX  
0040324D   .  E8 46FD0000   CALL <JMP.&MFC42.#4224>  
  
      ---------------------中间省略--------------------------  
  
004032B5   .  E8 D2FC0000   CALL <JMP.&MFC42.#6199>  
004032BA   .  EB 13         JMP SHORT Speed\_Vi.004032CF  
004032BC   >  6A 40         PUSH 40  
004032BE   .  68 C4C14100   PUSH Speed\_Vi.0041C1C4                   ;  sorry  
004032C3   .  68 98C14100   PUSH Speed\_Vi.0041C198                   ;  invalid username or registration code  
  
  
　F7 进入 0040320F 处,判断注册是否成功　  
  
00402DE0  /$  6A FF         PUSH -1  
00402DE2  |.  68 993C4100   PUSH Speed\_Vi.00413C99                   ;  SE 处理程序安装  
00402DE7  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]  
00402DED  |.  50            PUSH EAX  
00402DEE  |.  64:8925 00000>MOV DWORD PTR FS:\[0\],ESP  
00402DF5  |.  81EC 94000000 SUB ESP,94  
00402DFB  |.  8B8424 A40000>MOV EAX,DWORD PTR SS:\[ESP+A4\]            ;  用户名地址入 EAX  
00402E02  |.  53            PUSH EBX  
00402E03  |.  56            PUSH ESI  
00402E04  |.  50            PUSH EAX  
00402E05  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]  
00402E09  |.  C74424 60 B91>MOV DWORD PTR SS:\[ESP+60\],66C11BB9        |  
00402E11  |.  C74424 64 130>MOV DWORD PTR SS:\[ESP+64\],44A30D13        |  
00402E19  |.  C74424 68 6CB>MOV DWORD PTR SS:\[ESP+68\],D424BB6C        |  
00402E21  |.  C74424 6C 7B1>MOV DWORD PTR SS:\[ESP+6C\],9B43197B        |  这组数是 N  
00402E29  |.  C74424 70 CAF>MOV DWORD PTR SS:\[ESP+70\],3254F2CA        |  
00402E31  |.  C74424 74 45E>MOV DWORD PTR SS:\[ESP+74\],CEE8EC45        |  
00402E39  |.  C74424 78 572>MOV DWORD PTR SS:\[ESP+78\],EAF92557        |  
00402E41  |.  C74424 7C F2D>MOV DWORD PTR SS:\[ESP+7C\],5D79D4F2        |  
00402E49  |.  E8 5A000100   CALL <JMP.&MFC42.#537>  
00402E4E  |.  8B8C24 B00000>MOV ECX,DWORD PTR SS:\[ESP+B0\]             ;  假码地址入 ECX  
00402E55  |.  C78424 A40000>MOV DWORD PTR SS:\[ESP+A4\],0  
00402E60  |.  51            PUSH ECX  
00402E61  |.  8D4C24 0C     LEA ECX,DWORD PTR SS:\[ESP+C\]  
00402E65  |.  E8 3E000100   CALL <JMP.&MFC42.#537>  
00402E6A  |.  8B5424 0C     MOV EDX,DWORD PTR SS:\[ESP+C\]              ;  用户名地址入 EDX  
00402E6E  |.  8B35 04664100 MOV ESI,DWORD PTR DS:\[<&MSVCRT.\_mbs>  
00402E74  |.  68 60CE4100   PUSH Speed\_Vi.0041CE60                
00402E79  |.  52            PUSH EDX                              
00402E7A  |.  C68424 AC0000>MOV BYTE PTR SS:\[ESP+AC\],1            
00402E82  |.  FFD6          CALL NEAR ESI                             ;  检查是否输入用户名  
00402E84  |.  83C4 08       ADD ESP,8  
00402E87  |.  85C0          TEST EAX,EAX  
00402E89  |.  0F84 0F020000 JE Speed\_Vi.0040309E  
00402E8F  |.  8B4424 08     MOV EAX,DWORD PTR SS:\[ESP+8\]              ;  假码地址入 EAX  
00402E93  |.  68 60CE4100   PUSH Speed\_Vi.0041CE60  
00402E98  |.  50            PUSH EAX  
00402E99  |.  FFD6          CALL NEAR ESI                             ;  检查是否输入注册码  
00402E9B  |.  83C4 08       ADD ESP,8  
00402E9E  |.  85C0          TEST EAX,EAX  
00402EA0  |.  0F84 F8010000 JE Speed\_Vi.0040309E  
00402EA6  |.  57            PUSH EDI  
00402EA7  |.  6A 00         PUSH 0  
00402EA9  |.  8D4C24 44     LEA ECX,DWORD PTR SS:\[ESP+44\]  
00402EAD  |.  E8 6E730000   CALL Speed\_Vi.0040A220  
00402EB2  |.  6A 00         PUSH 0  
00402EB4  |.  8D4C24 4C     LEA ECX,DWORD PTR SS:\[ESP+4C\]  
00402EB8  |.  C68424 AC0000>MOV BYTE PTR SS:\[ESP+AC\],2  
00402EC0  |.  E8 5B730000   CALL Speed\_Vi.0040A220  
00402EC5  |.  B3 03         MOV BL,3  
00402EC7  |.  68 01000100   PUSH 10001                                 ; 加密密钥 E(10001)  
00402ECC  |.  8D4C24 5C     LEA ECX,DWORD PTR SS:\[ESP+5C\]  
00402ED0  |.  889C24 AC0000>MOV BYTE PTR SS:\[ESP+AC\],BL  
00402ED7  |.  E8 44730000   CALL Speed\_Vi.0040A220  
00402EDC  |.  8D4C24 58     LEA ECX,DWORD PTR SS:\[ESP+58\]  
00402EE0  |.  C68424 A80000>MOV BYTE PTR SS:\[ESP+A8\],4  
00402EE8  |.  51            PUSH ECX  
00402EE9  |.  8D4C24 4C     LEA ECX,DWORD PTR SS:\[ESP+4C\]  
00402EED  |.  E8 8E730000   CALL Speed\_Vi.0040A280  
00402EF2  |.  8D4C24 58     LEA ECX,DWORD PTR SS:\[ESP+58\]  
00402EF6  |.  889C24 A80000>MOV BYTE PTR SS:\[ESP+A8\],BL  
00402EFD  |.  E8 CE730000   CALL Speed\_Vi.0040A2D0  
00402F02  |.  8D5424 60     LEA EDX,DWORD PTR SS:\[ESP+60\]  
00402F06  |.  6A 08         PUSH 8  
00402F08  |.  52            PUSH EDX  
00402F09  |.  8D4C24 48     LEA ECX,DWORD PTR SS:\[ESP+48\]  
00402F0D  |.  E8 DE710000   CALL Speed\_Vi.0040A0F0  
00402F12  |.  B9 08000000   MOV ECX,8  
00402F17  |.  33C0          XOR EAX,EAX  
00402F19  |.  8D7C24 18     LEA EDI,DWORD PTR SS:\[ESP+18\]  
00402F1D  |.  8D5424 2C     LEA EDX,DWORD PTR SS:\[ESP+2C\]  
00402F21  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
00402F23  |.  8D4424 34     LEA EAX,DWORD PTR SS:\[ESP+34\]  
00402F27  |.  8D4C24 30     LEA ECX,DWORD PTR SS:\[ESP+30\]  
00402F2B  |.  50            PUSH EAX  
00402F2C  |.  51            PUSH ECX  
00402F2D  |.  8D4424 30     LEA EAX,DWORD PTR SS:\[ESP+30\]  
00402F31  |.  52            PUSH EDX  
00402F32  |.  8D4C24 30     LEA ECX,DWORD PTR SS:\[ESP+30\]  
00402F36  |.  50            PUSH EAX  
00402F37  |.  8D5424 30     LEA EDX,DWORD PTR SS:\[ESP+30\]  
00402F3B  |.  51            PUSH ECX  
00402F3C  |.  8D4424 30     LEA EAX,DWORD PTR SS:\[ESP+30\]  
00402F40  |.  52            PUSH EDX  
00402F41  |.  8B5424 24     MOV EDX,DWORD PTR SS:\[ESP+24\]       ;  假码地址入 EDX  
00402F45  |.  8D4C24 30     LEA ECX,DWORD PTR SS:\[ESP+30\]  
00402F49  |.  50            PUSH EAX  
00402F4A  |.  51            PUSH ECX  
00402F4B  |.  68 64C14100   PUSH Speed\_Vi.0041C164              ; %08lx-%08lx-%08lx-%08lx-%08lx-%08lx-%08lx-%08lx\\n  
00402F50  |.  52            PUSH EDX                              这是注册码的格式  
00402F51  |.  FF15 00664100 CALL NEAR DWORD PTR DS:\[<&MSVCRT.ss>  
00402F57  |.  8B4424 50     MOV EAX,DWORD PTR SS:\[ESP+50\]       ;  EAX=52345678  
00402F5B  |.  8B4C24 4C     MOV ECX,DWORD PTR SS:\[ESP+4C\]       ;  ECX=42345678  
00402F5F  |.  8B7C24 48     MOV EDI,DWORD PTR SS:\[ESP+48\]       ;  EDI=32345678  
00402F63  |.  8B5424 44     MOV EDX,DWORD PTR SS:\[ESP+44\]       ;  EDX=22345678  
00402F67  |.  03C1          ADD EAX,ECX                         ;  EAX=52345678+42345678=9468ACF0   
00402F69  |.  8B4C24 5C     MOV ECX,DWORD PTR SS:\[ESP+5C\]       ;  ECX=82345678  
00402F6D  |.  03C7          ADD EAX,EDI                         ;  EAX=9468ACF0+32345678=C69D0368   
00402F6F  |.  8B7C24 58     MOV EDI,DWORD PTR SS:\[ESP+58\]       ;  EDI=72345678  
00402F73  |.  03C2          ADD EAX,EDX                         ;  EAX=C69D0368+22345678=E8D159E0   
00402F75  |.  8B5424 40     MOV EDX,DWORD PTR SS:\[ESP+40\]       ;  EDX=12345678  
00402F79  |.  33C8          XOR ECX,EAX                         ;  ECX=82345678 xor E8D159E0=6AE50F98   
00402F7B  |.  8B4424 54     MOV EAX,DWORD PTR SS:\[ESP+54\]       ;  EAX=62345678  
00402F7F  |.  83C4 28       ADD ESP,28  
00402F82  |.  03C2          ADD EAX,EDX                         ;  EAX=62345678+12345678=7468ACF0   
00402F84  |.  894C24 34     MOV DWORD PTR SS:\[ESP+34\],ECX       ;  6AE50F98 替换 82345678  
00402F88  |.  33F8          XOR EDI,EAX                         ;  EDI=72345678 xor 7468ACF0=065CFA88   
00402F8A  |.  6A 00         PUSH 0  
00402F8C  |.  8D4C24 3C     LEA ECX,DWORD PTR SS:\[ESP+3C\]  
00402F90  |.  897C24 34     MOV DWORD PTR SS:\[ESP+34\],EDI       ;  065CFA88 替换 72345678  
00402F94  |.  E8 87720000   CALL Speed\_Vi.0040A220  
00402F99  |.  8D4C24 18     LEA ECX,DWORD PTR SS:\[ESP+18\]  
00402F9D  |.  6A 08         PUSH 8  
00402F9F  |.  51            PUSH ECX  
00402FA0  |.  8D4C24 40     LEA ECX,DWORD PTR SS:\[ESP+40\]  
00402FA4  |.  C68424 B00000>MOV BYTE PTR SS:\[ESP+B0\],5  
00402FAC  |.  E8 3F710000   CALL Speed\_Vi.0040A0F0  
00402FB1  |.  8D5424 38     LEA EDX,DWORD PTR SS:\[ESP+38\]  
00402FB5  |.  8D4424 50     LEA EAX,DWORD PTR SS:\[ESP+50\]  
00402FB9  |.  52            PUSH EDX  
00402FBA  |.  50            PUSH EAX  
00402FBB  |.  8D4C24 48     LEA ECX,DWORD PTR SS:\[ESP+48\]  
00402FBF  |.  E8 CC190000   CALL Speed\_Vi.00404990              　;  RSA 运算  
00402FC4  |.  B9 08000000   MOV ECX,8  
00402FC9  |.  33C0          XOR EAX,EAX  
00402FCB  |.  8D7C24 18     LEA EDI,DWORD PTR SS:\[ESP+18\]  
00402FCF  |.  6A 08         PUSH 8  
00402FD1  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
00402FD3  |.  8D4C24 1C     LEA ECX,DWORD PTR SS:\[ESP+1C\]  
00402FD7  |.  C68424 AC0000>MOV BYTE PTR SS:\[ESP+AC\],6  
00402FDF  |.  51            PUSH ECX  
00402FE0  |.  8D4C24 58     LEA ECX,DWORD PTR SS:\[ESP+58\]  
00402FE4  |.  E8 47710000   CALL Speed\_Vi.0040A130              　;  输出 RSA 运算结果 \[A44E60--70\]  
00402FE9  |.  B9 08000000   MOV ECX,8  
00402FEE  |.  33C0          XOR EAX,EAX  
00402FF0  |.  8DBC24 800000>LEA EDI,DWORD PTR SS:\[ESP+80\]  
00402FF7  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]           ;  堆栈空出空间  
00402FF9  |.  5F            POP EDI  
00402FFA  |>  8A5404 17     /MOV DL,BYTE PTR SS:\[ESP+EAX+17\]  
00402FFE  |.  8A4C04 16     |MOV CL,BYTE PTR SS:\[ESP+EAX+16\]  
00403002  |.  885404 7C     |MOV BYTE PTR SS:\[ESP+EAX+7C\],DL  
00403006  |.  8B5404 14     |MOV EDX,DWORD PTR SS:\[ESP+EAX+14\]  
0040300A  |.  884C04 7D     |MOV BYTE PTR SS:\[ESP+EAX+7D\],CL  
0040300E  |.  8A4C04 14     |MOV CL,BYTE PTR SS:\[ESP+EAX+14\]  
00403012  |.  C1EA 08       |SHR EDX,8  
00403015  |.  885404 7E     |MOV BYTE PTR SS:\[ESP+EAX+7E\],DL  
00403019  |.  884C04 7F     |MOV BYTE PTR SS:\[ESP+EAX+7F\],CL  
0040301D  |.  83C0 04       |ADD EAX,4  
00403020  |.  83F8 20       |CMP EAX,20  
00403023  |.^ 7C D5         \\JL SHORT Speed\_Vi.00402FFA           ; 循环重排序 RSA 运算结果  
00403025  |.  8D5424 7C     LEA EDX,DWORD PTR SS:\[ESP+7C\]  
00403029  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]  
0040302D  |.  52            PUSH EDX  
0040302E  |.  E8 75FE0000   CALL <JMP.&MFC42.#537>  
00403033  |.  8B4424 10     MOV EAX,DWORD PTR SS:\[ESP+10\]  
00403037  |.  8B4C24 0C     MOV ECX,DWORD PTR SS:\[ESP+C\]  
0040303B  |.  50            PUSH EAX  
0040303C  |.  51            PUSH ECX  
0040303D  |.  FFD6          CALL NEAR ESI                         ;  用户名 16 进制数与计算结果比较  
0040303F  |.  83C4 08       ADD ESP,8  
00403042  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]  
00403046  |.  85C0          TEST EAX,EAX                          ;  EAX=0 成功　EAX=1 失败  
00403048  |.  C68424 A40000>MOV BYTE PTR SS:\[ESP+A4\],6  
00403050      0F84 86000000 JE Speed\_Vi.004030DC                  ;  暴破点  
00403056  |.  E8 15FD0000   CALL <JMP.&MFC42.#800>  
  
  
二、算法小结  
  
1.注册码分为 8 组，s1、s2、s3、s4、s5、s6、s7、s8  
  验证前预处理：s7=(s1+s6) xor s7    s8=(s2+s3+s4+s5) xor s8  
  
2.RSA256 运算     
  n=5D79D4F2EAF92557CEE8EC453254F2CA9B43197BD424BB6C44A30D1366C11BB9  
  e=10001  
        
3. 重排序运算结果  
  B1BE436A　　　　　　　　6A43BEB1  
  F29961A1　 ------>　　　A16199F2               
  6A85B49E　　　　　　　　9EB4856A   
           
4. 重排序的运算结果和用户名 16 进制数比较，相等则注册成功，不等则失败。       
            
三、算法验证  
  
用 RSATool 工具，根据 n、e，求出 p、q、d   
　p=A4A4B5845A655DA9EF76DED6C373A31B  
  q=9157E97B62A6CFDD8AAA6FC9557355BB  
  d=2FD52823261A580196DF9A07CEB9A983654FAB5DD473BC2857780CFCF3D9AC01   
  
用户名 m=wzwgp -> wzwg-p -> p000wzwg (0x70000000777A7767)  
设：X=70000000777A7767   <------ m  
    Y=2FD52823261A580196DF9A07CEB9A983654FAB5DD473BC2857780CFCF3D9AC01 <------ d  
    Z=5D79D4F2EAF92557CEE8EC453254F2CA9B43197BD424BB6C44A30D1366C11BB9 <------ n  
用 Bigclc“X^Y%Z”计算出 C  
    c=1D033EF29AE89BBB1DD3C955D95D4215FEDDD89B3B41131F94F743B93A40440E     
   
变为：1D033EF2-9AE89BBB-1DD3C955-D95D4215-FEDDD89B-3B41131F-94F743B9-3A40440E  
　　　3A40440E-94F743B9-3B41131F-FEDDD89B-D95D4215-1DD3C955-9AE89BBB-1D033EF2  
计算：s7=58140D63 xor 9AE89BBB=C2FC96D8  
　　　s8=A8737188 xor 1D033EF2=B5704F7A  
　　　3A40440E-94F743B9-3B41131F-FEDDD89B-D95D4215-1DD3C955-C2FC96D8-B5704F7A  
  
  
用户名：wzwgp  
注册码：3A40440E-94F743B9-3B41131F-FEDDD89B-D95D4215-1DD3C955-C2FC96D8-B5704F7A  
  
注册信息保存在 Settings.ini 文件里。 

 4dd466d0e1e0cc123afbe504da2462ca