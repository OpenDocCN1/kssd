

# 魔术情书 6.55 破解过程+不脱壳打破解补丁【原创】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-117.htm`](https://www.pediy.com/kssd/pediy07/pediy7-117.htm)

【破解作者】 海风月影  
【作者邮箱】 zjhangtian@sohu.com  
【使用工具】 od 1.10d,PEid,LoadPE,winHex,eXeScope  
【破解平台】 WinXP  
【软件名称】 魔术情书 6.55  
【下载地址】 [www.coolboo.com](http://www.coolboo.com)  
【软件大小】 2M  
【加壳方式】 upx  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
一、破解过程  
  
1、peid 查壳，UPX 0.80 - 0.84 -> Markus & Laszlo, 是简单的压缩壳，脱壳过程就不写了，不会脱也没关系，反正后面我们要不脱壳打补丁  
  
2、用 od 载入，F9 运行，出现对话框，显示用了几次，因此，首先把 nag 去掉。如何确定调用 nag 的代码呢？我介绍一种自己的方法，打开内存（ALT+M），选中当前执行区域，这里是  
  
内存镜像，项目 28  
 地址=00401000  
 大小=00197000 (1667072.)  
 Owner=MSQingSh 00400000  
 区段=UPX0  
 类型=Imag 01001002  
 访问=R  
 初始访问=RWE  
  
因为我们执行的位置在 0056xxxx，属于上面那个区域。然后按 F2，此时，程序被中断下来了  
  
0044F25B    E8 F8F9FFFF       call MSQingSh.0044EC58  
0044F260    E8 1F46FBFF       call MSQingSh.00403884  
0044F265    807D FB 00        cmp byte ptr ss:\[ebp\-5\],0  
0044F269    74 05             je short MSQingSh.0044F270  
0044F26B    E8 FC85FBFF       call <jmp.&user32.WaitMessage>  
0044F270    33C0              xor eax,eax                             ; 中断在这  
0044F272    5A                pop edx  
0044F273    59                pop ecx  
0044F274    59                pop ecx  
0044F275    64:8910           mov dword ptr fs:\[eax\],edx  
0044F278    68 92F24400       push MSQingSh.0044F292  
0044F27D    8D45 F0           lea eax,dword ptr ss:\[ebp\-10\]  
0044F280    BA 02000000       mov edx,2  
0044F285    E8 7E4BFBFF       call MSQingSh.00403E08  
0044F28A    C3                retn  
  
然后一路按 F8，直到明显遇到一个循环，如下  
  
0044BC11    E8 36BBFBFF             call <jmp.&user32.SendMessageA>  
0044BC16    8B45 FC                 mov eax,dword ptr ss:\[ebp\-4\]  
0044BC19    33D2                    xor edx,edx  
0044BC1B    8990 34020000           mov dword ptr ds:\[eax+234\],edx  
0044BC21    8B03                    mov eax,dword ptr ds:\[ebx\]              ; 从这  
0044BC23    E8 D42D0000             call MSQingSh.0044E9FC  
0044BC28    8B03                    mov eax,dword ptr ds:\[ebx\]  
0044BC2A    80B8 8C000000 00        cmp byte ptr ds:\[eax+8C\],0  
0044BC31    74 0F                   je short MSQingSh.0044BC42  
0044BC33    8B45 FC                 mov eax,dword ptr ss:\[ebp\-4\]  
0044BC36    C780 34020000 02000000  mov dword ptr ds:\[eax+234\],2  
0044BC40    EB 14                   jmp short MSQingSh.0044BC56  
0044BC42    8B45 FC                 mov eax,dword ptr ss:\[ebp\-4\]  
0044BC45    83B8 34020000 00        cmp dword ptr ds:\[eax+234\],0  
0044BC4C    74 08                   je short MSQingSh.0044BC56  
0044BC4E    8B45 FC                 mov eax,dword ptr ss:\[ebp\-4\]  
0044BC51    E8 26FDFFFF             call MSQingSh.0044B97C  
0044BC56    8B45 FC                 mov eax,dword ptr ss:\[ebp\-4\]  
0044BC59    8B80 34020000           mov eax,dword ptr ds:\[eax+234\]  
0044BC5F    85C0                    test eax,eax  
0044BC61  ^ 74 BE                   je short MSQingSh.0044BC21              ; 到这，是消息循环  
0044BC63    8945 F8                 mov dword ptr ss:\[ebp\-8\],eax            ; 在这下断点  
0044BC66    6A 00                   push 0  
0044BC68    6A 00                   push 0  
0044BC6A    68 01B00000             push 0B001  
0044BC6F    8B45 FC                 mov eax,dword ptr ss:\[ebp\-4\]  
0044BC72    E8 D9ABFEFF             call MSQingSh.00436850  
  
这个循环是 windows 的消息循环，因为程序是用 delphi 写的，不容易看出来，当窗体卸载时就退出循环，因此在 0044BC63 下断点，F9 运行  
断在 44bc63 处，小心按 F8，一步一步走，不断地 ret ，遇到下面的这样的 ret 直接跳  
  
0044BC97    5A                      pop edx  
0044BC98    59                      pop ecx  
0044BC99    59                      pop ecx  
0044BC9A    64:8910                 mov dword ptr fs:\[eax\],edx  
0044BC9D    68 B2BC4400             push MSQingSh.0044BCB2                  ; 这是跳转地址  
0044BCA2    8B45 FC                 mov eax,dword ptr ss:\[ebp\-4\]  
0044BCA5    E8 7AFDFFFF             call MSQingSh.0044BA24  
0044BCAA    C3                      retn                                    ; 这个实际上是个跳转  
  
前面一个 push,后面一个 ret,实际上就是一个 jmp,因此只管跳，碰到下面的 ret 就要注意了，恢复 ebp 就代表这个过程结束了  
  
0044BD73    8B45 F8                 mov eax,dword ptr ss:\[ebp\-8\]  
0044BD76    5E                      pop esi  
0044BD77    5B                      pop ebx  
0044BD78    8BE5                    mov esp,ebp  
0044BD7A    5D                      pop ebp                                 ; 恢复 ebp 了，就要注意了  
0044BD7B    C3                      retn                                    ; 这个 ret 返回后的地址要注意  
  
ret 后来到 552379，上面一行的 call 就是调用 nag 的 call  
  
0055236A    A1 2C835600             mov eax,dword ptr ds:\[56832C\]  
0055236F    8B00                    mov eax,dword ptr ds:\[eax\]  
00552371    8B10                    mov edx,dword ptr ds:\[eax\]  
00552373    FF92 D8000000           call dword ptr ds:\[edx+D8\]         ; 这个 call 就是调用 nag 的 call  
00552379    33C0                    xor eax,eax                        ; 返回到这  
0055237B    5A                      pop edx  
0055237C    59                      pop ecx  
0055237D    59                      pop ecx  
0055237E    64:8910                 mov dword ptr fs:\[eax\],edx  
00552381    68 A6235500             push MSQingSh.005523A6  
00552386    A1 2C835600             mov eax,dword ptr ds:\[56832C\]  
0055238B    8B00                    mov eax,dword ptr ds:\[eax\]  
  
把 00552373 这行 nop 掉，nag 就没有了 ^\_^  
  
  
3、但还没有破解掉，因为虽然 nag 没了，但 20 次的限制还是有的，用了 20 次后碰到要注册的 nag，这就说明这个 nag 是在上面去掉的 nag 前出现的，用 od 载入脱壳后的文件（不是把上面去掉 nag 的文件），F9 运行，用刚才上面的方法确定 nag 的调用地址  
  
005635D2    8990 EC020000           mov dword ptr ds:\[eax+2EC\],edx  
005635D8    A1 CC825600             mov eax,dword ptr ds:\[5682CC\]  
005635DD    8B00                    mov eax,dword ptr ds:\[eax\]  
005635DF    E8 ACB5EEFF             call MSQingSh.0044EB90               ; nag 的 call  
005635E4    E8 9BCCFFFF             call MSQingSh.00560284               ; 返回到这行  
005635E9    E9 91070000             jmp MSQingSh.00563D7F                ; 跳走就 over 了  
  
然后向上看会发现很多这样的类似代码（call 44eb90 / call 560284 / jmp 563d7f），因此这段代码不能执行，一直向上看，代码类似就向上看直到看到代码不一样的时候，凭经验下断点（在跳转上下），然后重新运行，看看能否断下来。我找到一段可疑代码  
  
0056218D    33DB                    xor ebx,ebx                 ;最可疑的是这里 ebx=0  
0056218F    A1 54D05600             mov eax,dword ptr ds:\[56D054\]  
00562194    8B40 30                 mov eax,dword ptr ds:\[eax+30\]  
00562197    83C0 18                 add eax,18  
0056219A    8B15 B87E5600           mov edx,dword ptr ds:\[567EB8\]        ; MSQingSh.0056B5E8  
005621A0    8B52 24                 mov edx,dword ptr ds:\[edx+24\]  
005621A3    E8 901CEAFF             call MSQingSh.00403E38  
005621A8    8B15 B87E5600           mov edx,dword ptr ds:\[567EB8\]        ; MSQingSh.0056B5E8  
005621AE    8B52 20                 mov edx,dword ptr ds:\[edx+20\]  
005621B1    8BCB                    mov ecx,ebx  
005621B3    A1 54D05600             mov eax,dword ptr ds:\[56D054\]  
005621B8    E8 D358F0FF             call MSQingSh.00467A90  
.......  
中间有很多这样结构的代码，太长了，不帖了  
.......  
00562823    8B15 B87E5600           mov edx,dword ptr ds:\[567EB8\]        ; MSQingSh.0056B5E8  
00562829    8B52 28                 mov edx,dword ptr ds:\[edx+28\]  
0056282C    B9 674F0000             mov ecx,4F67  
00562831    A1 60D05600             mov eax,dword ptr ds:\[56D060\]  
00562836    E8 6DDDFFFF             call MSQingSh.005605A8  
0056283B    E9 52100000             jmp MSQingSh.00563892         ;这里跳走  
  
ebx = 0 ,猜测可能是清空计数器，从 0 开始计数（每次都清空，这样不就等于无限次数用了嘛^\_^），记住行号 0056218d，向上看有哪个跳转跳到这，很快就找到这里  
  
00561FDB    E8 B0CBEEFF         call MSQingSh.0044EB90  
00561FE0    E8 9FE2FFFF         call MSQingSh.00560284  
00561FE5    E9 951D0000         jmp MSQingSh.00563D7F  
00561FEA    E8 B1E2FFFF         call MSQingSh.005602A0           ; 上面是无条件跳转，记住这个行号  
00561FEF    803D 88D05600 00    cmp byte ptr ds:\[56D088\],0  
00561FF6    74 43               je short MSQingSh.0056203B  
00561FF8    803D 89D05600 00    cmp byte ptr ds:\[56D089\],0  
00561FFF    74 3A               je short MSQingSh.0056203B  
00562001    803D 8AD05600 00    cmp byte ptr ds:\[56D08A\],0  
00562008    74 31               je short MSQingSh.0056203B  
0056200A    803D 8BD05600 00    cmp byte ptr ds:\[56D08B\],0  
00562011    74 28               je short MSQingSh.0056203B  
00562013    803D 8CD05600 00    cmp byte ptr ds:\[56D08C\],0  
0056201A    74 1F               je short MSQingSh.0056203B  
0056201C    803D 8DD05600 00    cmp byte ptr ds:\[56D08D\],0  
00562023    74 16               je short MSQingSh.0056203B  
00562025    803D 8ED05600 00    cmp byte ptr ds:\[56D08E\],0  
0056202C    74 0D               je short MSQingSh.0056203B  
0056202E    803D 8FD05600 00    cmp byte ptr ds:\[56D08F\],0  
00562035    0F85 52010000       jnz MSQingSh.0056218D            ; 找到这了，上面的都不能跳，这个要跳  
0056203B    A1 CC825600         mov eax,dword ptr ds:\[5682CC\]  
00562040    8B00                mov eax,dword ptr ds:\[eax\]  
  
在 00561FEA 下断点发现断不下来，所以继续向上找跳到 00561FEA 的跳转  
  
00561B34    8B52 20             mov edx,dword ptr ds:\[edx+20\]  
00561B37    B1 01               mov cl,1  
00561B39    A1 54D05600         mov eax,dword ptr ds:\[56D054\]  
00561B3E    E8 C95DF0FF         call MSQingSh.0046790C  
00561B43    8BD8                mov ebx,eax  
00561B45    84DB                test bl,bl  
00561B47    0F84 F30C0000       je MSQingSh.00562840             ; 这是关键的跳转，不能跳  
00561B4D    8B0D B87E5600       mov ecx,dword ptr ds:\[567EB8\]    ; MSQingSh.0056B5E8  
00561B53    8B49 28             mov ecx,dword ptr ds:\[ecx+28\]  
00561B56    8D85 ECFEFFFF       lea eax,dword ptr ss:\[ebp\-114\]  
00561B5C    8B15 60D05600       mov edx,dword ptr ds:\[56D060\]  
00561B62    E8 4925EAFF         call MSQingSh.004040B0  
00561B67    8B85 ECFEFFFF       mov eax,dword ptr ss:\[ebp\-114\]  
00561B6D    E8 D676EAFF         call MSQingSh.00409248  
00561B72    84C0                test al,al  
00561B74    0F85 1E030000       jnz MSQingSh.00561E98         ;这些都不能跳，下面很多  
......  
中间很多这样的代码，省略  
......  
00561E69    75 2D               jnz short MSQingSh.00561E98     ;不能跳  
00561E6B    8B0D B87E5600       mov ecx,dword ptr ds:\[567EB8\]    ; MSQingSh.0056B5E8  
00561E71    8B49 70             mov ecx,dword ptr ds:\[ecx+70\]  
00561E74    8D85 A4FEFFFF       lea eax,dword ptr ss:\[ebp\-15C\]  
00561E7A    8B15 60D05600       mov edx,dword ptr ds:\[56D060\]  
00561E80    E8 2B22EAFF         call MSQingSh.004040B0  
00561E85    8B85 A4FEFFFF       mov eax,dword ptr ss:\[ebp\-15C\]  
00561E8B    E8 B873EAFF         call MSQingSh.00409248  
00561E90    84C0                test al,al  
00561E92    0F84 52010000       je MSQingSh.00561FEA             ; 找到这，这个要跳  
00561E98    A1 CC825600         mov eax,dword ptr ds:\[5682CC\]  
00561E9D    8B00                mov eax,dword ptr ds:\[eax\]  
00561E9F    E8 54CCEEFF         call MSQingSh.0044EAF8  
  
找到  
  
00561E92    0F84 52010000       je MSQingSh.00561FEA       
  
但上面有很多  
  
jnz short MSQingSh.00561E98  
  
在第一个处下断点发现断不下来，因此找到关键的跳转  
  
00561B47    0F84 F30C0000       je MSQingSh.00562840             ; 这是关键的跳转，不能跳  
  
这是能断下来的跳转，直接改成 jmp 0056218D 跳到  
  
0056218D    33DB                    xor ebx,ebx                 ;最可疑的是这里 ebx=0  
  
因为中间的代码都是判断是否是第一次运行的，没必要执行，直接跳不影响结果，F9 运行，果然出现了刚用了一次的 nag（用没有去掉这个 nag 的文件），然后在 nag 掉下面这行  
  
00552373    FF92 D8000000           call dword ptr ds:\[edx+D8\]         ; 这个 call 就是调用 nag 的 call  
  
保存一下 MSQingShu\_cRacKEd.exe，运行一下，呵呵，成功了^\_^！  
此时次数已不受限制了，下面的第四步可有可无，但是考虑到这篇文章第二部分是逆向的，所以干脆就写详细点，把自己的名字也写进程序里^\_^！  
  
4、先确定自己名字的字符，新建文本文件，写入“cRacKEd bY 海风月影\[NE365\]\[DFCG\]”，和“魔术情书 6.55  -  cRacKEd bY 海风月影\[NE365\]\[DFCG\]”，没有引号，前一个是放在 about 里的，后一个是标题栏。然后用 winHex 打开文件，看到 16 进制字符  
用 od 运行破解后的程序，然后打开内存（ALT+M），点右键选搜索－整个区域，填入"(Richie)"，没有引号，注意大小写，第一处  
  
005C8540  69 6F 6E 06 13 D7 F7 D5 DF A3 BA D6 EC 20 BD AD  ion 作者：朱 江  
005C8550  28 52 69 63 68 69 65 29 00 00 0F 54 62 73 53 6B  (Richie)..TbsSk  
  
这里是 delphi 的控件的属性，先记住偏移量 005C8548，用 loadPE 查一下在文件里的偏移 1c7148,用 winHex 修改此处的代码，保存，运行，发现没有用，这就说明在程序运行时修改了控件的属性，这条路走不通了。。。。。。  
  
继续搜索下一个，搜到  
  
00BB1B1C  C4 A7 CA F5 C7 E9 CA E9 20 36 2E 35 35 20 20 20  魔术情书 6.55  
00BB1B2C  20 20 D7 F7 D5 DF A3 BA D6 EC 20 BD AD 28 52 69    作者：朱 江(Ri  
00BB1B3C  63 68 69 65 29 20 20 20 20 20 CD F8 D5 BE A3 BA  chie)     网站：  
00BB1B4C  77 77 77 2E 43 6F 6F 6C 42 6F 6F 2E 63 6F 6D 00  [www.CoolBoo.com.](http://www.CoolBoo.com.)  
  
呵呵，这不是标题栏嘛，记住 00BB1B1C 这个地址，可惜，这个地址是动态生成的，原来程序文件里没有，因此要动态调试，用 od 重新载入，发现都没有 00BB1B1C 这个地址，看来要一步一步跟踪了，打开内存，把地址栏露出来能看到就行了，然后一步步跟踪，看看  
  
内存镜像，项目 36  
 地址=00BB0000  
 大小=00088000 (557056.)  
 Owner=         00BB0000 (itself)  
 区段=  
 类型=Priv 00021004  
 访问=RW  
 初始访问=  
  
这个页面什么时候出来  
  
00403AE4    8B04DF              mov eax,dword ptr ds:\[edi+ebx\*8\]   ; 这个是循环  
00403AE7    43                  inc ebx  
00403AE8    891D B0945600       mov dword ptr ds:\[5694B0\],ebx  
00403AEE    85C0                test eax,eax  
00403AF0    74 02               je short MSQingSh.00403AF4  
00403AF2    FFD0                call eax                           ; 运行到某一个时出现页面，下断点  
00403AF4    3BF3                cmp esi,ebx  
00403AF6  ^ 7F EC               jg short MSQingSh.00403AE4  
  
在 00403AF0 下断点，F9 慢慢按，发现页面出来了就转到 00BB1B1C 去看看，发现还全是 00（幸运^\_^），对 00BB1B1C 下内存写入断点，取消 call eax 的断点，F9 运行，断在  
  
00402980    FD                  std  
00402981    F3:A5               rep movsd                          ; 断在这，不能 F8 走  
00402983    89C1                mov ecx,eax                        ; 在这下断点，F9 运行，取消断点  
00402985    83E1 03             and ecx,3  
00402988    83C6 03             add esi,3  
0040298B    83C7 03             add edi,3  
0040298E    F3:A4               rep movsb  
00402990    FC                  cld  
00402991    5F                  pop edi  
00402992    5E                  pop esi  
00402993    C3                  retn  
  
不能用 F8 走，一走就飞，所以在下一行下断点，F9 运行，取消断点，然后一路 F8，来到  
  
0046A3C7    B8 30B25600         mov eax,MSQingSh.0056B230          ; 指针  
0046A3CC    BA 03000000         mov edx,3  
0046A3D1    E8 4E9DF9FF         call MSQingSh.00404124             ; 上面生成第一个字符串  
0046A3D6    FF35 14B25600       push dword ptr ds:\[56B214\]         ; 在这下断点，这里是第一个字符串  
0046A3DC    68 F8E64600         push MSQingSh.0046E6F8  
0046A3E1    FF35 24B25600       push dword ptr ds:\[56B224\]         ; 这里是 6.55  
0046A3E7    68 622C0000         push 2C62  
0046A3EC    8D45 DC             lea eax,dword ptr ss:\[ebp\-24\]  
0046A3EF    50                  push eax  
0046A3F0    B9 C6570000         mov ecx,57C6  
0046A3F5    BA 700A0000         mov edx,0A70  
0046A3FA    B8 04E74600         mov eax,MSQingSh.0046E704  
0046A3FF    E8 78E8FFFF         call MSQingSh.00468C7C             ; 生成第二个字符串  
0046A404    FF75 DC             push dword ptr ss:\[ebp\-24\]         ; 第二个字符串  
0046A407    B8 18B25600         mov eax,MSQingSh.0056B218          ; 连接后放入的地址（即指针）  
0046A40C    BA 04000000         mov edx,4  
0046A411    E8 0E9DF9FF         call MSQingSh.00404124             ; 连接两个字符  
0046A416    68 732C0000         push 2C73                          ; 来到这，然后向上看  
  
所以，找一个空白的地方，把 魔术情书 6.55  -  cRacKEd bY 海风月影\[NE365\]\[DFCG\] 复制过去，然后修改 0056B218 里的内容，指向我们的字符串，运行，出错，晕！！ 重来一次，断在 0046A3D6，仔细看每个字符串  
  
00BB1A8C                                      01 00 00 00              ...  
00BB1A9C  0D 00 00 00 C4 A7 CA F5 C7 E9 CA E9 20 36 2E 35  ....魔术情书 6.5  
00BB1AAC  35 00                                            5.  
  
  
00BB1ACC                          01 00 00 00 32 00 00 00          ...2...  
00BB1ADC  20 20 20 20 20 D7 F7 D5 DF A3 BA D6 EC 20 BD AD       作者：朱 江  
00BB1AEC  28 52 69 63 68 69 65 29 20 20 20 20 20 CD F8 D5  (Richie)     网  
  
在 字符串－8 偏移处有一个 01，在 字符串－4 偏移处的值正好是字符串的长度，应此我们把这两个信息也加上，再修改 0056B218 里的内容，指向我们的字符串，运行，发现标题已经变成 魔术情书 6.55  -  cRacKEd bY 海风月影\[NE365\]\[DFCG\] 了，下面就是保存了，在  
  
0046A416    68 732C0000         push 2C73                          ; 来到这，然后向上看  
  
处打个 smc 补丁，随便找一片空地，写代码  
  
push 2c73  
mov dword ptr ds:\[0056B218\],(放标题的地址)        ;smc 补丁  
jmp 46a41b                                            ;跳到 push 2c73 的下一行  
  
保存一下，标题栏就有我们的信息了。  
  
about 里的信息找法和上面一样，找到  
  
0046A764    68 9B2C0000         push 2C9B  
0046A769    8D45 88             lea eax,dword ptr ss:\[ebp\-78\]  
0046A76C    50                  push eax  
0046A76D    B9 FF570000         mov ecx,57FF  
0046A772    BA A90A0000         mov edx,0AA9  
0046A777    B8 3CE94600         mov eax,MSQingSh.0046E93C  
0046A77C    E8 FBE4FFFF         call MSQingSh.00468C7C  
0046A781    8B55 88             mov edx,dword ptr ss:\[ebp\-78\]      ; 在这里 00BB207C  
0046A784    B8 8CB25600         mov eax,MSQingSh.0056B28C          ; 指针  
0046A789    E8 AA96F9FF         call MSQingSh.00403E38  
  
修改方法可以用上面的方法，但这里没有连接两个字符串，所以我们可以这样改  
  
0046A764    BA 20805900         mov edx,MSQingSh.00598020          ; 假设字符串存在 00598020  
0046A769    8955 88             mov dword ptr ss:\[ebp\-78\],edx      ; 保存一下，以防程序有校验  
0046A76C    EB 16               jmp short MSQingSh.0046A784        ; 跳到保存指针  
0046A76E    90                  nop  
0046A76F    90                  nop  
。。。。中间的 nop 省略      
0046A782    90                  nop  
0046A783    90                  nop  
0046A784    B8 8CB25600         mov eax,MSQingSh.0056B28C          ; 指针  
0046A789    E8 AA96F9FF         call MSQingSh.00403E38  
  
上面要连接两个字符串，这里不用，所以让程序帮我们把指针填好，保存，这样就 ok 了。（注意要从 字符串－8 偏移处开始复制）  
我还把 about 里图片给去掉了，大家自己去试试看，或者看我下面 SMC 代码。  
  
  
  
二、不脱壳打破解补丁  
1、原理：未脱壳的程序文件执行代码是动态生成的，所以不能用静态修改保存，自然想到 SMC(Self Modify Code)来打补丁，关于 SMC 可以参考《加密与解密》第二版和《软件加密技术内幕》中关于 SMC 的介绍，在这我们直接用例子说明  
  
2、在哪打补丁：补丁位置很显然放在壳把程序代码还原后，进入程序执行之前。对于压缩壳，最简单的地方就是在跳入 oep 时打补丁，对于这个程序，是用 upx 加壳的，跳入 Oep 的代码一般是  
popad  
jmp oep  
那补丁打在哪呢? 打在 popad 前面，因为 popad 恢复寄存器前 寄存器是随便我们用的，而壳代码又结束了，不影响程序正常执行。  
3、下面要注意的问题就是空间问题，用 od 载入，跳入 oep 附近的代码  
  
0064880A  ^\\EB E2                    jmp short MSQingSh.006487EE  
0064880C    61                       popad  
0064880D  - E9 BE84F1FF              jmp MSQingSh.00560CD0  
00648812    0000                     add byte ptr ds:\[eax\],al  
00648814    0000                     add byte ptr ds:\[eax\],al  
00648816    0000                     add byte ptr ds:\[eax\],al  
00648818    0000                     add byte ptr ds:\[eax\],al  
0064881A    0000                     add byte ptr ds:\[eax\],al  
  
用 loadPE 查一下 64880d 在文件中的偏移是 b0c0d，属于第二区段，看一下区段表中第三区段的文件中偏移 b0e00，剩余空间 1f3  
（这里可以用 nbw 的剩余空间查看器自动算，我找不到了，所以手动算一下），如果空间不够，可以在后面加一个区段  
  
然后我们把我们的代码插在  
jmp short MSQingSh.006487EE  
和  
popad  
之间。  
  
  
  
打补丁后的代码如下，我尽量把我写的代码的注释写详细点：  
  
0064880A  ^\\EB E2                    jmp short MSQingSh.006487EE  
0064880C    A2 F37F5C00              mov byte ptr ds:\[5C7FF3\],al                  ; 修改属性 5 个  
00648811    A2 FD7F5C00              mov byte ptr ds:\[5C7FFD\],al                  ; about 里图片 height=0,width=0  
00648816    C605 07805C00 08         mov byte ptr ds:\[5C8007\],8                   ; about 里图片 autosize=false  
0064881D    C605 61845C00 67         mov byte ptr ds:\[5C8461\],67                  ; about 里 label.left  
00648824    C605 68845C00 5A         mov byte ptr ds:\[5C8468\],5A                  ; about 里 label.top  
0064882B    C705 471B5600 E9410600   mov dword ptr ds:\[561B47\],641E9              ; 爆破点，两个  
00648835    C705 73235500 EB049090   mov dword ptr ds:\[552373\],909004EB  
0064883F    C705 64A74600 BA898864   mov dword ptr ds:\[46A764\],648889BA           ; 修改代码  
00648849    C705 68A74600 00895588   mov dword ptr ds:\[46A768\],88558900           ; 让 about 里有我们的信息  
00648853    C705 6CA74600 EB169090   mov dword ptr ds:\[46A76C\],909016EB  
0064885D    C705 16A44600 E9CBE41D   mov dword ptr ds:\[46A416\],1DE4CBE9           ; 修改代码，让程序跳到我们的代码，跳转要用 5 个字节，但考虑到最后一个字节是 00，和原来的程序一样，所以我们只要改前面 4 个就行了  
  
00648867    61                       popad                                          
00648868  - E9 6384F1FF              jmp MSQingSh.00560CD0  
0064886D    0000                     add byte ptr ds:\[eax\],al  
0064886F    0000                     add byte ptr ds:\[eax\],al  
00648871    0000                     add byte ptr ds:\[eax\],al  
00648873    0000                     add byte ptr ds:\[eax\],al  
00648875    0000                     add byte ptr ds:\[eax\],al  
00648877    0000                     add byte ptr ds:\[eax\],al  
00648879    0000                     add byte ptr ds:\[eax\],al  
0064887B    0000                     add byte ptr ds:\[eax\],al  
0064887D    0000                     add byte ptr ds:\[eax\],al  
0064887F    0000                     add byte ptr ds:\[eax\],al  
00648881    0100                     add dword ptr ds:\[eax\],eax                   ; 这里是我们保存的数据  
00648883    0000                     add byte ptr ds:\[eax\],al  
00648885    2000                     and byte ptr ds:\[eax\],al  
00648887    0000                     add byte ptr ds:\[eax\],al  
00648889    6352 61                  arpl word ptr ds:\[edx+61\],dx                 ; 第一个字符串（about 里的）  
0064888C    634B 45                  arpl word ptr ds:\[ebx+45\],cx  
0064888F    64:2062 59               and byte ptr fs:\[edx+59\],ah  
00648893    20BA A3B7E7D4            and byte ptr ds:\[edx+D4E7B7A3\],bh  
00648899    C2 D3B0                  retn 0B0D3  
0064889C    5B                       pop ebx  
0064889D    4E                       dec esi  
0064889E    45                       inc ebp  
0064889F    3336                     xor esi,dword ptr ds:\[esi\]  
006488A1    35 5D5B4446              xor eax,46445B5D  
006488A6    43                       inc ebx  
006488A7    47                       inc edi  
006488A8    5D                       pop ebp  
006488A9    0001                     add byte ptr ds:\[ecx\],al  
006488AB    0000                     add byte ptr ds:\[eax\],al  
006488AD    0032                     add byte ptr ds:\[edx\],dh  
006488AF    0000                     add byte ptr ds:\[eax\],al  
006488B1    00C4                     add ah,al                                    ; 第二个字符串（标题栏的）  
006488B3    A7                       cmpsd  
006488B4    CA F5C7                  retf 0C7F5  
006488B7  - E9 CAE92036              jmp 36857286  
006488BC    2E:34 39                 xor al,39  
006488BF    2020                     and byte ptr ds:\[eax\],ah  
006488C1    2D 20206352              sub eax,52632020  
006488C6    61                       popad  
006488C7    634B 45                  arpl word ptr ds:\[ebx+45\],cx  
006488CA    64:2062 59               and byte ptr fs:\[edx+59\],ah  
006488CE    20BA A3B7E7D4            and byte ptr ds:\[edx+D4E7B7A3\],bh  
006488D4    C2 D3B0                  retn 0B0D3  
006488D7    5B                       pop ebx  
006488D8    4E                       dec esi  
006488D9    45                       inc ebp  
006488DA    3336                     xor esi,dword ptr ds:\[esi\]  
006488DC    35 5D5B4446              xor eax,46445B5D  
006488E1    43                       inc ebx  
006488E2    47                       inc edi  
006488E3    5D                       pop ebp  
006488E4    0000                     add byte ptr ds:\[eax\],al  
006488E6    68 732C0000              push 2C73                                    ; 保存原来代码  
006488EB    C705 18B25600 B2886400   mov dword ptr ds:\[56B218\],MSQingSh.006488B2  ; 打补丁  
006488F5  - E9 211BE2FF              jmp MSQingSh.0046A41B                        ; 跳回去  
  
4、改完后保存一下，因为这是在壳上打补丁，所以不用考虑可读可写可执行的问题，下面就用 keymaker 做一下文件补丁就行了，这里我用的是 dREAMtHEATER 的 XCell(R) Version 1.0 做的补丁  
  
  
  
\--------------------------------------------------------------------------------  
【破解总结】  
  
  
后记：本文难度不大，主要是做一个抛砖引玉的作用，最近很忙，而且上不了网，我是破解的魔术情书 6.49 版，刚破完发现出来了 6.55 版，好在同一作者编程思路相同，所以 6.55 版破解的点和 6.49 一样，还有作者的其他些软件都差不多。这篇文章写好时 6.56 版又出来了。。。。。。 作者动作真快，大家用 6.56 版对照这看吧，代码基本都是一样的。  
本文只是暴力破解，对于判断是否注册的地方一直没找到，希望有高手能指点一二。  
\--------------------------------------------------------------------------------  
【爆破地址】  
  
  
00552373    FF92 D8000000  call dword ptr ds:\[edx+D8\]；nop 掉  
00561B47    0F84 F30C0000  je MSQingSh.00562840；改成 jmp 0056218D  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!  
  
2004-12-4   
  
本代码的着色效果由 xTiNt 自动完成  
下载 xTiNt [`211.90.75.84/web/kanaun/download/xTiNt.rar`](http://211.90.75.84/web/kanaun/download/xTiNt.rar) 

 4dd466d0e1e0cc123afbe504da2462ca