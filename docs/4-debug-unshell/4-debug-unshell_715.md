

# hmimys-Packer V1.2、1.3 脱脱脱

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-543.htm`](https://www.pediy.com/kssd/pediy09/pediy09-543.htm)

> 娱乐一下，看论坛没朋友写这个，就忽悠一下！  
>   
> 今天有朋友叫我帮忙脱一壳，查壳后发现是 hmimys-Packer V1.2 -> hmimys，但感觉奇怪，似乎这个壳被某人修改过，代码有所不同（后面证明）：  
>   
> ★一、OD 载入目标程序：  
>   
> /\*1012CD8\*/  call PYG 脱客\_.01012D72           //EP  
> /\*1012CDD\*/  add byte ptr ds:\[eax\],dl  
> /\*1012CDF\*/  add dword ptr ds:\[eax\],eax  
> /\*1012CE1\*/  add byte ptr ds:\[eax\],dl  
> /\*1012CE3\*/  add byte ptr ds:\[ecx\],al  
> /\*1012CE5\*/  in al,dx  
> /\*1012CE6\*/  dec esp  
> /\*1012CE7\*/  add byte ptr ds:\[eax\],al  
> /\*1012CE9\*/  iretd  
>   
> ★二、Alt+M 打开内存镜像:  
>   
> Memory map  
> 地址       大小       属主                        区段       包含                  类型   访问      初始访问  已映射为  
> 01000000   00001000   PYG 脱客\_ 01000000 (自身)               PE 文件头             Imag   R         RWE  
> 01001000   00011000   PYG 脱客\_ 01000000           hmimys     代码                  Imag   R         RWE  
> 01012000   0000E000   PYG 脱客\_ 01000000           hmimys     SFX,输入表,资源       Imag   R         RWE  
> 01020000   00001000   PYG 脱客\_ 01000000           hmimys                           Imag   R         RWE  
>   
> 在第一“PE 文件头”区段上，F2 设置访问中断,F9 运行中断：  
>   
> Memory map, 条目 17  
>  地址=01000000  
>  大小=00001000 (4096.)  
>  属主=PYG 脱客\_ 01000000 (自身)  
>  区段=  
>  包含=PE 文件头  
>  类型=Imag 01001002  
>  访问=R  
>  初始访问=RWE  
>   
> 中断后来到这里：  
>   
> /\*7C930806\*/  cmp word ptr ds:\[ecx\],5A4D  
> /\*7C93080B\*/  jnz short ntdll.7C93082A  
> /\*7C93080D\*/  mov edx,dword ptr ds:\[ecx+3C\]  
> /\*7C930810\*/  cmp edx,10000000  
> /\*7C930816\*/  jnb short ntdll.7C93082A  
> /\*7C930818\*/  lea eax,dword ptr ds:\[edx+ecx\]  
> /\*7C93081B\*/  mov dword ptr ss:\[ebp-1C\],eax  
> /\*7C93081E\*/  cmp dword ptr ds:\[eax\],4550  
> /\*7C930824\*/  jnz ntdll.7C958BB5  
> /\*7C93082A\*/  or dword ptr ss:\[ebp-4\],FFFFFFFF  
> /\*7C93082E\*/  call ntdll.7C92EE02  
> /\*7C930833\*/  retn 4  
>   
> ★三、再次 Alt+M 打开内存镜像:  
>   
> 在第一区段上，F2 设置访问中断,F9 运行中断：  
>   
> Memory map, 条目 18  
>  地址=01001000  
>  大小=00011000 (69632.)  
>  属主=PYG 脱客\_ 01000000  
>  区段=hmimys  
>  包含=代码  
>  类型=Imag 01001002  
>  访问=R  
>  初始访问=RWE  
>   
> 中断后来到这里：  
>   
> /\*10102D3\*/  mov dword ptr ss:\[esp\],eax  
> /\*10102D6\*/  call PYG 脱客\_.010102E7  
> /\*10102DB\*/  inc ebx  
> /\*10102DC\*/  insb  
> /\*10102DD\*/  outsd  
> /\*10102DE\*/  jnb short PYG 脱客\_.01010345  
> /\*10102E0\*/  dec eax  
> /\*10102E1\*/  popad  
> /\*10102E2\*/  outsb  
> /\*10102E3\*/  insb  
> /\*10102E5\*/  add byte ptr gs:\[ebp-1\],dl  
> /\*10102E9\*/  salc  
> /\*10102EA\*/  call eax  
> /\*10102EC\*/  push 0  
> /\*10102EE\*/  call PYG 脱客\_.010102FE  
>   
> 四、Ctrl+S 搜索代码：  
>   
> popad  
> push eax  
>   
> 找到这里：  
>   
> /\*10106C9\*/  popad  
> /\*10106CA\*/  push eax  
> /\*10106CB\*/  push E82BBDF5  
> /\*10106D0\*/  pushad  
> /\*10106D1\*/  push C748640E  
> /\*10106D6\*/  sub esp,-4  
> /\*10106D9\*/  popad  
> /\*10106DA\*/  sub esp,-4  
> /\*10106DD\*/  pop eax  
> /\*10106DE\*/  call PYG 脱客\_.010106E3  
> /\*10106E3\*/  jmp short PYG 脱客\_.010106E6  
> /\*10106E5\*/  outsb  
> /\*10106E6\*/  add ebx,7B30DA6E  
> /\*10106EC\*/  sub ebx,7B30DA6E  
> /\*10106F2\*/  retn                   //在这里 F2 设置断点，经过 2 次中断后飞向 OEP  
>   
> 飞向 OEP:  
>   
> /\*1006420\*/  push ebp               //OEP  
> /\*1006421\*/  mov ebp,esp  
> /\*1006423\*/  push -1  
> /\*1006425\*/  push PYG 脱客\_.01001888  
> /\*100642A\*/  push PYG 脱客\_.010065D0  
> /\*100642F\*/  mov eax,dword ptr fs:\[0\]  
> /\*1006435\*/  push eax  
> /\*1006436\*/  mov dword ptr fs:\[0\],esp  
> /\*100643D\*/  add esp,-68  
> /\*1006440\*/  push ebx  
> /\*1006441\*/  push esi  
> /\*1006442\*/  push edi  
> /\*1006443\*/  mov dword ptr ss:\[ebp-18\],esp  
> /\*1006446\*/  mov dword ptr ss:\[ebp-4\],0  
> /\*100644D\*/  push 2  
> /\*100644F\*/  call dword ptr ds:\[1001160\]  
>   
> ★五、LoadPE 脱壳、Import REC 修复即可！  
>   
> OEP=1006420-1000000=0006420  
>   
> 为了证明我开场的观点，于是用真正的 hmimys-Packer V1.2 加了几个试炼程序（包括脱壳后我们的目标程序）：  
>   
> 脱壳前三步与以上的一样，接着我们来说第四步，也就是找 OEP：（这里目标为脱壳后再加壳的目标程序）  
>   
> ★一（略）  
>   
> ★二（略）  
>   
> ★三（略）  
>   
> 四、Ctrl+B 搜索代码：  
>   
> AB EB E4 AD 50 55 FF 53 04 AB EB E0 C3  
>   
> 来到：  
>   
> /\*1028DB8\*/  lodsb  
> /\*1028DB9\*/  test al,al  
> /\*1028DBB\*/  jnz short PYG 脱客\_.01028DB8  
> /\*1028DBD\*/  lodsw  
> /\*1028DBF\*/  test ax,ax  
> /\*1028DC2\*/  je short PYG 脱客\_.01028DAD  
> /\*1028DC4\*/  lodsb  
> /\*1028DC5\*/  sub esi,3  
> /\*1028DC8\*/  test al,al  
> /\*1028DCA\*/  je short PYG 脱客\_.01028DD4  
> /\*1028DCC\*/  push esi  
> /\*1028DCD\*/  push ebp  
> /\*1028DCE\*/  call dword ptr ds:\[ebx+4\]  
> /\*1028DD1\*/  stosd                       //找到这里  
> /\*1028DD2\*/  jmp short PYG 脱客\_.01028DB8  
> /\*1028DD4\*/  lodsd  
> /\*1028DD5\*/  push eax  
> /\*1028DD6\*/  push ebp  
> /\*1028DD7\*/  call dword ptr ds:\[ebx+4\]  
> /\*1028DDA\*/  stosd  
> /\*1028DDB\*/  jmp short PYG 脱客\_.01028DBD  
> /\*1028DDD\*/  retn                        //在这里 F2 设置断点，中断后 F8 飞向 OEP  
>   
> 飞向 OEP:  
>   
> /\*1006420\*/  push ebp  
> /\*1006421\*/  mov ebp,esp  
> /\*1006423\*/  push -1  
> /\*1006425\*/  push PYG 脱客\_.01001888  
> /\*100642A\*/  push PYG 脱客\_.010065D0  
> /\*100642F\*/  mov eax,dword ptr fs:\[0\]  
> /\*1006435\*/  push eax  
> /\*1006436\*/  mov dword ptr fs:\[0\],esp  
> /\*100643D\*/  add esp,-68  
> /\*1006440\*/  push ebx  
> /\*1006441\*/  push esi  
> /\*1006442\*/  push edi  
> /\*1006443\*/  mov dword ptr ss:\[ebp-18\],esp  
> /\*1006446\*/  mov dword ptr ss:\[ebp-4\],0  
> /\*100644D\*/  push 2  
> /\*100644F\*/  call dword ptr ds:\[1001160\]  
>   
> ★五（略）  
>   
> 原来 hmimys-Packer 1.30 也可以这么脱~~o(∩\_∩)o...  
>   
> 
> 代码:
> 
> * * *
> 
> `hmimys-Packer 1.30 主程序更为简便脱法（亦可应用于附件中的试炼品）：      壳主程序 EP：      /*   /*418360*/  call hmiPack1.004183FA   /*418365*/  add byte ptr ds:[eax+1],al   /*418368*/  add byte ptr ds:[eax],al   /*41836A*/  adc byte ptr ds:[eax],al   /*41836D*/  xlat byte ptr ds:[ebx+al]   /*41836E*/  add byte ptr gs:[eax],al   /*418371*/  push edi   /*418372*/  mov dword ptr ds:[ecx],eax   /*418375*/  add al,dh   /*418377*/  inc ecx   /*418378*/  add byte ptr ds:[eax+6B],dh   /*41837B*/  inc eax   /*41837C*/  add byte ptr ds:[eax],al   /*41837E*/  inc eax   /*41837F*/  inc ecx   /*418380*/  add byte ptr ds:[edi+1D],dh   /*418383*/  cmp byte ptr ds:[eax-53],80   /*418388*/  jl short hmiPack1.0041838A   /*41838A*/  add byte ptr ds:[eax],al   */      Alt+M 打开内存镜像，在第一区段 F2 设置访问断点：      /*   Memory map, 条目 18    地址=00401000    大小=00014000 (81920.)    属主=hmiPack1 00400000    区段=hmimys    包含=代码    类型=Imag 01001002    访问=R    初始访问=RWE   */      F9 运行中断：      /*   /*418719*/  mov byte ptr ds:[eax+ecx],bl   /*41871C*/  jge short hmiPack1.00418727   /*41871E*/  and dword ptr ss:[ebp-4],0   /*418722*/  jmp hmiPack1.00418944   /*418727*/  cmp dword ptr ss:[ebp-4],0A   /*41872B*/  jge short hmiPack1.00418736   /*41872D*/  sub dword ptr ss:[ebp-4],3   /*418731*/  jmp hmiPack1.00418944   /*418736*/  sub dword ptr ss:[ebp-4],6   /*41873A*/  jmp hmiPack1.00418944   */      Ctrl+B 搜索代码：5F 5E 5B C9 C2 14 00      /*   /*418950*/  pop edi   /*418951*/  pop esi   /*418952*/  pop ebx   /*418953*/  leave   /*418954*/  retn 14     // F2 设置访问断点，F9 运行中断后 F7 跟进   */      Ctrl+B 搜索代码：AB EB E4 AD 50 55 FF 53 04 AB EB E0 C3      /*   /*418459*/  stosd   /*41845A*/  jmp short hmiPack1.00418440   /*41845C*/  lodsd   /*41845D*/  push eax   /*41845E*/  push ebp   /*41845F*/  call dword ptr ds:[ebx+4]   /*418462*/  stosd   /*418463*/  jmp short hmiPack1.00418445   /*418465*/  retn        // F2 设置访问断点，F9 运行中断后 F7 跟进，飞向 OEP   */      找到 OEP，LoadPE 脱壳、Import REC 修复即可！      /*   /*406B70*/  push 60   /*406B72*/  push hmiPack1.0040B378   /*406B77*/  call hmiPack1.00407DFC   /*406B7C*/  mov edi,94   /*406B81*/  mov eax,edi   /*406B83*/  call hmiPack1.00407F60   /*406B88*/  mov dword ptr ss:[ebp-18],esp   /*406B8B*/  mov esi,esp   /*406B8D*/  mov dword ptr ds:[esi],edi   /*406B8F*/  push esi   */`
> 
> * * *

