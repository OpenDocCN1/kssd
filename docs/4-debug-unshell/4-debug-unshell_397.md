

# explor2000

> 原文：[`www.pediy.com/kssd/pediy03/forum889.htm`](https://www.pediy.com/kssd/pediy03/forum889.htm)

explor2000 的破解   
  
ASProtect1.2 以后版本加壳，ep=5f4690。   
用 bo2k 找不到入口。手动跟踪方法参见（1），脱壳方法参见（2）。   
  
explor2000 的破解方法如下。   
  
先运行 superbpm，再运行 trw，下 bpx 5f4690，g。   
trw 弹出后，下 pedump c:\\explor2k.exe 脱壳。   
用 w32dasm 对脱壳后的 explor2k.exe 进行反汇编。在 Refs/String Data Refrences 中找'trialversion'，找到后双击，看到下面这段(用 find text 找也可)   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:005D1D0E(U), :005D1D1A(C)   
|   
:005D1D2D 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:005D1D30 8B1520D05F00            mov edx, dword ptr \[005FD020\]   
:005D1D36 E87523E3FF              call 004040B0                ;这是一个关键的 call   
:005D1D3B 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:005D1D3E E83524E3FF              call 00404178                ;这里不是关键，进去就知道   
:005D1D43 85C0                    test eax, eax   
:005D1D45 0F8FD2000000            jg 005D1E1D                ；跳就对了   
:005D1D4B 833D28D05F0000          cmp dword ptr \[005FD028\], 00000000    ;\[5fd028\]中放的是剩余天数   
:005D1D52 7E47                    jle 005D1D9B   
:005D1D54 833D28D05F001E          cmp dword ptr \[005FD028\], 0000001E    ；试用期为 30 天   
:005D1D5B 7E07                    jle 005D1D64   
:005D1D5D B805000000              mov eax, 00000005   
:005D1D62 EB0B                    jmp 005D1D6F   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005D1D5B(C)   
|   
:005D1D64 B823000000              mov eax, 00000023   
:005D1D69 2B0528D05F00            sub eax, dword ptr \[005FD028\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005D1D62(U)   
|   
:005D1D6F 83F80A                  cmp eax, 0000000A   
:005D1D72 7E05                    jle 005D1D79   
:005D1D74 B80A000000              mov eax, 0000000A   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005D1D72(C)   
|   
:005D1D79 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:005D1D7C 52                      push edx   
:005D1D7D 68641F5D00              push 005D1F64   
:005D1D82 50                      push eax   
:005D1D83 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:005D1D86 8B90FC040000            mov edx, dword ptr \[eax+000004FC\]   
:005D1D8C B103                    mov cl, 03   
:005D1D8E 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:005D1D91 E812020000              call 005D1FA8   
:005D1D96 E982000000              jmp 005D1E1D   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005D1D52(C)   
|   
:005D1D9B E8140EE3FF              call 00402BB4   
:005D1DA0 B80A000000              mov eax, 0000000A   
:005D1DA5 E8CE10E3FF              call 00402E78   
:005D1DAA 83F802                  cmp eax, 00000002   
:005D1DAD 7550                    jne 005D1DFF   
:005D1DAF 6A30                    push 00000030   
  
\* Possible StringData Ref from Code Obj ->"EXPLOR2000"   
                                  |   
:005D1DB1 68E01E5D00              push 005D1EE0   
:005D1DB6 8D4DF4                  lea ecx, dword ptr \[ebp\-0C\]   
:005D1DB9 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:005D1DBC 8B8054050000            mov eax, dword ptr \[eax+00000554\]   
  
\* Possible StringData Ref from Code Obj ->"TrialVersion"   
                                  |   
:005D1DC2 BAF41E5D00              mov edx, 005D1EF4            ；显示试用字样   
:005D1DC7 E838B0F5FF              call 0052CE04   
  
下 bpx 5d1d36，trw 弹出后按 F8 进入   
0167:004040B0 31C9            XOR      ECX,ECX   
0167:004040B2 85D2            TEST    EDX,EDX   
0167:004040B4 7421            JZ      004040D7   
0167:004040B6 52              PUSH    EDX   
0167:004040B7 3A0A            CMP      CL,\[EDX\]    ；\[edx\]=\[1333405\]=0。把它改为 1，bd，g。   
0167:004040B9 7417            JZ      004040D2    ；嘿嘿，居然成为注册版了。不过用户名是乱码。   
0167:004040BB 3A4A01          CMP      CL,\[EDX+01\]   
0167:004040BE 7411            JZ      004040D1   
0167:004040C0 3A4A02          CMP      CL,\[EDX+02\]   
0167:004040C3 740B            JZ      004040D0   
0167:004040C5 3A4A03          CMP      CL,\[EDX+03\]   
0167:004040C8 7405            JZ      004040CF   
0167:004040CA 83C204          ADD      EDX,BYTE +04   
0167:004040CD EBE8            JMP      SHORT 004040B7   
0167:004040CF 42              INC      EDX   
0167:004040D0 42              INC      EDX   
0167:004040D1 42              INC      EDX   
0167:004040D2 89D1            MOV      ECX,EDX   
0167:004040D4 5A              POP      EDX   
0167:004040D5 29D1            SUB      ECX,EDX   
0167:004040D7 E904FFFFFF      JMP      00403FE0   
0167:004040DC C3              RET      
  
因为 4040b0 经常被调用，所以不能在这里修改。一种偷懒的方法是使用内存补丁 ppatcher3.93。   
把以下内容存为 ppatcher.ppc，连同 ppatcher.exe 考到 explor2000 安装目录下，运行 ppatcher.exe 即可，连脱壳也不用了。   
\------------------------------------------------   
#Process Patcher Configuration File   
Version=3.93   
WaitForWindowName=Explor2000   
  
PatchAuthor=kingtall   
  
DisplayName=Explor2000   
Filename=Explor2000.exe   
Filesize=933376   
Address=0x1333405:0x00:0x01   
#End of Configuration File   
\-----------------------------------------------   
  
程序中一定有直接对\[1333405\]进行修改的地方，懒得再找了。哪位大哥找到了，别忘说一声。   
  
（1）   
http://001.com.cn/forum/toye/14434.html   
标 题:如何跟踪 ASProtect 外壳加密过的程序？ (7 千字) 

*   作 者：ljtt  
*   时 间：2001-4-13 21:30:27   
      
    （2）   
    http://001.com.cn/forum/toye/15931.html   
    标 题:PicturesToExe3.51 的脱壳 (2 千字) *   作 者：hying  
    *   时 间：2001-4-22 19:18:35

asprotect 加壳程序又有新的变化   
  
我这里不打算讲如何脱壳和 IAT 的重建，因为 ljtt 兄的文章已经是很经典的了，也不打算讲如何破解，因为 kingtall 兄也已经说的很清楚了，我只想就我跟踪 sfx2000 这个程序看到的 asprotect 这个“伟大”的程序的新变化。   
  
  在 sfx2000.exe 里面，我们看到了 asprotect 较之以前的变化的地方   
1.增加了 OEP 的定位的难度   
  原来 asprotect 在处理完 IAT 以后出来，按 3-4 次 F8，然后按 2-3 次 F12，就快接近 OEP 了，一般是连着的 2-3 个 ret 命令，就到 OEP 了。   
  在 sfx2000 里面，不是这样的，在处理完 IAT 以后出来，对于 OEP 的定位又增加了极高的难度，在这里的程序是分段恢复的，每一段的恢复都用了很大的循环次数来完成的，给我的感觉是程序在这里并没有做什么，都用来恢复后面的程序了，这里的程序用了相当多的花指令，一部分是 jMP，还有一部分跳转是用 CALL 指令来完成的，所以指令必须找准循环的结束点，否则一不小心，从一个 CALL 就进到主程序里面了，又的从头开始。以前用冲击波来找 protect 压缩的程序的 OEP，几乎没有找不到的，但是这一次却不行了，只好用手动的方法来找了，发现了上面的新变化。如果大家跟踪的话，可以看到在 cs:135bb37 处的指令是这样的：   
    cs:135bb37  jmp  eax   
    就是从这里到 OEP 的，这里的 eax 是 5f4690。   
  
2.另外的变化是输入函数的处理又有了新的花招   
  asprotect 把一些输入函数在程序解压时首先运行，然后把得到的值存入一个变量里面，当解压后的程序去调用输入函数时，asprotect 把这个函数改成直接到对应的变量里面去取值就可以了。这个变化是这样发现的，在用 importREC.exe 来重建 IAT 时，有几个函数找不到正确的名称，我就随意给它们定了几个函数，看看程序能不能运行，结果是程序一运行老有非法操作，跟踪了很长时间也解决不了，最后想到是不是那几个没有修正的函数造成的，进去看那几个函数是干什么的，发现有的是到几个变量里面去取值赋给 EAX，也有的是垃圾函数，我就想到看看那几个变量里面的值最初是如何得到的，就用了 bpm 命令来看，一看使我大吃一惊，这几个变量里面的值是在运行了几个重要的函数后把得到的值赋进去的，实际上也就是输入函数的新的处理方法。我把得到的这几个函数的 RVA，在 asprotect 的解压缩程序的地址以及真实名称和得到的值的存储的地址列出来：   
    \[RVA\]        地址              函数名称            得到的值存储的地址   
  \[2192a4\]    133c86c            GetCommandlLineA      \[1343678\]   
  \[219488\]    133c834            GetVersion            \[13435D8\]   
  \[2194f8\]    133c864            GetCurrentProcessID  \[13435E0\]   
  当解压后的程序去调用这几个函数时，就到对应的变量里面去取就可以了。   
  另外还有几个函数，在程序里面不太用，大家有兴趣的话可以在下面的程序里面看到，在 cs:133c7da 打头的地方，大家可以跟踪去看一看。   
  当我搞定这个程序的时候，心里不免感叹 asprotect 真是太。。。。。。   
    
  就此打住。  4dd466d0e1e0cc123afbe504da2462ca