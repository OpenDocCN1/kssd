

# 菲菲 0502 版

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-223.htm`](https://www.pediy.com/kssd/pediy07/pediy7-223.htm)

突破两个实例限制  
                                               -----一次不完美的 PEDIY  
【破解作者】 winndy \[FCG\]\[PYG\]  
【作者邮箱】 CNwinndy@hotmail.com  
【使用工具】 PEID v0.93  OllyDbg v1.10 fly 修改版  
【破解平台】 Winxp SP2  
【软件名称】 天骄 2 外挂--菲菲 0502 版  
【官方网址】 http://bbs.tj2wg.com/  
【编写语言】 VC6  
【软件介绍】 不用多说了...  
【破解声明】 For study ,For Fun,  
【破解说明】 只能运行两个实例,无壳,0510 版的是 Armadillo 3.78 -> Silicon Realms Toolworks 加的壳,不会脱,  
             所以找这个 0502 来揉一揉...  
             玩到最后,可以运行 3 个实例,但是第 3 个实例运行,hook 住天骄后,不能启动热键 F12,...  
             最后实在找不出原因,不过想想.这次不完美的 pediy,也费了不少周折,自己写个总结吧...  
             失误之处还请各位老大指出!  
【破解过程】   
             OD 载入菲菲,反键,"搜索"--"所有参考文本串",  
             在下面设断点,然后观察和比较,实例一和实例二的代码的区别,以及寄存器的状态等值.  
  
004047DD    .  8B83 580F0000 mov eax,dword ptr ds:\[ebx+F58\]  
004047E3    .  8B4B 20       mov ecx,dword ptr ds:\[ebx+20\]  
004047E6    .  33ED          xor ebp,ebp  
004047E8    .  50            push eax                            ; /lParam  
004047E9    .  55            push ebp                            ; |wParam => 0  
004047EA    .  68 80000000   push 80                             ; |Message = WM\_SETICON  
004047EF    .  51            push ecx                            ; |hWnd  
004047F0    .  FFD6          call esi                            ; \\SendMessageA  
004047F2    .  8B35 24604100 mov esi,dword ptr ds:\[<&KERNEL32.Op>;  kernel32.OpenMutexA  
004047F8    .  68 ACA24100   push TJMan9.0041A2AC                ; /MutexName = "TJMANRunOnlyOneInstance1"  
004047FD    .  55            push ebp                            ; |Inheritable => FALSE  
004047FE    .  68 01001F00   push 1F0001                         ; |Access = 1F0001  
00404803    .  89AB 54020000 mov dword ptr ds:\[ebx+254\],ebp      ; |  
00404809    .  FFD6          call esi                            ; \\OpenMutexA  
0040480B    .  51            push ecx  
0040480C    .  3BC5          cmp eax,ebp  
0040480E    .  8983 00030000 mov dword ptr ds:\[ebx+300\],eax  
00404814    .  8BCC          mov ecx,esp  
00404816       75 7E         jnz short TJMan9.00404896                                ;实例 1 已运行,去运行实例 2  
  
00404818    .  896424 14              mov dword ptr ss:\[esp+14\],esp  
  
//d esp  
//00125C30  71 FB 92 7C              q 麙|.  
  
0040481C    .  68 A0A24100            push TJMan.0041A2A0                             ;  ASCII "外挂 1 开!"  
00404821    .  E8 5CF20000            call <jmp.&MFC42.#537>  
00404826    .  8BCB                   mov ecx,ebx                                     ; |  
00404828    .  E8 430B0000            call TJMan.00405370                             ; \\TJMan.00405370  
0040482D    .  BF 8CA24100            mov edi,TJMan.0041A28C                          ;  ASCII "TJ2HANDLE1byParker"  
00404832    .  83C9 FF                or ecx,FFFFFFFF  
00404835    .  33C0                   xor eax,eax  
00404837    .  8D5424 24              lea edx,dword ptr ss:\[esp+24\]  
0040483B    .  F2:AE                  repne scas byte ptr es:\[edi\]  
0040483D    .  F7D1                   not ecx  
0040483F    .  2BF9                   sub edi,ecx  
00404841    .  68 ACA24100            push TJMan.0041A2AC                             ; /MutexName = "TJMANRunOnlyOneInstance1"  
00404846    .  8BC1                   mov eax,ecx                                     ; |  
00404848    .  8BF7                   mov esi,edi                                     ; |  
0040484A    .  8BFA                   mov edi,edx                                     ; |  
0040484C    .  55                     push ebp                                        ; |InitialOwner  
0040484D    .  C1E9 02                shr ecx,2                                       ; |  
00404850    .  F3:A5                  rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  ; |  
00404852    .  8BC8                   mov ecx,eax                                     ; |  
00404854    .  55                     push ebp                                        ; |pSecurity  
00404855    .  83E1 03                and ecx,3                                       ; |  
00404858    .  F3:A4                  rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]    ; |  
0040485A    .  FF15 20604100          call dword ptr ds:\[<&KERNEL32.CreateMutexA>\]    ; \\CreateMutexA  
00404860    .  3BC5                   cmp eax,ebp  
00404862    .  8983 00030000          mov dword ptr ds:\[ebx+300\],eax  
00404868    .  74 11                  je short TJMan.0040487B  
0040486A    .  FF15 1C604100          call dword ptr ds:\[<&KERNEL32.GetLastError>\]    ; \[GetLastError  
00404870    .  3D B7000000            cmp eax,0B7  
00404875    .  0F85 C9000000          jnz TJMan.00404944  
0040487B    >  55                     push ebp  
0040487C    .  55                     push ebp  
0040487D    .  68 70A24100            push TJMan.0041A270                             ;  ASCII "系统错误 1，外挂不能运行!"  
00404882    .  E8 3FF00000            call <jmp.&MFC42.#1200>  
00404887    .  8B13                   mov edx,dword ptr ds:\[ebx\]  
00404889    .  8BCB                   mov ecx,ebx  
0040488B    .  FF92 CC000000          call dword ptr ds:\[edx+CC\]  
00404891    .  E9 AE000000            jmp TJMan.00404944  
  
//运行实例 2  
00404896    >  896424 14              mov dword ptr ss:\[esp+14\],esp  
  
//d esp  
//00125C30  C0 5B 12 00              繹..  
//这个 esp 好像很重要,在手动增加第 3 个实例的代码之后,esp 为 00125C34,运行出错了,  
//最后比较寄存器的状态,再往堆栈里压入一个参数,然后 esp 变为 00125C30,就可以正常启动第 3 个实例了.  
  
0040489A    .  68 68A24100            push TJMan.0041A268                             ;  ASCII "外挂 2 开"  
0040489F    .  E8 DEF10000            call <jmp.&MFC42.#537>  
004048A4    .  8BCB                   mov ecx,ebx                                     ; |  
004048A6    .  E8 C50A0000            call TJMan.00405370                             ; \\TJMan.00405370  
//通过后来的第三个实例可以知道,上面四行代码是类似用来初始化实例(ini instance)的代码,  
  
  
//////////////////  
//这段代码在实例一种是放在上面五行代码前面,现在把它放后面了,注意比较区别  
//也就是说,实例 2 是先初始化,然后调用 OpenMutexA 来看实例是否已运行,  
//而实例一是先调用 OpenMutexA 来看实例是否已运行,然后再来初始化,  
//显然,应该先判断实例是否存在,然后再来初始化.  
//在手动构造了实例 3 的代码之后,会发现实例 2 的这种顺序所带来的影响,在后来,我又手动调换了这两段小代码的顺序.  
  
  
  
004048AB    .  68 4CA24100            push TJMan.0041A24C                             ;  ASCII "TJMANRunOnlyOneInstance2"  
004048B0    .  55                     push ebp  
004048B1    .  68 01001F00            push 1F0001  
  
{  对照比较两个实例的代码:  
实例 1 中这里有一句  
00404803    .  89AB 54020000 mov dword ptr ds:\[ebx+254\],ebp      ; |  
  
}  
  
004048B6    .  FFD6                   call esi                            ;ESI 7C80EC1B kernel32.OpenMutexA  
{实例 1 中这里有一句  
0040480B    .  51            push ecx      
ECX 7C92FB71 ntdll.7C92FB71  
在后来建立实例 3 后,堆栈为 00125C34,运行错误,为了修正堆栈,压入了此 7C92FB71.  
不过当调换实例 2 的初始化和 openmutex 后,由于同样的堆栈原因,把 push 7C92FB71 NOP 了,  
运行正常(这个参数好像没用),只要保证 esp 和 ecx 为 00125C30 就行了.  
}  
  
  
004048B8    .  3BC5                   cmp eax,ebp  
004048BA    .  8983 00030000          mov dword ptr ds:\[ebx+300\],eax  
{  
实例 1 中这里有一句  
00404814    .  8BCC          mov ecx,esp  
ecx=00125C30  
d ecx  
00125C30  71 FB 92 7C              q 麙|.  
  
注意,这正是 0040480B 处 push 的 ecx.  
  
}  
004048C0    .  75 6C                  jnz short TJMan.0040492E                     
////第二个实例已运行,显示"只支持双开".  
//所需要作的就是在适当的位置增加一段创建实例 3 的代码,然后从这里跳去.基本思想就是这样.  
//创建实例 3 的代码,是从实例 2 的代码拷过去的,从 00404896 到 0040492C.  
///////////////////  
  
004048C2    .  BF 38A24100            mov edi,TJMan.0041A238                          ;  ASCII "TJ2HANDLE2byParker"  
004048C7    .  83C9 FF                or ecx,FFFFFFFF  
004048CA    .  33C0                   xor eax,eax  
004048CC    .  8D5424 24              lea edx,dword ptr ss:\[esp+24\]     ;edx=esp+24=00125C34+24=00125C58  
  
004048D0    .  F2:AE                  repne scas byte ptr es:\[edi\]  
004048D2    .  F7D1                   not ecx  
004048D4    .  2BF9                   sub edi,ecx  
004048D6    .  68 4CA24100            push TJMan.0041A24C                             ; /MutexName = "TJMANRunOnlyOneInstance2"  
004048DB    .  8BC1                   mov eax,ecx                                     ; |  
004048DD    .  8BF7                   mov esi,edi                                     ; |  
004048DF    .  8BFA                   mov edi,edx                                     ; |  
004048E1    .  55                     push ebp                                        ; |InitialOwner  
004048E2    .  C1E9 02                shr ecx,2                                       ; |  
004048E5    .  F3:A5                  rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  ; |  
004048E7    .  8BC8                   mov ecx,eax                                     ; |  
004048E9    .  55                     push ebp                                        ; |pSecurity  
004048EA    .  83E1 03                and ecx,3                                       ; |  
004048ED    .  F3:A4                  rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]    ; |  
004048EF    .  FF15 20604100          call dword ptr ds:\[<&KERNEL32.CreateMutexA>\]    ; \\CreateMutexA  
004048F5    .  3BC5                   cmp eax,ebp  
004048F7    .  8983 00030000          mov dword ptr ds:\[ebx+300\],eax  
004048FD    .  74 0D                  je short TJMan.0040490C  
004048FF    .  FF15 1C604100          call dword ptr ds:\[<&KERNEL32.GetLastError>\]    ; \[GetLastError  
00404905    .  3D B7000000            cmp eax,0B7  
0040490A    .  75 16                  jnz short TJMan.00404922  
  
  
0040490C    >  55                     push ebp  
0040490D    .  55                     push ebp  
0040490E    .  68 1CA24100            push TJMan.0041A21C                             ;  ASCII "系统错误 2，外挂不能运行!"  
00404913    .  E8 AEEF0000            call <jmp.&MFC42.#1200>  
00404918    .  8B13                   mov edx,dword ptr ds:\[ebx\]  
0040491A    .  8BCB                   mov ecx,ebx  
0040491C    .  FF92 CC000000          call dword ptr ds:\[edx+CC\]  
00404922    >  C783 54020000 01000000 mov dword ptr ds:\[ebx+254\],1,  //////////  
0040492C    .  EB 16                  jmp short TJMan.00404944  
  
//下面这段代码是用来显示信息的.  
//把下面全部 NOP 掉,然后用来做长转移的跳板!  
//然后把这段代码放在实例 3 的代码后面,最后再跳回来 00404944.  
//  
0040492E    >  55                     push ebp  
0040492F    .  55                     push ebp  
00404930    .  68 00A24100            push TJMan.0041A200                             ;  ASCII "对不起,外挂仅仅支持双开!"  
00404935    .  E8 8CEF0000            call <jmp.&MFC42.#1200>  
0040493A    .  8B03                   mov eax,dword ptr ds:\[ebx\]  
0040493C    .  8BCB                   mov ecx,ebx  
0040493E    .  FF90 CC000000          call dword ptr ds:\[eax+CC\]  
  
未修改前的代码就是上面这些了  
\=====================================================  
  
//下面就要在.text 段中找一块空白地方,把下面的代码(创建实例 3,从实例 2 拷贝而来,然后再修改部分代码,主要是一些 call,比如 call 00413A82,  
//在不同的位置调用,生成的机器代码不一样,因为采用相对寻址方式)加进去  
//修改 PE 的工具我采用了 raindy 汉化的 studyPE,很好用.向作者和汉化作者表示敬意!  
//用 studyPE 代开 Tjman.exe,点"区段",在 NO01,.text 区段这行点右键,"前往区段开头"  
//然后记住我们要拷贝的代码是从 00404896 到 0040492C,减去基址 00401000 后可得到相对位移,然后再右键,"copy"这段代码,  
//在这个区段的后面找个空白地方,点右键,"粘贴"过去.我选的开始位置是 000157B0.  
  
//000157B0  (000157B0)  
896424146868A24100E8DEF100008BCBE8C50A0000684CA24100556801001F00FFD63BC5898300030000756CBF38A2410083C9FF33C08D542424F2AEF7D12BF9684CA241008BC  
  
18BF78BFA55C1E902F3A58BC85583E103F3A4FF15206041003BC5898300030000740DFF151C6041003DB700000075165555681CA24100E8AEEF00008B138BCBFF92CC000000C7  
  
835402000001000000EB16  
//////  
  
下面要做的就是把前面提到的部分不准确的 call 修改过来.  
改完后看到的还是 ASCII "TJ2HANDLE2byParker",MutexName = "TJMANRunOnlyOneInstance2",ASCII "系统错误 2，外挂不能运行!",以及 ASCII "对不起,外挂仅仅  
  
支持双开!"  
  
因此,还需要做的就是在 NO03,区段.data 中为实例 3 增加显示的数据.这部分工作很简单,把实例 2 的数据拷贝过去,  
然后 32 改成 33,把二的内码改成 C8FD 就行了(但是 OD 还是不能正常显示'三',不知为什么).  
然而后来看到,事情不是想象那么简单.刚开始,我们这段数据放在.data 的便宜 000010C0 处,几乎紧跟在原来的数据后面了.  
后来调试发现,实例一运行后,这段数据就被修改了,如"/MutexName = "���""等,看来程序执行的时候还会用到这部分空间,  
于是这段数据又得搬家,这次远点搬吧,搬到了偏移 000018C0 处,后来就没问题了.  
  
后来运行第 3 个实例,报错,发现是堆栈的问题,  
见下面的代码,看它是在哪一行出错的  
  
00405370   /$  6A FF         push -1  
  
//eax=00125B40  
//d \[eax\]  
//00374040  CD E2 B9 D2 33 BF AA 00  外挂 3 开.  
//00374048  0D F0 AD BA 0D F0 AD BA  .瓠?瓠  
  
00405372   |.  68 D0484100   push TJMan3.004148D0                                ;  SE handler installation  
00405377   |.  64:A1 0000000>mov eax,dword ptr fs:\[0\]  
0040537D   |.  50            push eax  
0040537E   |.  64:8925 00000>mov dword ptr fs:\[0\],esp  
00405385   |.  83EC 14       sub esp,14  
00405388   |.  53            push ebx  
00405389   |.  55            push ebp  
0040538A   |.  56            push esi  
0040538B   |.  57            push edi  
0040538C   |.  8BF1          mov esi,ecx  
0040538E   |.  8D4C24 10     lea ecx,dword ptr ss:\[esp+10\]  
00405392   |.  C74424 2C 000>mov dword ptr ss:\[esp+2C\],0  
0040539A   |.  E8 75E50000   call <jmp.&MFC42.#540>  
0040539F   |.  8D4C24 20     lea ecx,dword ptr ss:\[esp+20\]  
004053A3   |.  C64424 2C 01  mov byte ptr ss:\[esp+2C\],1  
004053A8   |.  E8 67E50000   call <jmp.&MFC42.#540>  
004053AD   |.  68 80A14100   push TJMan3.0041A180                                ;  ASCII "  
"  
004053B2   |.  8D4C24 18     lea ecx,dword ptr ss:\[esp+18\]  
004053B6   |.  C64424 30 02  mov byte ptr ss:\[esp+30\],2  
004053BB   |.  E8 C2E60000   call <jmp.&MFC42.#537>  
004053C0   |.  8D4424 34     lea eax,dword ptr ss:\[esp+34\]  
004053C4   |.  8D4C24 10     lea ecx,dword ptr ss:\[esp+10\]  
004053C8   |.  50            push eax  
004053C9       C64424 30 03  mov byte ptr ss:\[esp+30\],3  
004053CE   |.  E8 23E50000   call <jmp.&MFC42.#858>    
  
//程序在 004053CE 这里报错了,跟进去,发现异常在 73D34973 处,是什么原因呢?  
原来在 73D3496F 处 edi=00125C34,而\[00125C34\]=0,所以 73D34973 处就是 mov eax,\[0\],  
显然出错,怎么修正呢?跟进实例 1 和实例 2 看看!发现它们在 73D3496F 处的 edi 都是 00125C30,  
照猫画虎看看,修改 edi,运行,这里不再报错.  
  
  
73D3496B M>  8BFF            mov edi,edi  
73D3496D     56              push esi  
73D3496E     57              push edi  
73D3496F     8B7C24 0C       mov edi,dword ptr ss:\[esp+C\]   
73D34973     8B07            mov eax,dword ptr ds:\[edi\]     
73D34975     8BF1            mov esi,ecx  
73D34977     8B0E            mov ecx,dword ptr ds:\[esi\]  
73D34979     3BC8            cmp ecx,eax  
73D3497B     74 39           je short MFC42.73D349B6  
73D3497D     8379 F4 00      cmp dword ptr ds:\[ecx-C\],0  
73D34981     7D 0B           jge short MFC42.73D3498E  
73D34983     83C1 F4         add ecx,-0C  
73D34986     3B0D C486E073   cmp ecx,dword ptr ds:\[73E086C4\]                     ; MFC42.73E086C8  
73D3498C     75 06           jnz short MFC42.73D34994  
73D3498E     8378 F4 00      cmp dword ptr ds:\[eax-C\],0  
73D34992     7D 0D           jge short MFC42.73D349A1  
  
然而这个 00125C34 是哪里传进来的呢,最后找到这里:  
  
004157B0    > \\896424 14     mov dword ptr ss:\[esp+14\],esp  
  
而这里是实例 3 的语句,它是从 00404934   长跳转过来的,  
查看实例 2 我们发现可以在这个长 jmp 前 push 一个字,便可纠正堆栈,  
在不知道这个压入的参数是什么意思的情况下,还是压入和实例 2 一样的字吧!  
push 7C92FB71  
然后再 jmp TJMan3.004157B0  
  
OK 了!  
  
实例 3 可以正常调出来.  
但是再面板里看到了两行文字  
"外挂 2 开  
外挂 3 开"  
应该只有一行"外挂 3 开"的,为什么呢?  
回头看看前面的代码,发现从实例 2 跳过来时,出了点问题.  
因为它先初始化,然后再 Openmutex,打开互斥量,这个顺序在逻辑上有问题.  
下面我们再交换这个两段小代码的顺序!  
  
还是用 StudyPE,copy,paste,copy,paste,...  
修改 call 的偏移量,运行,报错...  
再仔细来看看寄存器的状态,比较一下为什么出错.  
  
下面的 good 是在交换两段代码前的正常运行的状态,bad 是交换后的状态,  
通过修改可以发现,只要将 bad 的 ECX 该为 good 的 ECX 值,便可以正常运行了!  
看来这个 00125C30 真重要啊!  
  
\====  
good  
  
EAX 000000E8  
ECX 00125C30  
EDX 7C92EB94 ntdll.KiFastSystemCallRet  
EBX 00126320 ASCII "0nA"  
ESP 00125C30  
EBP 00000000  
ESI 7C80EC1B kernel32.OpenMutexA  
EDI 00000000  
  
  
bad  
  
EAX 00000000  
ECX 7C92FB71 ntdll.7C92FB71  
EDX 00000002  
EBX 00126320 ASCII "0nA"  
ESP 00125C30  
EBP 00000000  
ESI 7C80EC1B kernel32.OpenMutexA  
EDI 00000000  
  
  
仅修复 ecx=00125c30 就行了  
\============  
于是再想办法修复 ECX,通过观察发现,只要在 mov dword ptr ss:\[esp+14\],esp  
  
前面加一句 mov ecx,esp 就可以了(实例一中有这一句)  
  
再启动 StudyPE,把代码搬家,腾出空间...  
  
后来发现,mov ecx,esp 前面的 jnz short 00404934 得变成 jnz long 00404934 了,  
原来只有 2 字节,现在却是 6 字节,没办法,再搬家,累....  
  
搬完后还要修改 call 得偏移,  
最后实例 2 可以运行了!  
  
可是发现实例 3 又不能运行了,交换之前是  
push 7C92FB71  
mov ecx,esp  
jmp TJMan3.004157B0  
  
而交换后的 esp 已经为 00125C30,不要再修正堆栈了,  
这句 push 7C92FB71 NOP 掉!  
实例三也可以打开了!  
  
下面是修改后的代码:  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 004047F2    .  8B35 24604100 mov esi,dword ptr ds:\[<&KERNEL32.OpenMutexA>\]    ;  kernel32.OpenMutexA
> 004047F8    .  68 ACA24100   push TJMan23.0041A2AC                            ; /MutexName = "TJMANRunOnlyOneInstance1"
> 004047FD    .  55            push ebp                                         ; |Inheritable => FALSE
> 004047FE    .  68 01001F00   push 1F0001                                      ; |Access = 1F0001
> 00404803    .  89AB 54020000 mov dword ptr ds:\[ebx+254\],ebp                   ; |
> 00404809    .  FFD6          call esi                                         ; \\OpenMutexA
> 0040480B    .  51            push ecx
> 0040480C    .  3BC5          cmp eax,ebp
> 0040480E    .  8983 00030000 mov dword ptr ds:\[ebx+300\],eax
> 00404814    .  8BCC          mov ecx,esp
> 00404816       75 7E         jnz short TJMan23.00404896
> 00404818       896424 14     mov dword ptr ss:\[esp+14\],esp
> 0040481C    .  68 A0A24100   push TJMan23.0041A2A0                            ;  ASCII "外挂 1 开!"
> 00404821    .  E8 5CF20000   call <jmp.&MFC42.#537>
> 00404826    .  8BCB          mov ecx,ebx                                      ; |
> 00404828    .  E8 430B0000   call TJMan23.00405370                            ; \\TJMan23.00405370
> 0040482D    .  BF 8CA24100   mov edi,TJMan23.0041A28C                         ;  ASCII "TJ2HANDLE1byParker"
> 00404832    .  83C9 FF       or ecx,FFFFFFFF
> 00404835    .  33C0          xor eax,eax
> 00404837    .  8D5424 24     lea edx,dword ptr ss:\[esp+24\]
> 0040483B    .  F2:AE         repne scas byte ptr es:\[edi\]
> 0040483D    .  F7D1          not ecx
> 0040483F    .  2BF9          sub edi,ecx
> 00404841    .  68 ACA24100   push TJMan23.0041A2AC                            ; /MutexName = "TJMANRunOnlyOneInstance1"
> 00404846    .  8BC1          mov eax,ecx                                      ; |
> 00404848    .  8BF7          mov esi,edi                                      ; |
> 0040484A    .  8BFA          mov edi,edx                                      ; |
> 0040484C    .  55            push ebp                                         ; |InitialOwner
> 0040484D    .  C1E9 02       shr ecx,2                                        ; |
> 00404850    .  F3:A5         rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]   ; |
> 00404852    .  8BC8          mov ecx,eax                                      ; |
> 00404854    .  55            push ebp                                         ; |pSecurity
> 00404855    .  83E1 03       and ecx,3                                        ; |
> 00404858    .  F3:A4         rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]     ; |
> 0040485A    .  FF15 20604100 call dword ptr ds:\[<&KERNEL32.CreateMutexA>\]     ; \\CreateMutexA
> 00404860    .  3BC5          cmp eax,ebp
> 00404862    .  8983 00030000 mov dword ptr ds:\[ebx+300\],eax
> 00404868    .  74 11         je short TJMan23.0040487B
> 0040486A    .  FF15 1C604100 call dword ptr ds:\[<&KERNEL32.GetLastError>\]     ; \[GetLastError
> 00404870    .  3D B7000000   cmp eax,0B7
> 00404875    .  0F85 C9000000 jnz TJMan23.00404944
> 0040487B    >  55            push ebp
> 0040487C    .  55            push ebp
> 0040487D    .  68 70A24100   push TJMan23.0041A270                            ;  ASCII "系统错误 1，外挂不能运行!"
> 00404882    .  E8 3FF00000   call <jmp.&MFC42.#1200>
> 00404887    .  8B13          mov edx,dword ptr ds:\[ebx\]
> 00404889    .  8BCB          mov ecx,ebx
> 0040488B    .  FF92 CC000000 call dword ptr ds:\[edx+CC\]
> 00404891    .  E9 AE000000   jmp TJMan23.00404944
> 00404896    >  68 4CA24100   push TJMan23.0041A24C                            ;  ASCII "TJMANRunOnlyOneInstance2"
> 0040489B    .  55            push ebp
> 0040489C    .  68 01001F00   push 1F0001
> 004048A1    .  FFD6          call esi                                          ;OpenMutexA
> 004048A3    .  3BC5          cmp eax,ebp
> 004048A5    .  8983 00030000 mov dword ptr ds:\[ebx+300\],eax
> 004048AB       0F85 83000000 jnz TJMan23.00404934
> 004048B1    .  8BCC          mov ecx,esp
> 004048B3    .  896424 14     mov dword ptr ss:\[esp+14\],esp
> 004048B7    .  68 68A24100   push TJMan23.0041A268                            ;  ASCII "外挂 2 开"
> 004048BC    .  E8 C1F10000   call <jmp.&MFC42.#537>
> 004048C1    .  8BCB          mov ecx,ebx                                      ; |
> 004048C3    .  E8 A80A0000   call TJMan23.00405370                            ; \\TJMan23.00405370
> 004048C8    .  BF 38A24100   mov edi,TJMan23.0041A238                         ;  ASCII "TJ2HANDLE2byParker"
> 004048CD    .  83C9 FF       or ecx,FFFFFFFF
> 004048D0    .  33C0          xor eax,eax
> 004048D2    .  8D5424 24     lea edx,dword ptr ss:\[esp+24\]
> 004048D6    .  F2:AE         repne scas byte ptr es:\[edi\]
> 004048D8    .  F7D1          not ecx
> 004048DA    .  2BF9          sub edi,ecx
> 004048DC    .  68 4CA24100   push TJMan23.0041A24C                            ; /MutexName = "TJMANRunOnlyOneInstance2"
> 004048E1    .  8BC1          mov eax,ecx                                      ; |
> 004048E3    .  8BF7          mov esi,edi                                      ; |
> 004048E5    .  8BFA          mov edi,edx                                      ; |
> 004048E7    .  55            push ebp                                         ; |InitialOwner
> 004048E8    .  C1E9 02       shr ecx,2                                        ; |
> 004048EB    .  F3:A5         rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]   ; |
> 004048ED    .  8BC8          mov ecx,eax                                      ; |
> 004048EF    .  55            push ebp                                         ; |pSecurity
> 004048F0    .  83E1 03       and ecx,3                                        ; |
> 004048F3    .  F3:A4         rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]     ; |
> 004048F5    .  FF15 20604100 call dword ptr ds:\[<&KERNEL32.CreateMutexA>\]     ; \\CreateMutexA
> 004048FB    .  3BC5          cmp eax,ebp
> 004048FD    .  8983 00030000 mov dword ptr ds:\[ebx+300\],eax
> 00404903    .  74 0D         je short TJMan23.00404912
> 00404905    .  FF15 1C604100 call dword ptr ds:\[<&KERNEL32.GetLastError>\]     ; \[GetLastError
> 0040490B    .  3D B7000000   cmp eax,0B7
> 00404910    .  75 16         jnz short TJMan23.00404928
> 00404912    >  55            push ebp
> 00404913    .  55            push ebp
> 00404914    .  68 1CA24100   push TJMan23.0041A21C                            ;  ASCII "系统错误 2，外挂不能运行!"
> 00404919    .  E8 ACEF0000   call TJMan23.004138CA
> 0040491E    .  8B13          mov edx,dword ptr ds:\[ebx\]
> 00404920    .  8BCB          mov ecx,ebx
> 00404922    .  FF92 CC000000 call dword ptr ds:\[edx+CC\]
> 00404928       C783 54020000>mov dword ptr ds:\[ebx+254\],1
> 00404932    .  EB 10         jmp short TJMan23.00404944
> 00404934    >  90            nop
> 00404935    .  90            nop
> 00404936    .  90            nop
> 00404937    .  90            nop
> 00404938    .  90            nop
> 00404939    .  8BCC          mov ecx,esp
> 0040493B    .  E9 700E0100   jmp TJMan23.004157B0
> 00404940       90            nop
> 00404941       90            nop
> 00404942       90            nop
> 00404943       90            nop
> 
> 
> \------实例 3 的代码-----
> 004157B0    > \\896424 14     mov dword ptr ss:\[esp+14\],esp
> 004157B4    .  68 10B94100   push TJMan23.0041B910                            ;  ASCII "外挂 3 开"
> 004157B9    .  E8 C4E2FFFF   call <jmp.&MFC42.#537>
> 004157BE    .  8BCB          mov ecx,ebx                                      ; |
> 004157C0    .  E8 ABFBFEFF   call TJMan23.00405370                            ; \\TJMan23.00405370
> 004157C5    .  68 F5B84100   push TJMan23.0041B8F5                            ;  ASCII "JMANRunOnlyOneInstance3"
> 004157CA    .  55            push ebp
> 004157CB    .  68 01001F00   push 1F0001
> 004157D0    .  FFD6          call esi
> 004157D2    .  3BC5          cmp eax,ebp
> 004157D4    .  8983 00030000 mov dword ptr ds:\[ebx+300\],eax
> 004157DA    .  75 6C         jnz short TJMan23.00415848
> 004157DC    .  BF E0B84100   mov edi,TJMan23.0041B8E0                         ;  ASCII "TJ2HANDLE3byParker"
> 004157E1    .  83C9 FF       or ecx,FFFFFFFF
> 004157E4    .  33C0          xor eax,eax
> 004157E6    .  8D5424 24     lea edx,dword ptr ss:\[esp+24\]
> 004157EA    .  F2:AE         repne scas byte ptr es:\[edi\]
> 004157EC    .  F7D1          not ecx
> 004157EE    .  2BF9          sub edi,ecx
> 004157F0    .  68 F5B84100   push TJMan23.0041B8F5                            ; /MutexName = "JMANRunOnlyOneInstance3"
> 004157F5    .  8BC1          mov eax,ecx                                      ; |
> 004157F7    .  8BF7          mov esi,edi                                      ; |
> 004157F9    .  8BFA          mov edi,edx                                      ; |
> 004157FB    .  55            push ebp                                         ; |InitialOwner
> 004157FC    .  C1E9 02       shr ecx,2                                        ; |
> 004157FF    .  F3:A5         rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]   ; |
> 00415801    .  8BC8          mov ecx,eax                                      ; |
> 00415803    .  55            push ebp                                         ; |pSecurity
> 00415804    .  83E1 03       and ecx,3                                        ; |
> 00415807    .  F3:A4         rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]     ; |
> 00415809    .  FF15 20604100 call dword ptr ds:\[<&KERNEL32.CreateMutexA>\]     ; \\CreateMutexA
> 0041580F    .  3BC5          cmp eax,ebp
> 00415811    .  8983 00030000 mov dword ptr ds:\[ebx+300\],eax
> 00415817    .  74 0D         je short TJMan23.00415826
> 00415819    .  FF15 1C604100 call dword ptr ds:\[<&KERNEL32.GetLastError>\]     ; \[GetLastError
> 0041581F    .  3D B7000000   cmp eax,0B7
> 00415824    .  75 16         jnz short TJMan23.0041583C
> 00415826    >  55            push ebp
> 00415827    .  55            push ebp
> 00415828    .  68 C4B84100   push TJMan23.0041B8C4                            ;  ASCII "系统错误 3，外挂不能运行!"
> 0041582D    .  E8 94E0FFFF   call <jmp.&MFC42.#1200>
> 00415832    .  8B13          mov edx,dword ptr ds:\[ebx\]
> 00415834    .  8BCB          mov ecx,ebx
> 00415836    .  FF92 CC000000 call dword ptr ds:\[edx+CC\]
> 0041583C    >  C783 54020000>mov dword ptr ds:\[ebx+254\],1
> 00415846    .  EB 16         jmp short TJMan23.0041585E
> 
> 00415848    >  55            push ebp
> 00415849    .  55            push ebp
> 0041584A    .  68 00A24100   push TJMan23.0041A200                       ;这里应该是"对不起,不支持三开",不知怎么回事,不能显示'三'
> 0041584F    .  E8 72E0FFFF   call <jmp.&MFC42.#1200>
> 00415854    .  8B03          mov eax,dword ptr ds:\[ebx\]
> 00415856    .  8BCB          mov ecx,ebx
> 00415858    .  FF90 CC000000 call dword ptr ds:\[eax+CC\]
> 0041585E    >^ E9 E1F0FEFF   jmp TJMan23.00404944                      
> 
> * * *
> 
> 
> ```

  
  
