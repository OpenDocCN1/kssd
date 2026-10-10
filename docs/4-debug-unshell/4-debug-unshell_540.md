

# nspack3.5 主程序脱壳分析(Aspr SKE 2.X)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-818.htm`](https://www.pediy.com/kssd/pediy07/pediy7-818.htm)

nspack 3.5 主程序脱壳介绍  
xp sp2  
flyodbg  
Aspr SKE 2.X  
  
  
零  需要哪里就重新来过重点分析哪里  
come on let's go  
  
  
一  PEiD 可以不用， 但 LordPE 一定要先加载看看  
.rsrc 段上面有三个区段，没有名字，不过可以猜到是.text、.rdata 和.data 段，是 VC 的程序  
  
  
二  看看能不能在 OD 下跑起来  
OD 载入 nspack.exe，忽略所有异常，清除所有断点， 打上 IsDebuggerPresent 插件  
F9 运行  gogogo~  
正常情况下能跑起来，alt+e 看看加载的 dll，看到有 msvcrt.dll，没有发现 mfc 的 dll  
所以是普通 VC 或 MFC 静态  
我猜我猜我猜猜猜  
  
  
三  到 oep 看看  
重来，OD 截入，忽略所有...清除...打上..插件  
到 GetVersion 的末尾 retn 下断  
   
    7C8114AB kernel32.GetVersion      64:A1 18000000   mov eax,dword ptr fs:\[18\]  
    7C8114B1                          8B48 30          mov ecx,dword ptr ds:\[eax+30\]  
    7C8114B4                          8B81 B0000000    mov eax,dword ptr ds:\[ecx+B0\]  
    7C8114BA                          0FB791 AC000000  movzx edx,word ptr ds:\[ecx+AC\]  
    7C8114C1                          83F0 FE          xor eax,FFFFFFFE  
    7C8114C4                          C1E0 0E          shl eax,0E  
    7C8114C7                          0BC2             or eax,edx  
    7C8114C9                          C1E0 08          shl eax,8  
    7C8114CC                          0B81 A8000000    or eax,dword ptr ds:\[ecx+A8\]  
    7C8114D2                          C1E0 08          shl eax,8  
    7C8114D5                          0B81 A4000000    or eax,dword ptr ds:\[ecx+A4\]  
    7C8114DB                          C3               retn                              //这里下断  
  
F9 运行，断下，F8 返回，向上看看，看到 oep 了  
  
    00486C68                          55               push ebp  
    00486C69                          8BEC             mov ebp,esp  
    00486C6B                          6A FF            push -1  
    00486C6D                          68 38FB4A00      push nSpack.004AFB38  
    00486C72                          68 50554800      push nSpack.00485550  
    00486C77                          64:A1 00000000   mov eax,dword ptr fs:\[0\]  
    00486C7D                          50               push eax  
    00486C7E                          64:8925 00000000 mov dword ptr fs:\[0\],esp  
    00486C85                          83EC 58          sub esp,58  
    00486C88                          53               push ebx  
    00486C89                          56               push esi  
    00486C8A                          57               push edi  
    00486C8B                          8965 E8          mov dword ptr ss:\[ebp-18\],esp  
    00486C8E                          FF15 6C724A00    call dword ptr ds:\[4A726C\]                      ; kernel32.GetVersion  
    00486C94                          33D2             xor edx,edx                      // GetVersion 返回到这里  
  
VC6 会 GetVersion，VC7 会 GetVersionExA，可以都在末尾下断，到时候看哪个像 oep 附近就是了  
  
  
四  输入表  
GetVersion 是在\[4A726C\]，那么到那个地方向上看看，向下看看，找输入表的范围  
结果  
4A7000 到 4A7688  
输入表没有加密 :)  
有时候 aspr SKE 2.X 会把输入表加密，把一部分导入函数地址改的乱七八糟，可那些加密的地址是不存在的。  
那它怎么用那里的导入函数呢? 其实它把代码中所有对加密导入函数的调用从原先的 call \[IAT\]或 jmp \[IAT\]  
改成了 call 00EA0000 这种样子，从它自己的 call 00EA0000 进入导入函数，这样那些加密的导入函数就可以随便  
写一个不存在的地址了。  
  
