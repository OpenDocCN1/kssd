

# 通过 Hash 值计算 API 的名字

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-326.htm`](https://www.pediy.com/kssd/pediy09/pediy09-326.htm)

> 通过 Hash 值计算 API 的名字  
>    
> 2007.11.15  
>    
> by marxixing@tom.com  
>    
> 　  
>    
> 在分析 shellcode 的过程中，我们经常会遇到被编码后的 API 的 hash 值，因为 hash 算法是不可逆算法，我们不可能通过一个 DWORD 值得知原始的 api，为了知道 shellcode 都调用了哪些 api，你就得去不断的去调试，这个工作量是比较大的。我昨天就深有体会。今天，我写了个小工具，只要输入一个 DOWRD 值和 dll 文件的名字，就能计算出来 api 的名字了，呵呵，裤罢.  
>    
> 你可以这样来使用它： ByHashName EC0E4E8E kernel32 ,其中 ByHashName 是这个程序的名字，EC0E4E8E 是 api 的 hash 值，kernel32 就是该 api 所在的库了，玩 shellcode 的大家应该都知道吧。针对上面这个命令行，输出结果应该是 LoadLibraryA.  
>    
> 本文所列出的 hash 值的算法是 ror edi，0d， 也就是将每个字符都右移 13 位，名字的起始指针放在 esi 寄存器中。下面我列出了常见的 hash 值对应的函数名称，当你遇到了他们，就不用在程序中运算了，直接使用他们就是了。对于一个特定的 shellcode，这个算法是最常见的，如果你发现了其他的 hash 算法，只需要更改 dll 文件的输出函数的 hash 算法就可以了，shellcode 的 hash 算法不会很难，最多 10 行汇编代码就能搞定。  
>    
> 2007.11.21 晚上改进了一下，做了个纯汇编版本的，不用带个 dll 尾巴了。但原文件仍然保留。汇编版本附在本文后面。附件中是汇编版本的可执行文件。  
>    
> ; 0EC0E4E8Eh kernel32.dll LoadLibraryA  
> ; 016B3FE72h kernel32.dll CreateProcessA  
> ; 07c0dfcaah kernel32.dll GetProcAddress  
> ; 00c0397ech kernel32.dll GlobalAlloc  
>    
>    
> ; 702F1A36h urlmon.dll URLDownloadToFileA  
> ; 054FEF4Fh urlmon.dll URLDownloadToCacheFileA  
>    
> 　  
>    
> 这是个命令行工具，我使用了汇编语言编写 marxixing.dll 文件，供 c 语言编写的主程序调用。 其中 dll 文件的内容如下：  
>    
> .386  
> .model flat, stdcall  
> option casemap:none  
>    
>    
> include windows.inc   
> include kernel32.inc   
> includelib kernel32.lib   
>    
> .code  
>    
> DllEntry proc \_hInst: HINSTANCE, \_reason: DWORD, \_reserved: DWORD  
> mov eax, TRUE  
> ret  
> DllEntry Endp  
>    
>    
> ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
> ;得到 kernel32 在内存中的加载地址，无参数  
> ;eax 返回最终的加载地址  
> MXX\_GetKernelBase proc  
>    
> ASSUME FS:nothing  
> mov eax, fs:\[30h\]  
> mov eax, \[eax + 0ch\]  
> mov esi, \[eax + 1ch\]  
> lodsd  
> mov eax, \[eax + 8h\]  
> ret  
>    
> MXX\_GetKernelBase endp  
> ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
>    
> ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
> ;通过 hash 值得到转换前的 API 的名字  
> ;参数 DllBase: dll 文件的加载地址  
> ;参数 ApiHash: api 字符串的 Hash 值  
> ;返回值：eax 返回 API 名字字符串  
>    
> MXX\_GetFunctionAddress proc DllBase:dword, ApiHash:dword  
>    
> mov edx, DllBase  
> mov ebx, \[edx + 3ch\]  
> mov ebx, \[edx + ebx + 78h\]  
> add ebx, edx ;export table address  
>    
> mov ecx, \[ebx + 18h\] ;函数数目  
> mov edi, \[ebx + 20h\] ;api 名字数组首地址  
> add edi, edx  
>    
> ;;计算每个函数的 hash 值，获得我们需要的函数   
> NextFunc:  
> jecxz error  
> dec ecx  
> mov esi, \[edi + ecx \* 4h\] ;初始时指向最后一个函数的指针  
> add esi, edx  
>    
> push edi   
> push edx  
>    
> mov edx, esi ;先保存一下  
>    
> xor edi, edi  
> cld  
> Counthash: ;开始计算 hash 值  
> xor eax, eax  
> lodsb  
> cmp al, ah  
> jz Apiend  
>    
> ror edi, 0dh  
> add edi, eax  
>    
> jmp Counthash  
> Apiend:  
> cmp edi, ApiHash ;hash 值计算结束  
> mov esi, edx ;恢复 api 名字指针，用来返回  
> pop edx   
> pop edi  
> jnz NextFunc  
>    
> mov eax, esi  
> ret   
>    
> error:  
> mov eax, 0  
> ret  
>    
> MXX\_GetFunctionAddress endp  
> ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
>    
>    
> end DllEntry  
>    
> 　  
>    
> 该文件输出了两个函数，因此我们还需要给它加上一个 def 文件，命名为 marxixing.def,内容如下：  
>    
> LIBRARY marxixing  
> EXPORTS MXX\_GetKernelBase  
> MXX\_GetFunctionAddress  
>    
>    
> 之后，就可以用如下的命令进行编译和链接了，最终生成两个文件，marxixing.lib 和 marxixing.dll  
>    
> ml /c /coff marxixing.asm  
> link /DLL /subsystem:windows /Def:marxixing.def marxixing.obj  
>    
> 　  
>    
> 注意，我们不想用 loadlibrary，getproaddress 的方式来调用 dll 里面的函数，因此我写了下面的头文件 marxixing.h,这样，就可以直接使用我们的输出函数了。  
>    
> extern "C" DWORD \_\_stdcall MXX\_GetKernelBase();  
>    
> extern "C" char \* \_\_stdcall MXX\_GetFunctionAddress(HINSTANCE DllBase, DWORD HashSum);  
> 　  
>    
> 　  
>    
> 现在，该是我们的 c 语言上场了，我们用它来设计界面：  
>    
> 　  
>    
> //输入函数的 hash 值，求出对应的 api 名字  
> #include <stdio.h>  
> #include <windows.h>  
> #include "marxixing.h"  
>    
> #pragma comment(lib, "marxixing.lib")  
>    
> int main(int argc, char \* argv\[\])  
> {  
> //参数 1 为 api 的 hash 值  
> //参数 2 为 dll 文件的名字  
>    
> HINSTANCE dllbase;  
> char dllname\[32\] = {0};  
> char \* alloc = NULL;  
> DWORD hashsum;  
> DWORD apiname;  
>    
> if(argc <= 2 )  
> {  
> printf("USAGE: ByHashName hashsum dllname.\\n");  
> printf("return: ApiName string\\n");  
> return 0;   
> }  
> sscanf(argv\[1\], "%x", &hashsum);  
> strcpy(dllname, argv\[2\]);  
>    
> dllbase = LoadLibrary(dllname);  
> if(dllbase == NULL)  
> {  
> printf("%s load memory fail!", dllname);  
> return 0;  
> }  
> alloc = MXX\_GetFunctionAddress(dllbase, hashsum);  
> if(alloc == NULL)  
> {  
> printf("not find apiname!");  
> return 0;  
> }  
> printf("%s\\n",alloc);  
>    
> FreeLibrary(dllbase);  
>    
>    
> return 1;  
>    
> }  
>    
>    
>    
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> ;汇编版本的实现  
>    
> .386  
> .model flat, stdcall  
> option casemap:none  
>    
> include windows.inc   
> include kernel32.inc   
> includelib kernel32.lib   
> include user32.inc   
> includelib user32.lib   
> include masm32.inc  
> includelib masm32.lib  
> .data  
> libfail db "LoadLibrary fail!", 0  
> NotFindApi db "Not find apiname!", 0  
> .data?  
> Hashsum db 256 dup(?)  
> Libname db 64  dup(?)  
> ApiName   dd ?  
> HashDword dd ?  
> LibDword  dd ?  
>   
> MXX\_GetFunctionName proto :DWORD ,  :DWORD  
>   
> .code  
> start:  
>  invoke GetCL, 1, addr Hashsum  
>  invoke GetCL, 2, addr Libname  
>    
>    
>  invoke htodw, addr Hashsum  ;转换输入字符串为双字  
>  mov HashDword, eax  
>    
>  invoke LoadLibrary, addr Libname  
>  .IF eax == NULL  
>   mov ApiName, offset libfail  
>   jmp over  
>  .ELSE  
>   mov LibDword, eax  
>  .ENDIF  
>    
>  invoke MXX\_GetFunctionName, LibDword, HashDword  
>  .IF eax == NULL  
>   mov ApiName, offset NotFindApi  
>  .ELSE  
>   mov ApiName, eax  
>  .ENDIF  
>    
>    
> over:   
>  invoke MessageBox, NULL, ApiName, addr Libname, MB\_OK  
>  invoke FreeLibrary, LibDword  
>  invoke ExitProcess, NULL  
>    
> ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
> ;通过 Hash 值得到 API 的名字  
> ;参数 DllBase: dll 文件的加载地址  
> ;参数 ApiHash: api 字符串的 Hash 值  
> ;返回值：eax 返回 API 名字字符串  
> MXX\_GetFunctionName proc DllBase:dword, ApiHash:dword  
>  mov edx, DllBase  
>  mov ebx, \[edx + 3ch\]  
>  mov ebx, \[edx + ebx + 78h\]  
>  add ebx, edx  ;export table address  
>  mov ecx, \[ebx + 18h\]   ;函数数目  
>  mov edi, \[ebx + 20h\]  ;api 名字数组首地址  
>  add edi, edx  
>    
> ;;计算每个函数的 hash 值，获得我们需要的函数   
> NextFunc:  
>  jecxz error  
>  dec  ecx  
>  mov esi, \[edi + ecx \* 4h\]  ;初始时指向最后一个函数的指针  
>  add esi, edx  
>    
>  push edi  ;保存原来 edi,edx 的值，下面要用到它  
>  push edx  
>  mov  edx, esi  
>  xor edi, edi  
>  cld  
> Counthash:  ;开始计算 hash 值  
>  xor eax, eax  
>  lodsb  
>  cmp al,  ah  
>  jz  Apiend  
>  ror edi, 0dh  
>  add edi, eax  
>    
>  jmp Counthash  
> Apiend:  
>  cmp edi, ApiHash  ;hash 值计算结束  
>  mov esi, edx     ;恢复 api 名字指针，用来返回  
>  pop edx   ;恢复原来 edx,edi 的值  
>  pop edi  
>  jnz NextFunc  
>  mov eax, esi  
>  ret   
> error:  
>  mov eax, 0  
>  ret  
>    
> MXX\_GetFunctionName endp  
> ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
>   
> end start 

 4dd466d0e1e0cc123afbe504da2462ca