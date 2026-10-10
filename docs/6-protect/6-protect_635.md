

# ExeCryptor v2.2.6 虚拟机不完全分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-651.htm`](https://www.pediy.com/kssd/pediy07/pediy7-651.htm)

ExeCryptor v2.2.6 虚拟机不完全分析  
  
  
追 ExeCryptor 的 stolen codes 不大容易,跟一跟就会发现总是在某个循环里打转。碰到  
类似这样的代码就表示进入循环了:  
  
00587B19    BD 01000000             mov ebp,1  
00587B1E    F0:0FC12D FCA95500      lock xadd dword ptr ds:\[55A9FC\],ebp       ; LOCK prefix  
  
004FE66F    89EA                    mov edx,ebp  
004FE671    80F2 9D                 xor dl,9D  
004FE674    50                      push eax  
004FE675    68 B4F5DE83             push 83DEF5B4  
004FE67A    58                      pop eax  
004FE67B  - E9 72C90700             jmp dumped\_.0057AFF2  
...  
00519630    02D0                    add dl,al  
00519632  - 0F80 82BAFDFF           jo dumped\_.004F50BA  
00519638    58                      pop eax  
00519639    C0CA 07                 ror dl,7  
0051963C    53                      push ebx  
0051963D    68 EFBB226E             push 6E22BBEF  
00519642  - E9 8EBBFDFF             jmp dumped\_.004F51D5  
...  
00534234    23D3                    and edx,ebx  
00534236    5B                      pop ebx  
00534237    B8 00010000             mov eax,100  
0053423C    E9 E1650700             jmp dumped\_.005AA822  
  
  
ExeCryptor 的文档里使用了虚拟机一词:Protection options 中的 Virtualization level  
定义了虚拟机保护级别。关于 oep 保护,文档的说明是:  
  
Original entry point code will be transformed and protected like it is wrapped   
with crypt\_start/crypt\_end macros. Note: if you protect Delphi application you   
need to use EXECryptor.pas in your project for correct working of this option.   
  
也就是说,对 oep 代码的保护与使用 SDK 中的 crypt\_start/crypt\_end 宏是一样的。可以自己  
写几句代码,使用 SDK 试试。最好就用 Demo 版,不选择 Anti-debug,这样方便一些。Demo 下的  
虚拟机代码更简单,我没有细看,但想必不会因为 protection level 的不同而导致对 opcode  
进行不同的解释。  
  
