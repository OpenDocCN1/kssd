

# 监理通 2000

> 原文：[`www.pediy.com/kssd/pediy03/forum919.htm`](https://www.pediy.com/kssd/pediy03/forum919.htm)

Yinci 那个 RBDLL 什么都看得到，   
不过我没有原狗没试过，我想它的程序也是有价值的   
  
Sx32w.dll  (这是 5.051 版本的，6.xx 版本大同小异）   
  
没有找到狗后都转到这里出口：(改为找狗成功）   
改 00406C60 子程序   
:00406C60 668B442404              mov ax, word ptr \[esp+04\]   
:00406C65 663D0301                cmp ax, 0103   
:00406C69 7507 --->33C0          jne 00406C72   
:00406C6B 66B81201-->C2040090    mov ax, 0112   
:00406C6F C20400-->909090        ret 0004   
  
找 sx32w.dll QUERY 位置  0x00006E10   
改 Query   
:00407A10 53                      push ebx   
:00407A11 56                      push esi   
:00407A12 57                      push edi   
:00407A13 8B442410                mov eax, dword ptr \[esp+10\]   
:00407A17 0BC0                    or eax, eax   
:00407A19 750A                    jne 00407A25   
:00407A1B 66B80200                mov ax, 0002   
:00407A1F 5F                      pop edi   
:00407A20 5E                      pop esi   
:00407A21 5B                      pop ebx   
:00407A22 C21800                  ret 0018   
  
:00407A10 52                      push edx   
:00407A11 8B542414                mov edx, dword ptr \[esp+14\]   
:00407A15 B8xxxxxxxx              mov eax, xxxxxxxx   
:00407A1A 8902                    mov dword ptr \[edx\], eax   
:00407A1C 33C0                    xor eax, eax   
:00407A1E 33C0                    xor eax, eax   
:00407A20 5A                      pop edx   
:00407A21 C21800                  ret 0018   
:00407A24 90                      nop   
  
  xxxxxxxx 是某软件的关键数据,不同版本不一样   
  不过改一下上段代码，很容易通用的 

我搞破解不到一月，谢谢 Toye 兄   
sx32w.dll(V5.0.51)   
+++++++++++++++++++ EXPORTED FUNCTIONS ++++++++++++++++++   
Number of Exported Functions = 0017 (decimal)   
Addr:00407280 Ord:  1 (0001h) Name: RNBOsproInitialize   
Addr:00407620 Ord:  2 (0002h) Name: RNBOsproRead   
Addr:004076D0 Ord:  3 (0003h) Name: RNBOsproExtendedRead   
Addr:00407790 Ord:  4 (0004h) Name: RNBOsproWrite   
Addr:00407970 Ord:  5 (0005h) Name: RNBOsproActivate   
Addr:00407830 Ord:  6 (0006h) Name: RNBOsproOverwrite   
Addr:00407350 Ord:  7 (0007h) Name: RNBOsproGetVersion   
Addr:00407B10 Ord:  8 (0008h) Name: RNBOsproGetFullStatus   
Addr:00407220 Ord:  9 (0009h) Name: RNBOsproFormatPacket   
Addr:00406DC0 Ord:  10 (000Ah) Name: RNBOsproCfgLibParams   
Addr:00407450 Ord:  11 (000Bh) Name: RNBOsproFindFirstUnit   
Addr:00407540 Ord:  12 (000Ch) Name: RNBOsproFindNextUnit   
Addr:004078E0 Ord:  13 (000Dh) Name: RNBOsproDecrement   
Addr:00407A10 Ord:  14 (000Eh) Name: RNBOsproQuery   
Addr:00406C80 Ord:  15 (000Fh) Name: RNBOsproGetUnitInfo   
Addr:00406D10 Ord:  16 (0010h) Name: RNBOsproSetUnitInfo   
Addr:00407B40 Ord:  20 (0014h) Name: DllEntryPoint   
  
Exported fn(): RNBOsproQuery - Ord:000Eh   
:00407A10 9C                      pushfd   
:00407A11 52                      push edx   
:00407A12 8B542418                mov edx, dword ptr \[esp+18\]   
:00407A16 837C24102A              cmp dword ptr \[esp+10\], 0000002A   
:00407A1B 7515                    jne 00407A32   
:00407A1D 837C241030              cmp dword ptr \[esp+10\], 00000030   
:00407A22 7507                    jne 00407A2B   
:00407A24 xxxxxxxx(单机版          mov eax, xxxxxxxx   
              另一个可用数据）   
:00407A29 EB0C                    jmp 00407A37   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00407A22(C)   
|   
:00407A2B xxxxxxxx 单机版数据          mov eax, xxxxxxxx   
:00407A30 EB05                    jmp 00407A37   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00407A1B(C)   
|   
:00407A32 xxxxxxxx              mov eax, xxxxxxxx(网络版数据）   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00407A29(U), :00407A30(U)   
|   
:00407A37 8902                    mov dword ptr \[edx\], eax   
:00407A39 33C0                    xor eax, eax   
:00407A3B 33C0                    xor eax, eax   
:00407A3D 5A                      pop edx   
:00407A3E 9D                      popfd   
:00407A3F C21800                  ret 0018  4dd466d0e1e0cc123afbe504da2462ca