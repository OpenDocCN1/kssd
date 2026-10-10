

# telock 脱壳总结

> 原文：[`www.pediy.com/kssd/pediy04/kanxue068.htm`](https://www.pediy.com/kssd/pediy04/kanxue068.htm)

telock 脱壳总结   
  
  
telock 用 fi 查不出，所以如何判断是 telock 的壳，凭 ImportRECf 导出 it 的结果吧，乱七八糟的那种差不多就是了。初解 telock 的壳会感觉比较难，不过看了 2 个以后就不觉得有多难了。当然我略过了手动找入口的步骤，寻找程序的入口还是脱壳的关键所在也是难点。   
  
用 ShadowSecurityScaner Ver3.37 举个例子吧，讲的时候会比较方便   
  
1.寻找入口:   
  
(1)delphi:   
    快速寻找入口的方法:执行程序，接着用 winhex 打开程序所在的主内存，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，所以 delphi 程序的 oep 最好找了。   
  
脱壳详情请参考我最近写的(delphi 程序)   
  
脱 Flashfxp 1.4 的壳   
  
(2)vb：   
    快速寻找入口的方法:执行程序，接着用 winhex 打开程序所在的主内存，选择搜索文本，填程序的名字,执行搜索，搜到后,向前找到离名字最近的机器码为 68(后面就不一定了，一般很短，不超过 20h 字节)开头的地方就是程序的 oep，所以 vb 程序的 oep 也比较好找。   
  
(3)vc:   
    无任何规律，慢慢跟吧:(。手动脱壳方法最好熟练掌握，对付任何程序都应该有效吧:)，熟悉以后就没问题了。(难~~~)   
  
(4)其他:   
    没试过，不知道了。   
  
  
    ShadowSecurityScaner 是用 delphi 写的，hoho 运气真好:)。sss335 以前某版肯定是用 telock 加的壳，不过在 51 下的 337 版没有加壳，所以我用 telock0.90 给他加一个壳。用快速寻找 delphi 程序入口的方法：执行 sss，接着用 winhex 编辑 sss 的主内存，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，对应的地址是 59b9e4。   
  
  
2.脱壳：   
  
    telock 的壳在 dump 之前需要用 winhex 内存编辑功能把 400106-400107 填回原来的 section 数(telock 会改写为 ffff，使脱壳程序无法 dump)。然后从入口处 dump，如果不是从入口 dump 的，一般不能用。再有用 prodump dump 的程序，需要手动把 oep 改成正确的。用天意,trw 或 peditor dump 的程序稍大。其中天意在我的 winme 系统下执行 suspend 死机，所以极少用，peditor dump 出的程序超大。trw 只有娃娃改过的 1.03 版才能直接 g 到入口处，其余的版会非法操作的，而 s-ice 不会出任何问题的，cool。   
  
  
    ok，回到 sss。下面要在入口处脱壳，老规矩打开 SuperBPM，点 erase，用 trw1.03 娃娃修改过的版载入 sss，下 g 59b9e4，下 suspend，用 winhex 内存编辑功能把 400106-400107 填回原来的值 0800，再用 prodump 选 dump(full)就成功啦。   
  
  
3.重建 import table：   
  
以 sss 为例，执行 sss，然后用 ImportRECf 导出 it，是这个样子的：   
  
Target: D:\\TOOLS\\SHADOWSECURITYSCANER\\SSS.EXE   
OEP: 0019B9E4    IATRVA: 001AB1A0    IATSize: 000008A4   
  
FThunk: 001AB1A4    NbFunc: 00000036   
0    001AB1A4    ?    0000    009D0000   
0    001AB1A8    ?    0000    009D000A   
0    001AB1AC    ?    0000    009D0014   
0    001AB1B0    ?    0000    009D001E   
  
从 telock 重建 It 部分开始吧，用加了 superbpm 和 icedump 的 s-ice 重新载入 sss，下 bpm 9d0000，下 g，停在这里：   
  
