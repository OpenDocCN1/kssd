

# jv16 PowerTools RC3 0.0.0.115

> 原文：[`www.pediy.com/kssd/pediy04/kanxue438.htm`](https://www.pediy.com/kssd/pediy04/kanxue438.htm)

jv16 PowerTools RC3 0.0.0.115 在某些机器不能运行的解决方案   
  
    jv16 PowerTools 是 RegCleaner 的下一代版本，对注册表的管理真是一流的好，哈哈，可就是在某些   
机子的 win98 下运行不了，经多次跟踪发现程序出错在 4e655d 处,于是我把 4e6555 的 8bf0 改成 8cd6，程序   
得以运行,其它版本的只要找到这个位置改后一样也可以。   
    
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E63C1(C)   
|   
:004E653C 33C0                    xor eax, eax   
:004E653E 55                      push ebp   
:004E653F 68CE654E00              push 004E65CE   
:004E6544 64FF30                  push dword ptr fs:\[eax\]   
:004E6547 648920                  mov dword ptr fs:\[eax\], esp   
:004E654A 6A06                    push 00000006   
:004E654C 6A01                    push 00000001   
:004E654E 6A02                    push 00000002   
  
\* Reference To: wsock32.socket, Ord:0000h   
                                  |   
:004E6550 E803E2FFFF              Call 004E4758   
:004E6555 8BF0                    mov esi, eax          8bf0 改 8cd6 也就是(mov esi,ss)   
:004E6557 81FEFFFF0000            cmp esi, 0000FFFF      esi 不能为 ffffffff   
:004E655D 7605                    jbe 004E6564                
:004E655F E8F4C8F1FF              call 00402E58   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E655D(C)   
|   
:004E6564 6689B388020100          mov word ptr \[ebx+00010288\], si   
:004E656B 66C745FC0100            mov \[ebp\-04\], 0001   
:004E6571 66C745FE0000            mov \[ebp\-02\], 0000   
:004E6577 6A04                    push 00000004   
:004E6579 8D45FC                  lea eax, dword ptr \[ebp\-04\]   
:004E657C 50                      push eax   
:004E657D 6880000000              push 00000080   
  
\* Possible Reference to String Resource ID=65535: "Disk full"   
                                  |   
:004E6582 68FFFF0000              push 0000FFFF   
:004E6587 0FB7C6                  movzx eax, si   
:004E658A 50                      push eax  4dd466d0e1e0cc123afbe504da2462ca