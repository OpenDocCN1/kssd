

# Iparmor 木马克星 V5.40 Build 0414

> 原文：[`www.pediy.com/kssd/pediy05/pediy50644.htm`](https://www.pediy.com/kssd/pediy05/pediy50644.htm)

Iparmor 木马克星 V5.40 Build 0414 破解手记-算法分析  
作者：newlaos\[CCG\]\[DFCG\]

软件名称：木马克星 5.40 Build 0414  
下载地址：http://lpcsoft.yeah.net  
大    小：341Kb  
软件说明：本软件是动态监视网络与静态特征字扫描的完美结合，可以查杀 3759 种国际木马，保证查杀：冰河所有版本，黑洞 2001 所有版本等等国产木马，挑战安全极限，杀尽天下木马！木马克星采用监视硬盘技术，不占用 cpu 负荷，本软件操作更智能，占用系统资源更少，查杀木马更多!可以在线升级木马库，每天升级！ 

加密方式：注册码  
功能限制：未注册信息提示+功能次数限制  
PJ 工具：TRW20001.23 注册版，W32Dasm8.93 黄金版，FI2.5，eXeScope6.30  
PJ 日期：2003-04-23  
作者 newlaos 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

  
说明：Build 0414 与 Build 0307 的以机器码求注册码的方式不同，它现改为由注册名求注册码的方式，但用 Build 0307 的注册信息导入注册表的话，依然可以成为注册版

1、先用 fi2.5 发现此软件没有加壳

2、用 w32dasm 黄金版中文版对 Iparmor.exe(也就是木马克星 5.40 Build 0414 的主文件)静态态反汇编，再用“串式数据参考”，找到"注册失败"（多经典的句子呀），双击来到下面代码段：

3、再用 TRW20001.23 注册版进行动态跟踪，下断 BPX 0055CD74(通常在注册成功与否的前面一些下断，这样，才能找到关键部分)，  
先输入注册名：newlaos  
假码: 78787878

.......  
.......  
:0055CD74 8B55FC                  mov edx, dword ptr \[ebp\-04\]  
:0055CD77 8BC6                    mov eax, esi  
:0055CD79 E80E0EEEFF              call 0043DB8C  
:0055CD7E 8D55F0                  lea edx, dword ptr \[ebp\-10\]  
:0055CD81 8BB3D8020000            mov esi, dword ptr \[ebx+000002D8\]  
:0055CD87 8BC6                    mov eax, esi  
:0055CD89 E8BE0DEEFF              call 0043DB4C  
:0055CD8E 8B45F0                  mov eax, dword ptr \[ebp\-10\]  
:0055CD91 8D55F4                  lea edx, dword ptr \[ebp\-0C\]  
:0055CD94 E867D5EAFF              call 0040A300  
:0055CD99 8B55F4                  mov edx, dword ptr \[ebp\-0C\]  
:0055CD9C 8BC6                    mov eax, esi  
:0055CD9E E8E90DEEFF              call 0043DB8C  
:0055CDA3 8D95E8FEFFFF            lea edx, dword ptr \[ebp+FFFFFEE8\]  
:0055CDA9 8B83E0020000            mov eax, dword ptr \[ebx+000002E0\]  
:0055CDAF E8980DEEFF              call 0043DB4C  
:0055CDB4 8B85E8FEFFFF            mov eax, dword ptr \[ebp+FFFFFEE8\]  
:0055CDBA 8D95ECFEFFFF            lea edx, dword ptr \[ebp+FFFFFEEC\]  
:0055CDC0 E81FD2EAFF              call 00409FE4  
:0055CDC5 8B95ECFEFFFF            mov edx, dword ptr \[ebp+FFFFFEEC\]<===EDX\=NEWLAOS(自动转为大写)  
:0055CDCB 8D85F0FEFFFF            lea eax, dword ptr \[ebp+FFFFFEF0\]  
:0055CDD1 B9FF000000              mov ecx, 000000FF  
:0055CDD6 E83D7DEAFF              call 00404B18  
:0055CDDB 8D95F0FEFFFF            lea edx, dword ptr \[ebp+FFFFFEF0\]  
:0055CDE1 8B83D0020000            mov eax, dword ptr \[ebx+000002D0\]  
:0055CDE7 E8A854F6FF              call 004C2294  <===得出一个值在 EAX 里 19EC3494  
:0055CDEC 8D95E4FEFFFF            lea edx, dword ptr \[ebp+FFFFFEE4\]  
:0055CDF2 8B83D8020000            mov eax, dword ptr \[ebx+000002D8\]  
:0055CDF8 E84F0DEEFF              call 0043DB4C  <===  
:0055CDFD 8B85E4FEFFFF            mov eax, dword ptr \[ebp+FFFFFEE4\]  
:0055CE03 50                      push eax  
:0055CE04 8B83D0020000            mov eax, dword ptr \[ebx+000002D0\]  
:0055CE0A 8B8024020000            mov eax, dword ptr \[eax+00000224\] <===EAX\=650363A3  
:0055CE10 05EA040000              add eax, 000004EA     <===EAX\=6503688D(这时转为 10 进制，就是真正的注册码了)  
:0055CE15 99                      cdq       <===这里是一个符号位的扩展  
:0055CE16 33C2                    xor eax, edx <===而这个异或运算配合上面的符号位的扩展  
:0055CE18 2BC2                    sub eax, edx <===如果符号为负数，这里就等于加 1，正好是一个求绝对值的过程  
:0055CE1A 8D95E0FEFFFF            lea edx, dword ptr \[ebp+FFFFFEE0\]  
:0055CE20 E857D7EAFF              call 0040A57C  
:0055CE25 8B95E0FEFFFF            mov edx, dword ptr \[ebp+FFFFFEE0\] <===EDX 就是真注册码了  
:0055CE2B 58                      pop eax    <===EAX\=78787878(我输入的假码) ，在这一行可以用 KEYMAKE 制作内存注册机  
:0055CE2C E81B7EEAFF              call 00404C4C           <===将 EDX 和 EAX 进行对比的 CALL  
:0055CE31 755D                    jne 0055CE90           <===关键跳转，要想注册成功，则 ZF=1，即不跳转了  
:0055CE33 803DB117580000          cmp byte ptr \[005817B1\], 00 <===标志位是否为 0，是 0 则表明是英文版  
:0055CE3A 740C                    je 0055CE48

\* Possible StringData Ref from Code Obj ->"注册成功"  
                                 |  
:0055CE3C B818CF5500              mov eax, 0055CF18  
:0055CE41 E84EDDF0FF              call 0046AB94  
:0055CE46 EB0A                    jmp 0055CE52

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0055CE3A(C)  
|

\* Possible StringData Ref from Code Obj ->"Register ok!"  
                                 |  
:0055CE48 B82CCF5500              mov eax, 0055CF2C  
:0055CE4D E842DDF0FF              call 0046AB94

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0055CE46(U)  
|  
:0055CE52 E849FCFFFF              call 0055CAA0  
:0055CE57 33D2                    xor edx, edx  
:0055CE59 8B83D4020000            mov eax, dword ptr \[ebx+000002D4\]  
:0055CE5F E8340BEEFF              call 0043D998  
:0055CE64 33D2                    xor edx, edx  
:0055CE66 8B83D8020000            mov eax, dword ptr \[ebx+000002D8\]  
:0055CE6C E8270BEEFF              call 0043D998  
:0055CE71 33D2                    xor edx, edx  
:0055CE73 8B83DC020000            mov eax, dword ptr \[ebx+000002DC\]  
:0055CE79 E81A0BEEFF              call 0043D998

\* Possible StringData Ref from Code Obj ->"registed"  
                                 |  
:0055CE7E BA44CF5500              mov edx, 0055CF44  
:0055CE83 8B83E4020000            mov eax, dword ptr \[ebx+000002E4\]  
:0055CE89 E8FE0CEEFF              call 0043DB8C  
:0055CE8E EB1F                    jmp 0055CEAF

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0055CE31(C)  
|  
:0055CE90 803DB117580000          cmp byte ptr \[005817B1\], 00  
:0055CE97 740C                    je 0055CEA5

\* Possible StringData Ref from Code Obj ->"注册失败!"  
                                 |  
:0055CE99 B858CF5500              mov eax, 0055CF58  
:0055CE9E E8F1DCF0FF              call 0046AB94  
:0055CEA3 EB0A                    jmp 0055CEAF

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0055CE97(C)  
|

\* Possible StringData Ref from Code Obj ->"Register failure"  
                                 |  
:0055CEA5 B86CCF5500              mov eax, 0055CF6C  
:0055CEAA E8E5DCF0FF              call 0046AB94

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0055CE8E(U), :0055CEA3(U)  
|  
:0055CEAF 33C0                    xor eax, eax  
.......  
.......

\-------------0055CDE7 call 004C2294  得出一个值在 EAX 里 19EC3494--------------  
:004C2294 53                      push ebx  
:004C2295 56                      push esi  
:004C2296 57                      push edi  
:004C2297 81C400FFFFFF            add esp, FFFFFF00  
:004C229D 8BF2                    mov esi, edx  
:004C229F 8D3C24                  lea edi, dword ptr \[esp\]  
:004C22A2 33C9                    xor ecx, ecx  
:004C22A4 8A0E                    mov cl, byte ptr \[esi\]  
:004C22A6 41                      inc ecx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C223C(C)  
|  
:004C22A7 F3                      repz  
:004C22A8 A4                      movsb  
:004C22A9 8BD8                    mov ebx, eax  
:004C22AB 803C2447                cmp byte ptr \[esp\], 47  
:004C22AF 730C                    jnb 004C22BD  
:004C22B1 8D4324                  lea eax, dword ptr \[ebx+24\]  
:004C22B4 8BD4                    mov edx, esp  
:004C22B6 E8C111F4FF              call 0040347C  
:004C22BB EB17                    jmp 004C22D4

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C22AF(C)  
|  
:004C22BD 8D4324                  lea eax, dword ptr \[ebx+24\]  
:004C22C0 50                      push eax  
:004C22C1 B946000000              mov ecx, 00000046  
:004C22C6 BA01000000              mov edx, 00000001  
:004C22CB 8D442404                lea eax, dword ptr \[esp+04\]  
:004C22CF E8700DF4FF              call 00403044     

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C22BB(U)  
|  
:004C22D4 8BC3                    mov eax, ebx  
:004C22D6 E805010000              call 004C23E0   <===F8 跟进  
:004C22DB 81C400010000            add esp, 00000100  
:004C22E1 5F                      pop edi  
:004C22E2 5E                      pop esi  
:004C22E3 5B                      pop ebx  
:004C22E4 C3                      ret

\-----------004C22D6 call 004C23E0  按 F8 跟进来到下列代码段-------------------  
:004C23E0 53                      push ebx  
:004C23E1 83C4B8                  add esp, FFFFFFB8  
:004C23E4 8BD8                    mov ebx, eax  
:004C23E6 33C0                    xor eax, eax  
:004C23E8 8A4324                  mov al, byte ptr \[ebx+24\]  
:004C23EB 40                      inc eax  
:004C23EC 83F846                  cmp eax, 00000046  
:004C23EF 7F0B                    jg 004C23FC        <===如果注册名如满了 70 位，就省去下面的填\*循环

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C23FA(C)  
|  
:004C23F1 C64403242A              mov \[ebx+eax+24\], 2A  <===如果注册名不足 70 位的话，后面就用\*号补齐  
:004C23F6 40                      inc eax  
:004C23F7 83F847                  cmp eax, 00000047  
:004C23FA 75F5                    jne 004C23F1

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C23EF(C)  
|  
:004C23FC 33C0                    xor eax, eax  
:004C23FE 8BD4                    mov edx, esp

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C240B(C)  
|  
:004C2400 8A4C0324                mov cl, byte ptr \[ebx+eax+24\]  
:004C2404 880A                    mov byte ptr \[edx\], cl  
:004C2406 40                      inc eax  
:004C2407 42                      inc edx  
:004C2408 83F847                  cmp eax, 00000047  
:004C240B 75F3                    jne 004C2400   <===这个循环结构，完成长度为 46(10 进制为 70)的字符在内存的复制  
:004C240D 8BCC                    mov ecx, esp  
:004C240F 8B932C020000            mov edx, dword ptr \[ebx+0000022C\]  
:004C2415 8BC3                    mov eax, ebx  
:004C2417 E87CFFFFFF              call 004C2398  <===在这里 F8 再次跟进，因为这个 CALL 的作用就升成最后的 650363A3  
:004C241C 898324020000            mov dword ptr \[ebx+00000224\], eax  
:004C2422 33C0                    xor eax, eax  
:004C2424 8A8324010000            mov al, byte ptr \[ebx+00000124\]  
:004C242A 40                      inc eax  
:004C242B 83F846                  cmp eax, 00000046  
:004C242E 7F0E                    jg 004C243E

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C243C(C)  
|  
:004C2430 C68403240100002A        mov byte ptr \[ebx+eax+00000124\], 2A  
:004C2438 40                      inc eax  
:004C2439 83F847                  cmp eax, 00000047  
:004C243C 75F2                    jne 004C2430

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C242E(C)  
|  
:004C243E 33C0                    xor eax, eax  
:004C2440 8BD4                    mov edx, esp

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C2450(C)  
|  
:004C2442 8A8C0324010000          mov cl, byte ptr \[ebx+eax+00000124\]  
:004C2449 880A                    mov byte ptr \[edx\], cl  
:004C244B 40                      inc eax  
:004C244C 42                      inc edx  
:004C244D 83F847                  cmp eax, 00000047  
:004C2450 75F0                    jne 004C2442  
:004C2452 8BCC                    mov ecx, esp  
:004C2454 8B9330020000            mov edx, dword ptr \[ebx+00000230\]  
:004C245A 8BC3                    mov eax, ebx  
:004C245C E837FFFFFF              call 004C2398  
:004C2461 898328020000            mov dword ptr \[ebx+00000228\], eax  
:004C2467 83C448                  add esp, 00000048  
:004C246A 5B                      pop ebx  
:004C246B C3                      ret

  
\------------004C2417 call 004C2398  这里 F8 再次跟进，来到下列代码段------------------  
:004C2398 53                      push ebx  
:004C2399 56                      push esi  
:004C239A 57                      push edi  
:004C239B 83C4B8                  add esp, FFFFFFB8  
:004C239E 8BF1                    mov esi, ecx  
:004C23A0 8D3C24                  lea edi, dword ptr \[esp\]  
:004C23A3 B911000000              mov ecx, 00000011  
:004C23A8 F3                      repz  
:004C23A9 A5                      movsd  
:004C23AA 66A5                    movsw  
:004C23AC A4                      movsb  
:004C23AD B147                    mov cl, 47  <===计数器 CL 赋初值为 47(71)  
:004C23AF 8BC4                    mov eax, esp <===这里是 7NEWALAOS\*\*\*\*\*\*\*(前面有个注册名长度，注：前面的 7 是以内码的形式存放，也就是说后面的 N 其内码是 4E)

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004C23D5(C)  
|  
:004C23B1 8BDA                    mov ebx, edx <===EBX\=EFCA99(程序给的初始数值，为定值)  
:004C23B3 C1EB08                  shr ebx, 08  <===EBX\=EFCA  
:004C23B6 81E3FFFFFF00            and ebx, 00FFFFFF   
:004C23BC 0FB630                  movzx esi, byte ptr \[eax\] <===依次取出字符的 ASC 值放入 ESI，呵呵，第一个是注册码的长度  
:004C23BF 33D6                    xor edx, esi  <===EDX\=EFCA99 XOR 07 =EFCA9E  
:004C23C1 81E2FF000000            and edx, 000000FF <===EDX\=9E  
:004C23C7 8B1495AC645600          mov edx, dword ptr \[4\*edx+005664AC\] <===这里完全是天大的码表嘛(见下面)。共有 256 个数。根据 EDX 的不同所取的值也不同。  
:004C23CE 33DA                    xor ebx, edx <===EBX\=EFCA XOR 17B7BE43  
:004C23D0 8BD3                    mov edx, ebx <===最后的 EDX  
:004C23D2 40                      inc eax     <===EAX\=EAX+1  
:004C23D3 FEC9                    dec cl      <===CL\=CL\-1  
:004C23D5 75DA                    jne 004C23B1  <===这里是一个循环结构，直到 CL 计数器被减完  
:004C23D7 8BC2                    mov eax, edx <===最后的 EAX 值就是我们所要的值了。  
:004C23D9 83C448                  add esp, 00000048  
:004C23DC 5F                      pop edi  
:004C23DD 5E                      pop esi  
:004C23DE 5B                      pop ebx  
:004C23DF C3                      ret  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*码表如下，共有 256 个数\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
77073096 EE0E612C 990951BA 076DC419 706AF48F E963A535 9E6495A3 0EDB8832   
79DCB8A4 E0D5E91E 97D2D988 09B64C2B 7EB17CBD E7B82D07 90BF1D91 00000000   
6AB020F2 F3B97148 84BE41DE 1ADAD47D 6DDDE4EB F4D4B551 83D385C7 136C9856   
646BA8C0 FD62F97A 8A65C9EC 14015C4F 63066CD9 FA0F3D63 8D080DF5 3B6E20C8   
4C69105E D56041E4 A2677172 3C03E4D1 4B04D447 D20D85FD A50AB56B 35B5A8FA   
42B2986C DBBBC9D6 ACBCF940 32D86CE3 45DF5C75 DCD60DCF ABD13D59 26D930AC   
51DE003A C8D75180 BFD06116 21B4F4B5 56B3C423 CFBA9599 B8BDA50F 2802B89E   
5F058808 C60CD9B2 B10BE924 2F6F7C87 58684C11 C1611DAB B6662D3D 76DC4190   
01DB7106 98D220BC EFD5102A 71B18589 06B6B51F 9FBFE4A5 E8B8D433 7807C9A2   
0F00F934 9609A88E E10E9818 7F6A0DBB 086D3D2D 91646C97 E6635C01 6B6B51F4   
1C6C6162 856530D8 F262004E 6C0695ED 1B01A57B 8208F4C1 F50FC457 65B0D9C6   
12B7E950 8BBEB8EA FCB9887C 62DD1DDF 15DA2D49 8CD37CF3 FBD44C65 4DB26158   
3AB551CE A3BC0074 D4BB30E2 4ADFA541 3DD895D7 A4D1C46D D3D6F4FB 4369E96A   
346ED9FC AD678846 DA60B8D0 44042D73 33031DE5 AA0A4C5F DD0D7CC9 5005713C   
270241AA BE0B1010 C90C2086 5768B525 206F85B3 B966D409 CE61E49F 5EDEF90E   
29D9C998 B0D09822 C7D7A8B4 59B33D17 2EB40D81 B7BD5C3B C0BA6CAD EDB88320   
9ABFB3B6 03B6E20C 74B1D29A EAD54739 9DD277AF 04DB2615 73DC1683 E3630B12   
94643B84 0D6D6A3E 7A6A5AA8 E40ECF0B 9309FF9D 0A00AE27 7D079EB1 F00F9344   
8708A3D2 1E01F268 6906C2FE F762575D 806567CB 196C3671 6E6B06E7 FED41B76   
89D32BE0 10DA7A5A 67DD4ACC F9B9DF6F 8EBEEFF9 17B7BE43 60B08ED5 D6D6A3E8   
A1D1937E 38D8C2C4 4FDFF252 D1BB67F1 A6BC5767 3FB506DD 48B2364B D80D2BDA   
AF0A1B4C 36034AF6 41047A60 DF60EFC3 A867DF55 316E8EEF 4669BE79 CB61B38C   
BC66831A 256FD2A0 5268E236 CC0C7795 BB0B4703 220216B9 5505262F C5BA3BBE   
B2BD0B28 2BB45A92 5CB36A04 C2D7FFA7 B5D0CF31 2CD99E8B 5BDEAE1D 9B64C2B0   
EC63F226 756AA39C 026D930A 9C0906A9 EB0E363F 72076785 05005713 95BF4A82   
E2B87A14 7BB12BAE 0CB61B38 92D28E9B E5D5BE0D 7CDCEFB7 0BDBDF21 86D3D2D4   
F1D4E242 68DDB3F8 1FDA836E 81BE16CD F6B9265B 6FB077E1 18B74777 88085AE6   
FF0F6A70 66063BCA 11010B5C 8F659EFF F862AE69 616BFFD3 166CCF45 A00AE278   
D70DD2EE 4E048354 3903B3C2 A7672661 D06016F7 4969474D 3E6E77DB AED16A4A   
D9D65ADC 40DF0B66 37D83BF0 A9BCAE53 DEBB9EC5 47B2CF7F 30B5FFE9 BDBDF21C  
CABAC28A 53B39330 24B4A3A6 BAD03605 CDD70693 54DE5729 23D967BF B3667A2E   
C4614AB8 5D681B02 2A6F2B94 B40BBE37 C30C8EA1 5A05DF1B 2D02EF8D 004C3E50   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

4、算法注册机原码：  
(注：因为仅供学习之用，避免它人盗用从事不法活动，损坏作者利益，故此段程序中省去关键一句。如果是学习爱好者，可以 PM 给我询问此句)  
\--------------VB6.0 在 WIN98 下编译通过----------------------------

Private Sub Command1\_Click()  
Dim i As Integer  
Dim h As Integer  
Dim edx As Long  
Dim ebx As Long  
Dim startin As String  
Dim A As Variant  
Dim B As Variant  
startin = Text1.Text  
nlen = Len(startin)

A = Array(0, &H77073096, &HEE0E612C, &H990951BA, &H76DC419, &H706AF48F, &HE963A535, &H9E6495A3, &HEDB8832, &H79DCB8A4, &HE0D5E91E, &H97D2D988, &H9B64C2B, &H7EB17CBD, &HE7B82D07, &H90BF1D91, &H0 \_  
, &H6AB020F2, &HF3B97148, &H84BE41DE, &H1ADAD47D, &H6DDDE4EB, &HF4D4B551, &H83D385C7, &H136C9856, &H646BA8C0, &HFD62F97A, &H8A65C9EC, &H14015C4F, &H63066CD9, &HFA0F3D63, &H8D080DF5, &H3B6E20C8 \_  
, &H4C69105E, &HD56041E4, &HA2677172, &H3C03E4D1, &H4B04D447, &HD20D85FD, &HA50AB56B, &H35B5A8FA, &H42B2986C, &HDBBBC9D6, &HACBCF940, &H32D86CE3, &H45DF5C75, &HDCD60DCF, &HABD13D59, &H26D930AC \_  
, &H51DE003A, &HC8D75180, &HBFD06116, &H21B4F4B5, &H56B3C423, &HCFBA9599, &HB8BDA50F, &H2802B89E, &H5F058808, &HC60CD9B2, &HB10BE924, &H2F6F7C87, &H58684C11, &HC1611DAB, &HB6662D3D, &H76DC4190 \_  
, &H1DB7106, &H98D220BC, &HEFD5102A, &H71B18589, &H6B6B51F, &H9FBFE4A5, &HE8B8D433, &H7807C9A2, &HF00F934, &H9609A88E, &HE10E9818, &H7F6A0DBB, &H86D3D2D, &H91646C97, &HE6635C01, &H6B6B51F4 \_  
, &H1C6C6162, &H856530D8, &HF262004E, &H6C0695ED, &H1B01A57B, &H8208F4C1, &HF50FC457, &H65B0D9C6, &H12B7E950, &H8BBEB8EA, &HFCB9887C, &H62DD1DDF, &H15DA2D49, &H8CD37CF3, &HFBD44C65, &H4DB26158 \_  
, &H3AB551CE, &HA3BC0074, &HD4BB30E2, &H4ADFA541, &H3DD895D7, &HA4D1C46D, &HD3D6F4FB, &H4369E96A, &H346ED9FC, &HAD678846, &HDA60B8D0, &H44042D73, &H33031DE5, &HAA0A4C5F, &HDD0D7CC9, &H5005713C \_  
, &H270241AA, &HBE0B1010, &HC90C2086, &H5768B525, &H206F85B3, &HB966D409, &HCE61E49F, &H5EDEF90E, &H29D9C998, &HB0D09822, &HC7D7A8B4, &H59B33D17, &H2EB40D81, &HB7BD5C3B, &HC0BA6CAD, &HEDB88320 \_  
, &H9ABFB3B6, &H3B6E20C, &H74B1D29A, &HEAD54739, &H9DD277AF, &H4DB2615, &H73DC1683, &HE3630B12, &H94643B84, &HD6D6A3E, &H7A6A5AA8, &HE40ECF0B, &H9309FF9D, &HA00AE27, &H7D079EB1, &HF00F9344 \_  
, &H8708A3D2, &H1E01F268, &H6906C2FE, &HF762575D, &H806567CB, &H196C3671, &H6E6B06E7, &HFED41B76, &H89D32BE0, &H10DA7A5A, &H67DD4ACC, &HF9B9DF6F, &H8EBEEFF9, &H17B7BE43, &H60B08ED5, &HD6D6A3E8 \_  
, &HA1D1937E, &H38D8C2C4, &H4FDFF252, &HD1BB67F1, &HA6BC5767, &H3FB506DD, &H48B2364B, &HD80D2BDA, &HAF0A1B4C, &H36034AF6, &H41047A60, &HDF60EFC3, &HA867DF55, &H316E8EEF, &H4669BE79, &HCB61B38C \_  
, &HBC66831A, &H256FD2A0, &H5268E236, &HCC0C7795, &HBB0B4703, &H220216B9, &H5505262F, &HC5BA3BBE, &HB2BD0B28, &H2BB45A92, &H5CB36A04, &HC2D7FFA7, &HB5D0CF31, &H2CD99E8B, &H5BDEAE1D, &H9B64C2B0 \_  
, &HEC63F226, &H756AA39C, &H26D930A, &H9C0906A9, &HEB0E363F, &H72076785, &H5005713, &H95BF4A82, &HE2B87A14, &H7BB12BAE, &HCB61B38, &H92D28E9B, &HE5D5BE0D, &H7CDCEFB7, &HBDBDF21, &H86D3D2D4 \_  
, &HF1D4E242, &H68DDB3F8, &H1FDA836E, &H81BE16CD, &HF6B9265B, &H6FB077E1, &H18B74777, &H88085AE6, &HFF0F6A70, &H66063BCA, &H11010B5C, &H8F659EFF, &HF862AE69, &H616BFFD3, &H166CCF45, &HA00AE278 \_  
, &HD70DD2EE, &H4E048354, &H3903B3C2, &HA7672661, &HD06016F7, &H4969474D, &H3E6E77DB, &HAED16A4A, &HD9D65ADC, &H40DF0B66, &H37D83BF0, &HA9BCAE53, &HDEBB9EC5, &H47B2CF7F, &H30B5FFE9, &HBDBDF21C \_  
, &HCABAC28A, &H53B39330, &H24B4A3A6, &HBAD03605, &HCDD70693, &H54DE5729, &H23D967BF, &HB3667A2E, &HC4614AB8, &H5D681B02, &H2A6F2B94, &HB40BBE37, &HC30C8EA1, &H5A05DF1B, &H2D02EF8D, &H4C3E50)

'完成注册名的部分的前期工作  
B = Array(42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42)  
k = 1      '是否为汉字的标志位  
tlen = 0  
For h = 1 To nlen  
 sumtmp = Asc(UCase(Mid(startin, h, 1)))  
 If Abs(sumtmp) <> sumtmp Then   '对中文特别设计  
   k = 2  
 Else  
   k = 1  
 End If  
 For e = 1 To k  
 tlen = tlen + 1  
 B(tlen) = CInt("&H" + Mid(Hex(sumtmp), 2 \* e - 1, 2))  
 Next  
Next h  
B(0) = tlen  
If nlen < 1 Or tlen > 70 Then  
h = MsgBox("你输入的注册名有误或是还没有输入注册名", 0, "请确认你输入是否正确!")  
Else  
 ebx = 0  
 edx = &HEFCA99  
 For j = 0 To 70  
   strtmp = Hex(edx)  
   edx = (edx Xor B(j)) And &HFF  
   edx = A(edx)  
   ebx = ebx Xor edx  
   edx = ebx  
 Next j  
edx = Abs(edx + &H4EA)  
End If  
Text2.Text = edx  
End Sub

  
5、注册信息保存在注册表：  
\[HKEY\_USERS\\.DEFAULT\\Software\\AngelSoft\\iparmor\]  
"deta"="26"  
"pass"="XXXXXXXXX"                <===隐!  
"name536"="NEWLAOS"

 4dd466d0e1e0cc123afbe504da2462ca