0187:006ACE09 FF0424            INC    DWord Ptr \[ESP\]   
0187:006ACE0C 0483              ADD    AL,83   
0187:006ACE0E 2424              AND    AL,24   
0187:006ACE10 03833C240074      ADD    EAX,\[EBX+7400243C\]   
0187:006ACE16 36833C2401        CMP    DWord Ptr SS:\[ESP\],00000001   
0187:006ACE1B 744D              JZ      006ACE6A   
0187:006ACE1D 833C2402          CMP    DWord Ptr \[ESP\],00000002   
0187:006ACE21 7422              JZ      006ACE45   
0187:006ACE23 C144240C10        ROL    DWord Ptr \[ESP+0C\],10   
0187:006ACE28 668B44240C        MOV    AX,\[ESP+0C\]   
0187:006ACE2D 66AB              STOSW   
0187:006ACE2F 8BC3              MOV    EAX,EBX   
0187:006ACE31 C1E803            SHR    EAX,03   
0187:006ACE34 83E003            AND    EAX,00000003   
0187:006ACE37 8A440404          MOV    AL,\[ESP+1\*EAX+04\]   
0187:006ACE3B AA                STOSB   
0187:006ACE3C B0C3              MOV    AL,C3   
0187:006ACE3E AA                STOSB   
0187:006ACE3F 66B8FF35          MOV    AX,35FF   
0187:006ACE43 EB2D              JMP    006ACE72   
0187:006ACE45 8B442408          MOV    EAX,\[ESP+08\]   
0187:006ACE49 AB                STOSD   
0187:006ACE4A 4F                DEC    EDI   
0187:006ACE4B EBEF              JMP    006ACE3C   
0187:006ACE4D 668B44240C        MOV    AX,\[ESP+0C\]   
0187:006ACE52 66AB              STOSW   
0187:006ACE54 B0C3              MOV    AL,C3   
0187:006ACE56 AA                STOSB   
0187:006ACE57 8BC3              MOV    EAX,EBX   
0187:006ACE59 C1E803            SHR    EAX,03   
0187:006ACE5C 83E003            AND    EAX,00000003   
0187:006ACE5F 8A440404          MOV    AL,\[ESP+1\*EAX+04\]   
0187:006ACE63 AA                STOSB   
0187:006ACE64 66B8FF35          MOV    AX,35FF   
0187:006ACE68 EB08              JMP    006ACE72   
0187:006ACE6A 8B442410          MOV    EAX,\[ESP+10\]   
0187:006ACE6E AB                STOSD   
0187:006ACE6F 4F                DEC    EDI   
0187:006ACE70 EBCA              JMP    006ACE3C   
0187:006ACE72 E28C              LOOP    006ACE00   
0187:006ACE74 66AB              STOSW   
0187:006ACE76 83C414            ADD    ESP,00000014   
0187:006ACE79 61                POPAD   
0187:006ACE7A 89BD9AB04000      MOV    \[EBP+0040B09A\],EDI  //MOV    \[EBP+0040B09A\],EBX    
              899D9AB04000   
0187:006ACE80 8BBD9AB04000      MOV    EDI,\[EBP+0040B09A\]  //MOV     EDI,\[EBP+0040B0A2\]   
              8BBDA2B04000   