如果输入表加密了，你可以这样完整修复：  
OD 截入，忽略所有...清除...打上..插件  
随便对一个导入函数地址下内存写断点，比如这里 GetVersion 的 4A726C  
断了若干次后到这里  
    00C5764D        8902          mov dword ptr ds:\[edx\],eax                 ; // eax 指向 GetVersion 的地址，写入 ebx = 4A726C  
    00C5764F        E9 20010000   jmp 00C57774  
  
    00C57774        8B45 0C       mov eax,dword ptr ss:\[ebp+C\]  
    00C57777        8300 04       add dword ptr ds:\[eax\],4  
    00C5777A        8D85 FAFEFFFF lea eax,dword ptr ss:\[ebp-106\]  
    00C57780        3BF8          cmp edi,eax  
    00C57782        74 07         je short 00C5778B  
    00C57784        8BC7          mov eax,edi  
    00C57786        E8 D9ADFDFF   call 00C32564  
    00C5778B        5F            pop edi  
    00C5778C        5E            pop esi  
    00C5778D        5B            pop ebx  
    00C5778E        8BE5          mov esp,ebp  
    00C57790        5D            pop ebp  
    00C57791        C2 1000       retn 10                                    // F8 下来后这里返回  
  
返回后  
    00C5795A        E8 59FCFFFF   call 00C575B8                              //关键的 call 进去看  
    00C5795F        0FB707        movzx eax,word ptr ds:\[edi\]                //上面返回后是回到这里  
    00C57962        83C0 02       add eax,2  
    00C57965        03F8          add edi,eax  
    00C57967        8A1F          mov bl,byte ptr ds:\[edi\]  
    00C57969        47            inc edi  
    00C5796A        3A5E 34       cmp bl,byte ptr ds:\[esi+34\]  
    00C5796D      ^ 0F85 77FFFFFF jnz 00C578EA                               //继续当前 dll 的下一个导入函数  
    00C57973        8BDF          mov ebx,edi  
    00C57975        8B03          mov eax,dword ptr ds:\[ebx\]  
    00C57977        85C0          test eax,eax  
    00C57979      ^ 0F85 0AFFFFFF jnz 00C57889                               //下一个 dll  
  
C575B8 这个 call 就是对输入表的处理  
  
    00C575B8        55            push ebp  
    00C575B9        8BEC          mov ebp,esp  
    00C575BB        81C4 F8FEFFFF add esp,-108  
    00C575C1        53            push ebx  
    00C575C2        56            push esi  
    00C575C3        57            push edi  
    00C575C4        8B55 14       mov edx,dword ptr ss:\[ebp+14\]  
    00C575C7        8B5D 08       mov ebx,dword ptr ss:\[ebp+8\]  
    00C575CA        8DBD FAFEFFFF lea edi,dword ptr ss:\[ebp-106\]  
    00C575D0        8BC2          mov eax,edx  
    00C575D2        48            dec eax  
    00C575D3        83E8 02       sub eax,2  
    00C575D6        0FB630        movzx esi,byte ptr ds:\[eax\]  
    00C575D9        8B45 10       mov eax,dword ptr ss:\[ebp+10\]  
    00C575DC        83E8 02       sub eax,2  
    00C575DF        0FB600        movzx eax,byte ptr ds:\[eax\]  
    00C575E2        3B43 2C       cmp eax,dword ptr ds:\[ebx+2C\]  
    00C575E5        76 06         jbe short 00C575ED                         //上面不去管它，这个跳转肯定满足  
  
    00C575ED        33C0          xor eax,eax  
    00C575EF        8A43 3B       mov al,byte ptr ds:\[ebx+3B\]  
    00C575F2        3BF0          cmp esi,eax                                // 这里开始了 4 种情况的比较  
    00C575F4        75 5E         jnz short 00C57654  
  
C575F2 的 cmp esi, eax 开始了 4 种类型的比较  
当前导入函数的类型是放在 esi 中，你可以在这里下个断点，然后一个一个看下来  
第 1 种类型：用第 1 个密钥，还原真实导入函数地址，这里不防设 esi 值为 1  
第 2 种类型：用第 2 个密钥，还原真实导入函数地址，这里不防设 esi 值为 2  
第 3 种类型：用第 2 个密钥，不作任何处理，这里不防设 esi 值为 3  
第 4 种类型：GetProcAddress，这里不防设 esi 值为 4  
  
