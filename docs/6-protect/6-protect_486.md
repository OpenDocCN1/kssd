

# 通用且简单的扩充和改造 PE 中的 call 的功能的方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-525.htm`](https://www.pediy.com/kssd/pediy07/pediy7-525.htm)

技术基础：会编写加壳程序、了解 PE 结构、X86 指令系统。  
开发工具：MASM、VC++  
  
1、我们有时候在改造一个 PE 的时候，对其中的某些 call 要做大量的改动，这时  
候，往往通过手动增加一个节，在这个节中完成这个 call 的改造，再返回原来的地方继续执行。此法虽然可行，但工作量实在太大，且调试十分不方便。  
  
2、“通用简单的方法”之实现原理：  
    使用 jmp far address 和 call far address 的机器码长度相同，且仅仅第一字节不同的原则，jmp 第一字节是 0xE9，而 call 第一字节是 0xE8，后面 4 字节相同。  
    这种方法就是：通过给 PE 加个小壳，同时在加壳的时候，将 PE 中要改造的 call 修改成调用外壳中的 call，在外壳中的 call 中调用原来 PE 中的 call，而外壳中的 call 我们在编写的时候较容易实现功能扩充和改造。  
    在外壳中的 call 中，我们有两种扩充功能的方式：  
    (1)在调用原来 PE 中的 call 之前扩充功能；  
    (2)在调用原来 PE 中的 call 之后扩充功能；  
    扩充功能可以直接在外壳 call 中实现，但较难调试和复杂。  
    简单通用的方法是这个外壳中的新 call 仅仅完成：  
        1、加载一个外部补丁 DLL；  
        2、调用这个外部补丁 DLL 中的函数，完成原来 PE 中的 call 的功能扩充和改造。  
        3、调用原来 PE 中的 call 的；  
    是先调用原来 PE 中的 call 还是后调用，视具体需要决定。这样我们就可以很简单的写扩充的功能代码了，因为这个外部补丁 DLL 可以用高级语言来写，且调试十分简单。  
  
3、外壳中函数的具体实现：  
    (一) 首先，在外壳中设定几个变量：  
    (1) 保存外部 DLL 的各个函数地址的变量(DWORD)：  
        pfnExtPatchFun\_1--pfnExtPatchFun\_n，n 为多大具体看要改造 PE 中几个函数。  
    (2) 保存原来 PE 中的 call 的“地址差”变量(DWORD)：  
        dwOrgCallDifference\_1--dwOrgCallDifference\_n，n 为多大具体看要改造 PE 中几个函数。  
        dwOrgCallDifference\_x = 原来 call 的机器码的后 4 个字节(双字)。  
    (3) 保存原来 PE 中的 call 的 RVA 地址的变量(DWORD)：  
        dwOrgCallRVA\_1--dwOrgCallRVA\_n，n 为多大具体看要改造 PE 中几个函数。  
    (4) 保存外部 DLL 的 hModule 的变量(DWORD):  
        hExtPatchDll。  
    ;-----------------------------------------------------------------  
    举例说明：  
      如：原来 PE 的 ImageBase=0x00400000，且有下面代码(IDA 显示)：  
          .  
          .  
          004010CB 50                push    eax  
          004010CC 57                push    edi  
          004010CD E8 CE 08 00 00    call    sub\_4019A0 ;需要改造的 call，注意机器码后 4 字节  
          004010D2 85 C0             test    eax, eax  
          .  
          .  
      则：dwOrgCallRVA\_1 = 004010CD - 0x00400000 = 0x000010CD  
          dwOrgCallDifference\_1 = 0x000008CE  
   ;------------------------------------------------------------------  
   ;  
   (二) 在外壳程序中编写这些 call，需要改造几个函数，就有几个 call，格式如下：  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;---------------------------------------------------;  