对虚拟机代码的分析我用的是 dumped\_.exe,这样干扰少些。  
  
  
1. 虚拟机代码的基本结构  
  
   在壳代码中定义以下数组:  
  
   \_2fimh7mp:005A3A18 RegEbp          dd 100h dup(0)   
   \_2fimh7mp:005A3E18 RegEdi          dd 100h dup(0)  
   \_2fimh7mp:005A4220 RegEsi          dd 100h dup(0)   
   \_2fimh7mp:005777EC RegEdx          dd 100h dup(0)  
   \_2fimh7mp:00577BEC RegEbx          dd 100h dup(0)  
   \_2fimh7mp:005A525C RegEcx          dd 100h dup(0)  
   \_2fimh7mp:005A4E54 RegEax          dd 100h dup(0)  
   \_2fimh7mp:005A4A48 RegEsp          dd 100h dup(0)  
  
   用于保存进出虚拟机前后的寄存器。  
  
   \_2fimh7mp:005A463C DecryptedData   dd 100h dup(0)  
  
   用于保存解出的 opcode。  
  
   \_2fimh7mp:005A3908 busy            db 100  dup(0)  
  
   这是个字节数组。所有的数组都包含 256 个 item。ExeCryptor 使用了多线程,不同线程中  
   的代码都会调用虚拟机解释引擎。这个 byte 数组用于表示对应的 slot(这个词是我瞎编的;-)  
   是否可用,如果为 1,代码将选择别的 slot。  
  
   下面的代码用来选择 slot:  
  
   00587B19    BD 01000000             mov ebp,1  
   00587B1E    F0:0FC12D FCA95500      lock xadd dword ptr ds:\[55A9FC\],ebp       ; LOCK prefix  
   004FE66F    89EA                    mov edx,ebp  
   004FE671    80F2 9D                 xor dl,9D  
   00519630    02D0                    add dl,al  
   00519639    C0CA 07                 ror dl,7  
   00534234    23D3                    and edx,ebx  
   00534237    B8 00010000             mov eax,100  
   005496E9    F0:0FB0A2 08395A00      lock cmpxchg byte ptr ds:\[edx+5A3908\],ah   ; LOCK prefix  
   005496F1  - 0F84 E238F6FF           je <dumped\_.loc\_4ACFD9>  
   005496F7    E9 00F30000             jmp dumped\_.005589FC  
  
   如果可用,ebp 为 index,用来访问上面所有的数组。  
  
   虚拟机执行流程大致为:  
  
   . 保存寄存器  
   . 解密 opcode 及参数  
   . 执行 opcode  
   . 恢复寄存器  
  
   在执行过程中会检测 TF 标记,如果被跟踪,将破坏访问数据的指针。  
  
   调用虚拟机执行代码的形式为:  
  
   \_2fimh7mp:0054E10E                 call    l\_execute\_pcode\_2  
   \_2fimh7mp:0054E113                 dd 13F66194h  ; 密文 dword1            
   \_2fimh7mp:0054E117                 dd 652004C7h  ; 密文 dword2  
  
   call 时指向密文数据的指针作为返回地址入栈。有 1 个 dword 的,也有 2 个 dword。一般  
   在 IDA 中看看后续的代码可以分清。在解码后的判断中也可识别。  
   
   opcode 大约 30 种,我没有跟完,只看了追 stolen codes 相关部分(所以不完全;-)。  
  
   值得注意的是,在对密文 opcode 解码的过程中,执行到第 4 轮时使用了 call(即前面的  
   54E10E 处)时的 eflag 值。不知道这里我是否理解错了。开始以为是执行到判断处时  
   的 eflag,但看了几次,的确用的是 call 时进入虚拟机后 pushfd 的值。  
  
   虚拟机解释代码是固定的,但进入的路径(即前面 call 的地址)很多,我碰到了 10 余处。  
  
   本来想跟出所有的 PCode,写个程序 patch dumped\_.exe,这样一来就无法实现了。  
   有趣的是 stolen codes 本身(即被 crypt\_start/crypt\_end 保护的代码)基本是明文,  
   被虚拟机执行的都是壳代码,也许这可以映证前面的判断是对的,壳代码难以判断  
   被抽取代码的运行状态。  
  
   对这点我没有把握:-(。  
  
  
2. 下面跟一个 PCode 执行实例  
  
   \_2fimh7mp:0057915A                 call    near ptr l\_execute\_pcode\_1 ;   
   \_2fimh7mp:0057915F                 dd 0BAC63BB5h             
   \_2fimh7mp:00579163 dword\_579163    dd 0D08139CEh             
  
   从 call 处跟进。  
  
   0054D5E7  - 0F82 36BBFAFF           jb dumped\_.004F9123  
   0054D5ED    9C                      pushfd  
   0054D5EE    50                      push eax  
   0054D5EF    51                      push ecx  
   0054D5F0  ^ E9 3458FDFF             jmp dumped\_.00522E29  
   ...  
   005348F5    8BCA                    mov ecx,edx  
   005348F7    870C24                  xchg dword ptr ss:\[esp\],ecx  
   005348FA    56                      push esi  
   005348FB    8BF5                    mov esi,ebp  
   005348FD  - E9 0C48FCFF             jmp dumped\_.004F910E  
     
   004F910E    873424                  xchg dword ptr ss:\[esp\],esi  
   004F9111    BD 01000000             mov ebp,1  
   004F9116    F0:0FC12D FCA95500      lock xadd dword ptr ds:\[55A9FC\],ebp   ; LOCK prefix  
   004F911E  - E9 A62A0A00             jmp dumped\_.0059BBC9  
  
   寻找可用的 slot。  
  
   004FE66F    89EA                    mov edx,ebp  
   004FE671    80F2 9D                 xor dl,9D  
   004FE674    50                      push eax  
   004FE675    68 B4F5DE83             push 83DEF5B4  
   004FE67A    58                      pop eax  
   004FE67B  - E9 72C90700             jmp dumped\_.0057AFF2  
   ...  
   00519630    02D0                    add dl,al  
   00519632  - 0F80 82BAFDFF           jo dumped\_.004F50BA  
   00519638    58                      pop eax  
   00519639    C0CA 07                 ror dl,7  
   0051963C    53                      push ebx  
   0051963D    68 EFBB226E             push 6E22BBEF  
   00519642  - E9 8EBBFDFF             jmp dumped\_.004F51D5  
   ...  
   00534234    23D3                    and edx,ebx  
   00534236    5B                      pop ebx  
   00534237    B8 00010000             mov eax,100  
   0053423C    E9 E1650700             jmp dumped\_.005AA822  
  
   005496E9    F0:0FB0A2 08395A00      lock cmpxchg byte ptr ds:\[edx+5A3908\],ah    ; LOCK prefix  
   005496F1  - 0F84 E238F6FF           je <dumped\_.loc\_4ACFD9>  ; slot 空闲  
   005496F7    E9 00F30000             jmp dumped\_.005589FC  
  
   004ACFD9 >  C1E2 02                 shl edx,2               ; \*4,用于访问 dword 数组  
   004ACFDC    E9 1F010000             jmp <dumped\_.loc\_4AD100>  
  
   004AD100 >  89D5                    mov ebp,edx                      
   004AD102    8F85 183A5A00           pop dword ptr ss:\[ebp+5A3A18\]  ; 保存 ebp  
   004AD108    5A                      pop edx  
   004AD109    872C24                  xchg dword ptr ss:\[esp\],ebp  
   004AD10C    8BC5                    mov eax,ebp  
   004AD10E  - E9 22ED0A00             jmp dumped\_.0055BE35  
   ...  
   0055BE35    5D                      pop ebp  
   0055BE36    89BD 183E5A00           mov dword ptr ss:\[ebp+5A3E18\],edi ; 保存 edi  
   0055BE3C    E8 1752FEFF             call dumped\_.00541058  
  
   逐个保存 register,省略。进入是 pushfd 的值保存在 edi 内。  
     
   ...  
   0053B94F    8BC4                    mov eax,esp  
   0053B951    8985 484A5A00           mov dword ptr ss:\[ebp+5A4A48\],eax ; 保存 esp  
   0053B957    50                      push eax  
   0053B958    68 E8FFBEEE             push EEBEFFE8  
   0053B95D    58                      pop eax  
   0053B95E    81F0 51CB835E           xor eax,5E83CB51  
   0053B964    E9 CDAD0100             jmp dumped\_.00556736  
  
   00556736    81C0 B3B41B50           add eax,501BB4B3  
   0055673C    E8 0709FAFF             call dumped\_.004F7048  
  
   004F7048    871C24                  xchg dword ptr ss:\[esp\],ebx  
   004F704B    5B                      pop ebx  
   004F704C    870424                  xchg dword ptr ss:\[esp\],eax  ; push eax  
   004F704F  - E9 F5E40500             jmp dumped\_.00555549  
  
   这是被混淆过的代码,eax 最终为 58E96C,作为返回地址入栈。  
  
   00555549  ^\\E9 8C34FEFF             jmp dumped\_.005389DA ; 解码  
     
   005389DA    8B06                    mov eax,dword ptr ds:\[esi\]  ; 取密文 dword1  
   005389DC    8BD0                    mov edx,eax  
   005389DE    C1C8 10                 ror eax,10    ; \*  
   005389E1    66:C1C2 03              rol dx,3      ; \*  
   005389E5    E9 A3C50300             jmp dumped\_.00574F8D  
   ...  
   0057B8BF    03D6                    add edx,esi   ; \*  
   0057B8C1    5E                      pop esi  
   0057B8C2    57                      push edi  
   0057B8C3    E8 A7CAFBFF             call dumped\_.0053836F  
   ...  
   005A1129    81C7 E5BACC97           add edi,97CCBAE5  
   005A112F    9D                      popfd  
   005A1130    33D7                    xor edx,edi   ; \*  
   005A1132  ^ E9 2B05FDFF             jmp dumped\_.00571662  
  
   标有\*的为解码代码,这里执行完 1 轮。共 16 轮(不会是什么密码学算法吧,眼拙没看出来)。  
  
   004F8C79    57                      push edi  ; 第 4 轮时使用 eflag  
   004F8C7A    5A                      pop edx  
   ...  
   004F8AFD    84F1                    test cl,dh ; 检测 TF 位  
   ...  
   0054A6AA    0F95C1                  setne cl  
   0054A6AD    D3C8                    ror eax,cl ; eax 为上轮结果,搞点破坏  
   0054A6AF    E9 83480200             jmp dumped\_.0056EF37  
   ...  
   00589A79    23D7                    and edx,edi     ; 即 and edx,000008C1  
   005A96BE    81CA 3EF7FFFF           or edx,FFFFF73E ;  
   005A96C4    23C2                    and eax,edx  
    
   可以看到 eflag 为解码过程的确有影响。  
  
   ...  
  
   0057151F    33D7                    xor edx,edi  
   00571521    5F                      pop edi  
   00571522    66:33C2                 xor ax,dx  
   00571525    8985 3C465A00           mov dword ptr ss:\[ebp+5A463C\],eax ;  保存解码结果  
   0057152B    57                      push edi  
   0057152C  ^ E9 92FAFEFF             jmp dumped\_.00560FC3  
   ...  
   0058AE3C    03F7                    add esi,edi  ; add esi,4 移动指针->下 1 个密文 dword  
   ...  
   00599C43    85C1                    test ecx,eax ; test eax,E0000000h  
  
   检测解码结果,若真,代表某些特殊情况,会 mov eax,3F,跳过参数处理。我只  
   碰到过 1 次。3F 似乎等于 NOP。这里为 false。  
  
   00523DBB    85C2                    test edx,eax ; test edx,000000C0h  
   00523DBD    5A                      pop edx  
   00523DBE    0F84 356C0300           je dumped\_.0055A9F9  
   00523DC4  ^ E9 C5FDFFFF             jmp dumped\_.00523B8E  ; 走这里  
  
   若这个检测为 true,表示有第 2 个 dword 需要处理。这里为 true。  
  
   00565180    8B16                    mov edx,dword ptr ds:\[esi\] ; 取第 2 个 dword  
   00565182    C1E8 08                 shr eax,8  
   00565185    25 FF0F0000             and eax,0FFF  
   ...  
   00541AC7    23D1                    and edx,ecx ;  and  edx,000FFFFFh,取低 20 位  
   ...  
   00569C44    C1E2 0C                 shl edx,0C  
   00569C47    09D0                    or eax,edx  
   00569C49    8985 F87F5700           mov dword ptr ss:\[ebp+577FF8\],eax ;保存参数  
   ...  
   0051BF33    85C2                    test edx,eax ; test eax,00000080h  
   0051BF35    5A                      pop edx  
   0051BF36    0F84 BDEA0300           je dumped\_.0055A9F9  
   0051BF3C   /E9 02F10200             jmp dumped\_.0054B043  
  
   这里还有个检查,没看懂,如果 51BF3C 过去,似乎没什么实质的影响。  
  
   ...  
   0055A9F9    C3                      retn ; -> 58E96C 前面入栈的地址  
  
   00582560    8B00                    mov eax,dword ptr ds:\[eax\] ; 取解码结果  
   00582562    89C3                    mov ebx,eax ; 保存到 ebx 内  
   00582564  ^ E9 F083FEFF             jmp dumped\_.0056A959  
   ...  
   00584C9E    25 FF000000             and eax,0FF ; 最低字节为 opcode 类型  
   00584CA3    E8 8E5DF9FF             call dumped\_.0051AA36  
     
   0051AA36    871C24                  xchg dword ptr ss:\[esp\],ebx  
   0051AA39    5B                      pop ebx  
   0051AA3A    C1EB 08                 shr ebx,8 ; ebx 保留高 24 位  
   0051AA3D    83F8 00                 cmp eax,0 ; 开始逐个判断 opcode 类型  
   0051AA40    0F85 7C330500           jnz dumped\_.0056DDC2  
   0051AA46    E9 5DBB0100             jmp dumped\_.005365A8  
  
   这里 opcode = 0x44,到这里:  
  
   0057F98E    FFB5 F87F5700           push dword ptr ss:\[ebp+577FF8\] ; psuh 解出的参数  
   0057F994    83AD 484A5A00 04        sub dword ptr ss:\[ebp+5A4A48\],4 ; RegEsp 保存值减 4  
   0057F99B    33D2                    xor edx,edx  
   0057F99D    8BC7                    mov eax,edi  
   0057F99F    E9 71340000             jmp dumped\_.00582E15  
  
   下面再次检测 TF 标记,恢复各寄存器后退出虚拟机。  
  
   所以,opcode 0x44 = push imm32  
  
  
3. 解码程序及 stolen codes  
  
   我写了个程序来帮助分析,包含了跟 stolen codes 用到的 opcode。别的有兴趣的  
   朋友自己解决了;-)。  
  
     
   #include "stdafx.h"  
   #include <stdio.h>  
   #include <windows.h>  
  
   #pragma pack(push)  
   #pragma pack(1)  
  
   typedef struct \_SEED   
   {  
  DWORD  dwSeed1;  
  DWORD  dwSeed2;  
   } SEED,\*PSEED;  
  
   typedef struct \_CIPHER  
   {  
  DWORD  dwCipher1;  
  DWORD  dwCipher2;  
  DWORD  dwAddr;    // for debug,调用地址  
   } CIPHER,\*PCIPHER;  
  
   #pragma pack(pop)  
  
   SEED seeds\[\] = {  
    {0x0012D6B7,0x3BA8D443}, {0x0895445C,0x125AF2C9},  
    {0xA25E6A75,0x9261DE28}, {0xF967E9C6,0x931AB604},  
    {0xE74E7E14,0x5C0720F2}, {0x1B032895,0x070A5A10},  
    {0x429532EE,0xD3AA7A83}, {0x8E1FB211,0xB626113A},  
    {0x2EE5122A,0xE1719AC6}, {0xF3E3C5AB,0xED28B712},  
    {0x39502C58,0x536224AA}, {0x5B542ED3,0x9A2010C6},  
    {0xC28D35D5,0x01434C1A}, {0x4349B4B5,0x05092628},  
    {0x742FA3CB,0xF63BAA9C}, {0x8DCCEE88,0x962B5A4E},  
    {0xFDD60B98,0x41FD37AD}, {0x5687BC5F,0x4BA7B00D},  
    {0x2EE9B07D,0xDA1C3F7F}, {0xD533FEC6,0x64DF11FF},  
    {0x5B786F02,0x7CFD0370}, {0xF602CC3E,0x6EDA23B9},  
    {0x766CE221,0xFE351A14}, {0xB6C81829,0xA313B9D8},  
    {0xAA9CC06B,0xF64D7535}, {0xB6FA5ED0,0x2477B71B},  
    {0x263A9ED7,0x79A0668D}, {0x09E0F41A,0xEDC2AB89},  
    {0x8731DF1C,0x47B09DD0}, {0x4FA72153,0x2BE82BC9},  
    {0xACE42C68,0xA5875C45}, {0xF7CF2D22,0xC0CCC058}};  
      
  
   DWORD Decrypt(DWORD dwCipher,PSEED pSeed,int nNumOfSeeds,DWORD dwEFlag)  
   {  
  // dwCipher: 密文数据  
  // pSeed->dwSeed1: 用于 add 操作  
  // pSeed->dwSeed2: 用于 xor 操作  
  // dwEFlag: 执行 call 操作时的 eflag 值  
  
  DWORD  dwData = dwCipher;  
      
  for(int i = 0; i < nNumOfSeeds; i++)  
  {  
      
    DWORD dwSeed1 = (pSeed + i)->dwSeed1;  
    DWORD dwSeed2 = (pSeed + i)->dwSeed2;  
  
    \_asm  
    {  
      mov    eax,dwData  
      mov    edx,dwData  
        
      ror       eax,0x10    
      rol       dx,0x3  
      add    edx,dwSeed1  
      xor    edx,dwSeed2  
      xor       ax, dx  
      mov    dwData,eax  
    }  
      
    // 在第 4 轮后,eflag 参与了计算  
      
    if(3 == i)  
    {  
       // 8C1 = 1000 1100 0001 -> OF SF ZF CF  
       // 73E = 0111 0011 1110  
  
                   // 保留了 4 个标记位信息  
  
                   dwData &= (dwEFlag & 0x000008C1) | 0xFFFFF73E;  
        
    }  
  }  
    
  return dwData;  
   }  
  
  
   int main(int argc, char\* argv\[\])  
   {  
  CIPHER  stCipher;  
      
  int nNumOfSeeds = sizeof(seeds) / sizeof(SEED);  
  ZeroMemory(&stCipher,sizeof(stCipher));  
    
  //  
  
  stCipher.dwCipher1 = 0xBAC63BB5;  // 密文 dword1  
  stCipher.dwCipher2 = 0xD08139CE;  // 密文 dword2  
  stCipher.dwAddr = 0x57915F;    // 密文 dword1 地址,用于调试可以不设  
  
  //  
  DWORD dwEFlag = 0x00000246;    // 执行 call 时的 eflag  
  
  DWORD dwPlain = Decrypt(stCipher.dwCipher1,seeds,nNumOfSeeds,dwEFlag);  
  
  printf("解码结果 = %08X\\r\\n",dwPlain);    
  
  // 检测解码结果  
    
  DWORD  dwParameter = 0;  
  BOOL  bParamAvailable = FALSE;  
  //   
    
  if(dwPlain & 0xE0000000)  // 似乎是些特殊代码,跳过参数处理  
  {  
    dwPlain = 0x3F;  // 不需要参数处理  
  }  
  else  
  {  
    if(dwPlain & 0x000000C0)  
    {  
      bParamAvailable = TRUE;  
      dwParameter = ((dwPlain >> 0x8) & 0xFFF) |   
        ((stCipher.dwCipher2 & 0x000FFFFF) << 0xC);  
        
      // 注意还有个调整参数的动作(51BF33),似乎没有作用  
    }  
  }  
  
  //  
  
  BYTE cRegister = 0;  
  PCHAR lpszRegister\[\] = {"eax","ecx","edx","ebx","esp","ebp","esi","edi"};  
  
  
  switch (dwPlain & 0x000000FF)  
  {  
  case 0x0:  // push reg32  
  
    printf("%08X: opcode 0x00 -> push %s\\r\\n",  
      stCipher.dwAddr,  
      lpszRegister\[(dwPlain >> 0x8) & 0x7\]);  
  
    break;  
  
  case 0x1:  // pop reg32  
  
    printf("%08X: opcode 0x01 -> pop %s\\r\\n",  
      stCipher.dwAddr,  
      lpszRegister\[(dwPlain >> 0x8) & 0x7\]);  
      
    break;  
      
  case 0x2:  // mov reg32,reg32  
  
    printf("%08X: opcode 0x02 -> mov %s,%s\\r\\n",  
      stCipher.dwAddr,  
      lpszRegister\[(dwPlain >> 0xB) & 0x7\],  
      lpszRegister\[(dwPlain >> 0x8) & 0x7\]);  
      
    break;  
      
  case 0x3:  
    break;  
  
  case 0x4:  
    break;  
  
  case 0x5:  // xchg reg32,\[esp\]  
      
    printf("%08X: opcode 0x05 -> xchg %s,\[esp\] 返回%08X\\r\\n",  
      stCipher.dwAddr,  
      lpszRegister\[(dwPlain >> 0x8) & 0x7\],  
      stCipher.dwAddr + 4);  
  
    break;  
  
  case 0x6:  // ret  
  
    printf("%08X: opcode 0x06 -> ret\\r\\n",stCipher.dwAddr);  
    break;  
  
  case 0x7:  
    break;  
      
  case 0xC0:  // jmp imm32  
      
    printf("%08X: opcode 0xC0 -> jmp %08x\\r\\n",  
      stCipher.dwAddr,  
      dwParameter);  
      
    break;  
      
  case 0x82:  
    break;  
      
  case 0x44:  // push imm32  
      
    printf("%08X: opcode 0x44 -> push 立即数%08X,返回%08X\\r\\n",  
      stCipher.dwAddr,  
      dwParameter,  
      stCipher.dwAddr + 8);  
      
    break;  
  
  
  case 0x84:  // 似乎和 0x44 相同? push imm32  
  
    printf("%08X: opcode 0x84 -> push 立即数%08X,返回%08X\\r\\n",  
      stCipher.dwAddr,  
      dwParameter,  
      stCipher.dwAddr + 8);  
  
    break;  
  
  case 0x10:  
    break;  
  
  case 0x11:  
    break;  
  
  case 0x12:  
    break;  
  
  case 0x13:  
    break;  
  
  case 0x14:  
    break;  
      
  case 0x15:  // smc  
      
    printf("%08X: opcode 0x15-> SMC 修改%s 所指代码为%02x,返回%08x\\r\\n",  
      stCipher.dwAddr,  
      lpszRegister\[(dwPlain >> 0x8) & 0x7\],  
      (dwPlain >> 0xB) & 0x000000FF,  
      stCipher.dwAddr + 4);  
  
    break;  
  
  case 0x16:  
    break;  
  
  case 0x17:  
    break;  
  
  case 0x83:  
  
    // 这个不太肯定,处理代码直接测试 ebp(当前 slot 的 offset)而不是外界的数据  
    // 004AC8F7 test    ebp, 1  ; 把 offset 当 eflag 测试 CF,还是测试 ebp 的奇偶性(这样是 jp)  
    // 004AC8FD jz      loc\_53D934  ; 这里直接返回 call 的下一行  
    // 004AC903 jmp     loc\_52467F  ; 这里跳到解出的参数地址  
  
    // 不过对于 ExeCryptor 这种代码是干扰代码,最终会走到同样的地址  
    // 所以这里是否精确不大重要,但跟踪时要跟到 test 的结果才知道返回哪里  
  
    printf("%08X: opcode 0x83-> jc %08x\\r\\n",  
      stCipher.dwAddr,dwParameter);  
  
    break;  
  
    
  case 0x20:  // 与 0x20 的比较是>=,(在 5282C9)放在 case 内是否合适?   
    break;  
  
  case 0x3F:  
  
    printf("%08X: opcode 0x3F-> nop 返回%08x\\r\\n",  
      stCipher.dwAddr,  
      stCipher.dwAddr + 8);  
  
    break;  
  
      
  default:  
  
    if(0x8 == (dwPlain & 0x000000F8))    
    {  
      DWORD dwType = dwPlain & 0x7;  
      DWORD dwSrcRegister = (dwPlain >> 0x8) & 0x7;  
      DWORD dwDstRegister = (dwPlain >> 0xB) & 0x7;  
        
      switch(dwType)  // 这里还有别的标记没有跟  
      {  
      case 0:  // add reg32,reg32 (575755)  
  
        printf("%08X: opcode 0x8 -> add %s,%s,返回%08x\\r\\n",  
          stCipher.dwAddr,  
          lpszRegister\[dwDstRegister\],  
          lpszRegister\[dwSrcRegister\],  
          stCipher.dwAddr + 4);  
        break;  
  
  
      case 7:  // cmp reg32,reg32  
  
        printf("%08X: opcode 0x8 -> cmp %s,%s,返回%08x\\r\\n",  
          stCipher.dwAddr,  
          lpszRegister\[dwDstRegister\],  
          lpszRegister\[dwSrcRegister\],  
          stCipher.dwAddr + 4);  
          
        break;  
          
  
      default:  
        break;  
  
      }  
    }  
    else  
    {  
      DWORD dwConvert = (dwPlain + 0x40) & 0x87;  
  
      if(0x80 == dwConvert)  // add reg32,imm32  
      {  
        printf("%08X: opcode 0x48 -> add %s,%08x 返回%08x\\r\\n",  
          stCipher.dwAddr,  
          lpszRegister\[((dwPlain & 0x000000FF) >> 3) & 0x7\],  
          dwParameter,  
          stCipher.dwAddr + 8);  
      }  
      else if(0x81 == dwConvert)  // xor reg32,imm32  
      {  
        printf("%08X: opcode 0x49 -> xor %s,%08x 返回%08x\\r\\n",  
          stCipher.dwAddr,  
          lpszRegister\[((dwPlain & 0x000000FF) >> 3) & 0x7\],  
          dwParameter,  
          stCipher.dwAddr + 8);  
      }  
    }  
  
    break;  
  }  
  
  return 0;  
   }  
  
   代码可能有错,在与 0x11 比较后,对解码结果的最低字节做了些变换进行判断,  
   我放到 default 内了,可能不一定合适。不对的地方请包涵,指正。  
  
  
   追出的 ExeCryptor v2.2.6 主程序 stolen code 为:  
  
  
   004C74D0 >  55              push ebp                             
   004C74D1    8BEC            mov ebp,esp  
   004C74D3    83C4 F4         add esp,-0C  
   004C74D6    B8 20724C00     mov eax,dumped\_.004C7220  
   004C74DB    E8 6CF5F3FF     call <dumped\_.Sysinit::\_\_linkproc\_\_ InitExe(void)>  
   004C74E0 >  A1 D4634E00     mov eax,dword ptr ds:\[4E63D4\]                           
   004C74E5 >  8B00            mov eax,dword ptr ds:\[eax\]                                  
   004C74E7    E8 0CB8F8FF     call <dumped\_.Forms::TApplication::Initialize(void)>  
   004C74EC    E8 C7A6FFFF     call <dumped\_.sub\_4C1BB8>  
   004C74F1    A1 D4634E00     mov eax,dword ptr ds:\[4E63D4\]  
   004C74F6    8B00            mov eax,dword ptr ds:\[eax\]  
   004C74F8    BA 54754C00     mov edx,offset <dumped\_.aExecryptor>      ; ASCII "EXECryptor"  
   004C74FD    E8 FAB3F8FF     call <dumped\_.Forms::TApplication::SetTitle(System::AnsiString)>  
   004C7502    E8 B1A6FFFF     call <dumped\_.sub\_4C1BB8>  
   004C7507    E8 ACA6FFFF     call <dumped\_.sub\_4C1BB8>  
   004C750C >  8B0D B0614E00   mov ecx,dword ptr ds:\[4E61B0\]        
   004C7512    A1 D4634E00     mov eax,dword ptr ds:\[4E63D4\]  
   004C7517    8B00            mov eax,dword ptr ds:\[eax\]  
   004C7519    8B15 98214C00   mov edx,dword ptr ds:\[<off\_4C2198>\]     
   004C751F    E8 ECB7F8FF     call <dumped\_.Forms::TApplication::CreateForm(System::TMetaClass \*,void \*)>  
   004C7524 >  A1 D4634E00     mov eax,dword ptr ds:\[4E63D4\]     
   004C7529    8B00            mov eax,dword ptr ds:\[eax\]  
   004C752B    E8 60B8F8FF     call <dumped\_.Forms::TApplication::Run(void)>  
   004C7530    90              nop  
   004C7531    90              nop  
   004C7532    90              nop  
   004C7533    90              nop  
   004C7534    90              nop  
   004C7535 >  90              nop              
   004C7536    90              nop  
   004C7537    90              nop  
   004C7538    90              nop  
   004C7539    90              nop  
   004C753A    90              nop  
   004C753B    90              nop  
   004C753C    90              nop  
   004C753D    90              nop  
   004C753E    90              nop  
   004C753F    90              nop  
   004C7540    90              nop  
   004C7541    90              nop  
   004C7542    90              nop  
   004C7543    90              nop  
   004C7544    90              nop  
   004C7545    E8 D2C5F3FF     call <dumped\_.System::\_\_linkproc\_\_ Halt0(void)>  
   004C754A    0000            add byte ptr ds:\[eax\],al  
  
   最后,修复 oep 处的 stolen codes 不足以解决跨平台问题。主程序很多地方使用  
   了 SDK,还会返回壳内(因此也还会启动那些 anti-debug 线程)。要想剥干净,可以  
   把被抽掉的代码全部修复。  
  
   我是不想玩了,已经快精神错乱了...... 

 4dd466d0e1e0cc123afbe504da2462ca