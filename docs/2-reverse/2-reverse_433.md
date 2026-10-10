

# IDA 边界修改插件

> 原文：[`www.pediy.com/kssd/pediy06/pediy6747.htm`](https://www.pediy.com/kssd/pediy06/pediy6747.htm)

精华 5 中提到 IDA 工具使用都是关于修改边界的，试了试其中 bpx 大虾的插件法，总不成功。检查后发现  
是 IDA 升级后提供的 api 已经通过序列号调用，再无 MakeBorder 这样的 id 了。于是试着从应有一定“可  
持续性”出发，编了一个有相似功能的插件（其主要思想是替换 ExtTextOutA），不当之处请各位大侠多  
多指正。就算是抛砖引玉吧。  
  
以下是这个插件的主要代码，包括：  
自编两个函数 Checker 和 BorderPatcher，两个修改了内容的函数 init 和 term(编译好的文件下载)  

> ```
> 代码:static char \*strSeg={"; ※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※  "};
> static char \*strSub={"; 〓〓〓〓〓〓〓  S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  "};
> static char \*strBor={"; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  "};
> static char \*strStar={" ; ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  "};
> 
> static bool idaapi Checker(int Para)
> {
> bool t;
>     t=FALSE;
> \_asm{
>     lodsd
>     cmp ax,203Bh
>     jnz short retur
>     lodsd
>     cmp eax,Para
>     jnz short retur
>     lodsd
>     cmp eax,Para
>     jnz short retur
>     }
>     t=TRUE;
> retur:
>     return t;
> }
> static void idaapi BorderPatcher(HDC, int, int, UINT, CONST RECT \*,LPCSTR lpstr, UINT len, CONST INT \*)
> {
> \_asm{    push esi
>         push edi
>         push ecx
>         sub esp,20h
>         lea esi,\[ebp+8\]
>         mov edi,esp
>         mov ecx,8
>         rep movsd                //传递原调用 Ext 参数
>         mov esi, lpstr
> 
>         cmp len,50h
>         jbe short next\_1
>         mov \[esi+12h\],20h
>         jmp short Call\_ExtTextOut
> next\_1:
>         cmp len,40h
>         jl Call\_ExtTextOut
>         lodsd
>         cmp ax, 3B20h
>         jnz short CheckSub
>         rol eax,8
>         cmp al,0BAh
>         jnz short next\_2
>         mov byte ptr \[esi\-1\],20h
>         jmp Call\_ExtTextOut
> next\_2:
>         lodsd
>         cmp eax, 0cdcdcdcdh
>         jnz short CheckSub
>         mov eax,strStar            //Title border 替换
>         jmp Patche
> 
> CheckSub:
>         mov    esi, lpstr
>         push 0DBDBDBDBh
>         call Checker
>         test al,al
>         mov    esi, lpstr
>         jz short CheckBor
>         mov eax,strSub            //函数 border 替换
>         jmp Patche
> 
> CheckBor:
>         push 0C4C4C4C4h
>         call Checker
>         test al,al
>         jz short CheckSeg
>         mov eax,strBor            //逻辑边界替换
>         jmp Patche
> 
> CheckSeg:
>         mov    esi, lpstr
>         push 0CDCDCDCDh
>         call Checker
>         test al,al
>         jz short Call\_ExtTextOut
>         mov eax,strSeg            //段边界替换
> Patche:        
>         mov \[esp+14h\],eax
> Call\_ExtTextOut: 
>         call ExtTextOutEntry
>         pop ecx
>         pop edi
>         pop esi
>     }
> }
> int idaapi init(void)
> {
> \_asm{
>     push esi
>     push edi
>     push edx
>     push ecx
>     push eax
>     mov edi,\[ebp+4\]                 //得到调用返回地址(idag 中)
>     mov ecx,-1
>     mov al,0e8h
>     cld
> again:
>     repnz scasb                    //查找 call
>     jnz over
>     mov edx,\[edi\]
>     cmp edx,200000h
>     jae short again
>     cmp edx,0
>     jb short again
>     lea edx,\[edi+edx\-2\]            //获取调用目标地址
>     cmp word ptr\[edx\],25ffh            //是 jmp \[xxxxxxx\] ？
>     jnz short again
>     mov edi,\[edx+2\]                //得到 IAT
>     and edi,0fffff000h        
>     jmp short $+7
>     call ExtTextOutA
>     call $+5
>     pop eax
>     sub eax,9
>     add eax,\[eax\]
>     mov eax,\[eax+4\]                //得到 ExtTextOutA RAV
>     mov ecx,800h
>     repnz scasd                    //搜索 ExtTextOutA RAV 在 IAT 中的位置
>     jnz short over
>     sub edi,4            
>     mov ExtTextOutEntry,eax        //保存 ExtTextOutA RAV
>     mov ExtTextOutIAT,edi        //保存其指针
>     mov eax,offset BorderPatcher
>     stosd                        //移花接木：用 BorderPatcher 替换指针
> over:
>     pop eax
>     pop edx
>     pop ecx
>     pop edi
>     pop esi
>     }
>   return PLUGIN\_KEEP;
> }
> void idaapi term(void)
> {
>   unhook\_from\_notification\_point(HT\_UI, sample\_callback);
>   set\_user\_defined\_prefix(0, NULL);
> \_asm{
>     push edi
>     push eax
>     mov eax,ExtTextOutEntry
>     mov edi,ExtTextOutIAT
>     stosd                        //恢复指针
>     pop eax
>     pop edi
>     }  
> 
> }
> 
> ```
> 
>   

 4dd466d0e1e0cc123afbe504da2462ca