> 我这样脱的  
> 
> 代码:
> 
> * * *
> 
> `PEiD 插件查 OEP 为 1004260      OD 载入跑起来，找到输入表的最后项   010012EC   .  A9C43C76      dd      comdlg32.ChooseFontW   010012F0   .  D6483D76      dd      comdlg32.PageSetupDlgW   010012F4   .  CE003C76      dd      comdlg32.CommDlgExtendedError   010012F8      00            db      00      关掉 OD，重新载入（直接重载会退出）   下 bp GetModuleHandleA   数据窗口跟随到输入表最后项地址 010012F4 ，此时为全 0   F9 几次后断下 010012F4 已被填入 IAT   这个时候 CTRL+F9 多次直到返回到程序领空，      0101066D   .  FFD7          call    edi   0101066F   .  83EC 04       sub     esp, 4  //返回到里   01010672   .  50            push    eax   01010673   .  33DB          xor     ebx, ebx   01010675   >  8B45 00       mov     eax, dword ptr [ebp]   01010678   .  8B8418 000000>mov     eax, dword ptr [eax+ebx+1000000]   0101067F   .  A9 00000080   test    eax, 80000000   01010684   .  74 07         je      short 0101068D   01010686   .  25 FFFF0000   and     eax, 0FFFF   0101068B   .  EB 09         jmp     short 01010696   0101068D   >  85C0          test    eax, eax   0101068F   .  74 1D         je      short 010106AE   01010691   .  05 02000001   add     eax, 01000002   01010696   >  894424 04     mov     dword ptr [esp+4], eax   0101069A      FF            db      FF   0101069B   .  D6            salc   0101069C   .  83EC 08       sub     esp, 8   0101069F   .  8B4D 10       mov     ecx, dword ptr [ebp+10]   010106A2   .  898419 000000>mov     dword ptr [ecx+ebx+1000000], eax   010106A9   .  83C3 04       add     ebx, 4   010106AC   .^ EB C7         jmp     short 01010675   010106AE   >  83C4 08       add     esp, 8   010106B1   .  83C5 14       add     ebp, 14   010106B4   .  837D 10 00    cmp     dword ptr [ebp+10], 0   010106B8   .^ 75 A9         jnz     short 01010663   010106BA   .  45            inc     ebp   010106BB   .  68 4657A8E4   push    E4A85746   010106C0   .  EB 03         jmp     short 010106C5   010106C2      48            db      48                               ;  CHAR 'H'   010106C3      8C            db      8C   010106C4      B7            db      B7   010106C5   >  83EC FC       sub     esp, -4   010106C8   .  4D            dec     ebp   010106C9   .  61            popad   010106CA   .  50            push    eax   010106CB   .  68 F5BD2BE8   push    E82BBDF5   010106D0   .  60            pushad   010106D1   .  68 0E6448C7   push    C748640E   010106D6   .  83EC FC       sub     esp, -4   010106D9   .  61            popad   010106DA   .  83EC FC       sub     esp, -4   010106DD   .  58            pop     eax   010106DE   .  E8 00000000   call    010106E3   010106E3   $  EB 01         jmp     short 010106E6   010106E5   .  6E            outs    dx, byte ptr es:[edi]   010106E6   >  81C3 6EDA307B add     ebx, 7B30DA6E   010106EC   .  81EB 6EDA307B sub     ebx, 7B30DA6E   010106F2   .  C3            retn         最后这个 RETN 了两次，第二次指向 PEiD 插件找到的 OEP1004260   dump,fix,运行成功`
> 
> * * *

 4dd466d0e1e0cc123afbe504da2462ca