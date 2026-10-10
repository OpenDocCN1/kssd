

# 小菜写的第一个壳

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-511.htm`](https://www.pediy.com/kssd/pediy09/pediy09-511.htm)

> 【文章标题】: 一个超级简单的“壳”  
> 【文章作者】: bithaha  
> 【编写语言】: masm32  
> 【使用工具】: radasm  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   不敢说原创，载入原输入表的代码几乎都是抄袭《软件加密技术内幕》上面的，谢谢看雪论坛出品如此好的一本书。![](img/4ec4a010bb6e6d79731981cd94fbb640.jpg "尴尬")   
>   这个壳非常简单，对加壳以后的程序进行调试几乎发现不了壳的存在，汗一个。但是自己收获颇多。大家都喜欢说：来自看雪，回到看雪。  
>   .386  
>   .model flat,stdcall  
>   option casemap:none  
>   include windows.inc  
>   include user32.inc  
>   includelib user32.lib  
>   include kernel32.inc  
>   includelib kernel32.lib  
>   include comdlg32.inc  
>   includelib comdlg32.lib  
>   IDC\_EDIT1 EQU 1001  
>   IDC\_STATIC EQU 1002  
>   IDC\_PASS EQU  1003  
>   ICO\_ISCC EQU 1004  
>   .data?  
>   szFileName db MAX\_PATH dup(?)  
>   hFile dd ?  
>   szBuffer db 1024h dup(?)  
>   hInstance dd ?  
>   ReadNumber dd ?  
>   ReadBuffer dd ?  
>   HeaderBuffer dd ?  
>   peRva dd ? ;文件头在文件中的偏移地址  
>   peshell dd ?  
>   ShellSize dd ?  
>   Rav\_jiami1 dd ?  
>   Size\_jiami1 dd ?  
>   FileSize dd ?  
>   .const  
>   lpstrFilter db 'All Files(\*.\*)',0,'\*.\*',0,0  
>   szdlg db 'DLG\_MAIN',0  
>   .code  
>     
>   GetIntegral proc int1,int2;>>>>对齐数据  
>           mov eax,int1  
>           mov edx,0  
>           div int2  
>           mov eax,int1  
>           sub eax,edx  
>           add eax,int2  
>           ret  
>   GetIntegral endp  
>     
>   AddShell proc   
>       invoke CreateFile,addr szFileName,GENERIC\_READ or GENERIC\_WRITE,FILE\_SHARE\_WRITE,0,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,0  
>       mov hFile ,eax  
>       invoke GetFileSize,hFile,NULL  
>       mov FileSize,eax  
>       invoke SetFilePointer,hFile,3ch,0,0;3ch 指向"pe"标志  
>       invoke ReadFile,hFile,addr ReadBuffer,4,addr ReadNumber,0  
>       mov ecx,ReadBuffer  
>       mov peRva,ecx  
>       add ecx,50h  
>       invoke SetFilePointer,hFile,ecx,0,FILE\_BEGIN  
>       invoke ReadFile,hFile,addr ReadBuffer,4h,addr ReadNumber,0  
>       invoke VirtualAlloc,NULL,addr ReadBuffer,MEM\_COMMIT,PAGE\_READWRITE  
>       mov HeaderBuffer,eax  
>       invoke SetFilePointer,hFile,0,0,FILE\_BEGIN  
>       invoke ReadFile,hFile,HeaderBuffer,addr ReadBuffer,addr ReadNumber,0  
>       mov ecx,HeaderBuffer  
>       add ecx,peRva  
>       mov esi,ecx  
>       movzx ecx,word ptr \[esi+6h\];NumberOfSections  
>       movzx edi,word ptr \[esi+14h\];IMAGE\_OPTIONAL\_HEADER32 结构的大小  
>       add edi,esi  
>       add edi,18h;esi--->pe 头的位置,edi---->节表的位置  
>   GetSections:  
>       .repeat   
>       push ecx  
>       or dword ptr \[edi+24h\],0c0000000h  
>       mov eax,dword ptr \[edi+14h\];eax--->节在文件中的偏移  
>       invoke SetFilePointer,hFile,eax,NULL,FILE\_BEGIN  
>       mov ebx,dword ptr \[edi+0ch\];edi+0c--->该节的 rva  
>       add ebx,HeaderBuffer  
>       mov ecx,dword ptr \[edi+10h\];在文件中对齐的尺寸  
>       invoke ReadFile,hFile,ebx,ecx,addr ReadNumber,0  
>       cmp  dword ptr \[edi\],'xet.'  
>       jz jiami;加密.text 段  
>           add edi,28h  
>           pop ecx  
>       dec ecx  
>       jcxz next  
>       jmp GetSections  
>       ;>>>>>>>>>>>>>>>>>>>>>>>>>加密 text 节>>>>>>>>>>>>>>>>>>>>>>>>>>  
>   jiami:  
>            mov ecx,dword ptr \[edi+10h\]  
>            mov ebx,dword ptr \[edi+0ch\]  
>            add ebx,HeaderBuffer  
>   jiami\_loop:  
>            xor byte ptr \[ebx\],13h  
>       inc ebx  
>       loop jiami\_loop  
>       mov ebx,dword ptr \[edi+14h\]  
>       invoke SetFilePointer,hFile,ebx,NULL,FILE\_BEGIN  
>       mov ecx,dword ptr \[edi+10h\]  
>       mov ebx,dword ptr \[edi+0ch\]  
>       add ebx,HeaderBuffer  
>       invoke WriteFile,hFile,ebx,ecx,addr ReadNumber,NULL;写入文件  
>       mov eax,dword ptr \[edi+0ch\]  
>       mov Rav\_jiami1,eax  
>       mov ecx,dword ptr \[edi+10h\]  
>       mov Size\_jiami1,ecx  
>       add edi,28h  
>       pop ecx  
>       dec ecx  
>            jcxz next  
>            .until FALSE  
>    next:  
>          push edi ;新快表起点  
>          ;>>>>>>>>>>>>>>>>>>>>>>>>保存相关信息>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>          ;把部分数据保存到壳中  
>          ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>           push esi  
>           invoke VirtualAlloc,NULL,2000h,MEM\_COMMIT ,PAGE\_READWRITE  
>           mov peshell,eax  
>           lea esi,shellstart  
>           mov ecx,shellend-shellstart  
>           mov edi,peshell  
>           rep movsb  
>           pop esi;esi---->pe 头的位置  
>           mov edi,peshell  
>           add edi,OEP-shellstart  
>           mov eax,dword ptr \[esi+28h\];原文件 OEP  
>           mov dword ptr \[edi\],eax;保存 OEP RVA  
>           mov edi,peshell  
>           add edi,Import-shellstart  
>           mov eax,dword ptr \[esi+80h\]  
>           mov dword ptr \[edi\],eax;保存输入表 RVA  
>           mov edi,peshell  
>           add edi,RVA\_jiami-shellstart  
>          mov eax,Rav\_jiami1;加密段的 rva  
>           mov dword ptr \[edi\],eax  
>           mov eax,Size\_jiami1  
>           mov edi,peshell  
>           add edi,SIZE\_jiami-shellstart  
>           mov dword ptr \[edi\],eax  
>         ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>         ;添加新节  
>         ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>          pop edi   
>          mov dword ptr \[edi\],'csi.'  
>          mov ecx,shellend-shellstart  
>          invoke GetIntegral,ecx,dword ptr \[esi+38h\];esi+38h------->在内存中的对齐粒度  
>          mov dword ptr \[edi+08h\],eax  
>          mov eax,dword ptr \[esi+50h\];esi+50h---------->内存中整个 pe 文件尺寸  
>          mov dword ptr \[edi+0ch\],eax;----------->等于新节的 rva  
>          invoke GetIntegral,ecx,dword ptr \[esi+3ch\];esi+38h--------->文件中的对其粒度  
>          mov dword ptr \[edi+10h\],eax  
>          mov ShellSize,eax  
>          mov eax,dword ptr \[edi-14h\]  
>          add eax,dword ptr \[edi-18h\]  
>          mov dword ptr \[edi+14h\],eax  
>          mov dword ptr \[edi+24h\],0c0000040h  
>          inc word ptr \[esi+6h\]  
>         ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>         ;修改相关文件头  
>         ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>          mov eax,dword ptr \[edi+0ch\]  
>          mov dword ptr \[esi+28h\],eax;修改 Oep  
>          invoke GetIntegral,ecx,dword ptr \[esi+38h\]  
>          add eax,dword ptr \[esi+50h\]  
>          mov ebx,dword ptr \[esi+50h\]  
>          mov dword ptr \[esi+50h\],eax  
>          mov eax,ebx  
>          add eax,ImportTable-shellstart  
>          mov dword ptr \[esi+80h\],eax  
>          mov dword ptr \[esi+7ch\],ImportTableend-ImportTable  
>         ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>         ;修改外壳输入表部分相对地址  
>         ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>         mov ecx,dword ptr \[esi+80h\]  
>         mov eax,peshell  
>         add eax,ImportTable-shellstart  
>         add dword ptr \[eax\],ecx  
>         mov eax,peshell  
>         add eax,Import\_Name-shellstart  
>         add dword ptr \[eax\],ecx  
>         mov eax,peshell  
>         add eax,Import\_FirstThunk-shellstart  
>         add dword ptr \[eax\],ecx  
>         mov eax,peshell  
>         add eax,THUNK\_DATA1-shellstart  
>         add dword ptr \[eax\],ecx  
>         mov eax,peshell  
>         add eax,THUNK\_DATA2-shellstart  
>         add dword ptr \[eax\],ecx  
>         mov eax,peshell  
>         add eax,THUNK\_DATA3-shellstart  
>         add dword ptr \[eax\],ecx  
>       
>        ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>        ;把外壳写入文件,把修改的文件头写入文件  
>        ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>    
>         mov ecx,ShellSize  
>         invoke SetFilePointer,hFile, ecx,0,FILE\_END  
>         invoke SetEndOfFile,hFile  
>         mov eax,FileSize  
>         invoke SetFilePointer,hFile,eax,0,FILE\_BEGIN  
>         invoke WriteFile,hFile,peshell,ecx,addr ReadNumber,0  
>         invoke SetFilePointer,hFile,0,0,FILE\_BEGIN  
>         invoke WriteFile,hFile,HeaderBuffer,dword ptr \[esi+54h\],addr ReadNumber,0  
>         invoke CloseHandle,hFile  
>         invoke VirtualFree,peshell,0,MEM\_RELEASE  
>         invoke VirtualFree,HeaderBuffer,0,MEM\_RELEASE  
>     ret  
>       ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>pe 外壳程序>>>>>>>>>>>>>>>>>>>>>>>>  
>    shellstart:  ;shell 开始  
>       call @@1  
>       ImportTable     dd THUNK\_DATA1-ImportTable  
>           dd 0,0  
>       Import\_Name     dd  DllName-ImportTable  
>       Import\_FirstThunk dd  THUNK\_DATA1-ImportTable  
>           dd 0,0,0,0,0  
>       THUNK\_DATA1      dd  IMPORT\_BY\_NAME1-ImportTable  
>       THUNK\_DATA2     dd  IMPORT\_BY\_NAME2-ImportTable  
>       THUNK\_DATA3     dd  IMPORT\_BY\_NAME3-ImportTable  
>                       dd 0  
>       DllName         db 'Kernel32.dll',0  
>                          
>       IMPORT\_BY\_NAME1 dw 0  
>           db 'GetProcAddress',0  
>       IMPORT\_BY\_NAME2 dw 0  
>           db 'GetModuleHandleA',0  
>       IMPORT\_BY\_NAME3  dw 0  
>                  db 'LoadLibraryA',0  
>   ImportTableend:  
>           
>       ;>>>>>>>>>>>>>>>>>>>>>>>>>以上为输入表结构>>>>>>>>>>>>>>>>>>>>>>>>>>  
>       ;在把壳写入文件时，需要重新定位以上信息>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>       ;ebp 中保存着要加壳文件的 shellstart 的地址（不是相对虚拟地址，而是绝对）  
>       @@1:  
>           pop ebp;此时 ebp 中为 ImportTable 的地址  
>           sub ebp,(ImportTable-shellstart)  
>           mov edi, dword ptr \[ebp+(RVA\_jiami-shellstart)\]  
>           push 0  
>           call dword ptr \[ebp+(THUNK\_DATA2-shellstart)\]  
>           add edi,eax  
>           mov dword ptr \[ebp+(handle-shellstart)\],eax  
>           mov ecx, dword ptr \[ebp+(SIZE\_jiami-shellstart)\]  
>    jiemi:  
>        xor byte ptr \[edi\],13h  
>        inc edi  
>        loop jiemi  
>       ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>初始化输入表>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
>        mov edi,dword ptr \[ebp+(Import-shellstart)\]  
>        push 0  
>        call dword ptr \[ebp+(THUNK\_DATA2-shellstart)\]  
>        mov dword ptr \[ebp+(handle-shellstart)\],eax  
>        add edi,dword ptr \[ebp+(handle-shellstart)\]  
>          
>    GetDllAddr:  
>        mov esi,dword ptr \[edi+0ch\];esi--->dllname 的 rva  
>        .if esi==0  
>            jmp DllFinish  
>        .endif  
>        add esi,dword ptr \[ebp+(handle-shellstart)\]  
>        push esi  
>        call dword ptr \[ebp+(THUNK\_DATA2-shellstart)\];获取 dll 句柄  
>        .if eax==0  
>            push esi  
>            call dword ptr \[ebp+(THUNK\_DATA3-shellstart)\]  
>        .endif  
>         mov esi,eax  
>         mov edx,dword ptr \[edi\]  
>         .if edx==0  
>           mov edx,dword ptr \[edi+10h\]  
>         .endif  
>         add edx,dword ptr \[ebp+ (handle-shellstart)\]  
>         mov ebx,dword ptr \[edi+10h\]  
>         add ebx,dword ptr \[ebp+(handle-shellstart)\]  
>    GetFuncAddr:  
>        mov eax,dword ptr \[edx\] ;IMAGE\_THUNK\_DATA  
>        cmp eax,0  
>        jz FunctionFinish  
>        push ebx  
>        push edx  
>        cdq  
>        .if edx==0  
>          add eax,2h  
>          add eax,dword ptr \[ebp+(handle-shellstart)\]  
>        .else  
>          and eax,7fffffffh  
>        .endif  
>        push eax  
>        push esi;dll 句柄  
>        call dword ptr \[ebp+(THUNK\_DATA1-shellstart)\]  
>        mov dword ptr \[ebx\],eax  
>        pop edx  
>        pop ebx  
>        add edx,4h  
>        add ebx,4h  
>        jmp GetFuncAddr  
>   FunctionFinish:  
>       add edi,14h  
>       jmp GetDllAddr  
>   DllFinish:  
>       mov eax,dword ptr \[ebp+(OEP-shellstart)\]  
>       add eax,dword ptr \[ebp+(handle-shellstart)\]  
>       mov ebp,esp  
>       push ebp  
>       add esp,1  
>       inc ecx  
>       nop  
>       dec ecx  
>       pop ebp  
>       add esp,-1  
>       nop  
>       push eax  
>       retn  
>       RVA\_jiami    DD 0  
>       SIZE\_jiami   DD 0  
>       Import       DD 0  
>       handle       DD 0  
>       OEP          DD 0  
>   shellend:  
>   AddShell endp  
>     
>   \_DlgProcMain proc uses ebx edi esi hWnd,wMsg,wParam,lParam  
>       local @lpfn:OPENFILENAME  
>       local @dwBytesRead:dword  
>       invoke RtlZeroMemory,addr @lpfn,sizeof @lpfn  
>       mov eax,wMsg  
>       .if eax==WM\_CLOSE  
>         invoke EndDialog,hWnd,NULL  
>       .elseif eax==WM\_INITDIALOG  
>         invoke LoadString,hInstance,szErr101,addr szErr,sizeof szErr  
>         invoke LoadString,hInstance,szPass102,addr szPass,sizeof szPass  
>         invoke LoadString,hInstance,szStop103,addr szStop,sizeof szStop  
>         invoke LoadIcon,hInstance,ICO\_ISCC  
>          invoke SendMessage,hWnd,WM\_SETICON,ICO\_ISCC,eax  
>         invoke GetDlgItem,hWnd,IDC\_PASS  
>         invoke EnableWindow,eax,FALSE  
>       .elseif eax==WM\_COMMAND  
>         mov eax,wParam  
>         .if ax==IDOK  
>           mov @lpfn.lStructSize,sizeof @lpfn  
>           push hWnd  
>           pop @lpfn.hWndOwner  
>           mov @lpfn.lpstrFilter,offset lpstrFilter  
>           mov @lpfn.lpstrFile,offset szFileName  
>           mov @lpfn.nMaxFile,MAX\_PATH  
>           mov @lpfn.Flags,OFN\_FILEMUSTEXIST or OFN\_PATHMUSTEXIST  
>           invoke GetOpenFileName,addr @lpfn  
>           .if eax  
>             invoke SetDlgItemText,hWnd,IDC\_EDIT1,addr szFileName  
>             invoke GetDlgItem,hWnd,IDC\_PASS  
>             invoke EnableWindow,eax,TRUE  
>           .endif  
>          .elseif ax==IDC\_PASS  
>           invoke GetDlgItemText,hWnd,IDC\_EDIT1,addr szFileName,MAX\_PATH  
>           call AddShell  
>          .endif  
>        .else  
>          mov eax,FALSE  
>           ret  
>          .endif  
>         mov eax,TRUE  
>         ret  
>         
>   \_DlgProcMain endp  
>     
>   start:  
>       invoke GetModuleHandle,NULL  
>       mov hInstance,eax  
>       invoke DialogBoxParam,hInstance, addr szdlg,NULL,addr \_DlgProcMain,NULL  
>       invoke ExitProcess,eax  
>       end start  
>         
>                
>                
>                
>                
>                
>                
>                
>                
>               
>               
>               
>             
>     
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
>   
>                                                        2007 年 05 月 13 日 下午 06:23:02 

 4dd466d0e1e0cc123afbe504da2462ca