虽然实例 3 可以打开,但是进入游戏后,不能启动外挂的热键,  
而实例 1 和实例 2 都可以,什么原因呢?难道实例 3 的代码还有什么问题?  
这样来试试:先运行实例一,然后不运行实例 2,因为实例 2 和实例 3 的代码完全一致,  
区别仅在于数据(2 变成 3).  
00404816       75 7E         jnz short TJMan23.00404896  
这里是跳去实例 2,跳到 00404896 后再来个长跳转 jmp 004157B0,去运行实例 3.  
这里不在 00404816 直接长跳是为了避免数据再次搬家.  
  
结果实例 3 还是不能启动热键,这可是和实例 2 一样的代码啊!  
晕~~找不到原因了...  
还是写下了这篇总结,理一理自己的思路....  
  
【参考知识】从网上搜得,零零散散的  
1.  
在 InitApplication 中创建一个同步对象，如果 GETLASTERROR（）调用返回 ERROR\_ALREADY\_EXISTS 表明已经有一个了  
在 InitInstance 里可以调用：：OpenMutex 函数判断互斥量是否存在，如果没有，调用：：CreateMutex 建立一个互斥量，反之表明已经有一个实例了。退出程序  
  
时调用：：ReleaseMutex 释放创建的互斥量。  
  