; 外壳中的 call,用于替换原来 PE 中的 call               ;  
;---------------------------------------------------;  
align 4  
wjq\_API\_SMC\_Label\_1:  
GeneralPEShellCall\_1    proc  
;{  
    ;原 PE 中的 call 的入口参数，我们通过堆栈传递到 Dll 中的新 call，此时需注意 EBP 在本 call 中不要改变。  
    @nPara1 EQU \[ebp+0ch\] ;原来 PE 中的 Call 的参数，也可以通过 esp 取得  
    @nPara2 EQU \[ebp+8h\]  
    ;  
    pusha  
    call GetPatchDllFunctions ;获取外部 DLL 中的所有函数地址并保存  
    call GetAddressDifference ;获取地址差  
    .if \[pfnExtPatchFun\_1+eax\] != 0  ;如果功能函数 1 实现了，就调用  
       push  @nPara1 ;原来 PE 中的 Call 的参数  
       push  @nPara2  
       call  \[pfnExtPatchFun\_1+eax\] ;调用外部 DLL 中的补丁 call  
    .endif  
    ;------------调用原来的 call-------------------------  
    call GetAddressDifference ;获取地址差  
    lea esi,\[TempCall\_1+eax\]  ;现在 call 的 VA  
    sub esi,\[Image\_Base+eax\]  ;转换成 RVA  
    sub esi,\[dwOrgCallRVA\_1+eax\] ;减去原来 PE 中的 call 的 RVA = 两个 call 的地址差  
    ;  
    mov ebx,\[dwOrgCallDifference\_1+eax\]  ;原来 call xxxxxxxx 指令的偏移差  
    .if ebx < 0  ;负数，原来 call 向上调用  
        add ebx,esi  
    .else        ;正数，原来 call 向下调用  
        sub ebx,esi  
    .endif  
    ;  
    mov  \[TempCallDifference\_1+eax\],ebx  ;修正后的差值  
    popa  
    ;  
    ;下面等同于一个 jmp far xxxxxxxx 语句,调用原来的 call  
    ;这里之所以采用这种方式，而不是用 call，是为了通用，因为 PE 中的 call 有些是\_\_stdcall，有些是\_\_\_cdecl，如果一概采用 call 实现，要考虑出栈问题，不是很通用了。  
TempCall\_1           db 0E9h  
TempCallDifference\_1 dd  0  
    ret  
;}  
GeneralPEShellCall\_1 endp  
  
;---------------------------------------------------------------------  
; 获取外部 DLL 中的所有函数地址并保存  
; 此函数中使用的 W\_LoadLibraryA 等 W\_ 开始的函数，是外壳自建引入表是实现的  
;---------------------------------------------------------------------  
GetPatchDllFunctions  proc  
    pusha  
    call GetAddressDifference  
    mov  edx,eax  
    .if \[hExtPatchDll+edx\] == 0 ;如果已经装载了，就直接返回。  
        ;  
        lea ebx,\[pExtPatchDll+edx\]  
        push edx ;保存地址差  
        ;--------------------------  
        push ebx  
        call \[W\_LoadLibraryA+edx\]  
        ;--------------------------  
        pop  edx ;恢复地址差  
        mov  \[hExtPatchDll+edx\],eax  
        .if eax != 0  
           ;-------------;  
           ;  function\_1 ;  
           ;-------------;  
           push   edx  ;保存地址差  
           lea   ebx,\[pExtPatchFun\_1+edx\]  
           push   ebx  ; 功能名的字符偏移  
           push   dword ptr \[hExtPatchDll+edx\] ; 模块的句柄  
           call   \[W\_GetProcAddress+edx\] ; 调用 Kernel32!GetpProcAddress 以获得功能调用地址  
           pop   edx ;恢复地址差  
           mov   \[pfnExtPatchFun\_1+edx\],eax  
           ;-------------;  
           ;  function\_2 ;  
           ;-------------;  
           push   edx  ;保存地址差  
           lea   ebx,\[pExtPatchFun\_2+edx\]  
           push   ebx  ; 功能名的字符偏移  
           push   dword ptr \[hExtPatchDll+edx\] ; 模块的句柄  
           call   \[W\_GetProcAddress+edx\] ; 调用 Kernel32!GetpProcAddress 以获得功能调用地址  
           pop   edx ;恢复地址差  
           mov   \[pfnExtPatchFun\_2+edx\],eax  
           .  
           .  
           ;-直到取得 n 个函数地址。  
       .endif  
   .endif  
   popa  
   ret  
GetPatchDllFunctions  endp  
  
