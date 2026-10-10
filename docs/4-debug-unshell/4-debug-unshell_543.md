

# ASProtect 2.11 SKE 之 advanced import protection

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-573.htm`](https://www.pediy.com/kssd/pediy08/pediy8-573.htm)

【破解作者】 windycandy  
【使用工具】 ODdyk,LordPe,Peid 0.94,ImportREC1.6  
【破解平台】 Win XP sp1  
【软件名称】 ASProtect 2.11 SKE 之 advanced import protection  
【软件简介】 ASProtect SKE  2.1x build 03.13 在看雪工具下载区下载的对 98notepad 加壳时,  
             用了除了 protect original entrypoint 外的选项，没有 stolen code,有变形码   
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
1.前言  
  
   5 月看雪论坛热闹非凡，激情洋溢，先是 Asprotect，再到 themida 一锅端，众位前辈和高手出手，精品穷出不尽，  
放之论坛与众人分享，实在是我等菜鸟莫大的福分。惜我等菜鸟功力不深.今看论坛有关 Asprotect 的文章已经  
是一堆堆一箩箩，尤其是 Volx 大侠的脱壳脚本已经很完美了。本问只要是练习手动脱 Advanced Import protection  
，仅以此文献给和我一样的菜鸟。高手略过。声明本文是学习看雪老师的经典作《Asprotect SKE 2.2 的 Advanced   
Import protection 保护技术》完成的。  
  
2.过程  
  
首先，到 OEP 后进入 call XXXXXXXX,对照看雪和 8100303 两位老师的教程，找到以下几个关键点处  
  
00ACA7D6     8B7C82 68               mov edi,dword ptr ds:\[edx+eax\*4+68\]  
00ACA7DA     8B06                    mov eax,dword ptr ds:\[esi\]  
00ACA7DC     FFD7                    call edi  
00ACA7DE     8845 CA                 mov byte ptr ss:\[ebp-36\],al  
00ACA7E1     8B45 F4                 mov eax,dword ptr ss:\[ebp-C\]  
00ACA7E4     8A40 4A                 mov al,byte ptr ds:\[eax+4A\]  
00ACA7E7     3A45 EF                 cmp al,byte ptr ss:\[ebp-11\]-------得到 AL 的值，决定是 ff15 还是 ff25  
00ACA7EA     0F85 9C000000           jnz 00ACA88C                       这个程序 AL=CC(记住 AL 的值）  
  
后面需要下硬件断点的 3 个位置  
  
00ACA7F3     8B45 F4                 mov eax,dword ptr ss:\[ebp-C\]  
00ACA7F6     8B80 E0000000           mov eax,dword ptr ds:\[eax+E0\]  
00ACA7FC     0145 FC                 add dword ptr ss:\[ebp-4\],eax  
00ACA7FF    /EB 01                   jmp short 00ACA802----------这里是 B1  
  
00ACA8A5     8B45 F4                 mov eax,dword ptr ss:\[ebp-C\]  
00ACA8A8     8B80 E0000000           mov eax,dword ptr ds:\[eax+E0\]  
00ACA8AE     0145 FC                 add dword ptr ss:\[ebp-4\],eax  
00ACA8B1     8D45 0C                 lea eax,dword ptr ss:\[ebp+C\]-----这里是 B2  
  
00ACB807     8945 FC                 mov dword ptr ss:\[ebp-4\],eax  
00ACB80A     8B45 E0                 mov eax,dword ptr ss:\[ebp-20\]  
00ACB80D     8B00                    mov eax,dword ptr ds:\[eax\]  
00ACB80F     E8 C0E6FFFF             call 00AC9ED4  
00ACB814     8BD0                    mov edx,eax  
00ACB816     0255 DF                 add dl,byte ptr ss:\[ebp-21\]  
00ACB819     8B4D FC                 mov ecx,dword ptr ss:\[ebp-4\]-------这里是 A  
  
找到这 3 个地方了就可以动手了，OD 载入目标程序，停在入口处，设置 OD 忽略所有异常  
  
00401000 N>  68 01D04000             push NOTEPAD.0040D001  
00401005     E8 01000000             call NOTEPAD.0040100B  
0040100A     C3                      retn  
0040100B     C3                      retn  
0040100C     7D 30                   jge short NOTEPAD.0040103E  
0040100E     DA8A 5DEA5230           fimul dword ptr ds:\[edx+3052EA5D\]  
00401014   - 78 BE                   js short NOTEPAD.00400FD4  
00401016     9C                      pushfd  
  
下 bp GetModuleHandleA, shift+F9 中断 2 次后，取消断点，ALT+F9 返回  
  
00AE14AC     85C0                    test eax,eax  ---返回这里                               ; kernel32.77E40000  
00AE14AE     75 07                   jnz short 00AE14B7  
00AE14B0     53                      push ebx  
00AE14B1     FF95 F0314400           call dword ptr ss:\[ebp+4431F0\]  
00AE14B7     8985 4D294400           mov dword ptr ss:\[ebp+44294D\],eax  
00AE14BD     C785 51294400 00000000  mov dword ptr ss:\[ebp+442951\],0  
00AE14C7     8B95 D8304400           mov edx,dword ptr ss:\[ebp+4430D8\]  
00AE14CD     8B06                    mov eax,dword ptr ds:\[esi\]  
00AE14CF     85C0                    test eax,eax  
00AE14D1     75 03                   jnz short 00AE14D6  
00AE14D3     8B46 10                 mov eax,dword ptr ds:\[esi+10\]  
  
单击右键“搜索------所有字符串--------“85”，来到  
  
00ACEBF8     68 E8F4AC00             push 0ACF4E8                                 ; ASCII "85"  
00ACEBFD     E8 2A62FEFF             call 00AB4E2C  
00ACEC02     A1 1C37AD00             mov eax,dword ptr ds:\[AD371C\]  
00ACEC07     8B00                    mov eax,dword ptr ds:\[eax\]  
00ACEC09     E8 0A8CFFFF             call 00AC7818-----------这里 F2 下断  
00ACEC0E     84C0                    test al,al  
00ACEC10     75 0A                   jnz short 00ACEC1C  
00ACEC12     68 E8F4AC00             push 0ACF4E8                                 ; ASCII "85"  
  
shift+F9,中断在 00ACEC09，F7 跟进  
  
00AC7818     53                      push ebx  
00AC7819     56                      push esi  
00AC781A     57                      push edi  
00AC781B     55                      push ebp  
00AC781C     83C4 F4                 add esp,-0C  
00AC781F     8BF0                    mov esi,eax  
00AC7821     C60424 01               mov byte ptr ss:\[esp\],1  
  
向下拉动鼠标找到  
  
00AC7959     56                      push esi  
00AC795A     E8 59FCFFFF             call 00AC75B8----这个 CALL 是进行 IAT 处理，先在 00AC7989F2 下断，再 F4 到这里，F7 跟进  
00AC795F     0FB707                  movzx eax,word ptr ds:\[edi\]  
00AC7962     83C0 02                 add eax,2  
00AC7965     03F8                    add edi,eax  
00AC7967     8A1F                    mov bl,byte ptr ds:\[edi\]  
00AC7969     47                      inc edi  
00AC796A     3A5E 34                 cmp bl,byte ptr ds:\[esi+34\]  
00AC796D   ^ 0F85 77FFFFFF           jnz 00AC78EA-------每个 DLL 的函数是否处理  
00AC7973     8BDF                    mov ebx,edi  
00AC7975     8B03                    mov eax,dword ptr ds:\[ebx\]  
00AC7977     85C0                    test eax,eax  
00AC7979   ^ 0F85 0AFFFFFF           jnz 00AC7889-------------比较 DLL 是否处理完  
00AC797F     8A0424                  mov al,byte ptr ss:\[esp\]  
00AC7982     83C4 0C                 add esp,0C  
00AC7985     5D                      pop ebp  
00AC7986     5F                      pop edi  
00AC7987     5E                      pop esi  
00AC7988     5B                      pop ebx  
00AC7989     C3                      retn--------先在这里 F2 下断  
  
进入 00AC795A 的 call，来到  
  
00AC75D9     8B45 10                 mov eax,dword ptr ss:\[ebp+10\]  
00AC75DC     83E8 02                 sub eax,2  
00AC75DF     0FB600                  movzx eax,byte ptr ds:\[eax\]  
00AC75E2     3B43 2C                 cmp eax,dword ptr ds:\[ebx+2C\]  
00AC75E5     76 06                   jbe short 00AC75ED  
00AC75E7     8943 2C                 mov dword ptr ds:\[ebx+2C\],eax  
00AC75EA     EB 01                   jmp short 00AC75ED  
00AC75EC     6933 C08A433B           imul esi,dword ptr ds:\[ebx\],3B438AC0  
00AC75F2     3BF0                    cmp esi,eax----------这里是比较 ESI 的 3 个值  
00AC75F4     75 5E                   jnz short 00AC7654---这里下断  
  
每人的机子不一样数值也不一样,我这里是 ESI=61，A5，26，其中 A5，61 时 IAT 不加密,其中 esi 的值为 61,26 时 00AC75F4 处的  
跳转成立,但两个数中只有当 esi=26 时才会对 IAT 进行加密,因此只要将 26 改为 61 就可以避开 IAT 加密了.  
好,就在 00AC75F4 进行修改,先用 OD 插件 memory manage 申请一个内存空间，我申请的是 01640000  
  
00AC75F4 75F4  - E9 078A3500     JMP 01640000        然后下 F2 断点，F9 运行中断后,F7 跟进  
  
将 PATH 代码写进去：  
  
01640000   - 0F84 F37548FF           je 00AC75F9  
01640006     83FE 61                 cmp esi,61  
01640009   - 0F84 457648FF           je 00AC7654  
0164000F     BE 61000000             mov esi,61  
01640014   - E9 3B7648FF             jmp 00AC7654  
01640019     90                      nop  
  
写好代码后,取消 00AC75F2 及 00AC75F4 两处的断点,F9 运行中断在 00AC7989,中断后取消断点  
看看数据窗口,是不是得到所有的 IAT 了?起始 4062E4,终点 406e00.  
  
004062E4  77DA2410  ADVAPI32.RegQueryValueExA  
004062E8  77DA17D8  ADVAPI32.RegCloseKey  
004062EC  77DB63B1  ADVAPI32.RegSetValueExA  
004062F0  77DA23D9  ADVAPI32.RegOpenKeyA  
004062F4  77DA28BB  ADVAPI32.RegCreateKeyA  
004062F8  00000000  
004062FC  77C4513D  GDI32.GetObjectA  
00406300  77C44B71  GDI32.GetDeviceCaps  
00406304  77C4889D  GDI32.CreateFontIndirectA  
  
00406DEC  B91D3742  
00406DF0  0055F5A5  OLE32.0055F5A5  
00406DF4  D13637ED  
00406DF8  6422CE52  
00406DFC  387863B7  
00406E00  00000000  
  
将 1640000 处的修改代码及 00AC75F4 75F4  - E9 078A3500     JMP 01640000 的修改  
全部取消修改.  
  
CTRL+B 搜索 33，C0，8A，07，8D，04，40，8B，6C，83，68，8B，C6，找到  
  
00ACB956     33C0                    xor eax,eax---这里  
00ACB958     8A07                    mov al,byte ptr ds:\[edi\]  
00ACB95A     8D0440                  lea eax,dword ptr ds:\[eax+eax\*2\]  
00ACB95D     8B6C83 68               mov ebp,dword ptr ds:\[ebx+eax\*4+68\]  
00ACB961     8BC6                    mov eax,esi  
00ACB963     FFD5                    call ebp  
00ACB965     8BE8                    mov ebp,eax  
00ACB967     036B 24                 add ebp,dword ptr ds:\[ebx+24\]  
00ACB96A     03AB E0000000           add ebp,dword ptr ds:\[ebx+E0\]  
00ACB970     EB 01                   jmp short 00ACB973  
00ACB972     E8 33C08A47             call 483779AA  
00ACB977     098D 04408B54           or dword ptr ss:\[ebp+548B4004\],ecx  
00ACB97D     8368 8B C6              sub dword ptr ds:\[eax-75\],-3A  
00ACB981     FFD2                    call edx-----------直接 F4 到这里  
00ACB983     807B 20 00              cmp byte ptr ds:\[ebx+20\],0  
00ACB987     0F85 3D010000           jnz 00ACBACA  
  
看堆栈：  
0012FEF8   00000044--------有 68 个地址要处理  
0012FEFC   495732D5  
0012FF00   0653FC49  
0012FF04   00000000  
  
先在这句 00ACB987  jnz 00ACBACA 按回车跟随到这里  
  
00ACBACA     8B43 2C                 mov eax,dword ptr ds:\[ebx+2C\]  
00ACBACD     2BC5                    sub eax,ebp  
00ACBACF     83E8 05                 sub eax,5  
00ACBAD2     45                      inc ebp  
00ACBAD3     8945 00                 mov dword ptr ss:\[ebp\],eax  
00ACBAD6     6A 0A                   push 0A  
00ACBAD8     E8 7F9AFEFF             call 00AB555C  
00ACBADD     8BC8                    mov ecx,eax  
00ACBADF     038B E4000000           add ecx,dword ptr ds:\[ebx+E4\]  
00ACBAE5     8BD6                    mov edx,esi  
00ACBAE7     8BC3                    mov eax,ebx  
00ACBAE9     E8 8EE5FFFF             call 00ACA07C  
00ACBAEE     FF0C24                  dec dword ptr ss:\[esp\]  
00ACBAF1     03B3 E4000000           add esi,dword ptr ds:\[ebx+E4\]  
00ACBAF7     833C24 00               cmp dword ptr ss:\[esp\],0  
00ACBAFB   ^ 0F87 55FEFFFF           ja 00ACB956  
00ACBB01     53                      push ebx  
00ACBB02     E8 5D000000             call 00ACBB64  
00ACBB07     0183 EC000000           add dword ptr ds:\[ebx+EC\],eax  
00ACBB0D     B0 01                   mov al,1  
00ACBB0F     83C4 24                 add esp,24  
00ACBB12     5D                      pop ebp  
00ACBB13     5F                      pop edi  
00ACBB14     5E                      pop esi  
00ACBB15     5B                      pop ebx  
00ACBB16     C3                      retn-----------找到这里，F2 下断  
  
在 00ACBB16 下断后，返回 00ACB981  
将 00ACB981 call edx 修改为：  
  
00ACB981   - E9 7A46B700             jmp 01640000  
00ACB986     90                      nop  
  
写入 path 代码：  
  
01640000     FFD2                    call edx  
01640002     60                      pushad  
01640003     8B1D 30006401           mov ebx,dword ptr ds:\[1640030\]  
01640009     C1E0 1F                 shl eax,1F  
0164000C     03C5                    add eax,ebp  
0164000E     8903                    mov dword ptr ds:\[ebx\],eax  
01640010     83C3 04                 add ebx,4  
01640013     891D 30006401           mov dword ptr ds:\[1640030\],ebx  
01640019     61                      popad  
0164001A     807B 20 00              cmp byte ptr ds:\[ebx+20\],0  
0164001E   - E9 A7BA48FF             jmp 00ACBACA  
01640023     0000                    add byte ptr ds:\[eax\],al  
01640025     0000                    add byte ptr ds:\[eax\],al  
01640027     0000                    add byte ptr ds:\[eax\],al  
01640029     0000                    add byte ptr ds:\[eax\],al  
0164002B     0000                    add byte ptr ds:\[eax\],al  
0164002D     0000                    add byte ptr ds:\[eax\],al  
0164002F     0040 00                 add byte ptr ds:\[eax\],al  
01640032     64:0100                 add dword ptr fs:\[eax\],eax  
  
二进制代码：  
  
FF D2 60 8B 1D 30 00 64 01 C1 E0 1F 03 C5 89 03 83 C3 04 89 1D 30 00 64 01 61 80 7B 20 00 E9 A7  
BA 48 FF 00 00 00 00 00 00 00 00 00 00 00 00 00 40 00 64 01 00  
  
写好代码后，F9 运行，中断 00ACBB16，取消断点，返回 00ACB981 撤消修改，数据框得到 68 个地址的数据  
  
D3 10 40 00 31 11 40 00 4C 11 40 00 56 13 40 00 05 15 40 00 1D 15 40 00 79 1E 40 00 E5 1E 40 00  
C9 21 40 00 DA 22 40 00 83 23 40 00 BC 23 40 00 3A 24 40 00 59 24 40 00 69 24 40 00 7E 24 40 00  
B6 24 40 00 04 25 40 00 1B 25 40 00 B9 26 40 00 09 29 40 00 83 29 40 00 9D 30 40 00 8C 31 40 00  
C8 31 40 00 1B 32 40 00 34 32 40 00 FC 32 40 00 19 33 40 00 26 33 40 00 53 33 40 00 80 33 40 00  
CE 33 40 00 E0 33 40 00 69 34 40 00 85 34 40 00 F5 34 40 00 05 35 40 00 70 35 40 00 7C 35 40 00  
A0 35 40 00 07 37 40 00 DD 37 40 00 C3 39 40 00 69 3D 40 00 7A 3D 40 00 0A 41 40 00 63 46 40 00  
CB 46 40 00 6F 49 40 00 F8 49 40 00 BF 4A 40 00 95 4B 40 00 AC 4B 40 00 78 4C 40 00 AB 4C 40 00  
08 4D 40 00 10 4E 40 00 5B 4E 40 00 75 4E 40 00 EE 4E 40 00 47 4F 40 00 60 4F 40 00 B0 4F 40 00  
B6 4F 40 00 C2 4F 40 00 C8 4F 40 00 CE 4F 40 00  
  
将这些数据复制出来（后面用到），然后将 01640000 到 01640150 间的数据全部撤消修改  
  
然后 F8 从 retn 返回到  
  
00ACB8B6     84C0                    test al,al  
00ACB8B8     75 0A                   jnz short 00ACB8C4  
00ACB8BA     68 E4B8AC00             push 0ACB8E4                                 ; ASCII "108"  
00ACB8BF     E8 6895FEFF             call 00AB4E2C  
00ACB8C4     5B                      pop ebx  
00ACB8C5     5D                      pop ebp  
00ACB8C6     C2 0400                 retn 4  
  
不断交替使用 CTRL+F9 和 F8（重复几次），来到  
  
77F79BA4     64:8B25 00000000        mov esp,dword ptr fs:\[0\]  
77F79BAB     64:8F05 00000000        pop dword ptr fs:\[0\]  
77F79BB2     8BE5                    mov esp,ebp  
77F79BB4     5D                      pop ebp  
77F79BB5     C2 1400                 retn 14  
77F79BB8     8B4C24 04               mov ecx,dword ptr ss:\[esp+4\]  
  
到这里后改用 ALT+F9，几次后来到  
  
00AD095A     C700 EFCA5C85           mov dword ptr ds:\[eax\],855CCAEF  
00AD0960     67:64:8F06 0000         pop dword ptr fs:\[0\]  
00AD0966     83C4 04                 add esp,4  
00AD0969     83E8 AF                 sub eax,-51  
00AD096C     83C8 4B                 or eax,4B  
00AD096F     58                      pop eax  
00AD0970     8BC3                    mov eax,ebx  
00AD0972     E8 49A8FEFF             call 00ABB1C0  
00AD0977     8B15 FC37AD00           mov edx,dword ptr ds:\[AD37FC\]  
00AD097D     0302                    add eax,dword ptr ds:\[edx\]  
00AD097F     83C0 08                 add eax,8  
00AD0982     BA 00100000             mov edx,1000  
00AD0987     E8 DCCEFFFF             call 00ACD868  
00AD098C     E8 CBEEFFFF             call 00ACF85C  
00AD0991     A3 20B6AD00             mov dword ptr ds:\[ADB620\],eax  
  
到这里了就可以 ALT+M， code 段下断，shift+F9 到 OEP 了  
  
004010CC     55                      push ebp  
004010CD     8BEC                    mov ebp,esp  
004010CF     83EC 44                 sub esp,44  
004010D2     56                      push esi  
004010D3     E8 28EF8F00             call 00D00000  
004010D8     93                      xchg eax,ebx  
004010D9     7D 0A                   jge short NOTEPAD.004010E5  
004010DB     8A00                    mov al,byte ptr ds:\[eax\]  
004010DD     3C 22                   cmp al,22  
004010DF     75 1B                   jnz short NOTEPAD.004010FC  
004010E1     56                      push esi  
  
ALT+G 1640000，到 1640000 处写入 path 代码（用看雪老师的，已经包括 A 情况）  
  
01640000     A1 C0006401             mov eax,dword ptr ds:\[16400C0\]  
01640005     8B18                    mov ebx,dword ptr ds:\[eax\]  
01640007     81E3 FFFFFF7F           and ebx,7FFFFFFF  
0164000D     FFE3                    jmp ebx  
0164000F     0000                    add byte ptr ds:\[eax\],al  
01640011     0000                    add byte ptr ds:\[eax\],al  
01640013     0000                    add byte ptr ds:\[eax\],al  
01640015     0000                    add byte ptr ds:\[eax\],al  
01640017     0000                    add byte ptr ds:\[eax\],al  
01640019     0000                    add byte ptr ds:\[eax\],al  
0164001B     0000                    add byte ptr ds:\[eax\],al  
0164001D     0000                    add byte ptr ds:\[eax\],al  
0164001F     0000                    add byte ptr ds:\[eax\],al  
01640021     BF C0006401             mov edi,16400C0  
01640026     8B07                    mov eax,dword ptr ds:\[edi\]  
01640028     8B18                    mov ebx,dword ptr ds:\[eax\]  
0164002A     81FB FFFFFF7F           cmp ebx,7FFFFFFF  
01640030     79 49                   jns short 0164007B  
01640032     837D D4 FF              cmp dword ptr ss:\[ebp-2C\],-1  
01640036     74 0F                   je short 01640047  
01640038     8B47 04                 mov eax,dword ptr ds:\[edi+4\]  
0164003B     8B1F                    mov ebx,dword ptr ds:\[edi\]  
0164003D     8B1B                    mov ebx,dword ptr ds:\[ebx\]  
0164003F     8918                    mov dword ptr ds:\[eax\],ebx  
01640041     83C0 04                 add eax,4  
01640044     8947 04                 mov dword ptr ds:\[edi+4\],eax  
01640047     8B5D FC                 mov ebx,dword ptr ss:\[ebp-4\]  
0164004A     E8 46000000             call 01640095  
0164004F     B0 CC                   mov al,0CC------------------前面找到 AL 的值 CC  
01640051     66:B9 FF15              mov cx,15FF  
01640055     3A45 EF                 cmp al,byte ptr ss:\[ebp-11\]  
01640058     74 05                   je short 0164005F  
0164005A     66:81C1 0010            add cx,1000  
0164005F     8B07                    mov eax,dword ptr ds:\[edi\]  
01640061     8B18                    mov ebx,dword ptr ds:\[eax\]  
01640063     81E3 FFFFFF7F           and ebx,7FFFFFFF  
01640069     83C0 04                 add eax,4  
0164006C     8907                    mov dword ptr ds:\[edi\],eax  
0164006E     66:890B                 mov word ptr ds:\[ebx\],cx  
01640071     83C3 02                 add ebx,2  
01640074     8933                    mov dword ptr ds:\[ebx\],esi  
01640076   ^ EB 88                   jmp short 01640000  
01640078     90                      nop  
01640079     90                      nop  
0164007A     90                      nop  
0164007B     8B5D B4                 mov ebx,dword ptr ss:\[ebp-4C\]  
0164007E     E8 12000000             call 01640095  
01640083     B0 CC                   mov al,0CC  
01640085     66:B9 FF15              mov cx,15FF  
01640089     3AC2                    cmp al,dl  
0164008B   ^ 74 D2                   je short 0164005F  
0164008D   ^ EB CB                   jmp short 0164005A  
0164008F     0000                    add byte ptr ds:\[eax\],al  
01640091     0000                    add byte ptr ds:\[eax\],al  
01640093     0000                    add byte ptr ds:\[eax\],al  
01640095     BE E4624000             mov esi,4062E4----------------IAT 的起始  
0164009A     391E                    cmp dword ptr ds:\[esi\],ebx  
0164009C     74 0D                   je short 016400AB  
0164009E     83C6 04                 add esi,4  
016400A1     81FE 006E4000           cmp esi,406E00----------------IAT 的结束  
016400A7     77 03                   ja short 016400AC  
016400A9   ^ EB EF                   jmp short 0164009A  
016400AB     C3                      retn  
016400AC   - EB FE                   jmp short 016400AC  
  
写好上面的代码后，将前面复制出来的 68 个地址复制过来（从 16400D0 开始）  
  
016400D0  D3 10 40 00 31 11 40 00 4C 11 40 00 56 13 40 00  ?@.1@.L@.V@.  
016400E0  05 15 40 00 1D 15 40 00 79 1E 40 00 E5 1E 40 00  @.@.y&@.?@.  
016400F0  C9 21 40 00 DA 22 40 00 83 23 40 00 BC 23 40 00  ?@.?@.?@.?@.  
01640100  3A 24 40 00 59 24 40 00 69 24 40 00 7E 24 40 00  :$@.Y$@.i$@.~$@.  
01640110  B6 24 40 00 04 25 40 00 1B 25 40 00 B9 26 40 00  ?@.%@.%@.?@.  
01640120  09 29 40 00 83 29 40 00 9D 30 40 00 8C 31 40 00  .)@.?@.?@.?@.  
01640130  C8 31 40 00 1B 32 40 00 34 32 40 00 FC 32 40 00  ?@.2@.42@.?@.  
01640140  19 33 40 00 26 33 40 00 53 33 40 00 80 33 40 00  3@.&3@.S3@.€3@.  
01640150  CE 33 40 00 E0 33 40 00 69 34 40 00 85 34 40 00  ?@.?@.i4@.?@.  
01640160  F5 34 40 00 05 35 40 00 70 35 40 00 7C 35 40 00  ?@.5@.p5@.|5@.  
01640170  A0 35 40 00 07 37 40 00 DD 37 40 00 C3 39 40 00  ?@.7@.?@.?@.  
01640180  69 3D 40 00 7A 3D 40 00 0A 41 40 00 63 46 40 00  i=@.z=@..A@.cF@.  
01640190  CB 46 40 00 6F 49 40 00 F8 49 40 00 BF 4A 40 00  薋@.oI@.鳬@.縅@.  
016401A0  95 4B 40 00 AC 4B 40 00 78 4C 40 00 AB 4C 40 00  昁@.琄@.xL@.獿@.  
016401B0  08 4D 40 00 10 4E 40 00 5B 4E 40 00 75 4E 40 00  M@.N@.\[N@.uN@.  
016401C0  EE 4E 40 00 47 4F 40 00 60 4F 40 00 B0 4F 40 00  頝@.GO@.\`O@.癘@.  
016401D0  B6 4F 40 00 C2 4F 40 00 C8 4F 40 00 CE 4F 40 00  禣@.翺@.萇@.蜲@.  
016401E0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  
复制好后，手工在 16400C0 和 16400C4 输入两个数据：  
  
016400C0  D0 00 64 01 00 02 64 01 00 00 00 00 00 00 00 00  ?d.d........  
  
写好后可以在前面找到的地方下硬件断点了：  
  
00ACA7F3     8B45 F4                 mov eax,dword ptr ss:\[ebp-C\]  
00ACA7F6     8B80 E0000000           mov eax,dword ptr ds:\[eax+E0\]  
00ACA7FC     0145 FC                 add dword ptr ss:\[ebp-4\],eax  
00ACA7FF    /EB 01                   jmp short 00ACA802----------这里是 B1  
  
00ACA8A5     8B45 F4                 mov eax,dword ptr ss:\[ebp-C\]  
00ACA8A8     8B80 E0000000           mov eax,dword ptr ds:\[eax+E0\]  
00ACA8AE     0145 FC                 add dword ptr ss:\[ebp-4\],eax  
00ACA8B1     8D45 0C                 lea eax,dword ptr ss:\[ebp+C\]-----这里是 B2  
  
00ACB807     8945 FC                 mov dword ptr ss:\[ebp-4\],eax  
00ACB80A     8B45 E0                 mov eax,dword ptr ss:\[ebp-20\]  
00ACB80D     8B00                    mov eax,dword ptr ds:\[eax\]  
00ACB80F     E8 C0E6FFFF             call 00AC9ED4  
00ACB814     8BD0                    mov edx,eax  
00ACB816     0255 DF                 add dl,byte ptr ss:\[ebp-21\]  
00ACB819     8B4D FC                 mov ecx,dword ptr ss:\[ebp-4\]-------这里是 A  
  
把这 3 个地方设置硬件断点后，在第一个 call XXXXXXX 的地方（这里是 4010D3）新建 EIP，F9 运行，遇到硬件断点  
就运行以下脚本：  
  
mov eip,1640021   
run  
mov eip,1640021   
run  
mov eip,1640021   
run  
mov eip,1640021   
run  
mov eip,1640021   
run  
  
一直运行到出现“不知道如何继续运行，因为内存地址 00000000 处是不易读取的..................”  
这时就可以用 LordPe 纠正大小后 dump 了。然后用 ImportREC Fixed.  
可是运行时出现了错误。  
  
注意数据框：  
  
01640200  D3 10 40 00 1D 15 40 00 E5 1E 40 00 C9 21 40 00  ?@.@.?@.?@.  
01640210  3A 24 40 00 B6 24 40 00 B9 26 40 00 F5 34 40 00  :$@.?@.?@.?@.  
01640220  05 35 40 00 7C 35 40 00 00 00 00 00 00 00 00 00  5@.|5@.........  
  
即 4010D3，40151D，401EE5，4021C9，40243A，4024B6，4026B9，4034F5，403505，40357C  
看雪老师的教程指出这些地址的下一句代码有可能被变形了。  
  
载入没加壳前的 notepad 进行追踪，发现除了这几个地址的没有变形外，其他的都变形了。  
正常没有变形的地址：401E5E，40243A，4026B9，403505；  
  
变形的情况如下：  
  
4010D3   
  
004010D2     56                      push esi  
004010D3     FF15 E4634000           call dword ptr ds:\[<&kernel32.GetCommandLine>; kernel32.GetCommandLineA  
004010D9     7D 0A                   jge short dumped\_.004010E5---------脱壳后的变形  
004010DB     8A00                    mov al,byte ptr ds:\[eax\]  
  
004010D2   |.  56                    push esi  
004010D3   |.  FF15 E4634000         call dword ptr ds:\[<&kernel32.GetCommandLine>; \[GetCommandLineA  
004010D9   |.  8BF0                  mov esi,eax----------------没加壳的原代码  
004010DB   |.  8A00                  mov al,byte ptr ds:\[eax\]  
  
40151D   
  
0040151D     FF15 FC634000           call dword ptr ds:\[<&shell32.ShellAboutA>\]   ; shell32.ShellAboutA  
00401523     885D 6C                 mov byte ptr ss:\[ebp+6C\],bl---------脱壳后的变形  
00401526     AF                      scas dword ptr es:\[edi\]  
00401527     CD 6A                   int 6A  
00401529     00E8                    add al,ch  
0040152B     3B36                    cmp esi,dword ptr ds:\[esi\]  
  
  
0040151D   |.  FF15 FC634000         call dword ptr ds:\[<&shell32.ShellAboutA>\]   ; \\ShellAboutA  
00401523   |.  E9 9F030000           jmp 01.004018C7-----------没加壳的原代码  
00401528   |>  6A 00                 push 0                                       ; /Arg1 = 00000000; Case C of switch 00401283  
0040152A   |.  E8 3B360000           call 01.00404B6A                             ; \\01.00404B6A  
  
4021C9  
  
004021C9     FF15 C4634000           call dword ptr ds:\[<&kernel32.FindClose>\]    ; kernel32.FindClose  
004021CF     C00B 57                 ror byte ptr ds:\[ebx\],57---------脱壳后的变形  
004021D2     8D85 C4FDFFFF           lea eax,dword ptr ss:\[ebp-23C\]  
004021D8     50                      push eax  
  
                               
004021C9   |.  FF15 C4634000         call dword ptr ds:\[<&kernel32.FindClose>\]    ; \\FindClose  
004021CF   |.  EB 0E                 jmp short 01.004021DF----------没加壳的原代码  
004021D1   |>  57                    push edi                                     ; /String2  
004021D2   |.  8D85 C4FDFFFF         lea eax,dword ptr ss:\[ebp-23C\]               ; |  
  
4024B6   
  
004024B6     FF15 E4624000           call dword ptr ds:\[<&advapi32.RegQueryValueE>; advapi32.RegQueryValueExA  
004024BC     16                      push ss---------脱壳后的变形  
004024BD     D5 8B                   aad 8B  
004024BF     45                      inc ebp  
004024C0     FC                      cld  
004024C1     85C0                    test eax,eax  
  
  
004024B6   |.  FF15 E4624000         call dword ptr ds:\[<&advapi32.RegQueryValueE>; \\RegQueryValueExA  
004024BC   |.  EB 03                 jmp short 01.004024C1----------没加壳的原代码  
004024BE   |>  8B45 FC               mov eax,dword ptr ss:\[ebp-4\]  
  
4034F5   
  
004034F5     FF15 74634000           call dword ptr ds:\[<&kernel32.\_llseek>\]      ; kernel32.\_llseek  
004034FB     CE                      into---------脱壳后的变形  
004034FC     0B53 53                 or edx,dword ptr ds:\[ebx+53\]  
004034FF     A1 40564000             mov eax,dword ptr ds:\[405640\]  
  
004034F5   |.  FF15 74634000         call dword ptr ds:\[<&kernel32.\_llseek>\]      ; \\\_llseek  
004034FB   |.  8BF8                  mov edi,eax----------没加壳的原代码  
004034FD   |.  53                    push ebx                                     ; /Origin => FILE\_BEGIN  
004034FE   |.  53                    push ebx                                     ; |Offset => 0  
  
有关代码变形的情况，看雪老师的文章已经分析很清楚了。尽管没有 stolen code 感觉这个壳变形很厉害，本文是在有  
原程序的情况下找出变形代码的，如果没有程序很难找出变形的代码。况且如果它不是只变形一句两句，而是一大片呢？  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca