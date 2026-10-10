

# STEP 7-MicroWIN 32 V3.1.0.31E

> 原文：[`www.pediy.com/kssd/pediy03/forum506.htm`](https://www.pediy.com/kssd/pediy03/forum506.htm)

软件  ：STEP 7-MicroWIN 32 V3.1.0.31E(西门子公司做的工控软件)   
注册费：不知道！   
保护  ：VBOX 4.2 (NAG&60 天时间限制)   
目的  ：杀掉保护   
工具  ：freevirus，TRW2000 1.22 (LTT 是最伟大的，向他致敬！我要去注册)，   
  
开始：   
  
用 BPX GETLOCALTIME，F12 后来到 VBOXS420.DLL 中，再 F12 几下，就会来到 MicroWin.exe 里：   
  
\* Possible StringData Ref from Data Obj ->"STEP-7MICROWINVER310EVAL"   
                                          ^^^^^^^^^^^^^^^^^^^^^^^^   
                                          这是 VBOX 的试用注册码（我猜的）      
                                  |            
:004E8031 BFBCBD6200              mov edi, 0062BDBC   
:004E8036 8DB5E8FEFFFF            lea esi, dword ptr \[ebp+FFFFFEE8\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E805E(C)   
|   
:004E803C 8A16                    mov dl, byte ptr \[esi\]    ---------   
:004E803E 8A0F                    mov cl, byte ptr \[edi\]            |   
:004E8040 8AC2                    mov al, dl                        |   
:004E8042 3AD1                    cmp dl, cl                        |   
:004E8044 751E                    jne 004E8064                      |   
:004E8046 84C0                    test al, al                        |   
:004E8048 7416                    je 004E8060                  这是典型的比较程序   
:004E804A 8A4E01                  mov cl, byte ptr \[esi+01\]          |   
:004E804D 8A5701                  mov dl, byte ptr \[edi+01\]          |   
:004E8050 8AC1                    mov al, cl                        |   
:004E8052 3ACA                    cmp cl, dl                        |   
:004E8054 750E                    jne 004E8064                      |   
:004E8056 83C602                  add esi, 00000002                  |   
:004E8059 83C702                  add edi, 00000002                  |   
:004E805C 84C0                    test al, al                        |   
:004E805E 75DC                    jne 004E803C            -----------        
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E8048(C)   
|   
:004E8060 33C0                    xor eax, eax   
:004E8062 EB05                    jmp 004E8069   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004E8044(C), :004E8054(C)   
|   
:004E8064 1BC0                    sbb eax, eax   
:004E8066 83D8FF                  sbb eax, FFFFFFFF   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E8062(U)   
|   
:004E8069 33F6                    xor esi, esi   
:004E806B 3BC6                    cmp eax, esi   
:004E806D 7415                    je 004E8084   
:004E806F 8D5594                  lea edx, dword ptr \[ebp\-6C\]   
:004E8072 68F0166000              push 006016F0   
:004E8077 52                      push edx   
  
\* Possible Reference to Dialog: DialogID\_0069, CONTROL\_ID:03EC, "\*\*\*Some String"   
                                  |   
:004E8078 C74594EC030000          mov \[ebp\-6C\], 000003EC   
  
\* Reference To: MSVCRT.\_CxxThrowException, Ord:0041h   
                                  |   
:004E807F E878690000              Call 004EE9FC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E806D(C)   
|   
:004E8084 8D45BC                  lea eax, dword ptr \[ebp\-44\]   
:004E8087 56                      push esi   
:004E8088 50                      push eax   
  
\* Reference To: vboxs420.showMainDialog, Ord:0004h   
                                  |   
:004E8089 E82C580000              Call 004ED8BA    《---CALL VBOXS420.SHOWMAINDIALOG 因为过了这个 CALL 出来了 NAG,所以进去...   
:004E808E 3BC6                    cmp eax, esi   
:004E8090 741E                    je 004E80B0   
:004E8092 83F809                  cmp eax, 00000009   
:004E8095 0F8444090000            je 004E89DF   
:004E809B 8D4DB0                  lea ecx, dword ptr \[ebp\-50\]   
:004E809E 68F0166000              push 006016F0   
:004E80A3 51                      push ecx   
  
进入 CALL 004ED8BA 之后，来到下面：   
在 vboxs420.dll 中有以下语句:   
  
XXXX:00B52FA0            CALL NEAR \[EAX+2C\]   
XXXX:00B52FA3            CMP  EAX,EBX      <----EBX\=03EA   
XXXX:00B52FA5            JNZ  00B52FBA      <----这里一跳，也就完了！   
  
跟进 CALL NEAR \[EAX+2C\],最后他会有以下语句：   
  
XXXX:00B6B10C:8B4124    MOV EAX,\[ECX+24\]  <----什么都好，肯定不会是 03EA！不然还破啥！   
XXXX:00B6B10F:C3        RET   
  
这里有一种改法：   
XXXX:00B6B10C:          PUSH EBX   
XXXX:00B6B10D:          POP  EAX   
XXXX:00B6B10E:          NOP   
  
这种改法我想是可以很彻底的破了 Vbox4.20，在内存中修改试验，的确通过！也不用担心程序会崩溃。   
可惜，只能在内存中修改，而 VBOXS420.DLL 是加了外壳的，我没有功力去脱他，只好另寻出路了...   
  
再来看上面的程序：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E8048(C)   
|   
:004E8060 33C0                    xor eax, eax   
:004E8062 EB05                    jmp 004E8069   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004E8044(C), :004E8054(C)   
|   
:004E8064 1BC0                    sbb eax, eax   
:004E8066 83D8FF                  sbb eax, FFFFFFFF   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E8062(U)   
|   
:004E8069 33F6                    xor esi, esi   
:004E806B 3BC6                    cmp eax, esi   
:004E806D 7415                    je 004E8084                  《《《注意这一跳！我们可以利用   
:004E806F 8D5594                  lea edx, dword ptr \[ebp\-6C\]   
:004E8072 68F0166000              push 006016F0   
:004E8077 52                      push edx   
  
\* Possible Reference to Dialog: DialogID\_0069, CONTROL\_ID:03EC, "\*\*\*Some String"   
                                  |   
:004E8078 C74594EC030000          mov \[ebp\-6C\], 000003EC   
  
\* Reference To: MSVCRT.\_CxxThrowException, Ord:0041h   
                                  |   
:004E807F E878690000              Call 004EE9FC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E806D(C)   
|   
:004E8084 8D45BC                  lea eax, dword ptr \[ebp\-44\]   
:004E8087 56                      push esi   
:004E8088 50                      push eax   
  
\* Reference To: vboxs420.showMainDialog, Ord:0004h   
                                  |   
:004E8089 E82C580000              Call 004ED8BA    《---CALL VBOXS420.SHOWMAINDIALOG   
:004E808E 3BC6                    cmp eax, esi    《《《IF EAX\=ESI\=0,世界都是你的！   
:004E8090 741E                    je 004E80B0        
:004E8092 83F809                  cmp eax, 00000009   
:004E8095 0F8444090000            je 004E89DF   
:004E809B 8D4DB0                  lea ecx, dword ptr \[ebp\-50\]   
:004E809E 68F0166000              push 006016F0   
:004E80A3 51                      push ecx   
  
\* Possible Reference to Dialog: DialogID\_0069, CONTROL\_ID:03EC, "\*\*\*Some String"   
                                  |   
:004E80A4 C745B0EC030000          mov \[ebp\-50\], 000003EC   
  
\* Reference To: MSVCRT.\_CxxThrowException, Ord:0041h   
                                  |   
:004E80AB E84C690000              Call 004EE9FC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E8090(C)   
|   
:004E80B0 8B45BC                  mov eax, dword ptr \[ebp\-44\]  《《《如果过期，EAX=03EB,   
  
\* Possible Reference to Dialog: DialogID\_0072, CONTROL\_ID:03E8, ""   
                                  |   
:004E80B3 3DE8030000              cmp eax, 000003E8   
:004E80B8 7428                    je 004E80E2                  《《《天堂和地狱的分界   
  
\* Possible Reference to Dialog: DialogID\_0134, CONTROL\_ID:03E9, ""   
                                  |   
:004E80BA 3DE9030000              cmp eax, 000003E9   
:004E80BF 7421                    je 004E80E2                  《《《同上   
  
\* Possible Reference to Dialog: DialogID\_006E, CONTROL\_ID:03EA, "\*\*\*Enable Out (ENO)"   
                                  |   
:004E80C1 3DEA030000              cmp eax, 000003EA   
:004E80C6 741A                    je 004E80E2                  《《《同上   
  
\* Possible Reference to Dialog: DialogID\_0134, CONTROL\_ID:03EB, ""   
                                  |   
:004E80C8 3DEB030000              cmp eax, 000003EB   
:004E80CD 7531                    jne 004E8100   
:004E80CF 33C0                    xor eax, eax   
:004E80D1 8B4DF4                  mov ecx, dword ptr \[ebp\-0C\]   
:004E80D4 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:004E80DB 5F                      pop edi   
:004E80DC 5E                      pop esi   
:004E80DD 5B                      pop ebx   
:004E80DE 8BE5                    mov esp, ebp   
:004E80E0 5D                      pop ebp   
:004E80E1 C3                      ret   
  
由以上程序可见，Microwin.exe 只是 CALL VBOX 进行验证，之后返回参数以决定程序的流程，如果 vbox 没有在其它地方放地雷的话，我们可以...   
既然它 CALL 完 VBOX 之后正常的话会去到 004380B0，那我们何不这样（注意堆栈）：   
:004E806D 7415                    je 004E8084                  《《《注意这一跳！我们可以利用   
:004E906D 7441                    je 004380B0   
                                    
:004E80B0 8B45BC                  mov eax, dword ptr \[ebp\-44\]  《《《如果过期，EAX=03EB,                                    
由于没有 CALL VBOX，这里 EAX《》03EB 或其它比较有规律的东东，只是一个乱数！   
可以把下面三个天堂地狱分界线中的一个改为 JMP，你不改的话别指望进天堂！   
  
最后，事实证明是成功的！只改动两个字节，成功的从 VBOX 身边溜了过去!哈哈...   
但是得到一个经验，并不一定要去钻 VBOX 的这个牛角尖，达到目的，不择手段嘛！还有就是西门子公司的程序员太懒了！   
  
当然，以上的改法其实相当不好！成功是靠运气!本来这种方法是很容易造成程序崩溃的（堆栈或资源）！   
最好的改法我认为是改 Vboxs420.dll，这是治本的方法！但我没法改！   
高手们有兴趣不妨看看！   
http://www.ad.siemens.de/simatic/ftp/demosoft/ver3\_1/mw\_ges\_e.exe  4dd466d0e1e0cc123afbe504da2462ca