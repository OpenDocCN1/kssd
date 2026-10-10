

# CuteFTP Pro 2.0 Final

> 原文：[`www.pediy.com/kssd/pediy04/kanxue267.htm`](https://www.pediy.com/kssd/pediy04/kanxue267.htm)

去除 CuteFTP Pro 2.0 Final(2001.11.20)自校验   
  
  
    我们看看 CuteFTP Pro 2.0 Final 的自校验情况。   
    用 exescope 修改 cftppro.exe（1,966,080 字节）资源后保存为 test.exe。   
    运行后得到提示“CuteFTP ... check failed”。   
  
    用 smartcheck 载入 test.exe，程序运行,最后自动退出。   
    搜索 createfilea，看到如下：   
    CreateFileA（"E:\\CUTEFTP PRO\\TEST.EXE"）   
    (test.exe offset 0007F4D5)   
  
    wdasm 反编译 test.exe：   
\* Reference To: KERNEL32.CreateFileA, Ord:0034h   
                                  |   
:0047F4D4 FF1568945200            Call dword ptr \[00529468\]   
......   
......   
\* Reference To: KERNEL32.CreateFileMappingA, Ord:0035h   
                                  |   
:0047F4E3 FF1570945200            Call dword ptr \[00529470\]   
<-- 创建文件 test.exe 内存镜像   
......   
......   
\* Reference To: KERNEL32.MapViewOfFile, Ord:01D6h   
                                  |   
:0047F569 FF1574945200            Call dword ptr \[00529474\]   
<-- 分段读入   
......   
......   
//自校验开始   
:0047F588 8A0439                  mov al, byte ptr \[ecx+edi\]   
:0047F58B 8D54142C                lea edx, dword ptr \[esp+edx+2C\]   
:0047F58F 3002                    xor byte ptr \[edx\], al   
:0047F591 8B542410                mov edx, dword ptr \[esp+10\]   
:0047F595 41                      inc ecx   
:0047F596 42                      inc edx   
:0047F597 3BCE                    cmp ecx, esi   
:0047F599 89542410                mov dword ptr \[esp+10\], edx   
:0047F59D 72D8                    jb 0047F577   
//自校验结束   
<-- 文件 test.exe 自头开始，每 8 个字节为一组，依次进行“异或 XOR”计算，最后得到 8 字节的校验值   
  
\* Reference To: KERNEL32.UnmapViewOfFile, Ord:02B0h   
                                  |   
:0047F5A0 FF156C945200            Call dword ptr \[0052946C\]   
:0047F5A6 8B4C2424                mov ecx, dword ptr \[esp+24\]   
<-- 释放文件内存镜像   
......   
......   
:0047F5E8 8A4C041C                mov cl, byte ptr \[esp+eax+1C\]   
:0047F5EC 84C9                    test cl, cl   
:0047F5EE 7510                    jne 0047F600   
:0047F5F0 8BD0                    mov edx, eax   
:0047F5F2 48                      dec eax   
:0047F5F3 85D2                    test edx, edx   
:0047F5F5 75F1                    jne 0047F5E8   
<-- 对整个文件的校验值再进行一些计算，   
<-- 若最后 8 字节值为 00000000 00000000，则文件没有被修改   
:0047F5F7 B801000000              mov eax, 00000001   
<-- 所以放入记号，eax==1   
:0047F5FC 83C448                  add esp, 00000048   
:0047F5FF C3                      ret   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0047F5EE(C)   
|   
:0047F600 33C0                    xor eax, eax   
<--不然，eax==0   
:0047F602 83C448                  add esp, 00000048   
:0047F605 C3                      ret   
  
  
因此改:0047F5EC 84C9              test cl, cl   
    为          30C9              xor  cl, cl   
去除其自校验。   
  
  
End.   
  
From CCG   
  
2001.11.26  4dd466d0e1e0cc123afbe504da2462ca