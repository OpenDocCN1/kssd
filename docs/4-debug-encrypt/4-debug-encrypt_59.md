

# AdSubtract PRO Version 2.04

> 原文：[`www.pediy.com/kssd/pediy03/forum554.htm`](https://www.pediy.com/kssd/pediy03/forum554.htm)

谁能破解此软件：一个拦截网页广告的工具，此软件没有试用期，不注册就不让使用。本人水平太低，只能解除启动后的 NAG 框（在 60031004 处设断，将跳转改为 NOP）。此软件的下载地址为：http://www.adsubtract.com/bin/20082404/ad-pro.exe　请高手破解后指出此软件的关键断点，先谢了。 

  

破解 AdSubtract PRO Version 2.04   
其实主程序文件是 ctlpanel.dll 这个动态库文件.   
  
这是一个非常好的去广告窗程序！   
016F:6000A302  CMP      EDI,0403   
016F:6000A308  JZ      NEAR 6000A46B            不能跳跳了你就要数入注册码拉！   
016F:6000A30E  CMP      EDI,074C   
016F:6000A314  JNG      NEAR 6000A559   
016F:6000A31A  CMP      EDI,074F   
016F:6000A320  JG      NEAR 6000A559   
016F:6000A326  PUSH    DWORD 60052D50   
016F:6000A32B  LEA      ECX,\[ESP+2C\]   
016F:6000A32F  CALL    60010370   
016F:6000A334  LEA      EDX,\[ESP+10\]   
016F:6000A338  XOR      EBP,EBP   
016F:6000A33A  PUSH    EDX   
016F:6000A33B  PUSH    DWORD 60052D40   
016F:6000A340  LEA      ECX,\[ESP+30\]   
016F:6000A344  MOV      \[ESP+01CC\],EBP   
016F:6000A34B  CALL    600112D0   
016F:6000A350  CMP      DWORD \[ESP+10\],BYTE +04   
016F:6000A355  JNL      6000A370                  一定要跳转不然功能有限制！   
016F:6000A357  LEA      ECX,\[ESP+28\]   
016F:6000A35B  MOV      DWORD \[ESP+01C4\],FFFFFFFF   
016F:6000A366  CALL    60010440   
016F:6000A36B  JMP      6000A559   
016F:6000A370  OR      EBX,BYTE -01   
016F:6000A373  LEA      ECX,\[ESP+28\]   
016F:6000A377  MOV      \[ESP+01C4\],EBX   
016F:6000A37E  CALL    60010440   
016F:6000A383  MOV      ECX,\[ESI+3C\]   
016F:6000A386  CMP      ECX,EBP   
016F:6000A388  JZ      6000A3F6   
016F:6000A38A  CMP      WORD \[ESP+01CC\],03F0   
016F:6000A394  JNZ      6000A3C2   
016F:6000A396  PUSH    BYTE +01   
016F:6000A398  CALL    6002FD1B   
016F:6000A39D  MOV      EAX,\[ESI+3C\]   
016F:6000A3A0  MOV      ECX,\[EAX+1C\]   
016F:6000A3A3  PUSH    ECX   
016F:6000A3A4  CALL    \`USER32!SetForegroundWindow\`   
016F:6000A3AA  MOV      EAX,\[ESI+3C\]   
016F:6000A3AD  MOV      EDX,\[EAX+1C\]   
016F:6000A3B0  PUSH    EDX   
016F:6000A3B1  CALL    \`USER32!SetActiveWindow\`   
016F:6000A3B7  PUSH    EAX   
016F:6000A3B8  CALL    6002B848   
016F:6000A3BD  JMP      6000A559   
016F:6000A3C2  CALL    6000E220   
016F:6000A3C7  TEST    EAX,EAX   
016F:6000A3C9  JZ      NEAR 6000A570   
016F:6000A3CF  MOV      EAX,\[ESI+3C\]   
016F:6000A3D2  LEA      ECX,\[ESP+18\]   
016F:6000A3D6  PUSH    ECX   
016F:6000A3D7  MOV      EDX,\[EAX+1C\]   
016F:6000A3DA  PUSH    EDX   
016F:6000A3DB  CALL    \`USER32!GetWindowRect\`   
016F:6000A3E1  MOV      ECX,\[ESI+3C\]   
016F:6000A3E4  PUSH    BYTE +01   
016F:6000A3E6  MOV      EAX,\[ECX\]   
016F:6000A3E8  CALL    NEAR \[EAX+74\]   
016F:6000A3EB  MOV      ECX,\[ESI+3C\]   
016F:6000A3EE  MOV      EDX,\[ECX\]   
016F:6000A3F0  CALL    NEAR \[EDX+58\]   
016F:6000A3F3  MOV      \[ESI+3C\],EBP   
016F:6000A3F6  CMP      \[ESI+3C\],EBP   
016F:6000A3F9  JNZ      NEAR 6000A559   
016F:6000A3FF  PUSH    EDI   
016F:6000A400  PUSH    EBP   
016F:6000A401  PUSH    DWORD 60052214   
016F:6000A406  LEA      ECX,\[ESP+34\]   
016F:6000A40A  CALL    6000DE00   
016F:6000A40F  MOV      EAX,\[ESP+68\]   
016F:6000A413  MOV      DWORD \[ESP+01C4\],01   
016F:6000A41E  OR      AL,02   
016F:6000A420  MOV      \[ESP+68\],EAX   
016F:6000A424  CALL    6003C4E3   
016F:6000A429  MOV      EAX,\[EAX+08\]   
016F:6000A42C  PUSH    EBP   
016F:6000A42D  PUSH    BYTE +10   
016F:6000A42F  PUSH    BYTE +10   
016F:6000A431  PUSH    BYTE +01   
016F:6000A433  PUSH    DWORD 8E   
016F:6000A438  PUSH    EAX   
016F:6000A439  CALL    \`USER32!LoadImageA\`   
016F:6000A43F  MOV      \[ESP+74\],EAX   
016F:6000A443  LEA      EAX,\[ESP+28\]   
016F:6000A447  LEA      ECX,\[ESP+28\]   
016F:6000A44B  MOV      \[ESI+3C\],EAX   
016F:6000A44E  CALL    600387D8   
016F:6000A453  LEA      ECX,\[ESP+28\]   
016F:6000A457  MOV      \[ESI+3C\],EBP   
016F:6000A45A  MOV      \[ESP+01C4\],EBX   
016F:6000A461  CALL    6000E120

注册号：ab   
  
a 和 b 是 32 进制数。   
  
设 magic\[\]={0x5733a0c3, 0x155cf8be, 0xffc55d8d, 0x11cf424, 0x7117d5b1, 0x4baf5541, 0x83179f31, 0x20f3f593};   
  
md5init();   
md5padding(magic);   
md5padding(a);   
md5padding(magic);   
md5finish();   
s=md5\[0~4\];   
  
如果 s==b，则认为注册成功。 

象下面这样的即使用了不可逆算法又有什么难度呢？还不是轻而易举地写出注册机来？   
  
if ( MD5(UserName) == SerialNumber )   
{   
      Registered = TRUE;   
}   
else   
{   
      Registered = FALSE;   
}   
  
而象下面的即使软件作者自己是最天才伟大的数学家也无法弄出注册机来，这样有什么意义呢？   
  
if ( MD5(SerialNumber) == UserName )   
{   
      Registered = TRUE;   
}   
else   
{   
      Registered = FALSE;   
}   
  
若用 MD5 来对指令机器码进行运算，软件作者自己也无法搞定。  4dd466d0e1e0cc123afbe504da2462ca