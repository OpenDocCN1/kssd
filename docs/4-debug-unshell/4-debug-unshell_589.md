

# def

> 原文：[`www.pediy.com/kssd/pediy06/pediy6432.htm`](https://www.pediy.com/kssd/pediy06/pediy6432.htm)

DEF 分析与打造其脱壳软件    
初来贵地.献析文一篇. 请高手勿笑  .   
我不知道有这么个地方, 以后请大家多指教.   
我写的不怎么好的地方,请大家指正.   
我不怎么懂脱壳，请不要大家找我脱壳。。。      
  
作者：牛博威    
  
Email：advice107@sina.com    
  
[`nboy.cnwlt.com`](http://nboy.cnwlt.com)    
  
qq：  343538175   
       加壳软件一种可以对别的程序进行修改但是不影响其使用的工具软件。程序员经常利用加壳软件对自己的软件进行加壳，以加密或者压缩自己的软件。加壳后的软件一般来说难以被逆向分析，因此，脱壳软件应运而生。从某种角度来说，了解加壳和脱壳技术的原理是很有必要的。    
  
       下面我以 Def 为例说明其加壳原理，同时，我们改造 Def 使其具有针对自身的脱壳功能。    
  
Def 是一款源码开放的小型加壳软件，精炼易懂，便于学习。其源代码和程序可以去 http://protools.cjb.net 下载 或 点击此处下载（或鼠标右键另存为）  
  
       首先根据其源代码分析其加壳原理。它的加壳流程主要如下：    
  
1、  以文件内存映象方式打开被加壳文件，并判断是否是有效的 PE 文件且不是 Dll 文件；    
  
2、  遍历所有节表，根据节表名判断是否是输入表，资源节表或者其他节表，如果不是，定位到该节区，根据本节区大小对该节区的内容进行简单的异或加密，加密方式如下：    
  
\_encrypt:                              ;加密该节区    
  
              xor  byte ptr\[esi\],al        ;进行异或加密    
  
       inc   esi                        ;esi 指向节区下一个字节    
  
              dec  eax                       ;判断是否到节区尾部，同时也是下一次异或操作的变量    
  
       jne   \_encrypt    
  
              然后把节表名称修改为.def，对未进行加密的节表，添加未加密标志，具体见下文；    
  
3、  为被加壳文件添加自解密部分，用于被加壳文件运行时候自行解密。这一段可以参考源代码中的\_loader 部分。这段自解密代码被添加到文件节表的后面，同时修改文件的入口地址为此处的偏移地址。添加自解密部分的代码如下：    
  
              mov esi,offset \_loader           ; 初始化\_loader 的偏移地址    
  
       mov ecx,\_loader\_size            ; 初始化 loader 部分的大小    
  
              rep   movsb                          ; 把 loader 部分拷贝到被加壳文件的节表尾部    
  
                                                 ;其实这里最好判断一下是否有足够空间放置本段代码    
  
4、  最后关闭文件内存映象；    
  
根据上面的流程分析，再仔细研究一下其代码，相信你可以很快明白它的具体工作方式。下面我们来修改 def，使其具有脱壳功能。    
  
       Def 手工脱壳很简单，我以脱去 Def 自身的壳为例，说明其脱壳流程：    
  
1、  用 Trw 载入 Def；    
  
2、  F10 单步运行至 xxxx:400244   push   dword  ptr  00401000  处，其中 00401000 为 Def 原来的入口地址。下命令 Suspend，将进程挂起，F5 回到 windows 界面；    
  
3、  打开 Peditor，点 task，选择被挂起的 Def 进程，点右键 dump(full)，保存为 unDef；    
  
4、  用 Peditor 修改 unDef 文件的入口地址为 00001000，注意这里是 00401000 减去 00400000 得到的。    
  
好了，这样 unDef 就是 Def 的脱壳文件。我们现在要修改 Def 为自身的脱壳程序，也就是要对 unDef 进行处理。    
  
首先，脱壳程序应该可以判断一个文件是否被 Def 加壳，利用 Def 的判断方式，判断第一个节表名称是否为.def 便可以。故此，修改：    
  
:00401091 813A2E646566            cmp dword ptr \[edx\], 6665642E              ；.def ?    
  
:00401097 0F858B000000            je 00401128    
  
为：    
  
:00401091 813A2E646566            cmp dword ptr \[edx\], 6665642E    
  
:00401097 0F848B000000            jne 00401128       ；如果第一个节表名不为.def 则跳出    
  
                                                                                    ；请注意这里最好不要改成 jmp    
  
现在就可以对加壳文件进行脱壳了，由于 def 采用了简单的异或加密，因此解密部分不需要修改。找个加壳文件实验一下，竟然发生异常。为什么这样呢？因为 Def 并不是把所有的节表都加密，对于引入表，资源节表等，def 是不处理的。但我们的 unDef 会把所有的节表都进行解密，因此当然会发生错误。也许你会问，在程序中不是有 call \_is\_encryptable（也就是:004010A7 call 004011A0）用来判断是否应该被加密么？不错，虽然如此，但是仔细再看一下\_is\_encryptable 这个函数，它通过节表名来判断节表的修改合法性，而被加壳的文件所有节表名称都被改成了.def。因此，对这个函数来说加壳文件中所有的节表都应该被修改。    
  
难道我们就没有办法判断了么？当然不是，仔细研究一下加壳文件的引导部分，也就是 Def 源代码中的\_loader 部分，在程序自身解密的时候有 cmp byte ptr \[esi+07\], 00，其中 esi 指向节表头，如果\[edx+07\]=0 则跳过解密部分，否则进行解密。这样一来，我们修改 unDef 的解密部分。从 VA=004010A7 开始，修改后如下：    
  
:004010A7 807A0700                cmp byte ptr \[edx+07\], 00  ；判断节区修改标志    
  
:004010AB 90                      nop      ；保证文件后面的内容不变    
  
:004010AC 90                      nop    
  
:004010AD 90                      nop    
  
:004010AE 740F                    je 004010BF ；如果不该解密，便跳过解密部分    
  
到这里，就可以把所有加密部分恢复。但是脱壳文件仍然无法运行，因为我们还没有把文件的入口地址修改过来。看一下 def 如何修改入口地址，你可以很容易编写恢复入口地址的代码，如下：    
  
:004010D6 50                      push eax    
  
:004010D7 8B4225                  mov eax, dword ptr \[edx+25\]    ；edx 指向节表最后    
  
                                                                                           ；\[edx+25\]为真正的入口地址    
  
:004010DA 2D00004000              sub eax, 00400000            ；入口地址的 RVA->eax    
  
；减去 imagebase，我这里图省事，直接减去 00400000H，正规的应该从文件中读取    
  
:004010DF 894728                  mov dword ptr \[edi+28\], eax     ；edi+28 指向入口地址    
  
:004010E2 58                      pop eax    
  
:004010E3 E92B000000              jmp 00401113             ；跳转到添加\_loader 处    
  
:004010E8 90                      nop                            ；保证原来的文件内容不变    
  
最后，修改\_loader 处的代码为自己的信息（这就是上面为何跳到 401113）。把 "The Undef Made by NBW  QQ:37122085  http://nboy.cnwlt.com"的 ASCII 码覆盖到文件地址 1154 处，同时要修改：    
  
:00401118 B92A000000              mov ecx, 0000002A    
  
因为这个地方定义了添加的新代码的长度，如下：    
  
:00401118 B93A000000              mov ecx, 00000036     ；36H 就是我添加的信息的长度    
  
       到此，我们的 unDef 就制作完毕了，找一个被 Def 加壳的程序，用 unDef 进行脱壳，OK！很好用。    
  
       最后再多说几句，上面的这些地址除注明外都是程序的虚拟地址，在修改 unDef 的时候，不妨用 Hiew 打开，按 F3，再按 F2 进入 Asm 修改状态，你可以直接输入汇编程序进行修改，但是由于 Hiew 支持的 Asm 代码不全面，因此有些语句还需要在 16 进制状态进行修改。从这个修改过程也可以看到，由于加壳软件对原文件的破坏，我们做的脱壳软件并不可能 100%地恢复加壳软件原状，至少节表的名称和属性没有修改过来。因此，一些软件被专家级别的加壳软件所加壳，在脱壳的时候出现较大偏差是很正常的。    
  
下面是源代码分析篇：：：：：    
  
        Def 是一款典型的加壳软件，软件可以进行简单的异或加密，并且也未对输入表等进行处理，总体来说非常简单。    
      我在这里对其源代码写了详细的注释，仔细看一看会有不少收获。源代码也有注释，但是我现在还不明白那是哪国天书。要有高手明白，不妨告知一二，这里先谢过了。unDef 是我写对 Def 进行修改，使其可以脱自身的壳。     
  
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
; DEF - prostacki encryptor plikow PE EXE    
; kompatybilny z 95,NT    
;    
;原作： bart/xtreeme    
;    
;这是一款典型的加壳软件，软件可以进行简单的异或加密，并且也未对输入表等进行处理，总体来说非常简单。    
;其源代码是开放的，并且非常简练和易读，仔细看一看会有不少收获。源代码也有注释，但是我现在还不明白那是那国天书，    
;要有高手明白，不妨告知一二，这里先谢过了。    
;    
;    
;    
;注释：牛博威    
;http://nboy.cnwlt.com    
;QQ : 37122085    
;Email: advice107@sina.com    
;我的资料被人拿走了，因此有些地方应该还有问题，不足指出还请大家包涵和指出    
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
  
.586 ; 定义用的指令结合    
.mmx ; +mmx    
.model flat,stdcall    
  
includelib e:devmasmlibkernel32.lib ; 包含所需类库    
includelib e:devmasmlibuser32.lib    
includelib e:devmasmlibcomdlg32.lib    
  
include e:devmasmincludekernel32.inc    
include e:devmasmincludeuser32.inc    
include e:devmasmincludecomdlg32.inc    
  
include e:devmasmincludewindows.inc    
include pe.inc ;pe.inc 中定义了所需的常量，多与 PE 格式有关    
  
.data?    
lpFilename db 256 dup(?) ; 文件名    
lpFile MEMF <> ; 文件句柄    
lpPeheader dd ? ; PE 头    
lpSectionTable dd ? ;指向第一个节表的头部    
  
.data    
lpOfn dd lOfn ; 定义打开文件所需要的结构    
dd 0 ; hWnd    
dd 0 ; hInst    
dd offset szFilter ; filtr    
dd 0     
dd 0     
dd 0     
dd offset lpFilename     
dd 256     
dd 0 ; lpstrFileTitle;     
dd 0 ; nMaxFileTitle;     
dd 0 ; lpstrInitialDir;     
dd offset szCaption ; lpstrTitle;     
dd OFN\_HIDEREADONLY ; Flags;     
  
dw 0 ; nFileOffset;     
dw 0 ; nFileExtension;     
dd 0 ; lpstrDefExt;     
dd 0 ; lCustData;     
dd 0 ; lpfnHook;     
dd 0 ; lpTemplateName;     
lOfn equ $-lpOfn    
  
  
szCaption db 'DEF v1.0 by bart/xt - wybierz plik do zaszyfrowania',0    
szFilter db 'Pliki EXE (\*.exe)',0,'\*.exe',0    
db 'Wszystkie pliki (\*.\*)',0,'\*.\*',0,0    
  
szError db 'Nieprawidlowy format pliku!',0    
szDef db 'DEF v1.0 by bart/xt',0    
  
.code    
\_start:    
  
  
push offset lpOfn ; lpOfn 偏移入栈（废话）    
call GetOpenFileNameA ; 调用通用对话框打开文件    
test eax,eax ; 判断出错否？    
je \_exit    
  
push offset lpFilename ; 文件路径入栈    
call \_open\_file     
inc eax ; 这种判断方式也算一种技巧    
je \_exit     
  
test ecx,ecx    
je \_exit     
  
dec eax     
  
mov ebx,offset lpFile ;ebx->文件句柄，也相当于文件头    
assume ebx:ptr MEMF ;把 ebx 赋值为 MEMF 结构    
  
mov \[ebx\].file\_handle,eax ;保存句柄    
mov \[ebx\].file\_size,ecx ;保存大小    
  
push eax     
push ecx     
call \_map\_file ;以文件内存映像的方式打开文件，这样个方式的好处真是多多    
test eax,eax     
je \_exit\_close     
  
test ecx,ecx    
je \_exit\_close     
  
mov \[ebx\].mem\_ptr,eax     
mov \[ebx\].mem\_handle,ecx     
;下面是文件合法性判断    
cmp word ptr\[eax\],'ZM' ;判断文件头部的 Dos 标志“MZ”     
jne \_bad\_format    
  
add eax,dword ptr\[eax+3Ch\] ; eax->文件 NTHeader    
mov lpPeheader,eax     
xchg eax,edi ;edi->文件 NTHeader    
  
cmp dword ptr\[edi\],00004550h ; 判断是否是 PE 文件    
jne \_bad\_format     
  
  
cmp dword ptr\[edi+entrypointRVA\],0 ;判断文件的入口地址为 0 否？    
je \_bad\_format     
  
test word ptr\[edi+DllFlags\],2000h ; 判断是否是 Dll 文件    
jne \_bad\_format     
  
movzx edx,word ptr\[edi+NtHeaderSize\] ;edx->NtHeader 尾部，我记不清楚这里是否是节表开头了，你可以查一下    
lea edx,\[edi+edx+18h\] ; edx->节表头部    
  
cmp dword ptr\[edx\],'fed.' ; 判断该文件是否被 Def 加壳    
je \_bad\_format     
  
  
mov lpSectionTable,edx ; 保存节表头部    
  
movzx ecx,word ptr\[edi+numObj\] ; ecx->节表数目    
  
; edx <-- 节表头部    
; edi <-- PE 头    
; ecx <-- 节表数目    
  
\_encrypt\_sections:    
; 判断 edx 指向的节区是否该加密    
call \_is\_encryptable     
test eax,eax     
je \_encrypt\_skip     
  
mov esi,dword ptr\[edx+objpoff\] ;esi->节区文件偏移地址    
add esi,\[ebx\].mem\_ptr     
;esi->节区虚拟地址    
  
mov eax,dword ptr\[edx+objpsize\] ;eax->节区大小    
  
\_encrypt: ;加密该节区    
xor byte ptr\[esi\],al ;进行异或加密    
inc esi ;指向节区下一个字节    
dec eax ;判断是否到节区尾部，同时也是下一次异或操作的一个变量    
jne \_encrypt    
  
inc eax     
\_encrypt\_skip:     
mov dword ptr\[edx\],'fed.' ;修改节表名称    
bswap eax ;eax 由 01000000 改为 00000001，如果开始为 0，则仍然为 0    
mov dword ptr\[edx+4\],eax ;把 eax 写入节表名称后面,这样做是为了标志该节表是否被加密    
;1->被加密    
;0->未被加密    
mov dword ptr\[edx+objflags\],0C00000E0h ;修改节区属性为可写可读    
  
add edx,objlen ;edx->下一个节表    
loop \_encrypt\_sections ;转入下一个节表的处理     
  
  
; korygowanie offsetow loadera    
mov edx,dword ptr\[edi+imagebase\] ;读入基址    
mov dword ptr\[\_ldr\_imagebase\],edx ;保存基址，便于被加壳文件的载入    
  
mov eax,dword ptr\[edi+entrypointRVA\];读入入口地址    
add eax,edx ;入口地址的 VA    
mov dword ptr\[\_ldr\_host\],eax ;保存入口地址，便于被加壳文件的载入    
  
mov eax,lpSectionTable     
sub eax,\[ebx\].mem\_ptr    
add eax,edx ; VA    
mov dword ptr\[\_ldr\_sections\],eax ;保存节表头的地址    
  
movzx ecx,word ptr\[edi+numObj\] ;节表数目    
mov byte ptr\[\_ldr\_count\],cl    
  
imul ecx,objlen ;ecx->节表大小    
  
mov eax,lpSectionTable ;    
sub eax,\[ebx\].mem\_ptr    
  
add eax,ecx ;eax->节表尾部的 RVA    
  
mov dword ptr\[edi+entrypointRVA\],eax;再次保存入口地址，上面的保存其实多余    
  
add eax,\[ebx\].mem\_ptr    
xchg eax,edi     
mov esi,offset \_loader ; 初始化\_loader 的偏移地址    
mov ecx,\_loader\_size ; 初始化 loader 部分的大小    
rep movsb ; 把 loader 部分拷贝到被加壳文件的节表尾部    
;其实这里最好判断一下是否有足够空间放置本段代码    
  
push -1     
call MessageBeep ;响铃    
jmp \_exit\_unmap    
  
\_bad\_format:     
push 10h     
push offset szDef    
push offset szError    
push 0    
call MessageBoxA    
  
\_exit\_unmap:     
push \[ebx\].mem\_ptr    
push \[ebx\].mem\_handle    
call \_unmap    
  
\_exit\_close:    
push \[ebx\].file\_handle     
call CloseHandle    
  
\_exit:    
push -1    
call ExitProcess ; bye bye    
  
  
  
; 下面这一段被放入被加壳文件的节表尾部，作为程序运行时候的解密部分    
  
\_loader:    
mov esi,987654321 ;这里的 987654321 只是为了给\_ldr\_sections 留够 4 个字节的空间    
\_ldr\_sections equ dword ptr $-4 ;定义\_ldr\_sections 及其地址，请注意查找前面的设置    
  
push 0 ;节表数目    
\_ldr\_count equ byte ptr $-1    
pop ecx ;ecx->节表数目，在前面有设置    
  
\_ldr\_next: ;解密开始    
cmp byte ptr\[esi+7\],0 ;判断这个节区是否该解密，根据\[esi+7\]判断，请参考前面的注释    
je \_ldr\_not\_encrypted ;如果不该解密，越过下面的部分    
  
mov eax,dword ptr\[esi+objrva\] ;eax->节区的 Rva    
add eax,987654321 ;eax->节区的 VA    
\_ldr\_imagebase equ dword ptr $-4 ;请注意参考前面的设置    
  
mov edx,dword ptr\[esi+objpsize\] ;节区大小    
  
\_ldr\_decrypt: ;异或解密    
xor byte ptr\[eax\],dl    
inc eax ;下一个字节    
dec edx    
jne \_ldr\_decrypt    
  
\_ldr\_not\_encrypted:    
  
add esi,objlen ;esi->下一个节区    
loop \_ldr\_next ;开始下一个节区的解密    
  
push 987654321 ;程序原来的入口地址入栈，这样，下面的 ret 便返回到\_ldr\_host 处执行    
\_ldr\_host equ dword ptr $-4    
ret    
  
\_loader\_end:    
\_loader\_size equ $-\_loader    
  
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
;    
; mov edx,offset section\_header    
; call \_is\_rsrc    
;    
; sprawdza, czy sekcja zawiera zasoby    
;    
; na wyjsciu:    
; eax - 1 sekcja zasobow    
; 0 brak zasobow    
;    
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
  
\_is\_rsrc proc near ；判断节表名称是否为 rsr 或者 reso，因为这些节区不可以加密    
  
cmp dword ptr\[edx\],'rsr.' ; rsrc    
je \_rsrc\_section    
  
cmp dword ptr\[edx\],'oser' ; resource    
je \_rsrc\_section    
  
mov eax,dword ptr\[edx+objrva\]    
cmp eax,dword ptr\[ebx+resource\]     
je \_rsrc\_section     
  
sub eax,eax    
ret    
  
\_rsrc\_section:    
sub eax,eax    
inc eax    
ret    
  
\_is\_rsrc endp    
  
  
  
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
;    
; mov edx,offset section\_header    
; call \_is\_encryptable    
;    
; eax - 0 不可加密    
; 1 可加密    
;    
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
  
\_is\_encryptable proc near    
push ebx    
mov ebx,dword ptr\[lpPeheader\]    
  
call \_is\_rsrc     
dec eax    
je \_bad\_section    
  
  
cmp dword ptr\[edx\],'ler.' ; reloc    
je \_bad\_section    
  
cmp dword ptr\[edx\],'ade.' ; edata    
je \_bad\_section    
  
cmp dword ptr\[edx\],'ete.' ; etext    
je \_bad\_section    
  
cmp dword ptr\[edx\],'adr.' ; rdata    
je \_bad\_section    
  
cmp dword ptr\[edx\],'slt.' ; tls    
je \_bad\_section    
  
cmp dword ptr\[edx\],'adi.' ; idata    
je \_bad\_section    
  
  
sub eax,eax    
  
cmp dword ptr\[edx+objpoff\],eax ;判断节区的属性是否合法，下面的几个也是    
je \_bad\_section     
  
cmp dword ptr\[edx+objpsize\],eax     
je \_bad\_section     
  
  
mov eax,dword ptr\[edx+objrva\] ;保存 rva    
  
cmp eax,dword ptr\[ebx+resource\] ;判断节区的 rva 是否和资源的 rva 一样，如果一样则不可加密    
je \_bad\_section     
  
cmp eax,dword ptr\[ebx+edatadir\] ;我也不太清楚，你可以跟踪看一下    
je \_bad\_section     
  
cmp eax,dword ptr\[ebx+import\]    
je \_bad\_section    
  
cmp eax,dword ptr\[ebx+reloc\] ;reloc 是说重定位表    
je \_bad\_section     
  
cmp eax,dword ptr\[ebx+tls\] ;我也不知道    
je \_bad\_section ;    
  
sub eax,eax    
jmp \_check\_exit    
  
\_bad\_section:    
or eax,-1    
  
\_check\_exit:    
inc eax    
pop ebx    
  
ret    
  
\_is\_encryptable endp    
  
  
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
;下面是文件的打开和关闭部分，我就不献丑写注释了。大家要不太清楚，查以下书好了    
; push offset lpFilename    
; call \_open\_file    
;    
; Otwiera plik do zapisu i odczytu z podanej lokalizacji    
;    
; na wejsciu:    
; lpFilename - nazwa pliku do otworzenia    
;    
; na wyjsciu:    
; eax - uchwyt pliku lub -1 jesli blad    
; ecx - rozmiar pliku    
;    
; modyfikowane rejestry:    
; eax,ecx,edx    
;    
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
  
\_open\_file proc near    
pop eax    
pop edx    
push eax    
  
push edx    
  
push FILE\_ATTRIBUTE\_ARCHIVE ; atrybuty    
push edx ; nazwa pliku    
call SetFileAttributesA ; usun atrybuty blokujace dostep do zapisu    
  
pop edx ; pop nazwa pliku    
  
sub eax,eax    
push eax    
push FILE\_ATTRIBUTE\_NORMAL    
push OPEN\_EXISTING ; akcja, otworz plik    
push eax    
push eax    
push GENERIC\_READ + GENERIC\_WRITE ; tryb dostepu    
push edx ; nazwa pliku    
call CreateFileA    
  
cmp eax,-1 ; sprawdz wartosc zwrocona przez CreateFileA    
je \_open\_err ; jesli to -1 to znaczy, ze wystapil blad    
; wyjdz z takim wynikiem z funkcji    
  
push eax ; zapamietaj na stosie uchwyt pliku    
  
push 0 ; 0 dla plikow mniejszych niz 4gb    
push eax ; uchwyt pliku    
call GetFileSize ; pobierz rozmiar pliku    
  
pop ecx ; pop, uchwyt pliku do ecx    
xchg eax,ecx ; zamien wartosci w rejestrach, tak, ze    
; w eax znajdzie sie uchyt pliku a w ecx    
; rozmiar pliku    
\_open\_err:    
ret ; wyjscie z funkcji    
  
\_open\_file endp    
  
  
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
;    
; push hFile    
; push cbSize    
; call \_map\_file    
;    
; Mapuje plik do pamieci, wszystkie operacje dokonywane na pamieci maja fizyczne    
; odzwierciedlenie na dysku    
;    
; na wejsciu:    
; hFile - uchwyt pliku otwartego w trybie do odczytu i zapisu    
; cbSize - rozmiar pliku    
;    
; na wyjsciu:    
; eax - wskaznik do mapowanego pliku lub 0 jesli blad    
; ecx - uchwyt mapowanego pliku    
;    
; modyfikowane rejestry:    
; eax,ecx,edx    
;    
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
  
\_map\_file proc near    
  
pop eax ; adres powrotu    
pop ecx ; rozmiar pliku    
pop edx ; uchwyt pliku    
  
push eax    
  
sub eax,eax    
push eax    
push ecx ; rozmiar pliku(+opcjonalnie rozmiar    
; dodtkowego obszaru pamieci)    
  
push eax ;    
push PAGE\_READWRITE ; tryb dostepu do mapy pliku    
push eax    
push edx ; uchwyt pliku    
call CreateFileMappingA    
  
push eax ; zapamietaj uchwyt mapy pliku    
  
sub edx,edx ; zeruj edx    
  
push edx ; ilosc bajtow do mapowania, gdy 0    
; mapowany jest caly plik    
push edx ;    
push edx ;    
push FILE\_MAP\_WRITE ; flagi dostepu    
push eax ;    
call MapViewOfFile ; w eax wskaznik do mapy pliku    
  
pop ecx ; uchwyt mapy pliku do ecx    
ret    
\_map\_file endp    
  
  
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
;    
; push lpMap    
; push hMap    
; call \_unmap    
;    
; zamyka mapowany plik + zapisuje zmiany jakie dokonano na mapie pliku    
;    
; na wejsciu:    
; lpMap - wskaznik do mapy pliku    
; hMap - uchwyt mapy pliku    
;    
; na wyjsciu:    
; brak    
;    
; modyfikowane rejestry:    
; eax,ecx,edx    
;    
;哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪    
  
\_unmap proc near    
pop eax    
  
pop ecx ; hMap    
pop edx ; lpMap    
  
push eax    
  
push ecx ; zapamietaj parametr dla CloseHandle    
  
push edx ; lpMap    
call UnmapViewOfFile ; usuwa obraz mapy pliku z przestrzeni adresowej    
; naszego procesu    
  
call CloseHandle ; zamknij mape pliku    
  
ret    
\_unmap endp    
  
end \_start  
    
 4dd466d0e1e0cc123afbe504da2462ca