可见那些加密的导入函数地址，也就是第 3 种类型，与其说是加密，不如说是壳没有去处理  
既然它和第 2 种类型处理方式一样，可以在 cmp esi, eax 这个点，修改 esi 中的值，把第 3 种  
情况改成第 2 种情况就可以了  
或者你也可以跑下去，把一些 jnz 或 je 改成 magic jmp，让第 3 种情况跑到第 2 种情况也可以  
  
需要说明的是 esi 的对每个 aspr 加壳的程序都是随机的，只要多看几个，就知道是哪个改哪个了  
  
  
  
五  取得 call 00EA0000 的所有地址  
按照上面所说的，可以在 GetVerion 返回后 dump 出来，然后用 ImortREC 修复输入表，把 oep 86c68 写回去  
不妨叫做 unpack1.exe，用 od 载它跑一下，它会告诉你 call 00EA0000 挂了，然后按 F12(pause)，从堆栈的  
返回地址知道是这个让你挂了  
    00489AAB        E8 5065A100   call 00EA0000  
    00489AB0        1283 4E04FF6A adc al,byte ptr ds:\[ebx+6AFF044E\]  
  
EA0000 是什么呢？  
它是把导入函数调用的变形，原来的 call \[IAT\] 和 jmp \[IAT\]的变形  
EA0000 是壳用 VirtualAlloc 的空间，不在区段中  
在我的机机子上现在是 call 00EA0000，在你的机子上就可能是 call 1230000  
也就是说，call 00EA0000 是壳经过计算后写入的  
于是我想看看，在它写入 call 00EA0000 前是什么样子  
  
  
OD 载入 nspack.exe，忽略所有异常，清除所有断点， 打上 IsDebuggerPresent 插件  
对 489AAC 下内存写入断点 (因为 489AAB 是'E8'，我们要的是后 4 个字节)  
  
若干次后会断在这里  
    00C5BAD3        8945 00       mov dword ptr ss:\[ebp\],eax           // 断在这儿：ebp 指向 489AAC，eax 写入后，使那个地方变成 call 00EA0000  
    00C5BAD6        6A 0A         push 0A  
    00C5BAD8        E8 7F9AFEFF   call 00C4555C  
    00C5BADD        8BC8          mov ecx,eax  
    00C5BADF        038B E4000000 add ecx,dword ptr ds:\[ebx+E4\]  
    00C5BAE5        8BD6          mov edx,esi  
    00C5BAE7        8BC3          mov eax,ebx  
    00C5BAE9        E8 8EE5FFFF   call 00C5A07C  
    00C5BAEE        FF0C24        dec dword ptr ss:\[esp\]  
    00C5BAF1        03B3 E4000000 add esi,dword ptr ds:\[ebx+E4\]  
    00C5BAF7        833C24 00     cmp dword ptr ss:\[esp\],0  
    00C5BAFB      ^ 0F87 55FEFFFF ja 00C5B956                          //如果还有需要处理就跳上去  
    00C5BB01        53            push ebx  
    00C5BB02        E8 5D000000   call 00C5BB64  
    00C5BB07        0183 EC000000 add dword ptr ds:\[ebx+EC\],eax  
    00C5BB0D        B0 01         mov al,1  
    00C5BB0F        83C4 24       add esp,24  
    00C5BB12        5D            pop ebp  
    00C5BB13        5F            pop edi  
    00C5BB14        5E            pop esi  
    00C5BB15        5B            pop ebx  
    00C5BB16        C3            retn                                 //这里结束  
  
  
正如我所说，call 00EA0000 完全是在代码段解码后，申请空间，现在我申请到的是 EA0000  
那么它就将需要变形的地方计算后写成 call 00EA0000，如果你申请到的是 1230000，那么你  
是 call 1230000  
  
