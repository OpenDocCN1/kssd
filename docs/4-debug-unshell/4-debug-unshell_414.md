

# "看"壳

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-535.htm`](https://www.pediy.com/kssd/pediy08/pediy8-535.htm)

> 大话西游里好象有那么一句话是什么“你不看我怎么知道我看你---------”，呵呵，人老了，记不清了，套用它在脱壳里  
> “我不看你怎么脱你”：），这个不是什么脱壳文章，只是“看看”而已。手痒了，写些垃圾，娱乐娱乐，就看谁被我“忽悠”了，：）  
> 俺的这个蛮罗嗦的，包涵，包涵 

> 

【破解作者】 layper  
【作者邮箱】 layper2002@yahoo.com.cn  
【作者主页】 www.sy135.com  
【使用工具】 peid,od，  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 壳乱弹(2)单步分析篇  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
呵呵，终于进入 2006 年了，祝大家新年快乐！！心想事成！！  
我又来侃大山了！！！！：）  
其实我写壳乱弹篇与篇之间没什么联系性，主要是想到什么就写什么！！看得懂就看一下，看不懂就算了。别当真！  
  
学习了一段时间脱壳后，我希望有所提高，所以就想着试试单步走分析一下壳，我觉得这是提高技术的一个好办法。  
  
我选择分析 ASProtect 1.2x - 1.3x 加壳的程序，开工。  
  
00401000 >  68 01C05A00             push CWoool.005AC001  
00401005    E8 01000000             call CWoool.0040100B  
0040100A    C3                      retn  
0040100B    C3                      retn  
  
OD 载入后我们看见这几句代码。这个壳一开始就利用这几句代码做跳板，跳到执行壳的地方，你查查 ALT+M  
内存映射，项目 32  
 地址=005AC000  
 大小=0002A000 (172032.)  
 物主=CWoool   00400000  
 区段=.cwoool  
 包含=imports,relocations  
 类型=Imag 01001002  
 访问=R  
 初始访问=RWE  
看见没有包含 imports，知道是干什么的了吧：）  
  
走过这里后，  
005AC001    60                      pushad  
005AC002    E8 03000000             call CWoool.005AC00A     ；F7 入  
005AC007  - E9 EB045D45             jmp 45B7C4F7  
005AC00C    55                      push ebp  
005AC00D    C3                      retn  
  
F7 走入之后  
005AC00A    5D                      pop ebp                            ; CWoool.005AC007  
005AC00B    45                      inc ebp  
005AC00C    55                      push ebp  
005AC00D    C3                      retn  
  
看见没有代码为之一变,呵呵，这个是扰乱静态分析的一个手段，估计可能是花指令，我不是高手，估计它原本是  
    call lyp  
    db e9  
    jmp next  
lyp:  
    pop  ebp  
    inc  ebp  
    push ebp  
    retn  
next:  
    ....  
大概是这个样子(可能不太正确)。其实一些花指令就是插入一些指令的机器码后经过组合扰乱分析的.  
005AC008   /EB 04                   jmp short CWoool.005AC00E  ;跳转  
005AC00A   |5D                      pop ebp  
005AC00B   |45                      inc ebp  
005AC00C   |55                      push ebp  
005AC00D   |C3                      retn  
005AC00E   \\E8 01000000             call CWoool.005AC014      ;F7 进  
005AC013   /EB 5D                   jmp short CWoool.005AC072  
005AC015   |BB EDFFFFFF             mov ebx,-13  
005AC01A   |03DD                    add ebx,ebp  
005AC01C   |81EB 00C01A00           sub ebx,1AC000  
005AC022   |807D 4D 01              cmp byte ptr ss:\[ebp+4D\],1  
  
  
代码改变  
005AC014    5D                      pop ebp                            ;EBP=005AC013  
005AC015    BB EDFFFFFF             mov ebx,-13                        ;EBX=-13  
005AC01A    03DD                    add ebx,ebp  
005AC01C    81EB 00C01A00           sub ebx,1AC000                     ;EBX=400000 "MZP"  
005AC022    807D 4D 01              cmp byte ptr ss:\[ebp+4D\],1  
005AC026    75 0C                   jnz short CWoool.005AC034  
  
可以看出,这个壳最喜欢插入 EB\\E9 即 jmp 来扰乱静态分析.把 0050AC001~0050AC0014 代码还原后可能是这样  
    pushad  
    call lyp  
    db e9  
    jmp lyp1  
lyp:  
    pop  ebp  
    inc  ebp  
    push ebp  
    retn  
lyp1:  
    call lyp2  
    db   e9  
lyp3:  
    pop ebp  
    mov ebx,-13  
    sub ebx,1AC000  
看出它是做什么没有!它应该是取文件头部的"MZP"的地址,由于壳要先取得程序的控制权,必须先定位到被加壳的  
PE 文件的文件头处,到这里为止完成了壳常规运做的第一步:  
  
  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

>   

【破解作者】 layper  
【作者邮箱】 layper2002@yahoo.com.cn  
【作者主页】 www.sy135.com  
【使用工具】 peid,od，  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 壳乱弹(3)单步分析  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
接着上次的分析,上次我们分析到得到一个地址文件头部的"MZP"的地址,有个兄弟仙剑太郎说这是壳解码，  
谢谢这位兄弟，虽然我的观点不太同，但是他也给我好的启发。原来我还不敢肯定接下来作什么，听了仙剑太郎的  
说法跟我想的差不多，我们继续往下看：  
  
005AC022    807D 4D 01              cmp byte ptr ss:\[ebp+4D\],1         ,ss:\[005AC060\]=00  
与 1 比较  
005AC026    75 0C                   jnz short CWoool.005AC034           ,不等则调,这里不等  
005AC028    8B7424 28               mov esi,dword ptr ss:\[esp+28\]    ;\[ESP+28\]=\[12FFCC\]=FFFFFFFF  
005AC02C    83FE 01                 cmp esi,1                                       ,与 1 比较  
005AC02F    895D 4E                 mov dword ptr ss:\[ebp+4E\],ebx     ;EBX=400000  
005AC032    75 31                   jnz short CWoool.005AC065  
  
前面产生了两个分支,而且两个分支都是与 1 比较,估计是返回值之类的东西，(别怪我老是估计,我还很菜)  
005AC034    8D45 53                 lea eax,dword ptr ss:\[ebp+53\]        ;\[EBP+53\]=5AC0066  
005AC037    50                      push eax  
005AC038    53                      push ebx                                           ;ebx=00400000 (CWoool.00400000), ASCII "MZP"  
005AC039    FFB5 ED090000           push dword ptr ss:\[ebp+9ED\] ;ss:\[005ACA00\]=7C80B529 (kernel32.GetModuleHandleA)哈哈,终于出来了  
005AC03F    8D45 35                 lea eax,dword ptr ss:\[ebp+35\]      ;指向 005AC048  
005AC042    50                      push eax                           ;压入堆栈  
005AC043    E9 82000000             jmp CWoool.005AC0CA  
  
005AC0CA    8BF3                    mov esi,ebx                     ; CWoool.00400000  
005AC0CC    E8 08000000             call CWoool.005AC0D9            ;f7 跟进  
  
005AC0D9    50                      push eax                        ;005AC048  
005AC0DA    5F                      pop edi                         ; CWoool.005AC048,这两行代码也是常见的，利用堆栈传递参数  
005AC0DB    5A                      pop edx                         ;把 call 的返回地址出栈  
005AC0DC    8BFE                    mov edi,esi                     ;esi=00400000 (CWoool.00400000), ASCII "MZP"  
                                                                    ;edi=005AC048 (CWoool.005AC048)  
                                                                    ;奇怪,为什么传递参数没有用到,看来有可能是沉余代码  
005AC0DE    81C2 5C080000           add edx,85C                     ;edx=005AC0D1 (CWoool.005AC0D1)+85c=005AC92D,到 005AC92D 看看,后面全部是  
                                                                    ;零,看来 005AC92D 是一段代码的边界了.  
005AC0E4    0F8D 00000000           jge CWoool.005AC0EA             ;那么近也跳,垃圾  
005AC0EA    68 00000000             push 0  
005AC0EF    81E8 FC346B1E           sub eax,1E6B34FC                ;原 eax=005AC048,这个地址减去一个数值,运算后 eax=e1ef8b4c  
005AC0F5    5B                      pop ebx                         ;ebx 清零  
005AC0F6    66:81D6 A649            adc si,49A6                     ;注意这里,esi 指向了 004049A7  
005AC0FB    FF3413                  push dword ptr ds:\[ebx+edx\]     ;第一次取 005AC92D 的数据,注意前面已经提醒这个是一个边界  
005AC0FE    51                      push ecx                        ;ecx=0012FFB0  
005AC0FF    68 39A5C040             push 40C0A539                   ;又一个特殊数值  
005AC104    81F7 FB2FA22D           xor edi,2DA22FFB                ;edi=00400000 , ASCII "MZP" DOS 文件头地址异或  
005AC10A    5F                      pop edi                         ;40C0A539 传递给 EDI,奇怪,又是上一步好象又是垃圾  
005AC10B    5E                      pop esi                         ;esi 指向指针堆栈 \[0012FF8C\]=0012FFB0  
005AC10C    59                      pop ecx                         ;哈哈,看看这个是 005AC92D 中的数据  
005AC10D    66:B8 EB0D              mov ax,0DEB                     ;005AC048 运算结果改动低位为 E1EF0DEB  
005AC111    81E9 6CDCC50C           sub ecx,0CC5DC6C                ;005AC92D 的数据减去 0CC5DC6C=0212EC40  
005AC117    B8 C79DDA2F             mov eax,2FDA9DC7                ;EAX=2FDA9DC7 修改了 EAX 的值  
005AC11C    81F1 35F2EF20           xor ecx,20EFF235                ;005AC92D 的数据再次运算(异或)=22FD1E75  
005AC122    0F89 03000000           jns CWoool.005AC12B             ;SF=0 跳走  
005AC128    0FBFF6                  movsx esi,si      
005AC12B    81E9 CA76F46B           sub ecx,6BF476CA                ;005AC92D 的数据再次运算=B708A7AB  
005AC131    890C1A                  mov dword ptr ds:\[edx+ebx\],ecx  ;放回 005AC92D  
005AC134    E8 0C000000             call CWoool.005AC145            ;跟进  
  
005AC145    E8 0B000000             call CWoool.005AC155            ;跟进  
  
005AC155    80C0 16                 add al,16                       ;2FDA9DC7 低位加 16  
005AC158    5F                      pop edi                         ;返回地址出栈堆栈 \[0012FF8C\]=005AC14A (CWoool.005AC14A)  
005AC159    58                      pop eax                         ;再往上出栈堆栈 \[0012FF90\]=005AC139 (CWoool.005AC139)  
005AC15A    E8 08000000             call CWoool.005AC167  
005AC15F    F0:69EE 8F1C25FA        lock imul ebp,esi,FA251C8F      ; 不允许锁定前缀  
005AC166    AB                      stos dword ptr es:\[edi\]  
  
005AC167    BE 87E9C62D             mov esi,2DC6E987                ;ESI=2DC6E987  
005AC16C    58                      pop eax                         ;返回地址出栈 EAX=005AC15F  
005AC16D    83EB 04                 sub ebx,4                       ;EBX-4  
005AC170    80D0 4C                 adc al,4C                       ;005AC15F ADC 4C  
005AC173    81FB 7CF8FFFF           cmp ebx,-784                    ;EBX 为计数器  
005AC179    0F85 14000000           jnz CWoool.005AC193             ;为算完往下跳  
005AC17F    0FB7F2                  movzx esi,dx                    ;ESI 低字节为 005AC92D 的低字节  
005AC182    E9 26000000             jmp CWoool.005AC1AD             ;循环出口  
  
005AC193    0FB7C3                  movzx eax,bx                   ;  
005AC196  ^ E9 60FFFFFF             jmp CWoool.005AC0FB            ;往回跳  
  
看到这里我们就知道是一个算法循环,我这样叙述:  
      
    以 005AC92D 为起始地址,往低每隔四个字节取地址数据进行运算,先是减去 0CC5DC6C,然后与 20EFF235 异或,接着减去  
6BF476CA 所得的值放回原处,如此循环直到地址 005AC1AD 为止!  
  
继续往下看:  
005AC1AD    66:81C2 C5D5            add dx,0D5C5                  ;EDX=5A9EF2  
005AC1B2    E8 0F000000             call CWoool.005AC1C6          ;跟进  
  
005AC1C6    68 58BB343C             push 3C34BB58                 ;3C34BB58 入栈  
005AC1CB    E9 0C000000             jmp CWoool.005AC1DC           ;跳转  
005AC1D0    04 ED                   add al,0ED  
005AC1D2    22B3 70E96E0F           and dh,byte ptr ds:\[ebx+F6EE970>  
005AC1D8    9C                      pushfd  
005AC1D9    A5                      movs dword ptr es:\[edi\],dword p>  
005AC1DA    7A 2B                   jpe short CWoool.005AC207  
005AC1DC    59                      pop ecx                       ;ECX=3C34BB58  
005AC1DD    58                      pop eax                       ;返回地址出栈堆栈 \[0012FF90\]=005AC1B7 (CWoool.005AC1B7)  
005AC1DE    E8 0C000000             call CWoool.005AC1EF          ;跟进  
005AC1E3    34 5D                   xor al,5D  
005AC1E5    D2A3 A0591EFF           shl byte ptr ds:\[ebx+FF1E59A0\],>  
  
005AC1EF   /E9 0B000000             jmp CWoool.005AC1FF           ;跳转  
005AC1F4   |F764CD 82               mul dword ptr ss:\[ebp+ecx\*8-7E\]  
  
005AC1FF    5E                      pop esi                         ; 返回地址出栈堆栈 \[0012FF90\]=005AC1E3 (CWoool.005AC1E3)  
005AC200    81C0 75070000           add eax,775                     ;005AC1B7+775=005AC92C  
005AC206    66:81D9 940F            sbb cx,0F94                     ;3C34BB58 低位减 0F94  
005AC20B    2BFF                    sub edi,edi                     ;EDI 清零  
005AC20D    BE 39EDC518             mov esi,18C5ED39                ;ESI=18C5ED39  
005AC212    8B1C38                  mov ebx,dword ptr ds:\[eax+edi\]  ;ds:\[005AC92C\]=08A7AB8A  
005AC215    66:81D2 FB37            adc dx,37FB                     ;地址 005A9EF2 ADC 37FB =005AD6ED  
005AC21A    81C3 F860357D           add ebx,7D3560F8                ;EBX=ds:\[005AC92C\]-7D3560F8=85DD0C82  
005AC220    66:8BF0                 mov si,ax                       ;005AC92C 低位入 si  
005AC223    81F3 D109160B           xor ebx,0B1609D1                ;EBX=(ds:\[005AC92C\]-7D3560F8) XOR 0B1609D1=8ECB0053  
005AC229    81EB 360C0618           sub ebx,18060C36                ;EBX=((ds:\[005AC92C\]-7D3560F8) XOR 0B1609D1)-18060C36=76C4F91D  
005AC22F    8ACE                    mov cl,dh                       ;FB 放入 cl  
005AC231    891C38                  mov dword ptr ds:\[eax+edi\],ebx  ;计算值放回原处 005AC92C  
005AC234    E9 0C000000             jmp CWoool.005AC245             ;跳  
005AC239    F4                      hlt  
005AC23A    1D 92636019             sbb eax,19606392  
005AC23F    DEBF 8CD5EADB           fidivr word ptr ds:\[edi+DBEAD58C\]  
005AC245    0FB7F2                  movzx esi,dx                    ;dx=D6ED  
005AC248    83EF 04                 sub edi,4                       ;D6ED-4  
005AC24B    0FB7F3                  movzx esi,bx  
005AC24E    81FF 60F9FFFF           cmp edi,-6A0                    ;EDI 为计数器  
005AC254    0F85 13000000           jnz CWoool.005AC26D             ;不等则跳  
005AC25A    B9 A713045D             mov ecx,5D0413A7                ;ECX=5D0413A7  
005AC25F    E9 2C000000             jmp CWoool.005AC290             ;出口  
  
005AC26D    68 D804010D             push 0D0104D8                   ;0D0104D8  
005AC272    66:BA F096              mov dx,96F0                     ;DX=96F0  
005AC276    5E                      pop esi                         ;ESI=0D0104D8  
005AC277  ^ E9 96FFFFFF             jmp CWoool.005AC212             ;往回跳  
  
这里也是一个循环:  
  
    以 005AC92C 为起始地址,往低每隔四个字节取地址数据进行运算,按以下运算  
  
((ds:\[005AC92C\]-7D3560F8) XOR 0B1609D1)-18060C36  
  
所得的值放回原处,如此循环直到地址 005AC290 为止!  
  
注意 005AC215 和 005AC21A 时 EDX=005ACEEB ASCII "iantChangeTypeEx",呵呵,这个有问题啊,我们分析一下他怎么来的,  
我们用回溯法:  
  
005AC272    66:BA F096              mov dx,96F0                     ;DX=96F0  
005AC215    66:81D2 FB37            adc dx,37FB                     ;  
005AC1AD    66:81C2 C5D5            add dx,0D5C5                  ;EDX=5A9EF2  
005AC0DE    81C2 5C080000           add edx,85C                     ;edx=005AC0D1 (CWoool.005AC0D1)+85c=005AC92D,到 005AC92D 看看,后面全部是  
                                                                    ;零,看来 005AC92D 是一段代码的边界了.  
看到没有这个 iantChangeTypeEx 事先准备好后经过找到他的!  
  
继续分析：  
005AC290    8BF2                    mov esi,edx                       ;edx=005ACEEB ASCII "iantChangeTypeEx"  
005AC292    E8 12000000             call CWoool.005AC2A9              ;跟进  
005AC297    22B3 70E96E0F           and dh,byte ptr ds:\[ebx+F6EE970\]  
005AC29D    9C                      pushfd  
  
005AC2A9    5A                      pop edx                           ;返回地址出栈 EDX=0059C297  
005AC2AA    E8 10000000             call CWoool.005AC2BF              ;跟进  
005AC2AF    B8 91F6F764             mov eax,64F7F691  
005AC2B4    CD 82                   int 82  
005AC2B6    93                      xchg eax,ebx  
005AC2B7    D0C9                    ror cl,1  
005AC2B9    CE                      into  
005AC2BA    EF                      out dx,eax  
005AC2BB    FC                      cld  
005AC2BC    85DA                    test edx,ebx  
005AC2BE    0B5F 81                 or ebx,dword ptr ds:\[edi-7F\]  
005AC2C1    C2 9506                 retn 695  
  
005AC2BF    5F                      pop edi                           ; 返回地址出栈堆栈 \[0012FF90\]=005AC2AF  
005AC2C0    81C2 95060000           add edx,695                       ;EDX=0059C297+695=0059C92C  
005AC2C6    BE F5B53274             mov esi,7432B5F5                  ;ESI=7432B5F5  
005AC2CB    BB 00000000             mov ebx,0                         ;EBX=0  
005AC2D0    E9 06000000             jmp CWoool.005AC2DB  
  
005AC2DB    8B041A                  mov eax,dword ptr ds:\[edx+ebx\]    ;又来了,肯定又是算法部分\[0059C92C\]=76C4F91D  
005AC2DE    0F8D 17000000           jge CWoool.005AC2FB               ;跳走  
  
005AC2FB    81E8 BE7EF45B           sub eax,5BF47EBE                  ;\[0059C92C\]-5BF47EBE=1AD07A5F  
005AC301    81E8 1F0BAC6A           sub eax,6AAC0B1F                  ;\[0059C92C\]-5BF47EBE-6AAC0B1F=B0246F40  
005AC307    66:8BF2                 mov si,dx                         ;SI=C92C,ESI=7432C92C  
005AC30A    81C0 6C5E9E4F           add eax,4F9E5E6C                  ;\[0059C92C\]-5BF47EBE-6AAC0B1F+4F9E5E6C=FFC2CDAC  
005AC310    66:8BCF                 mov cx,di                         ;CX=C2AF  
005AC313    50                      push eax                          ;FFC2CDAC 入栈  
005AC314    68 66FEC94B             push 4BC9FE66                     ;4BC9FE66 入栈  
005AC319    80C5 C0                 add ch,0C0                        ;5D0482AF  
005AC31C    5F                      pop edi                           ;EDI=4BC9FE66  
005AC31D    8F041A                  pop dword ptr ds:\[edx+ebx\]        ;换一种方法把\[0059C92C\]-5BF47EBE-6AAC0B1F+4F9E5E6C 传回去  
005AC320    66:8BF9                 mov di,cx                         ;cx=82AF,di=FE66  
005AC323    0F8E 08000000           jle CWoool.005AC331               ;  
005AC329    68 97364C1D             push 1D4C3697  
005AC32E    B1 A5                   mov cl,0A5  
005AC330    5E                      pop esi  
005AC331    83EB 01                 sub ebx,1  
005AC334    4B                      dec ebx  
005AC335    4B                      dec ebx  
005AC336    4B                      dec ebx                           ;EBX=-4,其实就是上面的四个字节的变形  
005AC337    E9 0C000000             jmp CWoool.005AC348               ;这里是出口  
  
005AC348    81FB 24FAFFFF           cmp ebx,-5DC                      ;比较是否为-5DC  
005AC34E  ^ 0F85 87FFFFFF           jnz CWoool.005AC2DB               ;往回跳  
005AC354    59                      pop ecx  
005AC355    3102                    xor dword ptr ds:\[edx\],eax  
005AC357    77 71                   ja short CWoool.005AC3CA  
005AC359    E4 00                   in al,0  
005AC35B    D7                      xlat byte ptr ds:\[ebx+al\]  
005AC35C    1D A1683647             sbb eax,473668A1  
  
  
第三个算法是非常 good 的部分:  
这个算法的起始地址为 0059C92C,而结束地址为 0059C354,呵呵,结束时正好紧接在我们这个算法之后,假如运算完变成了  
  
  
005AC354    E8 05000000             call CWoool.005AC35E  
005AC359    B9 FE5FAC75             mov ecx,75AC5FFE  
005AC35E    66:BF D6AA              mov di,0AAD6  
005AC362    5E                      pop esi  
005AC363    B3 9E                   mov bl,9E  
005AC365    81C6 D2050000           add esi,5D2  
005AC36B    B9 6B767804             mov ecx,478766B  
005AC370    81E9 21757804           sub ecx,4787521  
005AC376    B0 A7                   mov al,0A7  
005AC378    FF36                    push dword ptr ds:\[esi\]  
  
知道这三个算法是什么了吗?就是解码,所以说那位兄弟说这三个算法之前的是解码部分,呵呵,应该是这里.总结第三个  
算法为:  
\[0059C92C\]-5BF47EBE-6AAC0B1F+4F9E5E6C  
  
这部分的总结:  
  
这部分利用了 SMC 技术加密代码后,再在壳中解码.(呵呵,不知下面还有没有)  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

【破解作者】 layper  
【作者邮箱】 layper2002@yahoo.com.cn  
【作者主页】 www.sy135.com  
【使用工具】 peid,od，  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 壳乱弹(4)单步分析  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
我们继续分析 ASProtect 1.2x - 1.3x 壳,经过三个算法解码之后,005AC354 变成了以下这个样子  
  
005AC354    E8 05000000             call CWoool.005AC35E          ;F7 跟进  
005AC359    B9 FE5FAC75             mov ecx,75AC5FFE  
005AC35E    66:BF D6AA              mov di,0AAD6                        ;EDI=4BC9AAD6  
005AC362    5E                      pop esi                                           ;返回地址出栈,  
005AC363    B3 9E                   mov bl,9E                                   ;EBX=FFFFFA9E  
005AC365    81C6 D2050000           add esi,5D2                       ;指向 005AC359+5D2=5AC92B  
005AC36B    B9 6B767804             mov ecx,478766B               ;ECX=478766B  
005AC370    81E9 21757804           sub ecx,4787521               ;ECX=478766B-4787521=14A  
005AC376    B0 A7                   mov al,0A7                                ;EAX=000005A7  
005AC378    FF36                    push dword ptr ds:\[esi\]               ;ds:\[005AC92B\]=C2CDACD5  
005AC37A    50                      push eax                                     ;EAX=000005A7  
005AC37B    BB 6A63D365             mov ebx,65D3636A          ;EBX=65D3636A  
005AC380    5F                      pop edi                                        ;EDI=000005A7  
005AC381    5A                      pop edx                                      ;EDX=ds:\[005AC92B\]=C2CDACD5  
005AC382    66:BF C2F5              mov di,0F5C2                       ;EDI=0000F5C2  
005AC386    81EA 055CEE21           sub edx,21EE5C05         ;EDX=ds:\[005AC92B\]-21EE5C05=A0DF50D0  
005AC38C    E8 0E000000             call CWoool.005AC39F     ;F7 跟进  
005AC391    0E                      push cs  
005AC392    2F                      das  
005AC393    3C C5                   cmp al,0C5  
005AC395    1A4B 28                 sbb cl,byte ptr ds:\[ebx+28\]  
005AC398    41                      inc ecx  
005AC399    E6 27                   out 27,al  
005AC39B    D4 7D                   aam 7D  
005AC39D  ^ 72 C3                   jb short CWoool.005AC362  
005AC39F    66:B8 1F89              mov ax,891F                        ;EAX=891F  
005AC3A3    5B                      pop ebx                                     ;堆栈 \[0012FF90\]=005AC391 (CWoool.005AC391)  
005AC3A4    81EA 5AE46403           sub edx,364E45A           ;EDX=ds:\[005AC92B\]-21EE5C05-364E45A=9D7A6C76  
005AC3AA    E9 0D000000             jmp CWoool.005AC3BC    ;跳走  
005AC3AF    B1 96                   mov cl,96     
005AC3B1    17                      pop ss  
005AC3B2    04 ED                   add al,0ED  
005AC3B4    22B3 70E96E0F           and dh,byte ptr ds:\[ebx+F6EE970\]  
005AC3BA    9C                      pushfd  
005AC3BB    A5                      movs dword ptr es:\[edi\],dword ptr>  
005AC3BC    81C2 8BD62427           add edx,2724D68B              ;EDX=ds:\[005AC92B\]-21EE5C05-364E45A+2724D68B=C49F4301  
005AC3C2    81C8 4618CF13           or eax,13CF1846                  ;891F XOR 13CF1846=13CF995F  
005AC3C8    52                      push edx                                         ;计算结果入栈  
005AC3C9    E8 07000000             call CWoool.005AC3D5          ;F7 跟进  
005AC3CE    A3 A0591EFF             mov dword ptr ds:\[FF1E59A0\],eax  
005AC3D3    CC                      int3  
005AC3D4    15 0F850500             adc eax,5850F  
005AC3D9    0000                    add byte ptr ds:\[eax\],al  
005AC3D5   /0F85 05000000           jnz CWoool.005AC3E0    
005AC3DB   |66:81EF C9BF            sub di,0BFC9  
005AC3E0   \\5F                      pop edi                                 ;返回地址果出栈到 EDI  
005AC3E1    8F06                    pop dword ptr ds:\[esi\]         ;计算结果放回原处  
005AC3E3    83EE 04                 sub esi,4                          ;遗向下一个地址  
005AC3E6    E8 08000000             call CWoool.005AC3F3    ;F7 跟进  
005AC3EB    0039                    add byte ptr ds:\[ecx\],bh         ;  
005AC3ED  ^ 7E DF                   jle short CWoool.005AC3CE  
005AC3EF    2C F5                   sub al,0F5  
005AC3F1    8AFB                    mov bh,bl  
005AC3F3    0FB7FF                  movzx edi,di                      ;到这里 EDI=C3F3  
005AC3F6    58                      pop eax                                 ;返回地址出栈  
005AC3F7    81E9 01000000           sub ecx,1                    ;ECX=14A,ECX 为计数器  
005AC3FD  ^ 0F85 75FFFFFF           jnz CWoool.005AC378  
  
算法如下:  
从 005AC92B 到 5AC407  
EDX=ds:\[005AC92B\]-21EE5C05-364E45A+2724D68B  
  
005AC403    66:BB 5C09              mov bx,95C                    ;指向 5AC095C  
005AC407    E8 00000000             call CWoool.005AC40C          ;F7  
005AC40C    5D                      pop ebp                       ;EBP=005AC40C  
005AC40D    5B                      pop ebx                       ;005AC048  
005AC40E    895D 5B                 mov dword ptr ss:\[ebp+5B\],ebx ;\[005AC467\]=005AC048  
005AC411    5B                      pop ebx                       ;堆栈 \[0012FF98\]=7C80B529 (kernel32.GetModuleHandleA)  
005AC412    895D 5F                 mov dword ptr ss:\[ebp+5F\],ebx ;\[005AC46B\]=7C80B529 (kernel32.GetModuleHandleA)  
005AC415    58                      pop eax                       ;堆栈 \[0012FF9C\]=00400000 (CWoool.00400000), ASCII "MZP"  
005AC416    8985 0D040000           mov dword ptr ss:\[ebp+40D\],eax ;SS:\[005AC819\]=00400000 (CWoool.00400000), ASCII "MZP"  
005AC41C    58                      pop eax                       ;堆栈 \[0012FFA0\]=005AC066 (CWoool.005AC066)  
005AC41D    807D 5A 01              cmp byte ptr ss:\[ebp+5A\],1    ;比较 ss:\[005AC466\]=00 是否是 0  
005AC421    75 59                   jnz short CWoool.005AC47C     ;不是则跳,我们是跳走  
005AC423    8985 25040000           mov dword ptr ss:\[ebp+425\],eax  
005AC429    60                      pushad  
005AC42A    8D45 33                 lea eax,dword ptr ss:\[ebp+33\]  
005AC42D    50                      push eax  
005AC42E    33C0                    xor eax,eax  
005AC430    64:FF30                 push dword ptr fs:\[eax\]       ;用到 FS,可能是异常处理  
005AC433    64:8920                 mov dword ptr fs:\[eax\],esp  
005AC436    8BC3                    mov eax,ebx  
005AC438    E8 E4020000             call CWoool.005AC721  
005AC43D    EB 1C                   jmp short CWoool.005AC45B  
  
005AC47C    E8 9C020000             call CWoool.005AC71D         ;跳到这里,F7 跟进  
  
005AC71D    8B4424 24               mov eax,dword ptr ss:\[esp+24\]     ; kernel32.7C816D4F  
005AC721    25 0000FFFF             and eax,FFFF0000                  ;EAX AND FFFF0000 =7C810000  
005AC726    05 00000100             add eax,10000                     ; UNICODE "=::=::\\"  
005AC72B    2D 00000100             sub eax,10000                     ; UNICODE "=::=::\\"  
005AC730    66:8138 4D5A            cmp word ptr ds:\[eax\],5A4D        ;ds:\[7C810000\]=0109 比较是否是"ZM"  
005AC735  ^ 75 F4                   jnz short CWoool.005AC72B         ;不是则跳回,这三步就是寻找"ZM",最后文件头是在 7C8000  
005AC737    60                      pushad                            ;寄存器值全部压入栈  
005AC738    8985 F8030000           mov dword ptr ss:\[ebp+3F8\],eax    ;eax=7C800000 (kernel32.7C800000) ss:\[005AC804\]=00000000 保存 kernel32 文件头地址  
005AC73E    8BD0                    mov edx,eax                       ;EDX=7C800000 (kernel32.7C800000)  
005AC740    8BD8                    mov ebx,eax                       ;EBX=7C800000 (kernel32.7C800000)  
005AC742    0340 3C                 add eax,dword ptr ds:\[eax+3C\]     ;指向 7C8000C3,文件头"PE"地址  
005AC745    0358 78                 add ebx,dword ptr ds:\[eax+78\]     ;指向 ds:\[7C800160\]=0000262C,输出表偏移  
005AC748    899D 0D030000           mov dword ptr ss:\[ebp+30D\],ebx    ;保存输出表偏移在 005AC719 处  
005AC74E    8D9D CC030000           lea ebx,dword ptr ss:\[ebp+3CC\]    ;EBX 指向 005AC7D8  
005AC754    8DBD E4030000           lea edi,dword ptr ss:\[ebp+3E4\]    ;EDI 指向 005AC7F0  
005AC75A    8B33                    mov esi,dword ptr ds:\[ebx\]        ;取 005AC7D8 的数据=B72551A7  
005AC75C    89B5 7C030000           mov dword ptr ss:\[ebp+37C\],esi    ;esi=B72551A7,ss:\[005AC788\]=12345678 奇怪，竟然是 12345678  
005AC762    E8 0B000000             call CWoool.005AC772              ;F7 进  
005AC767    AB                      stos dword ptr es:\[edi\]           ;把 AX(字)中的数据存储到 DI 为目的串地址指针所寻址的存储器单元中去。  
005AC768    83C3 04                 add ebx,4  
005AC76B    833B 00                 cmp dword ptr ds:\[ebx\],0  
005AC76E  ^ 75 EA                   jnz short CWoool.005AC75A  
005AC770    61                      popad  
005AC771    C3                      retn                              ;返回 005AC481  
  
005AC772    60                      pushad                            ;来到这里  
005AC773    8B9D 0D030000           mov ebx,dword ptr ss:\[ebp+30D\]    ;输出表地址出栈  
005AC779    8B4B 20                 mov ecx,dword ptr ds:\[ebx+20\]     ;指向输出表函数名地址表的 rva=ds:\[7C80264C\]=00003528  
005AC77C    03CA                    add ecx,edx                       ;指向函数名地址表 7c803528  
005AC77E    8B31                    mov esi,dword ptr ds:\[ecx\]        ;取函数名偏移  
005AC780    03F2                    add esi,edx                       ;指向函数名 edx=7C804B73  
005AC782    E8 2F000000             call CWoool.005AC7B6              ;跟进  
  
005AC7B6    52                      push edx                           ;到这里,保存"ZM"文件头地址  
005AC7B7    BA 8E243B9C             mov edx,9C3B248E                   ;EDX=9C3B248E  
005AC7BC    AC                      lods byte ptr ds:\[esi\]             ;送字符串字节到 AL  
005AC7BD    0AC0                    or al,al                           ;是否是 0  
005AC7BF    74 14                   je short CWoool.005AC7D5           ;是则跳走  
005AC7C1    32D0                    xor dl,al                          ;与 DL 异或后放回 DL,EDX=9C3B24CF  
005AC7C3    B0 08                   mov al,8                           ;al=8  
005AC7C5    D1EA                    shr edx,1                          ;逻辑右移一位,EDX=4E1D9267  
005AC7C7    73 06                   jnb short CWoool.005AC7CF          ;不低于则跳  
005AC7C9    81F2 9AF3A7C1           xor edx,C1A7F39A                   ;再次异或,8FBA61FD  
005AC7CF    FEC8                    dec al                             ;AL 减 1  
005AC7D1  ^ 75 F2                   jnz short CWoool.005AC7C5          ;循环 8 次  
005AC7D3  ^ EB E7                   jmp short CWoool.005AC7BC          ;  
005AC7D5    92                      xchg eax,edx                       ;计算结果与文件头地址交换  
005AC7D6    5A                      pop edx                            ;保证是文件头地址  
005AC7D7    C3                      retn                               ;返回 005AC787  
  
005AC7B6 到 005AC7D7 的算法叙述如下:  
1)取函数地址名的一位字符串  
2)与 EDX(初始为 9C3B248E)的 DL 数据异或  
3)EDX 逻辑左移一位  
4)不低于的话在左移一位  
5)低于的话 EDX 在与 C1A7F39A 异或,结果放回 EDX  
6)3--5 步总共循环八次  
7)完之后跳向函数名的下一位  
  
005AC77E    8B31                    mov esi,dword ptr ds:\[ecx\]        ;取函数名偏移  
005AC780    03F2                    add esi,edx                       ;指向函数名 edx=7C804B73  
005AC782    E8 2F000000             call CWoool.005AC7B6              ;跟进  
005AC787    BF A75125B7             mov edi,B72551A7                ;跳回这里,EDI=B72551A7，这个数据随着 005AC75C 的 esi 变化，而变化。  
005AC78C    3BC7                    cmp eax,edi                     ;比较刚才的结果是否是 B72551A7  
005AC78E    74 05                   je short CWoool.005AC795        ;是则往下  
005AC790    83C1 04                 add ecx,4                        ;不是则移向下一个保存函数名地址的偏移地址  
005AC793  ^ EB E9                   jmp short CWoool.005AC77E        ;跳回继续  
005AC795    2B4B 20                 sub ecx,dword ptr ds:\[ebx+20\]   
005AC798    2BCA                    sub ecx,edx                      ;这两步得出函数名 rva 相对于第一个函数地址表的位置  
005AC79A    D1E9                    shr ecx,1                        ;逻辑左移一位  
005AC79C    034B 24                 add ecx,dword ptr ds:\[ebx+24\]    ;\[ebx+24\]指向函数名序号表的起始 RVA=ds:\[7C802650\]=000043FC,这一步加法运算某个函数的 RVA  
005AC79F    03CA                    add ecx,edx                      ;获得函数序号数组索引值  
005AC7A1    0FB709                  movzx ecx,word ptr ds:\[ecx\]      ;取相对于起始项的偏移  
005AC7A4    C1E1 02                 shl ecx,2  
005AC7A7    034B 1C                 add ecx,dword ptr ds:\[ebx+1C\]    ;获得函数地址表中的某项的偏移  
005AC7AA    03CA                    add ecx,edx                      ;指向那项的地址  
005AC7AC    8B09                    mov ecx,dword ptr ds:\[ecx\]       ;取函数入口 RVA  
005AC7AE    03CA                    add ecx,edx                      ;函数入口地址  
005AC7B0    894C24 1C               mov dword ptr ss:\[esp+1C\],ecx    ;保存函数入口地址  
005AC7B4    61                      popad  
005AC7B5    C3                      retn  
  
其实 005AC77E 到 005AC793 这几步是找函数名,这里是获取几个函数的入口，上面这一部分，总共在 Kernel32.dll 中得到了  
以下这几个函数入口：  
005AC7F0  7C80AC28  kernel32.GetProcAddress  
005AC7F4  7C80B529  kernel32.GetModuleHandleA  
005AC7F8  7C801D77  kernel32.LoadLibraryA  
005AC7FC  7C809A81  kernel32.VirtualAlloc  
005AC800  7C809B14  kernel32.VirtualFree  
  
  
005AC481    FC                      cld                            ;DF 置 0  
005AC482    8DB5 8C000000           lea esi,dword ptr ss:\[ebp+8C\]  ;esi=005AC498,  
005AC488    AD                      lods dword ptr ds:\[esi\]        ;把 SI 寻址的源串的数据字节送 AL 或数据字送 AX 中去,  
                                                                   ;EAX=ds:\[esi\]=\[005AC498\]=00401000,看来是原程序的 code 段的起始地址  
005AC489    0BC0                    or eax,eax                     ;是否是 0  
005AC48B    74 1B                   je short CWoool.005AC4A8  
005AC48D    8BF8                    mov edi,eax                    ;EDI=EAX=00401000  
005AC48F    B9 0C000000             mov ecx,0C                     ;ECX=0C  
005AC494    F3:A4                   rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\] ; 字符串传送,CX 为次数,修改了 00401000 至 0040100C 的数据  
005AC496    EB 10                   jmp short CWoool.005AC4A8  
  
005AC4A8    89A5 29040000           mov dword ptr ss:\[ebp+429\],esp    
005AC4AE    6A 40                   push 40                         ;设置提交的物理空间为可读\\可写\\可执行属性  
005AC4B0    68 00100000             push 1000                       ;为指定的地址空间提交物理内存  
005AC4B5    FFB5 08040000           push dword ptr ss:\[ebp+408\]     ;大小为 4300  
005AC4BB    6A 00                   push 0                          ;自行保留  
005AC4BD    FF95 F0030000           call dword ptr ss:\[ebp+3F0\]     ;ss:\[005AC7FC\]=7C809A81 (kernel32.VirtualAlloc),保留或提交一段地址空间  
                                                                    ;0012FF90   005AC4C3     /CALL 到 VirtualAlloc 来自 CWoool.005AC4BD  
                                                                    ;|Address = NULL  
                                                                    ;|Size = 43000 (274432.)  
                                                                    ;00001000     |AllocationType = MEM\_COMMIT  
                                                                    ;00000040     \\Protect = PAGE\_EXECUTE\_READWRITE  
005AC4C3    8985 CC010000           mov dword ptr ss:\[ebp+1CC\],eax  ;保存提交地址中的第一页的起始线程地址 00F50000  
005AC4C9    8B9D 00040000           mov ebx,dword ptr ss:\[ebp+400\]  ;ss:\[005AC80C\]=001AF7D4  
005AC4CF    039D 0D040000           add ebx,dword ptr ss:\[ebp+40D\]  ;ss:\[005AC819\]=00400000 (CWoool.00400000), ASCII "MZP",原程序的文件头加上 001AF7D4=005AF7D4  
005AC4D5    50                      push eax                        ;保存地址  
005AC4D6    53                      push ebx                        ;保存地址  
005AC4D7    E8 04010000             call CWoool.005AC5E0            ;跟进  
  
005AC5E0    55                      push ebp                           ; CWoool.005AC40C  
005AC5E1    8BEC                    mov ebp,esp  
005AC5E3    60                      pushad                          ;所有寄存器值压入栈  
005AC5E4    55                      push ebp                       ;EBP=12FFA94  
005AC5E5    8B75 08                 mov esi,dword ptr ss:\[ebp+8\]   ;ESI=文件头+001AF7D4 地址=005AF7D4  
005AC5E8    8B7D 0C                 mov edi,dword ptr ss:\[ebp+C\]   ;EDI=分配的内存地址=00F50000  
005AC5EB    FC                      cld                            ;DF 置 0  
005AC5EC    B2 80                   mov dl,80                      ;EDX=7C92EB80  
005AC5EE    8A06                    mov al,byte ptr ds:\[esi\]       ;ds:\[005AF7D4\]=A4,EAX=00F500A4  
005AC5F0    46                      inc esi                        ;ESI=005AF7D5  
005AC5F1    8807                    mov byte ptr ds:\[edi\],al       ;起始地址放入 A4  
005AC5F3    47                      inc edi                        ;EDI=00F50001  
005AC5F4    02D2                    add dl,dl                      ;EDX=7C92EB00  
005AC5F6    75 05                   jnz short CWoool.005AC5FD  
005AC5F8    8A16                    mov dl,byte ptr ds:\[esi\]       ;ds:\[005AF7D5\]=38 ('8')  
005AC5FA    46                      inc esi                        ;ESI=005AF7D6  
005AC5FB    12D2                    adc dl,dl                      ;EDI=7C92EB71  
005AC5FD  ^ 73 EF                   jnb short CWoool.005AC5EE      ;循环之后变成 00F50000  000418A4  
005AC5FF    02D2                    add dl,dl                      ;7C92EB88  
005AC601   /75 05                   jnz short CWoool.005AC608      ;不等则跳走  
005AC603   |8A16                    mov dl,byte ptr ds:\[esi\]  
005AC605   |46                      inc esi  
005AC606   |12D2                    adc dl,dl  
005AC608   \\73 4A                   jnb short CWoool.005AC654      ;不低于跳走,最里边循环的出口  
005AC60A    33C0                    xor eax,eax                    ;清空 EAX  
005AC60C    02D2                    add dl,dl                      ;7C92EB10  
005AC60E    75 05                   jnz short CWoool.005AC615      ;不等则跳  
005AC610    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC612    46                      inc esi  
005AC613    12D2                    adc dl,dl  
005AC615    0F83 D6000000           jnb CWoool.005AC6F1            ;外面的循环出口  
005AC61B    02D2                    add dl,dl                      ;7c92eb20                        
005AC61D    75 05                   jnz short CWoool.005AC624  
005AC61F    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC621    46                      inc esi  
005AC622    12D2                    adc dl,dl  
005AC624    13C0                    adc eax,eax  
005AC626    02D2                    add dl,dl  
005AC628    75 05                   jnz short CWoool.005AC62F  
005AC62A    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC62C    46                      inc esi  
005AC62D    12D2                    adc dl,dl  
005AC62F    13C0                    adc eax,eax  
005AC631    02D2                    add dl,dl  
005AC633    75 05                   jnz short CWoool.005AC63A  
005AC635    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC637    46                      inc esi  
005AC638    12D2                    adc dl,dl  
005AC63A    13C0                    adc eax,eax  
005AC63C    02D2                    add dl,dl  
005AC63E    75 05                   jnz short CWoool.005AC645  
005AC640    8A16                    mov dl,byte ptr ds:\[esi\]       ;7C92EB70  
005AC642    46                      inc esi  
005AC643    12D2                    adc dl,dl  
005AC645    13C0                    adc eax,eax  
005AC647    74 06                   je short CWoool.005AC64F  
005AC649    57                      push edi  
005AC64A    2BF8                    sub edi,eax  
005AC64C    8A07                    mov al,byte ptr ds:\[edi\]  
005AC64E    5F                      pop edi  
005AC64F    8807                    mov byte ptr ds:\[edi\],al  
005AC651    47                      inc edi  
005AC652  ^ EB A0                   jmp short CWoool.005AC5F4  
005AC654    B8 01000000             mov eax,1  
005AC659    02D2                    add dl,dl  
005AC65B    75 05                   jnz short CWoool.005AC662  
005AC65D    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC65F    46                      inc esi  
005AC660    12D2                    adc dl,dl  
005AC662    13C0                    adc eax,eax  
005AC664    02D2                    add dl,dl  
005AC666    75 05                   jnz short CWoool.005AC66D  
005AC668    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC66A    46                      inc esi  
005AC66B    12D2                    adc dl,dl  
005AC66D  ^ 72 EA                   jb short CWoool.005AC659  
005AC66F    83E8 02                 sub eax,2  
005AC672    75 28                   jnz short CWoool.005AC69C  
005AC674    B9 01000000             mov ecx,1  
005AC679    02D2                    add dl,dl  
005AC67B    75 05                   jnz short CWoool.005AC682  
005AC67D    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC67F    46                      inc esi  
005AC680    12D2                    adc dl,dl  
005AC682    13C9                    adc ecx,ecx  
005AC684    02D2                    add dl,dl  
005AC686    75 05                   jnz short CWoool.005AC68D  
005AC688    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC68A    46                      inc esi  
005AC68B    12D2                    adc dl,dl  
005AC68D  ^ 72 EA                   jb short CWoool.005AC679  
005AC68F    56                      push esi  
005AC690    8BF7                    mov esi,edi  
005AC692    2BF5                    sub esi,ebp  
005AC694    F3:A4                   rep movs byte ptr es:\[edi\],byte pt>  
005AC696    5E                      pop esi  
005AC697  ^ E9 58FFFFFF             jmp CWoool.005AC5F4  
005AC69C    48                      dec eax  
005AC69D    C1E0 08                 shl eax,8  
005AC6A0    8A06                    mov al,byte ptr ds:\[esi\]  
005AC6A2    46                      inc esi  
005AC6A3    8BE8                    mov ebp,eax  
005AC6A5    B9 01000000             mov ecx,1  
005AC6AA    02D2                    add dl,dl  
005AC6AC    75 05                   jnz short CWoool.005AC6B3  
005AC6AE    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC6B0    46                      inc esi  
005AC6B1    12D2                    adc dl,dl  
005AC6B3    13C9                    adc ecx,ecx  
005AC6B5    02D2                    add dl,dl  
005AC6B7    75 05                   jnz short CWoool.005AC6BE  
005AC6B9    8A16                    mov dl,byte ptr ds:\[esi\]  
005AC6BB    46                      inc esi  
005AC6BC    12D2                    adc dl,dl  
005AC6BE  ^ 72 EA                   jb short CWoool.005AC6AA  
005AC6C0    3D 007D0000             cmp eax,7D00  
005AC6C5    73 1A                   jnb short CWoool.005AC6E1  
005AC6C7    3D 00050000             cmp eax,500  
005AC6CC    72 0E                   jb short CWoool.005AC6DC  
005AC6CE    41                      inc ecx  
005AC6CF    56                      push esi  
005AC6D0    8BF7                    mov esi,edi  
005AC6D2    2BF0                    sub esi,eax  
005AC6D4    F3:A4                   rep movs byte ptr es:\[edi\],byte pt>  
005AC6D6    5E                      pop esi  
005AC6D7  ^ E9 18FFFFFF             jmp CWoool.005AC5F4  
005AC6DC    83F8 7F                 cmp eax,7F  
005AC6DF    77 03                   ja short CWoool.005AC6E4  
005AC6E1    83C1 02                 add ecx,2  
005AC6E4    56                      push esi  
005AC6E5    8BF7                    mov esi,edi  
005AC6E7    2BF0                    sub esi,eax  
005AC6E9    F3:A4                   rep movs byte ptr es:\[edi\],byte pt>  
005AC6EB    5E                      pop esi  
005AC6EC  ^ E9 03FFFFFF             jmp CWoool.005AC5F4  
005AC6F1    8A06                    mov al,byte ptr ds:\[esi\]  
005AC6F3    46                      inc esi  
005AC6F4    33C9                    xor ecx,ecx  
005AC6F6    C0E8 01                 shr al,1  
005AC6F9    74 12                   je short CWoool.005AC70D         ;第三个出口  
005AC6FB    83D1 02                 adc ecx,2  
005AC6FE    8BE8                    mov ebp,eax  
005AC700    56                      push esi  
005AC701    8BF7                    mov esi,edi  
005AC703    2BF0                    sub esi,eax  
005AC705    F3:A4                   rep movs byte ptr es:\[edi\],byte pt>  
005AC707    5E                      pop esi  
005AC708  ^ E9 E7FEFFFF             jmp CWoool.005AC5F4  
005AC70D    5D                      pop ebp                         ;0012FF94 指针出栈  
005AC70E    2B7D 0C                 sub edi,dword ptr ss:\[ebp+C\]    ;堆栈 ss:\[0012FFA0\]=00F50000,edi=00F6B800-00F50000=1B800  
005AC711    897D FC                 mov dword ptr ss:\[ebp-4\],edi    ;把 1B800 放入 12FF90  
005AC714    61                      popad                           ;恢复以前的寄存器  
005AC715    5D                      pop ebp                         ;堆栈 \[0012FF94\]=005AC40C (CWoool.005AC40C),ebp=0012FF94  
005AC716    C2 0800                 retn 8  
  
这个 call 还是解码,把前面一些代码还原到刚申请的地址中.  
  
005AC4DC    6A 40                   push 40                        ;设置可读\\可写\\可执行属性  
005AC4DE    68 00100000             push 1000                      ;为指定的地址空间提交物理内存  
005AC4E3    FFB5 08040000           push dword ptr ss:\[ebp+408\]    ;\[005AC814\]=00043000 申请内存大小  
005AC4E9    6A 00                   push 0                         ;指定需要保留或提交空间的位置为自行保留  
005AC4EB    FF95 F0030000           call dword ptr ss:\[ebp+3F0\]    ;保留或提交一段地址空间 VirtualAlloc  
005AC4F1    8985 31040000           mov dword ptr ss:\[ebp+431\],eax ;\[005AC83D\]=eax=00FA0000  
005AC4F7    8985 D0010000           mov dword ptr ss:\[ebp+1D0\],eax ;ss:\[005AC5DC\]=00FA0000  
005AC4FD    64:67:A1 0000           mov eax,dword ptr fs:\[0\]       ;eax=fs:\[00000000\]=\[7FFDF000\]=0012FFE0  
005AC502    8985 2D040000           mov dword ptr ss:\[ebp+42D\],eax ;ss:\[005AC839\]=0012FFE0 异常指针  
005AC508    8B55 5B                 mov edx,dword ptr ss:\[ebp+5B\]  ;EDX=ss:\[005AC467\]=005AC048 (CWoool.005AC048)  
005AC50B    8B85 D0010000           mov eax,dword ptr ss:\[ebp+1D0\] ;EAX=ss:\[005AC5DC\]=00FA0000  
005AC511    8902                    mov dword ptr ds:\[edx\],eax     ;ds:\[005AC048\]=eax=00FA0000  
005AC513    8B85 08040000           mov eax,dword ptr ss:\[ebp+408\] ;eax=ss:\[005AC814\]=00043000 申请内存大小  
005AC519    8942 04                 mov dword ptr ds:\[edx+4\],eax   ;eax=ds:\[005AC04C\]=00043000  
005AC51C    8D85 9F030000           lea eax,dword ptr ss:\[ebp+39F\] ;eax=地址=005AC7AB  
005AC522    8B40 55                 mov eax,dword ptr ds:\[eax+55\]  ;eax=ds:\[005AC800\]=7C809B14 (kernel32.VirtualFree)  
005AC525    8942 08                 mov dword ptr ds:\[edx+8\],eax   ;ds:\[005AC050\]=eax=7C809B14 (kernel32.VirtualFree)  
005AC528    8B85 EC030000           mov eax,dword ptr ss:\[ebp+3EC\] ;eax=ss:\[005AC7F8\]=7C801D77 (kernel32.LoadLibraryA)  
005AC52E    8942 10                 mov dword ptr ds:\[edx+10\],eax  ;ds:\[005AC058\]=eax=7C801D77 (kernel32.LoadLibraryA)  
005AC531    8B85 E8030000           mov eax,dword ptr ss:\[ebp+3E8\] ;eax=ss:\[005AC7F4\]=7C80B529 (kernel32.GetModuleHandleA)  
005AC537    8942 14                 mov dword ptr ds:\[edx+14\],eax  ;ds:\[005AC05C\]=eax=7C80B529 (kernel32.GetModuleHandleA)这几步其实就是用 EAX 传递入口地址  
005AC53A    8B95 CC010000           mov edx,dword ptr ss:\[ebp+1CC\] ;edx=ss:\[005AC5D8\]=00F50000  
005AC540    BB F8010000             mov ebx,1F8                    ;ebx=1F8  
005AC545    8B7C1A 0C               mov edi,dword ptr ds:\[edx+ebx+C\];edi=ds:\[00F50204\]=00001000  
005AC549    0BFF                    or edi,edi                      ;  
005AC54B    74 1E                   je short CWoool.005AC56B  
005AC54D    8B4C1A 10               mov ecx,dword ptr ds:\[edx+ebx+10\];ecx=ds:\[00F50208\]=00017800  
005AC551    0BC9                    or ecx,ecx  
005AC553    74 11                   je short CWoool.005AC566  
005AC555    03BD D0010000           add edi,dword ptr ss:\[ebp+1D0\]   ;edi=00FA0000+00001000  
005AC55B    8B741A 14               mov esi,dword ptr ds:\[edx+ebx+14\];esi=ds:\[00F5020C\]=00000400  
005AC55F    03F2                    add esi,edx                      ;esi=00F50400  
005AC561    C1F9 02                 sar ecx,2                        ; 算术右移两位  
005AC564    F3:A5                   rep movs dword ptr es:\[edi\],dword >;ECX=0000000,ESI=00F67C00,EDI=00FB8800  
005AC566    83C3 28                 add ebx,28                         ;EBX=220  
005AC569  ^ EB DA                   jmp short CWoool.005AC545          ;这几步是把代码读入 00FA0000  
005AC56B    8B85 CC010000           mov eax,dword ptr ss:\[ebp+1CC\]     ;ss:\[005AC5D8\]=00F50000,eax=7C80B529 (kernel32.GetModuleHandleA)  
005AC571    50                      push eax                           ;保存 EAX=00F50000 在堆栈  
005AC572    8B95 D0010000           mov edx,dword ptr ss:\[ebp+1D0\]     ;edx=ss:\[005AC5DC\]=00FA0000  
005AC578    52                      push edx                           ;保存 00FA0000 在堆栈  
005AC579    8B18                    mov ebx,dword ptr ds:\[eax\]         ;ebx=ds:\[00F50000\]=000418A4  
005AC57B    03DA                    add ebx,edx                        ;edx=00FA0000,ebx=000418A4  
005AC57D    8B85 E4030000           mov eax,dword ptr ss:\[ebp+3E4\]  
005AC583    8903                    mov dword ptr ds:\[ebx\],eax     ;ds:\[00FE18A4\]=eax=7C80AC28 (kernel32.GetProcAddress)把这几个函数入口放入  
005AC585    8B85 E8030000           mov eax,dword ptr ss:\[ebp+3E8\]  
005AC58B    8943 04                 mov dword ptr ds:\[ebx+4\],eax   ;ds:\[00FE18A8\]=eax=7C80B529 (kernel32.GetModuleHandleA)  
005AC58E    8B85 EC030000           mov eax,dword ptr ss:\[ebp+3EC\]  
005AC594    8943 08                 mov dword ptr ds:\[ebx+8\],eax   ;00FE18AC=7C801D77  kernel32.LoadLibraryA  
005AC597    5F                      pop edi                        ;EDI=00FA0000  
005AC598    5E                      pop esi                        ;ESI00F50000  
005AC599    8B46 04                 mov eax,dword ptr ds:\[esi+4\]   ;00041000  
005AC59C    03C7                    add eax,edi                    ;EAX=00FE1000  
005AC59E    8985 C7010000           mov dword ptr ss:\[ebp+1C7\],eax ;ss:\[005AC5D3\]=eax=00FE1000  
005AC5A4    8B55 5B                 mov edx,dword ptr ss:\[ebp+5B\]  ;edx=ss:\[005AC467\]=005AC048 (CWoool.005AC048)  
005AC5A7    8B85 C7010000           mov eax,dword ptr ss:\[ebp+1C7\] ;eax=ss:\[005AC5D3\]=00FE1000,005AC5D3 是下面的地址  
005AC5AD    8942 0C                 mov dword ptr ds:\[edx+C\],eax   ;ds:\[005AC054\]=eax=00FE1000  
005AC5B0    8D9D 0D040000           lea ebx,dword ptr ss:\[ebp+40D\] ;ebx=地址=005AC819,文件头"MZP"的地址  
005AC5B6    53                      push ebx  
005AC5B7    6A 00                   push 0  
005AC5B9    6A 00                   push 0  
005AC5BB    6A 01                   push 1  
005AC5BD    57                      push edi                      ;EDI=00FA0000  
005AC5BE    8B5E 08                 mov ebx,dword ptr ds:\[esi+8\]  ;ebx=ds:\[00F50008\]=0003062C  
005AC5C1    03DF                    add ebx,edi                   ;EBX=00FD062C  
005AC5C3    53                      push ebx  
005AC5C4    68 00800000             push 8000  
005AC5C9    6A 00                   push 0  
005AC5CB    56                      push esi                      ;esi=00F50000  
005AC5CC    FF95 F4030000           call dword ptr ss:\[ebp+3F4\]   ;ss:\[005AC800\]=7C809B14 (kernel32.VirtualFree)释放地址空间  
                                                                  ;0012FF7C   005AC5D2     /CALL 到 VirtualFree 来自 CWoool.005AC5CC  
                                                                  ;0012FF80   00F50000     |Address = 00F50000 指定的地址  
                                                                  ;0012FF84   00000000     |Size = 0        大小  
                                                                  ;0012FF88   00008000     \\FreeType = MEM\_RELEASE 释放保留的地址空间  
005AC5D2    68 00000000             push 0                        ;这里变成了  
005AC5D2    68 0010FE00             push 0FE1000  
005AC5D7    C3                      retn                          ;这里就变成了返回 0FE1000  
  
呵呵,这个部分呢把代码动态解码到 00FA0000 处,并且用了一个手段跳到 00FE1000,在 005AC5A7 把 005AC5D3 写入返回的 00FE1000 地址,动态修改了 push 后压入的地址,然后在后面用一个 retn  
就返回了 00FE1000,真够精彩!!  
  
  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

【破解作者】 layper  
【作者邮箱】 layper2002@yahoo.com.cn  
【作者主页】 www.sy135.com  
【使用工具】 peid,od，  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 壳乱弹单步分析(5)  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
写到这篇,我感觉在分析这个壳的过程中学到了很多东西(虽然很多错漏),我很早就想单步分析壳了,但一直下不了决心,谁知道在这个胡  
侃的文章中我竟然实现了这个想法.呵呵,太好了.其实在国外,我们 crack 经常研究的一个项目是逆向工程(Reverse Engineer),  
就是把程序的源码用 crack 工具逆向出来,这是我们学习编程和系统低层的一种好办法,但可惜,在我们国内似乎非常少看  
见这种文章,我这种单步分析的办法虽然不是逆向工程,经过前几篇的分析后,我确实感受到了逆向工程的好处,真希望逆向工程  
在我们这里发扬光大.  
废话少说,继续分析我们的 ASProtect 1.2x - 1.3x 壳,经过上次的代码之后,我们返回了这里  
00FE1000    90                      nop                      ;这个 nop 估计是增加系统的兼容性吧  
00FE1001    60                      pushad                   ;保存所有寄存器的数据  
00FE1002    E8 40060000             call 00FE1647            ;跟进  
00FE1007    EB 44                   jmp short 00FE104D  
  
00FE1647    8B2C24                  mov ebp,dword ptr ss:\[esp\] ;ebp=堆栈 ss:\[0012FF68\]=00FE1007  
00FE164A    81ED 4B294400           sub ebp,44294B             ;ebp=00B9E6BC  
00FE1650    C3                      retn                       ;返回  
  
00FE1007   /EB 44                   jmp short 00FE104D         ;返回这里,跳  
  
00FE104D   \\BB 44294400             mov ebx,442944             ;ebx=442944  
00FE1052    03DD                    add ebx,ebp                ;EBX=00FE1007-44294B+442944=00FE1000,奸诈,竟然是移向 00FE1000  
00FE1054    2B9D 71294400           sub ebx,dword ptr ss:\[ebp+442971\] ;ebx=00FA0000  
00FE105A    83BD D8304400 00        cmp dword ptr ss:\[ebp+4430D8\],0   ;ss:\[00FE1794\]=00000000  
00FE1061    899D 2F2E4400           mov dword ptr ss:\[ebp+442E2F\],ebx ;ss:\[00FE14EB\]=ebx=00FA0000  
00FE1067    0F85 3E050000           jnz 00FE15AB                      ;不等则跳  
00FE106D    8D85 E0304400           lea eax,dword ptr ss:\[ebp+4430E0\] ;eax=地址=010F179C, (ASCII "kernel32.dll")  
00FE1073    50                      push eax                          ;压入栈  
00FE1074    FF95 EC314400           call dword ptr ss:\[ebp+4431EC\]    ;ss:\[010F18A8\]=7C80B529 (kernel32.GetModuleHandleA)返回 kernel32.dll 模块的句柄  
00FE107A    8985 DC304400           mov dword ptr ss:\[ebp+4430DC\],eax ;ss:\[010F1798\]=EAX=78C00000  
00FE1080    8BF8                    mov edi,eax                       ;EDI=EAX=78C0000  
00FE1082    8D9D ED304400           lea ebx,dword ptr ss:\[ebp+4430ED\] ;ebx=地址=010F17A9, (ASCII "VirtualAlloc")  
00FE1088    53                      push ebx  
00FE1089    50                      push eax  
00FE108A    FF95 E8314400           call dword ptr ss:\[ebp+4431E8\]  
00FE1090    8985 79294400           mov dword ptr ss:\[ebp+442979\],eax  
00FE1096    8D9D FA304400           lea ebx,dword ptr ss:\[ebp+4430FA\]  
00FE109C    53                      push ebx  
00FE109D    57                      push edi  
00FE109E    FF95 E8314400           call dword ptr ss:\[ebp+4431E8\]  
00FE10A4    8985 7D294400           mov dword ptr ss:\[ebp+44297D\],eax  
00FE10AA    8B85 2F2E4400           mov eax,dword ptr ss:\[ebp+442E2F\]  
00FE10B0    8985 D8304400           mov dword ptr ss:\[ebp+4430D8\],eax  
00FE10B6    6A 04                   push 4  
00FE10B8    68 00100000             push 1000  
00FE10BD    68 46050000             push 546  
00FE10C2    6A 00                   push 0  
00FE10C4    FF95 79294400           call dword ptr ss:\[ebp+442979\]  
00FE10CA    8985 75294400           mov dword ptr ss:\[ebp+442975\],eax  
00FE10D0    8D9D 452A4400           lea ebx,dword ptr ss:\[ebp+442A45\]  
00FE10D6    50                      push eax  
00FE10D7    53                      push ebx  
00FE10D8    E8 74050000             call 00FE1651  
00FE10DD    8BC8                    mov ecx,eax  
00FE10DF    8DBD 452A4400           lea edi,dword ptr ss:\[ebp+442A45\]  
00FE10E5    8BB5 75294400           mov esi,dword ptr ss:\[ebp+442975\]  
00FE10EB    F3:A4                   rep movs byte ptr es:\[edi\],byte ptr d>  
00FE10ED    8B85 75294400           mov eax,dword ptr ss:\[ebp+442975\]  
00FE10F3    68 00800000             push 8000  
00FE10F8    6A 00                   push 0  
00FE10FA    50                      push eax  
00FE10FB    FF95 7D294400           call dword ptr ss:\[ebp+44297D\]  
00FE1101    8D0E                    lea ecx,dword ptr ds:\[esi\]  
00FE1103    8551 2C                 test dword ptr ds:\[ecx+2C\],edx  
00FE1106    44                      inc esp  
00FE1107    07                      pop es  
00FE1108    50                      push eax  
00FE1109    C3                      retn  
  
  
  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!