0187:006ACE86 8B85A2B04000      MOV    EAX,\[EBP+0040B0A2\]   
0187:006ACE8C 03859AAF4000      ADD    EAX,\[EBP+0040AF9A\]   
0187:006ACE92 8B8D9EB04000      MOV    ECX,\[EBP+0040B09E\]   
0187:006ACE98 8908              MOV    \[EAX\],ECX   
0187:006ACE9A 83859EB040000A    ADD    DWord Ptr \[EBP+0040B09E\],0000000A   
0187:006ACEA1 EB08              JMP    006ACEAB   
0187:006ACEA3 838D9AB04000FF    OR      DWord Ptr \[EBP+0040B09A\],FFFFFFFF   
0187:006ACEAA 61                POPAD   
0187:006ACEAB 03BD9AAF4000      ADD    EDI,\[EBP+0040AF9A\]   
0187:006ACEB1 85DB              TEST    EBX,EBX   
0187:006ACEB3 0F84B4000000      JZ      006ACF6D   
0187:006ACEB9 F7C300000080      TEST    EBX,80000000   
0187:006ACEBF 6A00              PUSH    00000000   
0187:006ACEC1 7506              JNZ    006ACEC9   
0187:006ACEC3 8D5C1302          LEA    EBX,\[EBX+1\*EDX+02\]   
0187:006ACEC7 EB3C              JMP    006ACF05   
0187:006ACEC9 FF0424            INC    DWord Ptr \[ESP\]   
0187:006ACECC 8B8596AF4000      MOV    EAX,\[EBP+0040AF96\]   
0187:006ACED2 3B858AB04000      CMP    EAX,\[EBP+0040B08A\]   
0187:006ACED8 752B              JNZ    006ACF05   
0187:006ACEDA 81E3FFFFFF7F      AND    EBX,7FFFFFFF   
0187:006ACEE0 8BD3              MOV    EDX,EBX   
0187:006ACEE2 8D1495FCFFFFFF    LEA    EDX,\[FFFFFFFC+4\*EDX\]   
0187:006ACEE9 8B9D96AF4000      MOV    EBX,\[EBP+0040AF96\]   
0187:006ACEEF 8B433C            MOV    EAX,\[EBX+3C\]   
0187:006ACEF2 8B441878          MOV    EAX,\[EAX+1\*EBX+78\]   
0187:006ACEF6 035C181C          ADD    EBX,\[EAX+1\*EBX+1C\]   
0187:006ACEFA 8B041A            MOV    EAX,\[EDX+1\*EBX\]   
0187:006ACEFD 038596AF4000      ADD    EAX,\[EBP+0040AF96\]   
0187:006ACF03 EB13              JMP    006ACF18   
0187:006ACF05 81E3FFFFFF7F      AND    EBX,7FFFFFFF   
0187:006ACF0B 53                PUSH    EBX   
0187:006ACF0C FFB596AF4000      PUSH    DWord Ptr \[EBP+0040AF96\]   
0187:006ACF12 FF9504AF4000      CALL    Near \[\`KERNEL32!GetProcAddress\`\]  //取函数名   
0187:006ACF18 40                INC    EAX   
0187:006ACF19 48                DEC    EAX   
0187:006ACF1A 7532              JNZ    006ACF4E   
0187:006ACF1C 58                POP    EAX   
0187:006ACF1D F9                STC      
0187:006ACF1E 0F829BFDFFFF      JB      006ACCBF   
0187:006ACF24 47                INC    EDI   
0187:006ACF25 44                INC    ESP   
0187:006ACF26 49                DEC    ECX   
0187:006ACF27 3332              XOR    ESI,\[EDX\]   
0187:006ACF29 2E44              INC    ESP   
0187:006ACF2B 4C                DEC    ESP   
0187:006ACF2C 4C                DEC    ESP   
0187:006ACF2D 55                PUSH    EBP   
0187:006ACF2E 53                PUSH    EBX   
0187:006ACF2F 45                INC    EBP   
0187:006ACF30 52                PUSH    EDX   
0187:006ACF31 3332              XOR    ESI,\[EDX\]   
0187:006ACF33 2E44              INC    ESP   
0187:006ACF35 4C                DEC    ESP   
0187:006ACF36 4C                DEC    ESP   
0187:006ACF37 53                PUSH    EBX   
0187:006ACF38 48                DEC    EAX   
0187:006ACF39 45                INC    EBP   
0187:006ACF3A 4C                DEC    ESP   
0187:006ACF3B 4C                DEC    ESP   
0187:006ACF3C 3332              XOR    ESI,\[EDX\]   
0187:006ACF3E 2E44              INC    ESP   
0187:006ACF40 4C                DEC    ESP   
0187:006ACF41 4C                DEC    ESP   
0187:006ACF42 4B                DEC    EBX   
0187:006ACF43 45                INC    EBP   
0187:006ACF44 52                PUSH    EDX   
0187:006ACF45 4E                DEC    ESI   
0187:006ACF46 45                INC    EBP   
0187:006ACF47 4C                DEC    ESP   
0187:006ACF48 3332              XOR    ESI,\[EDX\]   
0187:006ACF4A 2E44              INC    ESP   
0187:006ACF4C 4C                DEC    ESP   
0187:006ACF4D 4C                DEC    ESP   
0187:006ACF4E 8907              MOV    \[EDI\],EAX  //将函数名放回正确的位置   
0187:006ACF50 58                POP    EAX   
0187:006ACF51 48                DEC    EAX   
0187:006ACF52 740D              JZ      006ACF61   
0187:006ACF54 40                INC    EAX   
0187:006ACF55 F8                CLC      
0187:006ACF56 668943FE          MOV    \[EBX\-02\],AX   
0187:006ACF5A 8803              MOV    \[EBX\],AL   
0187:006ACF5C 43                INC    EBX   
0187:006ACF5D 3803              CMP    \[EBX\],AL   
0187:006ACF5F 75F9              JNZ    006ACF5A   
0187:006ACF61 83859AAF400004    ADD    DWord Ptr \[EBP+0040AF9A\],00000004   
0187:006ACF68 E9D4FDFFFF        JMP    006ACD41   
0187:006ACF6D 83C614            ADD    ESI,00000014   
0187:006ACF70 8B95AEAF4000      MOV    EDX,\[EBP+0040AFAE\]   
0187:006ACF76 E9B1FCFFFF        JMP    006ACC2C    //循环   
0187:006ACF7B 61                POPAD   
0187:006ACF7C C3                RET      
  
  
按我的方法把 006ACE7A 和 006ACE80 改成这个样子，就可以用 ImportRECf 了。   
0187:006ACE7A 89BD9AB04000      MOV    \[EBP+0040B09A\],EDI  //MOV    \[EBP+0040B09A\],EBX   
              899D9AB04000   
0187:006ACE80 8BBD9AB04000      MOV    EDI,\[EBP+0040B09A\]  //MOV    EDI,\[EBP+0040B0A2\]    
              8BBDA2B04000   
  
这样做完所有的函数都可以认出，但还有一个问题，就是该分段的地址还是被 telock 写入了东西，这样 ImportRECf 就无法识别，还好按我的改法认不出来的地方都是该分段的地方。   
  
类似这样   
1    001AB24C    KERNEL32.dll    0181    GetFileType   
1    001AB250    KERNEL32.dll    00BA    CreateFileA   
1    001AB254    KERNEL32.dll    00A1    CloseHandle   
0    001AB258    ?    0000    009D01C2   
1    001AB25C    USER32.dll    0115    GetKeyboardType   
1    001AB260    USER32.dll    01AB    LoadStringA   
1    001AB264    USER32.dll    01BA    MessageBoxA   
1    001AB268    USER32.dll    0027    CharNextA   
0    001AB26C    ?    0000    009E0028   
1    001AB270    ADVAPI32.dll    00F8    RegQueryValueExA   
1    001AB274    ADVAPI32.dll    00F0    RegOpenKeyExA   
1    001AB278    ADVAPI32.dll    00D9    RegCloseKey   
  
只需手工把它改成如下就可以了，比原来的方法要快些:)，具体的动手试试吧。   
  
1    001AB24C    KERNEL32.dll    0181    GetFileType   
1    001AB250    KERNEL32.dll    00BA    CreateFileA   
1    001AB254    KERNEL32.dll    00A1    CloseHandle   
  
FThunk: 001AB25C    NbFunc: 00000004   
1    001AB25C    USER32.dll    0115    GetKeyboardType   
1    001AB260    USER32.dll    01AB    LoadStringA   
1    001AB264    USER32.dll    01BA    MessageBoxA   
1    001AB268    USER32.dll    0027    CharNextA   
  
FThunk: 001AB270    NbFunc: 00000003   
1    001AB270    ADVAPI32.dll    00F8    RegQueryValueExA   
1    001AB274    ADVAPI32.dll    00F0    RegOpenKeyExA   
1    001AB278    ADVAPI32.dll    00D9    RegCloseKey   
  
做完这些后用 ImportRECf 修复后的程序可以正常运行了。   
  
  
完活，哪位大哥有其他好方法可用的话，千万记得告诉我呀，谢啦。   
另外可以参考 hying 和 fpc 大哥关于 telock 的脱文。   
  
感谢你能够坚持下来 hoho:)   
telock 的壳除了手动找入口比较难外，其它的还是比较简单的。   
  
  
关于 sss 的注册：   
  
这是我对付以前用 telock 加过壳的那个 sss 的 patch，sss 任意版都只需改 2 个字节变成无限制版   
  
#Process Patcher Configuration File   
Version=3.93   
  
DisplayName=My Test Program   
Filename=sss.exe   
Filesize=1043968   
Arguments=/quiet   
Address=0x58ec5d:0x75:0xeb   
  
#End of Configuration File   
  
  
再有用这个 key 强制注册后，就是注册版。   
  
UserName=SSSRussian   
DigitalSignature=Vu3xB7ar69mR3Vt6hru4piTtDpPOVALxT+SXLg5nALwNVGDHbzWyEA4UAxQktR2arkd5klsvjHCTQVvO0zz9RU   
  
  
  
2001.9.27   
zombieys\[CCG\]   
  
———————————————————————————————>   
                .-"      "-.      unpacked by zombieys\[CCG\] >   
                /　      　　\\      qq:1789655                >   
              |      ★　　  |    http://zombieys.yeah.net  >   
              |,  .-.  .-.  ,|    http://zombieys.126.com  >   
                |)(\_\_/  \\\_\_)(|      zombieys.cn.hongnet.com  >   
                |/    /\\    \\|                                >   
      (@\_@)    (\_    ^^    \_)      Thanks for your supports  >   
    \_  )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_\_\_                          >   
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_China Crack Group\_zombieys\_\_\_> 

提示：所有加密过的代码或数据段都会在内存里解密，并完整的存在一段   
时间，只要用 SOFT-ICE + ICEDUMP 就足够了！   
只要跳过 VirtualProtectEx 这个 API 就可以正确的 DUMP 了  4dd466d0e1e0cc123afbe504da2462ca