;-------------------------------------------------------------------  
;该函数返回：地址差  
;-------------------------------------------------------------------  
align 4  
GetAddressDifference proc  
   call  @F  
@@:  
   pop  eax             ;获得实际偏移  
   sub  eax, offset @B  ;减去偏移=地址差  
   ret  
GetAddressDifference endp  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
4、加壳部分的实现：  
   相对来说，加壳部分的实现加简单，只要按照上面计算 call 的偏移差的方法，计算出新的偏移差，改写这个 call 的机器码的后 4 字节为新的偏移差，使其调用我们外壳中的 call 就可以了。  
  ;  
  具体计算过程如下：  
  mov  eax,dwOrgCallRVA\_1    ;原来 PE 中 call 处的 RVA，对应上面例子就是：dwOrgCallRVA\_1 = 0x000010CD  
  mov  edx,Sheller\_RVA       ;外壳程序入口的 RVA  
  add  edx,API\_CALL\_ADJUST\_1 ;API\_CALL\_ADJUST\_1 = wjq\_API\_SMC\_Label\_1 - Sheller\_Start  
  sub  edx,eax   ;差值  
  sub  edx,5     ;减去 Call 指令的长度 5  
  mov  eax,edx   ;此时后的 eax 就是要修改的原 PE 中的 call 的后 4 字节(偏移差)，对应上面例子就是：004010CD E8 CE 08 00 00 的 CE 08 00 00，变为双字就是 0x000008CE，这个值改为 eax 中的值。  
  
  需要注意的是：API\_CALL\_ADJUST\_1 的计算要使用 wjq\_API\_SMC\_Label\_1 标号，而不是函数名字 GeneralPEShellCall\_1。  
  
  
5、外部 DLL 的编写格式(使用高级语言，快速编写，十分方便的调试)：  
//具体参数格式，请根据原来 PE 中的 call 的入口参数确定  
//调用约定请参照外壳中的 call，主要看出栈规则，我用的是\_\_stdcall，也可以用\_\_\_cdecl，主要根据外壳 call 中的使用方式，这个不能错，否则堆栈就乱套了。  
extern "C" DWORD \_\_stdcall ExtPatchFun\_1(DWORD nPara1,DWORD nPara2)  
{  
  //实现功能扩充。。。  
  return;  
}  
//  
extern "C" DWORD \_\_stdcall ExtPatchFun\_2(DWORD nPara1,DWORD nPara2)  
{  
  //实现功能扩充。。。  
  return;  
}  
  
6、关于补丁 DLL 的卸载  
   我们在外壳 call 中只加载外部 DLL，但不去卸载。这会不会有问题呢？回答是否定的，因为 PE 在结束进程时候，会自动卸载所有加载到本进程的 DLL，所以，不用关心这个 DLL 不会卸载掉。而且，我们通过只加载方式使用这个 DLL，会给我们带来很大方便，因为，只要在一处加载了这个外部 DLL，我们就可以在原 PE 的任何地方调用其中的函数。  
  
7、上面介绍的是先调用功能扩展部分，再调用原来的 call。反之实现的方法类似，可自行发挥。  
  
  排版不是很好，凑合着看吧。。呵呵。  
                                                       Spring.W  
                                                       2005.3.14 

不错，详细介绍了与外壳代码通信的一种可行方案  
补充一下，如何在原 PE 文件中选择合适的 call  
call XXXXXXXX  
搜索 0E8h，并确定相应指令的 RVA，进而计算出 XXXXXXXX 确定是否位于代码段。不知道误码的概率是多少，是否还有其他的方法搜索这种有效替换指令呢？  
如果在外壳中这样处理还有一个缺陷，那就是脱壳以后外壳代码仍然有效，被替换的 call 无需修复，但是若外壳申请动态内存保存附加的功能代码，则势必要同时修补这些被替换的 call 指令的相对偏移量，从而把修复的线索保留在了外壳的执行过程中。  
  
记得 Spring.W 兄在以前的精华里提过一个与外壳通信的简单思路，好像是替换 add eax,XXXXXXXX。  
相比之下，如今的想法更成熟可行度也更高了![](img/710b5589904d0d2bf58a67748fca40e1.jpg)   

 4dd466d0e1e0cc123afbe504da2462ca