2.  
创建互斥量：  
HANDLE CreateMutex(  
  LPSECURITY\_ATTRIBUTES lpMutexAttributes,// 安全信息  
  BOOL bInitialOwner,  // 最初状态，  
  //如果设置为真，则表示创建它的线程直接拥有了该互斥量，而不需要再申请  
  LPCTSTR lpName       // 名字，可以为 NULL，但这样一来就不能被其他线程/进程打开  
);  
打开一个存在的互斥量：  
HANDLE OpenMutex(  
  DWORD dwDesiredAccess,  // 存取方式  
  BOOL bInheritHandle,    // 是否可以被继承  
  LPCTSTR lpName          // 名字  
);  
释放互斥量的使用权，但要求调用该函数的线程拥有该互斥量的使用权：  
BOOL ReleaseMutex(//作用如同 LeaveCriticalSection  
  HANDLE hMutex   // 句柄  
);  
关闭互斥量：  
BOOL CloseHandle(  
  HANDLE hObject   // 句柄  
);  
  
3.  
CreateFileMapping 的函数为指定的文件创建一个文件映射对象，该函数的原形如下：   
HANDLE CreateFileMapping(HANDLE hFile,//用于映射的文件句柄 LPSECURITY?ATTRIBUTES FileMappingAttributes,//内存映射文件的安全描述符 DWORD   
  
Flprotect,//文件映射对象的最大长度的高 32 位 DWORD dwMaximumSizelow,//最大长度的低 32 位 LPCTSTR IPNAME//指定这个内存映射文件的名字)  
  
值得注意的是，参数如果是 OXFFFFFFFF，将在操作系统虚拟内存页面替换文件中创建文件映射对象，而不是使用磁盘文件，同时必须给出这个映射对象的大小。  
  
  
【破解总结】   None  
  
【Greetings】  看雪论坛,FCG 论坛,DFCG 论坛等  
  
  
【完稿时间等】2005.05.13,22:31,天气:晴,广州 

 4dd466d0e1e0cc123afbe504da2462ca