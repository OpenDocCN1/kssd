
# HeapMagic

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-830.htm`](https://www.pediy.com/kssd/pediy08/pediy8-830.htm)

`Themida1800` demo 的 `HeapMagic`

`VM` 保护代码里面检测了 1 个标记，是在这里设置的:

```asm
00B27354    83BD AD244F03 0>cmp     dword ptr [ebp+34F24AD], 1  
00B2735B    0F85 C2000000   jnz     00B27423  
00B27361    83BD C1054F03 0>cmp     dword ptr [ebp+34F05C1], 0  
00B27368    0F85 B5000000   jnz     00B27423  
00B2736E    83BD C92F4F03 0>cmp     dword ptr [ebp+34F2FC9], 0  
00B27375    0F85 A8000000   jnz     00B27423  
00B2737B    8D85 04D66003   lea     eax, dword ptr [ebp+360D604]  
00B27381    90              nop  
00B27382    50              push    eax  
00B27383    90              nop  
00B27384    64:FF35 0000000>push    dword ptr fs:[0]  
00B2738B    64:8925 0000000>mov     dword ptr fs:[0], esp  
00B27392    64:A1 30000000  mov     eax, dword ptr fs:[30]  
00B27398    8B40 0C         mov     eax, dword ptr [eax+C]  
00B2739B    B9 30000000     mov     ecx, 30  
00B273A0    40              inc     eax  
00B273A1    8138 EEFEEEFE   cmp     dword ptr [eax], FEEEFEEE  
00B273A7   ^ 0F85 F3FFFFFF  jnz     00B273A0  
00B273AD    49              dec     ecx  
00B273AE   ^ 0F85 ECFFFFFF  jnz     00B273A0  
00B273B4    8BC0            mov     eax, eax  
00B273B6    83BD 5D2C4F03 0>cmp     dword ptr [ebp+34F2C5D], 0  
00B273BD    75 09           jnz     short 00B273C8  
00B273BF    83BD 4D0C4F03 0>cmp     dword ptr [ebp+34F0C4D], 0  
00B273C6    74 13           je      short 00B273DB  
00B273C8    50              push    eax  
00B273C9    53              push    ebx  
00B273CA    8BC0            mov     eax, eax  
00B273CC    B8 D1040000     mov     eax, 4D1  
00B273D1    8D9D 674A5D03   lea     ebx, dword ptr [ebp+35D4A67]  
00B273D7    FFD3            call    ebx  
00B273D9    5B              pop     ebx  
00B273DA    58              pop     eax  
00B273DB    8BC0            mov     eax, eax  
00B273DD    C785 8D1D4F03 0>mov     dword ptr [ebp+34F1D8D], 1  ; 这里  
00B273E7    E9 2D000000     jmp     00B27419  
00B273EC    8B5C24 0C       mov     ebx, dword ptr [esp+C]  
00B273F0    50              push    eax  
00B273F1    892C24          mov     dword ptr [esp], ebp  
00B273F4    E8 00000000     call    00B273F9  
00B273F9    5D              pop     ebp  
00B273FA    81ED 11D66003   sub     ebp, 360D611  
00B27400    8B83 B8000000   mov     eax, dword ptr [ebx+B8]  
00B27406    8D85 31D66003   lea     eax, dword ptr [ebp+360D631]  
00B2740C    8983 B8000000   mov     dword ptr [ebx+B8], eax  
00B27412    5D              pop     ebp  
00B27413    B8 00000000     mov     eax, 0  
00B27418    C3              retn  
00B27419    64:8F05 0000000>pop     dword ptr fs:[0]  
00B27420    83C4 04         add     esp, 4  
00B27423    8BC0            mov     eax, eax  
00B27425    83BD 5D2C4F03 0>cmp     dword ptr [ebp+34F2C5D], 0  
00B2742C    75 09           jnz     short 00B27437  
00B2742E    83BD 4D0C4F03 0>cmp     dword ptr [ebp+34F0C4D], 0  
00B27435    74 19           je      short 00B27450
```

如果没被调试，会产生异常，这里的代码和 `ap0x` 的例子一字不差;-) 

老早的时候写的，当时 `anti Hero` 玩的   

附件下载：`anti.rar` 

`shoooo` 的 `anti` 很好玩，  

第一个是 `heapmagic`（我只知道一种方法过，强制跳转）  

第三个是调用 `API`，`ZwQueryInformationProcess`，两种方法过  

1. 强制跳
2. `hook sdt`

```asm
00401470  /.  55            push    ebp  
00401471  |.  8BEC          mov     ebp, esp  
00401473  |.  83EC 10       sub     esp, 10  
00401476  |.  68 40304000   push    00403040                    ; /ProcNameOrOrdinal = "ZwQueryInformationProcess"  
0040147B  |.  68 34304000   push    00403034                    ; |/pModule = "ntdll.dll"  
00401480  |.  FF15 04204000 call    dword ptr [<&KERNEL32.GetModu>; |\GetModuleHandleA  
00401486  |.  50            push    eax                           ; |hModule  
00401487  |.  FF15 00204000 call    dword ptr [<&KERNEL32.GetProc>; \GetProcAddress  
0040148D  |.  8945 F4       mov     dword ptr [ebp-C], eax  
00401490  |.  B8 01000000   mov     eax, 1  
00401495  |.  8945 FC       mov     dword ptr [ebp-4], eax  
00401498  |.  8945 F8       mov     dword ptr [ebp-8], eax  
0040149B  |.  8D85 FFFFFFF  lea     eax, dword ptr [ebp-8]  
004014A1  |.  50            push    eax  
004014A2  |.  6A 04         push    4  
004014A4  |.  8D85 FFFFFFF  lea     eax, dword ptr [ebp-4]  
004014AA  |.  50            push    eax  
004014AB  |.  6A 07         push    7  
004014AD  |.  6A FF         push    -1  
004014AF  |.  FF55 F4       call    dword ptr [ebp-C]  
004014B2  |.  8945 F0       mov     dword ptr [ebp-10], eax  
004014B5  |.  8B45 FC       mov     eax, dword ptr [ebp-4]  
004014B8  |.  85C0          test    eax, eax  
004014BA  |.  75 18         jnz     short 004014D4  
004014BC  |.  837D F8 04    cmp     dword ptr [ebp-8], 4  
004014C0  |.  75 12         jnz     short 004014D4  
004014C2  |.  6A 00         push    0  
004014C4  |.  6A 00         push    0  
004014C6  |.  68 20304000   push    00403020                    ;   ASCII "ok"  
004014CB  |.  E8 14020000   call    <jmp.&MFC42.#1200>  
004014D0  |.  8BE5          mov     esp, ebp  
004014D2  |.  5D            pop     ebp  
004014D3  |.  C3            retn  
004014D4  |>  6A 00         push    0  
004014D6  |.  6A 00         push    0  
004014D8  |.  68 24304000   push    00403024                    ;   ASCII "Debugger Found"  
004014DD  |.  E8 02020000   call    <jmp.&MFC42.#1200>  
004014E2  |.  8BE5          mov     esp, ebp  
004014E4  |.  5D            pop     ebp  
004014E5  \\.  C3            retn
```

第二个看不懂，![](img/70b1b1b49da6fb968b36803b1ec24864.jpg)

1. 遍历堆，`BAADF00D`，`FEEEFEEE` 个数，小于等于 `0x0A` 正常  

   ```asm
   MOV EAX,FS:[18]  
   MOV EAX,[EAX+30]  
   MOV EAX,[EAX+18]
   ```

2. 异常 `0x64` 次后 `7FFE0000` 的内容有变化，在我的 `2003sp1` 下没变化，正常  

   ```
   7FFE0000  00 00 00 00                              ....
   ```

3. `api` 有 `5` 个字节长就可以 `hook`，判断参数返回非调试状态  

   `ZwQueryInformationProcess` `B8 A1000000` `MOV EAX,0A1`  

非常好的东西，谢谢 `shoooo`![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)

引用: *最初由** `kanxue` **发布*  

`softworm` 对 `Themida` 虚拟机研究的很深，这段代码这么完美地还原了。  

另外，`ap0x` 的例子是这页面上的吗？  

`http://www.openrce.org/reference_library/anti_reversing` 

是呀，`ap0x` 给它取的名字是 `Ring3 Debugger Detection via LDR_MODULE`  

代码如下:  

Description:  

```asm
; ######################################################################### 
    .586 
    .model flat, stdcall 
    option casemap :none   ; case sensitive 
; ######################################################################### 
    include \masm32\include\windows.inc 
    include \masm32\include\user32.inc 
    include \masm32\include\kernel32.inc 
    include \masm32\include\comdlg32.inc 
    includelib \masm32\lib\user32.lib 
    includelib \masm32\lib\kernel32.lib 
    includelib \masm32\lib\comdlg32.lib 
; #########################################################################    
    .data 
DbgFoundTitle db "Debugger found:",0h 
DbgFoundText db "Debugger has been found!",0h 
DbgNotFoundTitle db "Debugger not found:",0h 
DbgNotFoundText db "Debugger not found!",0h 
Tries db 30 
Alloc dd ? 
    .code 
start: 
; MASM32 antiRing3Debugger example 
; coded by ap0x 
; Reversing Labs: http://ap0x.headcoders.net 
ASSUME FS:NOTHING 
PUSH offset _SehExit 
PUSH DWORD PTR FS:[0] 
MOV FS:[0],ESP 
; Get NtGlobalFlag 
MOV EAX,DWORD PTR FS:[30h] 
; Get LDR_MODULE 
MOV EAX,DWORD PTR[EAX+12] 
; The trick is here ;) If ring3 debugger is present memory will be allocated 
; and it will contain 0xFEEEFEEE bytes at the end of alloc. This will only 
; happen if ring3 debugger is present! 
; If there is no debugger SEH will fire and take control. 
; Note: This code works only on NT systems! 
_loop: 
INC EAX 
CMP DWORD PTR[EAX],0FEEEFEEEh 
JNE _loop 
DEC [Tries] 
JNE _loop 
PUSH 30h
```



```
PUSH offset DbgFoundTitle  

PUSH offset DbgFoundText  

PUSH 0  

CALL MessageBox  

PUSH 0  

CALL ExitProcess  

RET  

\_Exit:  

PUSH 40h  

PUSH offset DbgNotFoundTitle  

PUSH offset DbgNotFoundText  

PUSH 0  

CALL MessageBox  

PUSH 0  

CALL ExitProcess  

RET  

\_SehExit:  

POP FS:[0]  

ADD ESP,4  

JMP \_Exit  

end start 
```

这段代码在 `Themida` 里是明文，早就注意到了，只是没看过 `ap0x` 的例子，所以没有想到这就是传说中的 `heapMagic`，一直围绕着 `heapalloc`，`heapsize` 那里推敲，以为是调试堆那里出了问题，始终没有进展.......

```
01244152               8BC0            mov     eax, eax  
01244154               83BD D92A3509 0>cmp     dword ptr [ebp+9352AD9], 1  
0124415B               0F85 C3000000   jnz     01244224  
01244161               83BD 7D193509 0>cmp     dword ptr [ebp+935197D], 0  
01244168               0F85 B6000000   jnz     01244224  
0124416E               83BD B9233509 0>cmp     dword ptr [ebp+93523B9], 0  
01244175               0F85 A9000000   jnz     01244224  
0124417B               8D85 D4BF4409   lea     eax, dword ptr [ebp+944BFD4]  
01244181               50              push    eax  
01244182               890424          mov     dword ptr [esp], eax  
01244185               64:FF35 0000000>push    dword ptr fs:[0]  
0124418C               64:8925 0000000>mov     dword ptr fs:[0], esp  
01244193               64:A1 30000000  mov     eax, dword ptr fs:[30]  
01244199               8B40 0C         mov     eax, dword ptr [eax+C]  
0124419C               B9 30000000     mov     ecx, 30  
012441A1               40              inc     eax  
012441A2               8138 EEFEEEFE   cmp     dword ptr [eax], FEEEFEEE  
012441A8              ^ 0F85 F3FFFFFF   jnz     012441A1  
012441AE               49              dec     ecx  
012441AF              ^ 0F85 ECFFFFFF   jnz     012441A1  
012441B5               8BC0            mov     eax, eax  
012441B7               83BD 85273509 0>cmp     dword ptr [ebp+9352785], 0  
012441BE               75 09           jnz     short 012441C9  
012441C0               83BD 0D103509 0>cmp     dword ptr [ebp+935100D], 0  
012441C7               74 13           je      short 012441DC  
012441C9               50              push    eax  
012441CA               53              push    ebx  
012441CB               8BC0            mov     eax, eax  
012441CD               B8 D1040000     mov     eax, 4D1  
012441D2               8D9D A2734109   lea     ebx, dword ptr [ebp+94173A2]  
012441D8               FFD3            call    ebx  
012441DA               5B              pop     ebx  
012441DB               58              pop     eax  
012441DC               8BC0            mov     eax, eax  
012441DE               C785 4D1D3509 0>mov     dword ptr [ebp+9351D4D], 1  
012441E8               E9 2D000000     jmp     0124421A  
012441ED               8B5C24 0C       mov     ebx, dword ptr [esp+C]  
012441F1               50              push    eax  
012441F2               892C24          mov     dword ptr [esp], ebp  
012441F5               E8 00000000     call    012441FA  
012441FA               5D              pop     ebp  
012441FB               81ED E1BF4409   sub     ebp, 944BFE1  
01244201               8B83 B8000000   mov     eax, dword ptr [ebx+B8]  
01244207               8D85 01C04409   lea     eax, dword ptr [ebp+944C001]  
0124420D               8983 B8000000   mov     dword ptr [ebx+B8], eax  
01244213               5D              pop     ebp  
01244214               B8 00000000     mov     eax, 0  
01244219               C3              retn  
0124421A               64:8F05 0000000>pop     dword ptr fs:[0]  
01244221               83C4 04         add     esp, 4  
01244224               8BC0            mov     eax, eax  
01244226               83BD 85273509 0>cmp     dword ptr [ebp+9352785], 0  
0124422D               75 09           jnz     short 01244238  
0124422F               83BD 0D103509 0>cmp     dword ptr [ebp+935100D], 0  
01244236               74 19           je      short 01244251  
01244238               50              push    eax  
01244239               53              push    ebx  
0124423A               8BC0            mov     eax, eax  
0124423C               B8 D1040000     mov     eax, 4D1  
01244241               8985 95283509   mov     dword ptr [ebp+9352895], eax  
01244247               8D9D 9C764109   lea     ebx, dword ptr [ebp+941769C]  
0124424D               FFD3            call    ebx  
0124424F               5B              pop     ebx  
01244250               58              pop     eax  
01244251               83BD 0D103509 0>cmp     dword ptr [ebp+935100D], 0  
01244258               74 17           je      short 01244271  
0124425A               50              push    eax  
0124425B               53              push    ebx  
0124425C               B8 D1040000     mov     eax, 4D1  
01244261               8985 95283509   mov     dword ptr [ebp+9352895], eax  
01244267               8D9D 3D734109   lea     ebx, dword ptr [ebp+941733D]  
0124426D               FFD3            call    ebx  
0124426F               5B              pop     ebx  
01244270               58              pop     eax  
01244271               8BC0            mov     eax, eax  
01244273               E9 54010000     jmp     012443CC
```