断在这里，我当然想看一看在改写成 call 00EA0000 之前，那些地址是不是正常的  
很可惜，那里在改写成 call 00EA0000，本身就是乱掉的。  
或者在某个时候能知道那些变形地址原先的真实情况，可惜我找不到。  
也许只有作者知道在哪里  
也许根本就找不到  
因为根本就不需要  
对于 call 00EA0000，它加密前只要知道 2 件事，1.本身所在的地址 2.IAT 中的位置  
对于 call 00EA0000，现在也只要知道 2 件事，1.本身所在的地址 2.最后要去的导入函数的地址  
它没有理由记录 IAT 中的位置  
我们要做的是找出最后到达的导入函数的地址，然后找出它在 IAT 中的位置  
改成原先的 call \[IAT\] 或 jmp \[IAT\]  
  
回到正题，当我们断下时，前面可能已经处理若干个了  
要想得到全部的表  
你有好几种选择  
1. 到 oep 后，写一段代码搜索出所有的 call 00EA0000 的地址  
2. 想办法第一时间断在上面这个地方，即 00C5BAD3，ebp-1 就是变形的地址，保存所有的 ebp-1  
3. 也许内存中本身存在这张表，我没有去找，你可以找找  
  
要找全他们并不难:)  
  
啊，还有一个要说明的  
在写入每一处的 call 00EA0000 时，上面的流程会经过这里  
    00C5B981        FFD2          call edx                           //call edx 结果在 eax  
    00C5B983        807B 20 00    cmp byte ptr ds:\[ebx+20\],0         // eax 可能是 1 或 0  
    00C5B987        0F85 3D010000 jnz 00C5BACA  
  
如果是 1，当前这个 call 00EA0000 处运行时，会重新回到调用地址，再进入导入函数  
如果是 0，当前这个 call 00EA0000 进入导入函数后出来(好像是废话)，不过这种方式比较邪恶，它可能做更多的事情  
下面我会讲到  
  
  
  
六  call 00EA0000 的修复  
有没有想过一个有意思的问题，所有这样的调用都是进入 EA0000 一个地方，但是壳却知道最后  
目的地址是哪一个导入函数，它是怎么判断的呢？  
当到了 EA0000，壳能看到什么？  
1. 参数  
2. 返回地址  
第 1 种情况：鬼知道我会传什么参数，多少个参数，它不能作为评判标准的  
第 2 种情况：只有你了，Aspr 存着一张表，它记录了所有 call 00EA0000 的返回地址和最后导入函数的 1 对 1 关系  
  
它是加密的  
我们要做的是找出这张表，或者找到 1 个点能确定它们 1 对 1 的关系  
  
简单说一下进入 EA0000 后发生了什么，一共三层  
  
