

# 新春大奉献\_9 城名将三国的分析

> 原文：[`www.pediy.com/kssd/pediy11/107013.html`](https://www.pediy.com/kssd/pediy11/107013.html)

【文章标题】: 新春大奉献\_9 城名将三国的分析  
【文章作者】: thomasyzh  
【作者邮箱】: machinesy@gmail.com  
【作者 QQ 号】: 516674080  
【软件名称】: \_9 城名将三国的分析  
【下载地址】: 自己搜索下载  
【保护方式】: HS  
【编写语言】: c++  
【软件介绍】: 第 9 城市的一个游戏  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  传闻写外挂很赚钱.于是我搞了搞.之前没有灰色产业链的商业积累.身边的朋友也越来越少.基本上外挂这块由于牵扯到钞票  
  的关系,所以很少有人讨论.  
  其实我一开始的主要目的是像赚钱.也没打算发出来.但是觉得自己现在的编程技术和水平想吃这个票子确实来的有点太累了  
  所以痛下决心,再花半年打基础.至少怎完 vmp.然后把写脚本,写插件那些能力再提高一翻翻.再来吃这个票子.  
  首先向在背后默默鼓励我的商务朋友表示歉意.东西我还是会慢慢完成.就当作练习.只是觉得,忙活了半天没让你们赚到钱实  
  在有点不好意思.  
  谈技术了:  
  1.hs 保护:  
  我打 log 分析出了第九城 wof 的 hs 只需要调其 hs 的 7 号函数  
    HMODULE hEhsvc = NULL;   
    hEhsvc = (HMODULE)GetModuleHandleA("EHSvc.dll");  
    if (hEhsvc == INVALID\_HANDLE\_VALUE)  
    {  
      printf("<<<<<<<<<<<<<<<get EHSvc.dll handle error<<<<<<<<<<<\\r\\n");  
    }  
    else  
    {  
      UnStallAntiDebug  UnStall = NULL;  
      UnStall = (UnStallAntiDebug)GetProcAddress( hEhsvc,"7");  
      if (UnStall == NULL)  
      {  
        printf("<<<<<<<<<<<<<<<get EHSvc.dll get fun 7 Error<<<<<<<<<<<\\r\\n");  
        MessageBoxA(NULL,"abc","abc",MB\_OK);  
      }  
      else  
      {  
        UnStall();  
      }  
        
    }  
  流程如上代码.  
    
  执行后就卸载掉保护了.  
  2.还原封包  
  2.1  
     非动作封包还原.  
     因为动作封包在包上有个 index 所以我把这个游戏的封包分成非动作封包和动作封包.具体哪个 index 的意义,我还没有  
  分析.  
     打开商店的封包明文  
     BYTE   OpenShop\[\] = {0x1b,0x00,0x00,0x00,0x08,0x00,0x01,0x12,0x00,0x00,0x00,0x00};--这个包完整的说,不因  
  该算打开的全明文.世纪其头部有个经过另外一个函数算出来的效验值.  
     打开商店的全代码:  
       RtlZeroMemory(&g\_packCmd,sizeof(PACK\_CMD));  
    BYTE   OpenShop\[\] = {0x1b,0x00,0x00,0x00,0x08,0x00,0x01,0x12,0x00,0x00,0x00,0x00};  
    g\_packCmd.dwCmdAddr = (DWORD)OpenShop;  
    g\_packCmd.dwCmdLen = 0x0c;  
    g\_packCmd.dwEncodeTableLen = 0x10;  
    if (dwEncodeTable!=0)  
    {  
      g\_packCmd.dwEncodeTable = dwEncodeTable;  
    }  
    else  
    {  
      MessageBoxA(NULL,"Dot get table","Dot get table",MB\_OK);  
      return;  
    }  
    \_\_asm  
    {  
            
      mov    eax,g\_packCmd.dwEncodeTableLen  
      push  eax  
      mov    eax,g\_packCmd.dwEncodeTable  
      sub     eax,0x10  
      push    eax  
      mov     esi,eax  
      mov     eax,edi  
      mov     eax,g\_packCmd.dwCmdLen  
      push    eax  
      mov    eax,g\_packCmd.dwCmdAddr  
      push    eax  
      mov     eax,Package\_Data  
      call  eax  
      add    esp,0x10  
      xor    eax,eax  
      mov     edi,esi  
  table\_change:  
      add byte ptr \[eax+edi\],al  
      add eax,1  
      cmp eax,0x10  
      jl  table\_change  
    }  
    BYTE  SendEncodeTable\[0x10\] = {0};  
    DWORD \*pdwpointer = (DWORD\*)SendEncodeTable;   
    \*pdwpointer = (DWORD)dwHead+0x10;  
    memcpy((SendEncodeTable+4),(void\*)g\_packCmd.dwCmdAddr,0x0c);  
    send(g\_send,(char\*)SendEncodeTable,0x10,0);  
    
  这个代码中起重函数 Package\_Data 是风暴的加密函数.加密方式我就不多说了.是垃圾.  
  给出封包加密函数的汇编代码  
          \_Package\_Data   proc    near  
          mov    eax,dword ptr \[esp+08h\]  
          xor     ecx, ecx  
          xor     edx, edx  
          test    eax, eax  
          jbe     short FLA\_1  
          push    ebx  
          push    ebp  
          mov     ebp, dword ptr \[esp+14h\]  
          push    esi  
          mov     esi, dword ptr \[esp+10h\]  
          push    edi  
          mov     edi, dword ptr \[esp+20h\]  
          lea     esp, dword ptr \[esp\]  
  FLA\_3:          
          mov     bl, byte ptr \[ecx+ebp\]  
          xor     byte ptr \[edx+esi\], bl  
          add     ecx, 1  
          cmp     ecx, edi  
          jb      short FLA\_2  
          xor     ecx, ecx  
  FLA\_2:  
          add     edx, 1  
          cmp     edx, eax  
          jb      short FLA\_3  
          pop     edi  
          pop     esi  
          pop     ebp  
          pop     ebx  
  FLA\_1:  
          retn      
  \_Package\_Data  endp  
     具体数据结构,请自己分析分析.  
    
  算封包效验的函数代码  
  \_Package\_Head  proc    near  
           movzx   edx,word ptr \[ecx\]  
           xor     eax,eax  
           test    dx,dx  
           jbe     short FLA\_1  
           movzx   edx, dx  
           push    esi  
           mov     edi, edi  
  FLA\_2:           
           movzx   esi, byte ptr \[ecx\]  
           add     eax, esi  
           add     ecx, 1  
           sub     edx, 1  
           jnz     short FLA\_2  
           pop     esi  
  FLA\_1:           
           retn  
  \_Package\_Head endp   
  具体我也不说了.这个在有 index 的风暴就要自己去算下效验.比如说攻击.  
  下边我贴出一个攻击的发包代码  
    
    BYTE   AttackPack\[\] = {0xaf,0x05,0x00,0x00,0x10,0x00,0x17,0x08,0x0a,0xa1,0x34,0x00,0x00,0x21,0x24,0x60,0xff,0xff,0xff,0xff};  
    g\_packCmd.dwCmdAddr = (DWORD)AttackPack;  
    g\_packCmd.dwCmdLen = 0x14;  
    g\_packCmd.dwEncodeTableLen = 0x10;  
      
      
    if (g\_packIndex == 0)  
    {  
      MessageBoxA(NULL,"packIndex error","packIndex error",MB\_OK);  
      return;  
    }  
    if (dwEncodeTable!=0)  
    {  
      g\_packCmd.dwEncodeTable = dwEncodeTable;  
    }  
    else  
    {  
      MessageBoxA(NULL,"Dot get table","Dot get table",MB\_OK);  
      return;  
    }  
    \_\_asm  
    {  
      lea  ecx,AttackPack  
      mov  eax,g\_packIndex  
      mov  eax,dword ptr \[eax+0x04\]  
      mov  dword ptr \[ecx+0x09\],eax  
      add  ecx,4  
      mov  eax,Package\_Head  
      call eax  
      lea  ecx,AttackPack  
      mov  dword ptr \[ecx\],eax  
    }  
    \_\_asm  
    {  
      mov    eax,g\_packCmd.dwEncodeTableLen  
      push  eax  
      mov    eax,g\_packCmd.dwEncodeTable  
      push    eax  
      mov     esi,eax  
      mov     edi,eax  
      mov     eax,0x14  
      push    eax  
      mov     eax,g\_packCmd.dwCmdAddr  
      push    eax  
      mov    eax,Package\_Data  
      call  eax  
      add    esp,0x10  
      xor     eax,eax  
      mov     edi,esi  
      change\_table:  
      add byte ptr \[eax+edi\],al  
      add eax,1  
      cmp eax,0x10  
      jl  change\_table  
    }  
    
    BYTE attackPakc\[0x18\] = {0};  
    DWORD\* dwordPointer = (DWORD\*)attackPakc;  
    \*dwordPointer = (DWORD)dwHead+0x18;  
    memcpy((attackPakc+0x04),(void\*)g\_packCmd.dwCmdAddr,0x14);  
    send(g\_send,(char\*)attackPakc,0x18,0);  
    
    
    //index+14  
    \_\_asm  
    {  
      mov eax,g\_packIndex  
      mov ebx,dword ptr \[eax+0x04\]  
      add ebx,0x14  
      mov dword ptr \[eax+0x04\],ebx  
    }  
    return;  
       
  具体过程就是.  
  1.取 index  
  2.算效验  
  3.加密.  
  4.加个包长度头  
  5.发包.  
    
  到这里,有心的人可以作脱机挂了.只是还由很多的过程要分析.只能说还原了封包.重要的还是分析后边的协议.不过分析  
  协议不复杂.  
    
    
    
  下边谈内挂.  
    
  我一开是想做变态的全屏秒杀挂.难免要找一些数据结构.  
  具体我分析到如下结果了.  
    
  其主程序很多时候使用一片内存作为参数传递,那我们就适时的在一些特定的点动态的修改其内存.实现修改攻击等数据.  
  我找到了攻击坐标.  
  实现了全屏打.但是一旦攻击一下.其就会切换验证模式.然后就失效了.总的来说,就是可以在单人模式下.攻击一次.  
    
  我的修改和我的修改的地方的特征地址大概如下代码.具体游戏版本可能有更新.不过按照我的思路,修改那片内存,一样可  
  以实现很多功能.不过一定要在适当的时候修改.  
  代码如下.特征码在注释片断.  
  //attack x  
  /\*  
  005CF942  |.  51            push    ecx  
  005CF943  |.  8B4C24 78     mov     ecx, dword ptr \[esp+78\]  
  005CF947  |.  50            push    eax  
  005CF948  |.  51            push    ecx  
  005CF949  |.  8BCF          mov     ecx, edi  
  005CF94B  |.  E8 B071E6FF   call    00436B00  
  005CF950  |.  8B5424 3C     mov     edx, dword ptr \[esp+3C\]  
  005CF954  |.  52            push    edx  
  005CF955  |.  894424 54     mov     dword ptr \[esp+54\], eax  
  005CF959  |.  C78424 C40000>mov     dword ptr \[esp+C4\], -1  
  005CF964  |.  E8 A7F4FFFF   call    005CEE10  
  005CF969  |.  8BF8          mov     edi, eax  
  005CF96B  |.  83C4 04       add     esp, 4  
    
    
    
    
  \*/  
  /\*  
    if 0x7ff73fa0  
    real 0x7ff73fd0 +0x04  
    EDI  
  \*/  
    
  extern BOOL bGetAttackXBaseAddr;  
  extern int g\_attackXValue;//set value  
  #define ATTACX\_X\_CODE       0x005CF950  
  void  attackXInline();  
    
  extern BOOL bEnableY;  
  extern int g\_attackY1Value;  
  extern int g\_attackY2Value;  
  extern int g\_attackY3Value;  
    
  /\*  
    attack Test point 2  
    004374F0   .  56            push    esi  
    004374F1   .  57            push    edi  
    004374F2   .  8B7C24 0C     mov     edi, dword ptr \[esp+C\]  
    004374F6   .  57            push    edi  
    004374F7   .  8BF1          mov     esi, ecx  
    004374F9   .  E8 B2D1FEFF   call    004246B0  
    004374FE   .  57            push    edi  
    004374FF   .  E8 6CBAFEFF   call    00422F70  
    00437504   .  57            push    edi  
    00437505   .  8946 1C       mov     dword ptr \[esi+1C\], eax  
    00437508   .  E8 63BAFEFF   call    00422F70  
    0043750D   .  57            push    edi  
    0043750E   .  8946 24       mov     dword ptr \[esi+24\], eax  
    00437511   .  E8 5ABAFEFF   call    00422F70  
    00437516   .  57            push    edi  
    00437517   .  8946 28       mov     dword ptr \[esi+28\], eax  
    0043751A   .  E8 51BAFEFF   call    00422F70  
    0043751F   .  57            push    edi  
    00437520   .  8946 2C       mov     dword ptr \[esi+2C\], eax  
    00437523   .  E8 48BAFEFF   call    00422F70  
    00437528   .  57            push    edi  
    00437529   .  8946 30       mov     dword ptr \[esi+30\], eax  
    0043752C   .  E8 3FBAFEFF   call    00422F70  
    00437531   .  57            push    edi  
    00437532   .  8946 34       mov     dword ptr \[esi+34\], eax  
    00437535   .  E8 36BAFEFF   call    00422F70  
    0043753A   .  57            push    edi  
    0043753B   .  8946 38       mov     dword ptr \[esi+38\], eax  
    0043753E   .  E8 2DBAFEFF   call    00422F70  
    00437543   .  57            push    edi  
    00437544   .  8946 3C       mov     dword ptr \[esi+3C\], eax  
    00437547   .  E8 24BAFEFF   call    00422F70  
    0043754C   .  57            push    edi  
    0043754D   .  8946 50       mov     dword ptr \[esi+50\], eax  
    00437550   .  E8 1BBAFEFF   call    00422F70  
    00437555   .  57            push    edi  
    00437556   .  8946 44       mov     dword ptr \[esi+44\], eax  
    00437559   .  E8 12BAFEFF   call    00422F70  
    0043755E   .  57            push    edi  
    0043755F   .  8946 48       mov     dword ptr \[esi+48\], eax  
    00437562   .  E8 09BAFEFF   call    00422F70  
    00437567   .  57            push    edi  
    00437568   .  8946 20       mov     dword ptr \[esi+20\], eax  
    0043756B   .  E8 00BAFEFF   call    00422F70  
    00437570   .  83C4 34       add     esp, 34  
    00437573   .  6A 01         push    1  
    00437575   .  8BCF          mov     ecx, edi  
    00437577   .  8946 54       mov     dword ptr \[esi+54\], eax  
    0043757A   .  E8 11B8FEFF   call    00422D90  
    0043757F   .  85C0          test    eax, eax  
    00437581   .  0F95C0        setne   al  
    00437584   .  6A 01         push    1  
    00437586   .  8BCF          mov     ecx, edi  
    00437588   .  8846 40       mov     byte ptr \[esi+40\], al  
    0043758B   .  E8 00B8FEFF   call    00422D90  
    00437590   .  85C0          test    eax, eax  
    00437592   .  0F95C1        setne   cl  
    00437595   .  5F            pop     edi  
    00437596   .  884E 4C       mov     byte ptr \[esi+4C\], cl  
    00437599   .  5E            pop     esi  
    0043759A   .  C2 0400       retn    4  
  \*/  
    
    
    
  /\*  
    attack call point 3  
    00853DF0   $  55            push    ebp  
    00853DF1   .  8BEC          mov     ebp, esp  
    00853DF3   .  83E4 C0       and     esp, FFFFFFC0  
    00853DF6   .  6A FF         push    -1  
    00853DF8   .  68 E3F8A700   push    00A7F8E3  
    00853DFD   .  64:A1 0000000>mov     eax, dword ptr fs:\[0\]  
    00853E03   .  50            push    eax  
    00853E04   .  81EC E8000000 sub     esp, 0E8  
    00853E0A   .  53            push    ebx  
    00853E0B   .  56            push    esi  
    00853E0C   .  57            push    edi  
    00853E0D   .  A1 20E2CF00   mov     eax, dword ptr \[CFE220\]  
    00853E12   .  33C4          xor     eax, esp  
    00853E14   .  50            push    eax  
    00853E15   .  8D8424 F80000>lea     eax, dword ptr \[esp+F8\]  
    00853E1C   .  64:A3 0000000>mov     dword ptr fs:\[0\], eax  
    00853E22   .  894C24 24     mov     dword ptr \[esp+24\], ecx  
    00853E26   .  8B5D 08       mov     ebx, dword ptr \[ebp+8\]  
    00853E29   .  8B43 24       mov     eax, dword ptr \[ebx+24\]  
    00853E2C   .  50            push    eax  
    00853E2D   .  E8 CED5FCFF   call    00821400  
    00853E32   .  8B4B 28       mov     ecx, dword ptr \[ebx+28\]  
    00853E35   .  8BF0          mov     esi, eax  
    00853E37   .  51            push    ecx  
    00853E38   .  897424 30     mov     dword ptr \[esp+30\], esi  
    00853E3C   .  E8 3FABD7FF   call    005CE980  
    00853E41   .  83C4 08       add     esp, 8  
    00853E44   .  85F6          test    esi, esi  
    00853E46   .  8BF8          mov     edi, eax  
    00853E48   .  0F84 8F050000 je      008543DD  
    00853E4E   .  85FF          test    edi, edi  
    00853E50   .  0F84 87050000 je      008543DD  
  \*/  
    
  /\*  
    attack X Inline Here  
    008233F1   .  8B5C24 08     mov     ebx, dword ptr \[esp+8\]  
    008233F5   .  55            push    ebp  
    008233F6   .  8B6C24 10     mov     ebp, dword ptr \[esp+10\]  
    008233FA   .  56            push    esi  
    008233FB   .  57            push    edi  
    008233FC   .  8B7C24 1C     mov     edi, dword ptr \[esp+1C\]  
    00823400   .  57            push    edi  
    00823401   .  55            push    ebp  
    00823402   .  53            push    ebx  
    00823403   .  8BF1          mov     esi, ecx  
    00823405   .  E8 D62B0300   call    00855FE0  
    0082340A   .  53            push    ebx  
    0082340B   .  55            push    ebp  
    0082340C   .  57            push    edi  
    
    
    
    
    
    
    
  \*/  
  #define ATTACK\_X\_DEBUG 0x0082340A  
  extern int g\_attackXDebugValue;  
  void attackXDebugInline();  
    
  /\*  
  005C0E6B   .  8B13          mov     edx, dword ptr \[ebx\]  
  005C0E6D   .  8B52 1C       mov     edx, dword ptr \[edx+1C\]  
  005C0E70   .  8BC6          mov     eax, esi  
  005C0E72   .  50            push    eax  
  005C0E73   .  8BCB          mov     ecx, ebx  
  005C0E75   .  FFD2          call    edx  
  005C0E77   .  8B75 18       mov     esi, dword ptr \[ebp+18\]  
  005C0E7A   .  C745 FC 00000>mov     dword ptr \[ebp-4\], 0  
  005C0E81   .  8B46 08       mov     eax, dword ptr \[esi+8\]  
  005C0E84   .  8D48 10       lea     ecx, dword ptr \[eax+10\]  
  \*/  
    
  #define ATTACK\_X\_DEBUG\_1 0x005C0E7A  
  extern int g\_attackDebugValue1;  
  void attackXDebugInline1();  
    
    
    
  具体 inline 后的修改代码如下:  
  1  
  InlineHook g\_attackX;  
  int       g\_attackXValue;  
    
  BOOL bEnableY;  
    
  int g\_attackY1Value;  
  int g\_attackY2Value;  
  int g\_attackY3Value;  
    
  BOOL bGetAttackXBaseAddr;  
  DWORD dwAttackXBaseAddr;  
    
  void attackXInline()  
  {    
    dwAttackXBaseAddr = 0;  
    bEnableY = FALSE;  
    bGetAttackXBaseAddr = FALSE;  
    g\_attackXValue =0;  
    g\_attackY1Value = 0;  
    g\_attackY2Value = 0;  
    g\_attackY3Value = 0;  
    g\_attackX.dwsrcaddress = 0;  
    \_\_asm{  
        mov eax,real\_fun  
        mov g\_attackX.dwdesaddress ,eax  
    }  
    
    HMODULE hNtdll = NULL;  
    
    g\_attackX.dwsrcaddress = ATTACX\_X\_CODE;  
    if (g\_ZwSetInfomationThread.dwsrcaddress == 0)  
    {  
    
    }  
    HookDestFunc(g\_attackX.dwsrcaddress,g\_attackX.dwdesaddress,5);  
    goto END\_PROC;  
    
    
    \_\_asm  
    {  
  real\_fun:  
        pushad  
        pushfd  
    }  
    if (bGetAttackXBaseAddr)  
    {  
      \_\_asm  
      {  
        mov dwAttackXBaseAddr,edi  
      }  
    }  
    if (dwAttackXBaseAddr == 0x00)  
    {  
      \_\_asm jmp end\_fun  
    }  
      
      
      
    \_\_asm  
    {  
        mov eax,dwAttackXBaseAddr  
        add eax,0x30  
        mov ecx,g\_attackXValue  
        mov dword ptr \[eax+0x04\],ecx;  
    }  
    //是否测试坐标 y  
    if (bEnableY)  
    {  
      \_\_asm  
      {  
        mov eax,dwAttackXBaseAddr  
        add eax,0x30  
        mov ecx,g\_attackY1Value  
        mov dword ptr \[eax-0x04\],ecx  
        mov ecx,g\_attackY2Value  
        mov dword ptr \[eax+0x10\],ecx  
        mov ecx,g\_attackY3Value  
        mov dword ptr \[eax+0x1c\],ecx   
      }  
        
    }  
    \_\_asm  
    {  
  end\_fun:  
        popfd  
        popad  
    }  
    \_\_asm{  
          
        mov     edx, dword ptr \[esp+0x3C\]    
        push    edx  
        mov  edx,g\_attackX.dwsrcaddress  
        add  edx,0x05  
        jmp  edx  
      }  
  END\_PROC:  
    return;  
  }  
    
  2  
    
  void attackXDebugInline()  
  {  
    g\_attackXDebugValue = 0;  
    \_\_asm  
    {  
      mov eax,real\_fun  
      mov g\_attackXDebugInline.dwdesaddress,eax  
    }  
    g\_attackXDebugInline.dwsrcaddress = ATTACK\_X\_DEBUG;  
    
    HookDestFunc(g\_attackXDebugInline.dwsrcaddress,g\_attackXDebugInline.dwdesaddress,5);  
    goto END\_PROC;  
    \_\_asm  
    {  
  real\_fun:  
      pushad  
      pushfd  
      mov eax,dwAttackXBaseAddr  
      add eax,0x30  
      /\*mov ecx,g\_attackXDebugValue\*/  
      mov ecx,g\_attackXValue  
      mov dword ptr \[eax+0x04\],ecx;  
      popfd  
      popad  
      push ebx  
      push ebp  
      push edi  
      mov  ecx,esi  
      mov  eax,g\_attackXDebugInline.dwsrcaddress  
      add  eax,0x05  
      jmp  eax  
    }  
  END\_PROC:  
    return;  
  }  
  3  
  InlineHook g\_attackDebugInline1;  
  int g\_attackDebugValue1;  
  void attackXDebugInline1()  
  {  
    \_\_asm  
    {  
      mov eax,real\_fun  
      mov g\_attackDebugInline1.dwdesaddress,eax  
    }  
    g\_attackDebugInline1.dwsrcaddress = ATTACK\_X\_DEBUG\_1;  
      
    HookDestFunc(g\_attackDebugInline1.dwsrcaddress,g\_attackDebugInline1.dwdesaddress,0x07);  
    goto END\_PROC;  
    \_\_asm{  
  real\_fun:  
        pushad  
        pushfd  
        mov eax,dwAttackXBaseAddr  
        add eax,0x30  
         /\*mov ecx,g\_attackDebugValue1\*/  
        mov ecx,g\_attackXValue  
         mov dword ptr \[eax+0x04\],ecx;  
        popfd  
        popad  
        mov     dword ptr \[ebp-0x4\], 0x0  
        mov eax,g\_attackDebugInline1.dwsrcaddress  
        add eax,0x07  
        jmp eax  
    }  
  END\_PROC:  
    return;  
  }  
    
    
  这三处同时挂之修改之.实现 xxx 之.分析出来的方法有很多种.  
    
    
  我的分析就到这了.  
    
    
    
    
    
    
\--------------------------------------------------------------------------------  
【经验总结】  
  总的来说,制作外挂的方法有很多种.应该要先学习好调试技巧,和调试工具.外挂的制作是没有技术含量的.我就是调试技太  
  差.造成我觉得现在来搞这玩艺,只能使土炮打分析.打得相当辛苦.只能说能做.不能做做好.  
  祝各位新春快乐  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2010 年 02 月 09 日 16:56:57 

貌似已经不是 7 号了吧。  
int \_\_usercall 7<eax>(int a1<eax>)  
{  
  signed int v2; // \[sp-14h\] \[bp-30h\]@4  
  int v3; // \[sp+0h\] \[bp-1Ch\]@1  
  int v4; // \[sp+Ch\] \[bp-10h\]@1  
  int (\_\_cdecl \*v5)(int, int, int); // \[sp+10h\] \[bp-Ch\]@1  
  int v6; // \[sp+14h\] \[bp-8h\]@1  
  int v7; // \[sp+18h\] \[bp-4h\]@1  
  
  v6 = (int)dword\_100BA8F8;  
  v5 = unknown\_libname\_24;  
  v4 = a1;  
  v3 = 0;  
  v7 = 0;  
  if ( (unsigned \_\_int8)byte\_100DCB8D == 1 )  
  {  
    EnterCriticalSection(&stru\_100DCB50);  
    v2 = 117954820;  
    JUMPOUT(\*(unsigned int \*)loc\_101936F2);  
  }  
  v3 = 3;  
  sub\_1000D7E0(0, dword\_100CB9D0, dword\_100CB9D4, dword\_100CB9D8, dword\_100CB9DC, &Default);  
  v7 = -1;  
  sub\_100112C5();  
  return v3;  
}