第一层：保存所有当前寄存器 (出来后还要继续运行的，不能影响后面，不过它不是明目张胆的 pushad)  
第二层：1. 决定是哪一种方式的导入函数调用  
           a. 第一种方式：将 call 00EA0000 变成 call F00004 之类，出来后再次从原地进入 F00004 进入导入函数  
           b. 第二种方式：直接带着参数进入导入函数  
        2. 决定这个调用是 call (ff15)还是 jmp (ff25)  
           不要以为 C 的都是 call，delphi 的都是 jmp  
           c. 如果是 call (ff15)，返回地址要+1 ，比如 inc \[esp\]，因为 call 00EA0000 占 5 个字节，call (ff15)占 6 个字节  
           d. 如果是 jmp (ff25)，要 esp+4，想一下就知道原因了:)  
        3. 如果是 1.b 的情况，可能有更邪恶的对下一行的偷代码，我一直没有找到好的方式解决它:(  
第三层：恢复所有的寄存器返回  
  
  
对于第一层的和第三层的操作，只要一路 F7 即可  
当你看到  
    00EA0166        2BDA          sub ebx,edx  
    00EA0168        FFD3          call ebx                  //F7 进入第二层  
就知道要 F7 进入第二层了，当然别的 aspr 的壳可能是 call eax 或 call esi 等等  
到了第二层，代码比较工整了，可以一路 F8  
最后  
    00EB00B9        5C            pop esp  
    00EB00BA        FF6424 FC     jmp dword ptr ss:\[esp-4\]  //从第三层返回  
是第三层回来，上面已提到，回来可能是回到原处 call 到一个新的地方进入导入函数，也可能就是完成回来  
  
  
因此重点讲讲第二层  
一路 F8  
可以看到这里  
    00C5B48F       /75 63         jnz short 00C5B4F4                       //比较 call 00EA0000 返回地址的密文，不是就跳上去继续找  
    00C5B491       |807B 20 00    cmp byte ptr ds:\[ebx+20\],0               //找到了当前 call 00EA0000 的处理情况了  
    00C5B495       |74 3C         je short 00C5B4D3  
    00C5B497       |8B45 E4       mov eax,dword ptr ss:\[ebp-1C\]  
    00C5B49A       |0FB640 09     movzx eax,byte ptr ds:\[eax+9\]  
    00C5B49E       |8D0440        lea eax,dword ptr ds:\[eax+eax\*2\]  
    00C5B4A1       |8B5483 68     mov edx,dword ptr ds:\[ebx+eax\*4+68\]  
    00C5B4A5       |8B45 FC       mov eax,dword ptr ss:\[ebp-4\]  
    00C5B4A8       |FFD2          call edx                                 //和第五章最后说的情况一下，再次比较是哪一种方式  
    00C5B4AA       |3C 01         cmp al,1                                 //eax 为 1 则是 a 情况，为 0 则是 b 情况  
    00C5B4AC       |75 25         jnz short 00C5B4D3  
    00C5B4AE       |56            push esi  
    00C5B4AF       |8D45 FC       lea eax,dword ptr ss:\[ebp-4\]  
    00C5B4B2       |50            push eax  
    00C5B4B3       |8B45 14       mov eax,dword ptr ss:\[ebp+14\]  
    00C5B4B6       |50            push eax  
    00C5B4B7       |8B45 18       mov eax,dword ptr ss:\[ebp+18\]  
    00C5B4BA       |50            push eax  
    00C5B4BB       |8B45 0C       mov eax,dword ptr ss:\[ebp+C\]  
    00C5B4BE       |50            push eax  
    00C5B4BF       |8B45 F0       mov eax,dword ptr ss:\[ebp-10\]  
    00C5B4C2       |50            push eax  
    00C5B4C3       |8B4D 1C       mov ecx,dword ptr ss:\[ebp+1C\]  
    00C5B4C6       |8B55 10       mov edx,dword ptr ss:\[ebp+10\]  
    00C5B4C9       |8BC3          mov eax,ebx  
    00C5B4CB       |E8 C0010000   call 00C5B690                           // a 情况这里 F7 进去  
    00C5B4D0       |EB 01         jmp short 00C5B4D3  
    00C5B4D2       |E8 8D45FC50   call 51C1FA64  
    00C5B4D7       |8B45 14       mov eax,dword ptr ss:\[ebp+14\]  
    00C5B4DA       |50            push eax  
    00C5B4DB       |8B45 18       mov eax,dword ptr ss:\[ebp+18\]  
    00C5B4DE       |50            push eax  
    00C5B4DF       |8B45 0C       mov eax,dword ptr ss:\[ebp+C\]  
    00C5B4E2       |50            push eax  
    00C5B4E3       |8B45 F0       mov eax,dword ptr ss:\[ebp-10\]  
    00C5B4E6       |50            push eax  
    00C5B4E7       |8B4D 1C       mov ecx,dword ptr ss:\[ebp+1C\]  
    00C5B4EA       |8B55 10       mov edx,dword ptr ss:\[ebp+10\]  
    00C5B4ED       |8BC3          mov eax,ebx  
    00C5B4EF       |E8 64F1FFFF   call 00C5A658                           // b 情况这里 F7 进去  
  
  
先看 a 情况吧，进去后一路 F8  
  
很快到了这里  
    00C5B7DD        8B45 F4       mov eax,dword ptr ss:\[ebp-C\]  
    00C5B7E0        8B80 E0000000 mov eax,dword ptr ds:\[eax+E0\]  
    00C5B7E6        0345 E4       add eax,dword ptr ss:\[ebp-1C\]  
    00C5B7E9        8945 FC       mov dword ptr ss:\[ebp-4\],eax            //到了这里 eax 的值就是导函数的地址了:)  
不过我觉得这个点不太好，再往下 F8  
    00C5B7EC        33C0          xor eax,eax  
    00C5B7EE        8AC3          mov al,bl  
    00C5B7F0        0145 10       add dword ptr ss:\[ebp+10\],eax  
    00C5B7F3        57            push edi  
    00C5B7F4        6A 00         push 0  
    00C5B7F6        8D4D E0       lea ecx,dword ptr ss:\[ebp-20\]  
    00C5B7F9        8B45 F4       mov eax,dword ptr ss:\[ebp-C\]  
    00C5B7FC        8B40 3C       mov eax,dword ptr ds:\[eax+3C\]  
    00C5B7FF        8B55 FC       mov edx,dword ptr ss:\[ebp-4\]  
    00C5B802        E8 6DB9FFFF   call 00C57174  
    00C5B807        8945 FC       mov dword ptr ss:\[ebp-4\],eax  
    00C5B80A        8B45 E0       mov eax,dword ptr ss:\[ebp-20\]  
    00C5B80D        8B00          mov eax,dword ptr ds:\[eax\]  
    00C5B80F        E8 C0E6FFFF   call 00C59ED4  
    00C5B814        8BD0          mov edx,eax  
    00C5B816        0255 DF       add dl,byte ptr ss:\[ebp-21\]  
    00C5B819        8B4D FC       mov ecx,dword ptr ss:\[ebp-4\]          //这个点比较好  
  
到了这里 \[ebp-4C\]是我们需要的导入函数的地址，dl 中的值决定了是 call(ff15)还是 jmp(ff25)  
dl 中的值不同的程序是随机，找几个 call 00EA0000 进去出来看一下就知道当前的程序中哪个对应 ff15，哪个对应 ff25 了  
  
  
再来看看 b 情况，进去后也是一路 F8  
  
    00C5A7E7        3A45 EF       cmp al,byte ptr ss:\[ebp-11\]          //al 和 a 情况中的 dl 一样，决定是 ff15 还是 ff25  
    00C5A7EA        0F85 9C000000 jnz 00C5A88C  
    00C5A7F0        EB 01         jmp short 00C5A7F3  
  
ff15 和 ff25 产生的分支分别能到下面  
   00C5A7F3        8B45 F4       mov eax,dword ptr ss:\[ebp-C\]  
   00C5A7F6        8B80 E0000000 mov eax,dword ptr ds:\[eax+E0\]  
   00C5A7FC        0145 FC       add dword ptr ss:\[ebp-4\],eax  
  
    00C5A8A5        8B45 F4       mov eax,dword ptr ss:\[ebp-C\]  
    00C5A8A8        8B80 E0000000 mov eax,dword ptr ds:\[eax+E0\]  
    00C5A8AE        0145 FC       add dword ptr ss:\[ebp-4\],eax  
C5A7FC 或 C5A8AE 做完后\[ebp-4\] 是我们需要的导入函数的地址  
再看看\[ebp-2c\]，如果它是 FFFFFFFF，说明这个导入函数调用是干净的  
如果它有值，表示它的下一行也偷了。具体处理可能对它下个硬件访问断点再跟踪  
不过我比较没耐心  
我喜欢把不干净的这些地方扣出来  
然后跑过去猜  
一般偷的都是  
mov esi,eax 或 mov edi,eax 等等  
找到了这些点，写脚本也好，写代码恢复也好，修复就不难了  
  
  
  
七  stolen oep  
这个例子中没有 stolen oep，所以没什么好讲，有兴趣的看看 loveboom 的文章  
文章可能比较老，但是现在还是适用的  
  
  
  
八  最后一些说明  
到了这里差不多结束了，你可以像 syscom 那样，扫描所有有导入函数变形地址进行修复了  
其实把原理搞清楚了，修复的时候就算碰到状况也就能知道怎么处理  
脱 aspr 并不需要从头跟到尾，只要重点的地方耐心分析就可以了，只要耐心，你能发现更多一些东西:)  
  
  
shoooo  
12 月 11 日 

 4dd466d0e1e0cc123afbe504da2462ca