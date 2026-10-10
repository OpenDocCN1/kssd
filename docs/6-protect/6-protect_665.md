

# Code Virtualizer 1.3.8.0 小窥。

> 原文：[`www.pediy.com/kssd/pediy11/125140.html`](https://www.pediy.com/kssd/pediy11/125140.html)

似乎分析 cv 的东西不多，还好找到了 Ryosuke 兄弟的一个详细的分析，非常强大的。  
http://bbs.pediy.com/showthread.php?t=62447&highlight=Virtualizer  
  
1.3.8.0 这版和 Ryosuke 分析的那版没有太多变化，vm\_context 中多了一个字段，增加到了 0x48，其他的变化都不大，个别 dispatch 函数有些变化。选择不同的加密强度测试看，似乎在代码混淆方面并没明显加强，也许测试代码少的原因，分析的不到位。当时看的时候有些地方还没看懂，还是动手自己琢磨一下，顺便记录下的笔记，这个分析只是婴儿奶粉，不能补钙，高手略过吧。  
  
  
    先把 Code Virtualizer 的虚拟机简单说一下，这里把我们自己的程序运行空间叫做宿主吧。cv 虚拟机的所有运算的操作都在宿主的堆栈中完成。  
  
    cv 自己的存放宿主程序的寄存器及解析例程的结构叫 vm\_context，所有操作的都是由一个叫 vm\_data 数据块来驱动的。这里面的数据会指示 cv 的虚拟机如何取指令计算出新的数据。  
  
    程序运行进入 cv 虚拟机后，edi 的值始终不变，一直指向 vm\_context（在 Ryosuke 的里面叫 VM\_BLOCK 都是一个意思）的首地址。  
  
    从偏移 0 ~ 0x1c 存放的是宿主的寄存器，但每次都会变，这个是按某种顺序的变化的。  
      
    esi 指向 vm\_data 结构，每执行一个 dispatch，esi 不断的增加。  
      
    eax 的用处就是从 esi 中读取数据，进行运算，获得解析例程的 dispatch 号码。或者是 vm\_context 的某个相对偏移地址(cv 里面为了操作某个结构)  
      
    ebx 就是参与计算用的，和 vm\_data 中的数据运算，用来获得有效的 eax  
      
    edx 是个临时的变量，cv 在宿主堆栈的运算中，常用 edx 做临时变量来存放数据  
      
    ebp，esp 基本没有太大用处，参与一些混淆代码的计算。  
      
   struct vm\_context  
   {  
  DWORD  vm\_off\_00;     +0  
  DWORD  vm\_off\_04;     +4  
  DWORD  vm\_off\_08;     +8  
  DWORD  vm\_off\_0c;     +0c  
  DWORD  vm\_off\_10;     +10  
  DWORD  vm\_off\_14;     +14  
  DWORD  vm\_off\_18;     +18  
  DWORD  vm\_off\_1c;     +1c  
  DWORD  vm\_off\_20;  
  DWORD  VM\_off\_24;  
  DWORD  VM\_ecx\_value;  
  DWORD  VM\_off\_2c;    
  DWORD  VM\_busy\_flag;  
  DWORD  VM\_off\_34;  
  DWORD  VM\_off\_38;  
  DWORD  VM\_off\_3c;  
  DWORD  VM\_off\_40;  
  DWORD  VM\_off\_44;  
  DWORD  VM\_off\_48;  
  DWORD  vm\_context\_xxx1;  
  DWORD  VM\_dispatch\_xxx2;  
  DWORD  VM\_dispatch\_xxx3;  
  ...  
   }  
      
  下面就测试这么一个例子：  
   
  .data   
  t\_buff  db 100 dup(090h)   
  t\_txt   db "test",0  
  t\_title db "codevirtualizer",0  
  .code  
  
  START:    
    
  mov     eax,offset t\_buff  
    
  call    VirtualizerStart      
  mov     dword ptr\[eax\],01234567h  ; 既然小窥，先只测试这一条语句吧，对 cv 的分析可以在里加入不同的语句一点一点分析      
  call    VirtualizerEnd  
    
  invoke  MessageBox,0,offset t\_txt,offset t\_title,0  
  invoke  ExitProcess,0  
  
  end START  
    
  分别在 Protection Options 中选择这两种个加一次  
  
          Virtual Opcodes Obfuscation :Low ,Highest    
          Virtual Machine Complexity  :Low ,Highest   
          Opcodes Mutation            :Low ,Highest   
            
  分析后可以发现再混淆方面差别并不是太大。            
  
  进入 VM 之前先保存程序最初的环境  
  
  EAX 00403000    
  ECX 0013FFB0  
  EDX 7C92E514    
  EBX 7FFDE000  
  ESP 0013FFC0  
  EBP 0013FFF0  
  ESI 00000068  
  EDI 0013B790  
  EIP 004034EC CV\_TES~3.004034EC  
     
  EFL 00000246 (NO,NB,E,BE,NS,PE,GE,LE)  
   
    
  软后先压入 VM\_DATA  
  00407083     68 1A704000        push CV\_TES~3.0040701A  -----VM\_DATA  
  00407088   ^ E9 5FC4FFFF        jmp CV\_TES~3.004034EC  
  
  
  VM\_DATA 内容每次加壳后都变化  
  0040701A  3F 1E E1 D0 F8 5F A0 8F B7 A4 5B 4A 72 E4 1B 0A  ?嵝筏\[Jr?.  
  0040702A  32 2A D5 C4 5F D9 C0 9C 63 52 7A DE 21 10 38 21  2\*漳\_ 倮 Rz?8!  
  0040703A  DE CD F5 65 9A 89 11 74 22 9F 40 20 7A DD 43 EF  尥 t" z  
  0040704A  45 05 C3 7B DD 74 54 43 A9 25 6B BD 85 1A EF 01  ETC?k?  
  0040705A  00 00 00 FB 59 04 FB 59 C7 93 6C CA 38 21 DE 3C  ...l?!?  
  0040706A  AA AE 51 AF 1D 3F C0 1E 8C D0 2F 8D FB 5B A4 02  Q???/\[?  
  0040707A  70 EB 14 72 E0 77 88 E6 B8 68 1A 70 40 00 E9 5F  p?rp@.  
  0040708A  C4 FF FF 00 00 00 00 00 00 00 00 00 00 00 00 00  ?.............  
  
  通过重定位获得自身数据及处理例程的偏移，设置重定位标志，如果是 dll 还要重新定位。  
  
  004034EF     E8 00000000        call CV\_TES~3.004034F4   ; 重定位自身  
  004034F4     5F                 pop edi  
  004034F5     81EF F4344000      sub edi,CV\_TES~3.004034F4  
  004034FB     8BC7               mov eax,edi  
  004034FD     81C7 00324000      add edi,offset <CV\_TES~3.vm\_dispatch> ;定位 edi 为 vm\_context   
  00403503     3B47 2C            cmp eax,dword ptr ds:\[edi+2C\]  
    
    
  00403556     AC              lods byte ptr ds:\[esi\]  ;此处进入虚拟机，知道遇到 vm\_exit，才退出。  
  00403557     04 81           add al,81  
  00403559     04 90           add al,90  
  0040355B     00D8            add al,bl  
  0040355D     2C 90           sub al,90  
  0040355F     E9 080A0000     jmp CV\_TES~3.00403F6C  
     
  1.3.8 版的 vm\_dispatch  
  
  00403200 >00000000  
  00403204  00000000  
  00403208  00000000  
  0040320C  00000000  
  00403210  00000000  
  00403214  00000000  
  00403218  00000000  
  0040321C  00000000  ----->eflag 标志  
  00403220  00000000  
  00403224  00000000  
  00403228  00000000  
  0040322C  00000000  
  00403230  00000000  ----->是否是忙的标志  
  00403234  00000000  
  00403238  00000000  
  0040323C  00000000  
  00403240  00000000  
  00403244  00000000  
  00403248  00000000 ------->相比前版本增加的字段  
  0040324C  0040469C  CV\_TES~3.0040469C  --- > 解析例程  
  00403250  00403843  CV\_TES~3.00403843  --- > 解析例程  
  00403254  00403B82  CV\_TES~3.00403B82  --- > 解析例程  
  00403258  00403FE9  CV\_TES~3.00403FE9  --- > 解析例程  
  0040325C  00403608  CV\_TES~3.00403608  --- > 解析例程  
  00403260  004037E2  CV\_TES~3.004037E2  --- > 解析例程  
  00403264  00403723  CV\_TES~3.00403723  --- > 解析例程  
  00403268  004069F1  CV\_TES~3.004069F1  --- > 解析例程  
  0040326C  00406241  CV\_TES~3.00406241  --- > 解析例程  
  00403270  004064F3  CV\_TES~3.004064F3  --- > 解析例程  
  00403274  00405C54  CV\_TES~3.00405C54  --- > 解析例程  
  ...  
    
  
  先开始分析进入虚拟机情况第一次情况：  
  
  EAX 0000003F  
  ECX 00000001  
  EDX 7C92E514 ntdll.KiFastSystemCallRet  
  EBX 0040701A CV\_TES~3.0040701A           -----> vm\_data\[0\] 数据  
  ESP 0013FF9C  
  EBP 0013FFF0  
  ESI 0040701B CV\_TES~3.0040701B           -----> vm\_data\[n+1\] 以后 esi 依次递增  
  EDI 00403200 offset <CV\_TES~3.vm\_context>  
  EIP 00403557 CV\_TES~3.00403557  
   
  EFL 00000246 (NO,NB,E,BE,NS,PE,GE,LE)  
   
//进入运算前，就从 VM\_DATA 中读取的 al->0x3f    
//其实运算的内容都是为了混淆变乱而已，为了做到自动的去除这些指令，还得分析清楚这些内容  
  
//跟踪的时候，盯住 al 寄存器，ebx 及堆栈数据的的变化就可以。  
//al   - 计算得知最后的解析例程的索引值  
//ebx  - 参与中间计算的用途，用来计算 al 时，所需要的值，每次都变化  
  
//下面的代码是为了计算一个 dispatch 的索引，跟踪一遍明白原来后，其他的情况都类似  
  
00403556     AC                 lods byte ptr ds:\[esi\]  --- 读取 VM\_DATA 中的数据，此时 al->0x3f  
00403557     04 81              add al,81  
00403559     04 90              add al,90  
0040355B     00D8               add al,bl  
0040355D     2C 90              sub al,90  
0040355F    /E9 080A0000        jmp CV\_TES~3.00403F6C  
  
eax --> 0xDA  
//--------------------------------------  
  
// 00403F6C  
00403F6C     53                 push ebx                       ; CV\_TES~3.0040701A  
00403F6D     B7 57              mov bh,57  
00403F6F     80EF D6            sub bh,0D6  
00403F72     28F8               sub al,bh  
00403F74     E9 4A230000        jmp CV\_TES~3.004062C3  
  
eax -> 0x59  
//--------------------------------------  
  
//004062C3  
004062C3     5B                 pop ebx                        ; CV\_TES~3.0040701A  
004062C4     68 C5250000        push 25C5  
004062C9     891C24             mov dword ptr ss:\[esp\],ebx     ; 这里注意堆栈中记录了 vm\_data 位置，也就是 ebx 的值   
004062CC     B7 D2              mov bh,0D2                     ; 修改了 ebx 值  
004062CE   ^ E9 97E8FFFF        jmp CV\_TES~3.00404B6A  
  
eax-> 0x59  
esp-4 -> 0x40701A (vm\_data offset)  
   
// 继续跟踪  
  
00404B6A     30F8               xor al,bh  
00404B6C     8B1C24             mov ebx,dword ptr ss:\[esp\]  ; --> 又把 ebx 恢复了  
00404B6F     81C4 04000000      add esp,4                   ; 恢复堆栈了，不再用\[esp\]保留,ebx 值了。  
00404B75     52                 push edx                    ; 又把 edx 压入堆栈保存起来，开始折腾 edx 值了  
00404B76   ^ E9 06FDFFFF        jmp CV\_TES~3.00404881  
eax-> 0x8B  
edx-> 7C92E514  
  
//--------------------------------------  
//00404881  
  
00404881     B6 30              mov dh,30  
00404883     80EE 70            sub dh,70  
00404886     80C6 34            add dh,34  
00404889     80EE 98            sub dh,98  
0040488C     F6D6               not dh  
0040488E   ^ E9 E5F2FFFF        jmp CV\_TES~3.00403B78  
// eax 不变，   
// edx -> 7C92A314  
//--------------------------------------  
  
//00403B78  
00403B78     80F6 52            xor dh,52  
00403B7B     2C 34              sub al,34  
00403B7D     E9 E5130000        jmp CV\_TES~3.00404F67  
  
// eax->0x57  
// edx->7C92F114  
  
//--------------------------------------  
//00404F67  
00404F67     28F0               sub al,dh  
00404F69     51                 push ecx             ; ecx 又压入栈中折腾去了  
00404F6A     E9 390D0000        jmp CV\_TES~3.00405CA8  
  
// eax->66  
// edx 不变  
// ecx->0x01  
//--------------------------------------  
00405CA8     B1 12              mov cl,12  
00405CAA     C0E1 05            shl cl,5  
00405CAD     E9 210B0000        jmp CV\_TES~3.004067D3  
  
// eax 不变  
// edx 不变  
// ecx->40  
//--------------------------------------  
//004067D3   
  
004067D3     80C1 F4            add cl,0F4  
004067D6     00C8               add al,cl  
004067D8     59                 pop ecx  
004067D9     5A                 pop edx  
004067DA   ^ E9 B8FFFFFF        jmp CV\_TES~3.00406797  
  
// 堆栈已经被恢复  
// eax -> 0x9A  
// edx -> 变为最初值 0x7C92E514  
// ecx -> 变为最初值 0x00000001  
  
//--------------------------------------  
// 修改 ebx 值   
  
//00406797  
00406797     80C3 4C            add bl,4C  
0040679A     80C3 BE            add bl,0BE  
0040679D     00C3               add bl,al  
0040679F     80EB BE            sub bl,0BE  
004067A2   ^ E9 E8FFFFFF        jmp CV\_TES~3.0040678F  
  
// eax -> 0x9A  
// ebx -> 00407000  
  
//--------------------------------------  
// 0040678F  
0040678F     51                 push ecx  
00406790     B5 CC              mov ch,0CC  
00406792   ^ E9 F2DEFFFF        jmp CV\_TES~3.00404689  
  
// eax -> 0x9A  
// ebx -> 00407000  
// ecx -> 0000CC01  
  
//--------------------------------------  
00404689  
00404689     D0ED               shr ch,1  
0040468B     80E5 95            and ch,95  
0040468E     80F5 48            xor ch,48  
00404691     28EB               sub bl,ch  
00404693     59                 pop ecx      ; 恢复了 ecx 为 0x00000001  
00404694     0FB6C0             movzx eax,al ; eax -> 0x0000009A  
00404697   ^\\FF2487             jmp dword ptr ds:\[edi+eax\*4\]  ; 调向了真正的解析例程中  
  
// ebx -> 0x004070b4  
  
dword ptr ds:\[edi+eax\*4\]  相当于  
ds:\[00403468\]=00403FFE (CV\_TES~3.00403FFE)  
  
403468 是 vm\_dispatch 的数据范围，里面存放的时候 00403FFE 解析例程  
到此时我们已经知道，这一路跟踪下来的代码仅为了计算一个 eax 的值，用来定位解析例程的索引。  
  
但 al 从 0x3F 如何变成 0x9A 的？只要查看这一路跟下了起变化的过程即可。  
  
al = vm\_data\[0 ~ n\] + 0x81 + 0x90 + bl - 0x90 - 0x81 ^ 0xD2  - 0x34 - 0xF1  + 0x34   
al = vm\_data\[0 ~ n\] + bl ^ 0xD2 - 0xF1   
al = (0x3f + 0x1a ) ^ 0xD2 - 0xF1  
al = 0x9A  
  
所以固定的计算就是   
index = (al + bl ) ^ 0xD2 - 0xF1 ;  
同时更新 ebx 的值，  
bl = bl + al  
  
还是用这个程序加壳生成另外一个时，会发现和前面过程中执行的指令一摸一样，仅是利用大量的 jmp 打乱原有的指令  
而已,也就是说指令的运算及执行条数是固定的，然后用 jmp 来打乱。只有设置不同的加密等级时才变得不一样。  
  
  
x0:                                x0:  
  xxx1                               xxx1  
  xxx2                               xxx2  
  xxx3            混乱为           jmp x1  
jmp x1         --------------->           
   
x1:                                x1:  
  xxx4                               xxx3  
  xxx5                             jmp x2  
  xxx6  
jmp x2                             x2:  
...                                  xxx4  
                                     xxx5  
                                     xxx6   
                                     jmp x3  
                                     ...  
                                       
// 继续看处理例程，还是那套路子，不过这次计算的 al 是用来获得 vm\_context 中要操作的一个寄存器的索引标志  
// 简化后 al = al - bl - 0x2b - 0x38   
  
00403FFE     AC                 lods byte ptr ds:\[esi\] ；取 vm\_data 下一个数据  
00403FFF     2C 36              sub al,36  
00404001     E9 F81A0000        jmp CV\_TES~3.00405AFE  
  
00405AFE     28D8               sub al,bl  
00405B00     52                 push edx  
00405B01     66:53              push bx  
00405B03   ^ E9 71F9FFFF        jmp CV\_TES~3.00405479  
  
00405479     B7 AB              mov bh,0AB  
0040547B     88FE               mov dh,bh  
0040547D     66:5B              pop bx  
0040547F     C0EE 06            shr dh,6  
00405482     FEC6               inc dh  
00405484     80F6 35            xor dh,35  
00405487     00F0               add al,dh  
00405489     E9 0B190000        jmp CV\_TES~3.00406D99  
  
00406D99     5A                 pop edx                                    
00406D9A   ^ E9 9CD0FFFF        jmp CV\_TES~3.00403E3B  
  
00403E3B     51                 push ecx  
00403E3C     B5 C2              mov ch,0C2  
00403E3E     F6DD               neg ch  
00403E40     FECD               dec ch  
00403E42     F6D5               not ch  
00403E44     E9 8C0E0000        jmp CV\_TES~3.00404CD5  
  
00404CD5     80E5 6A            and ch,6A  
00404CD8   ^ E9 47F0FFFF        jmp CV\_TES~3.00403D24  
  
00403D24     F6D5               not ch  
00403D26     80C5 6E            add ch,6E  
00403D29     28E8               sub al,ch  
00403D2B     8B0C24             mov ecx,dword ptr ss:\[esp\]  
00403D2E     83C4 04            add esp,4  
00403D31     53                 push ebx  
00403D32     B7 A7              mov bh,0A7  
00403D34     50                 push eax  
00403D35     B4 B3              mov ah,0B3  
00403D37     80C4 26            add ah,26  
00403D3A     80EC 91            sub ah,91  
00403D3D     E9 3F1F0000        jmp CV\_TES~3.00405C81  
  
00405C81     80C7 D1            add bh,0D1  
00405C84     28E7               sub bh,ah  
00405C86   ^ E9 27E4FFFF        jmp CV\_TES~3.004040B2  
  
004040B2     80EF D1            sub bh,0D1  
004040B5     58                 pop eax  
004040B6     F6D7               not bh  
004040B8     E9 6D2E0000        jmp CV\_TES~3.00406F2A  
  
00406F2A     C0EF 05            shr bh,5  
00406F2D     F6DF               neg bh  
00406F2F     C0EF 05            shr bh,5  
00406F32     80C7 31            add bh,31  
00406F35     28F8               sub al,bh  
00406F37     5B                 pop ebx  
00406F38     51                 push ecx  
00406F39     B1 F7              mov cl,0F7  
00406F3B   ^ E9 26D9FFFF        jmp CV\_TES~3.00404866  
  
00404866     D0E1               shl cl,1  
00404868   ^ E9 5BFAFFFF        jmp CV\_TES~3.004042C8  
  
004042C8     80C1 4F            add cl,4F  
004042CB     80E1 E7            and cl,0E7  
004042CE     E9 40140000        jmp CV\_TES~3.00405713  
  
00405713     80E1 BD            and cl,0BD  
00405716     80E9 2D            sub cl,2D  
00405719     00CB               add bl,cl  
0040571B     59                 pop ecx  
0040571C     80EB 68            sub bl,68  
0040571F   ^ E9 F2E2FFFF        jmp CV\_TES~3.00403A16  
  
00403A16     00C3               add bl,al  
00403A18     E9 56000000        jmp CV\_TES~3.00403A73  
  
00403A73     80C3 68            add bl,68  
00403A76     80EB F8            sub bl,0F8  
00403A79     0FB6C0             movzx eax,al  
  
// eax == 7   
00403A7C     8D0487             lea eax,dword ptr ds:\[edi+eax\*4\]   
  
// 上面执行完 eax == 0040321C,是 vm\_context 结构中的偏移 1c 的位置，经过分析知道是 eflag 标志  
  
00403A7F     68 7E320000        push 327E  ; 压入一个临时数字，后面要改写这个数字               
00403A84     E9 270D0000        jmp CV\_TES~3.004047B0  
  
004047B0     890424             mov dword ptr ss:\[esp\],eax ; 修改这个数字  
  
// 这相当于把虚拟机的 eflag 标志的地址进栈  
// ebx - > 004070BB  
// esi - > 0040701C -- 此时指向 vm\_data+3 处  
  
004047B3   ^ E9 9EEDFFFF        jmp CV\_TES~3.00403556    ;再次跳回 vm 起始位置  
  
相当于执行：push eflag  
  
// 至此我们已经知道刚才的那个 00403FFE 例程是 cv 虚拟机压入 eflag 标志进栈操作,但该例程并不是就单纯完成 push eflag  
// 因为 eflag 的偏移是根据 vm\_data 中的数据计算而来的，所以这个例程应该是 push imm32 功能，imm32 是根据传入的参数  
// 来确定的。  
  
// 继续看下面的例程，al = 0xe1 ,计算后得到的例程地址为 5D，\[edi + eax \* 4\] = 0x403f8f  
// 分析可知 0x403f8f 的解析例程目的是执行 pop edx，执行过程如下，\*表示垃圾指令  
  
00403F8F     8B1424             mov edx,dword ptr ss:\[esp\] ; CV\_TES~3.0040321C  
00403F92     55                 push ebp      \*   
00406EC7     89E5               mov ebp,esp   \*  
00406EC9     81C5 04000000      add ebp,4     \*  
00406C51     51                 push ecx      \*  
00406C52     B9 04000000        mov ecx,4     \*  
00406BEA     01CD               add ebp,ecx   \*   
00406BEC     59                 pop ecx       \*  
00406BED     872C24             xchg dword ptr ss:\[esp\],ebp \*  
00406BF0     5C                 pop esp       \*  
  
执行；pop edx  
  
大家可以在 00404697  jmp dword ptr ds:\[edi+eax\*4\]  这个地方大家可以下断点，因为每一个 cv 的 dispatch 执  
行完毕后，都要调向这里去执行下一个 dispatch   
  
// 继续获得解析例程地址为 ds:\[00403324\] = 0x00405F70   
00405F70     8F02               pop dword ptr ds:\[edx\]    
00405F72   ^ E9 DFD5FFFF        jmp CV\_TES~3.00403556  --- 跳回 vm\_start  
  
// 也就是 0x00405F70 例程  
执行 ：pop \[edx\]  
  
// 下一条解析指令 ds:\[00403468\]= 0x00403FFE ，这是已知的，上面分析过  
执行 ：push vm\_off\_04  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 EDI 的值，所以可以推断出 vm\_off\_04 是 cv 虚拟机的存放 edi 的地方  
  
//--------------------------------------  
  
// 下一条解析指令 ds:\[00403468\]=00403FFE ,这是已知的，上面分析过  
执行：push vm\_off\_14  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 ESI 的值，所以可以推断出 vm\_off\_14 是 cv 虚拟机的存放 esi 的地方  
  
//--------------------------------------  
  
// 下一条解析指令 ds:\[00403468\]= 0x00403FFE ，这是已知的，上面分析过  
执行 ：push vm\_off\_00  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 ebp 的值，所以可以推断出 vm\_off\_00 是 cv 虚拟机的存放 ebp 的地方  
  
//--------------------------------------    
  
// 下一条解析指令 ds:\[00403468\]= 0x00403FFE ，这是已知的，上面分析过  
执行 ：push vm\_off\_18  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 Esp 的值，所以可以推断出 vm\_off\_18 是 cv 虚拟机的存放 esp 的地方  
  
//--------------------------------------  
  
// 下一条解析指令 ds:\[00403468\]= 0x00403FFE ，这是已知的，上面分析过  
执行 ：push vm\_off\_18  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 Esp 的值，所以可以推断出 vm\_off\_18 是 cv 虚拟机的存放 esp 的地方  
  
//--------------------------------------  
接着对 vm\_off\_28 赋值 0x03 ，在 vm\_context 中索引是 0x03 的寄存器是 ecx   
  
//紧接着又给 vm\_off\_18 赋值 ebx 值  
   
//--------------------------------------  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 EDX 的值，所以可以推断出 vm\_off\_08 是 cv 虚拟机的存放 eDX 的地方  
  
//--------------------------------------  
  
// 下一条解析指令 ds:\[00403468\]= 0x00403FFE ，这是已知的，上面分析过  
执行 ：push vm\_off\_08  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 EDX 的值，所以可以推断出 vm\_off\_08 是 cv 虚拟机的存放 eDX 的地方  
  
//--------------------------------------  
  
// 下一条解析指令 ds:\[00403468\]= 0x00403FFE ，这是已知的，上面分析过  
执行 ：push vm\_off\_0c  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 EcX 的值，所以可以推断出 vm\_off\_0c 是 cv 虚拟机的存放 ecx 的地方  
  
/--------------------------------------  
  
// 下一条解析指令 ds:\[00403468\]= 0x00403FFE ，这是已知的，上面分析过  
执行 ：push vm\_off\_10  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F ，这是已知的，上面分析过  
执行 : pop edx  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，这是已知的，上面分析过  
执行： pop \[edx\]  
此时堆栈弹出来的值原 EAX 的值，所以可以推断出 vm\_off\_10 是 cv 虚拟机的存放 eax 的地方  
  
//--------------------------------------  
此时可以整理的 vm\_data 结构是  
  
struct vm\_data  
{  
  DWORD  vm\_ebp;     +0  
  DWORD  vm\_edi;     +4  
  DWORD  vm\_edx;     +8  
  DWORD  vm\_ecx;     +0c  
  DWORD  vm\_eax;     +10  
  DWORD  vm\_esi;     +14  
  DWORD  vm\_ebx;     +18  
  DWORD  vm\_eflag;   +1c  
  DWORD  VM\_off\_20;  
  DWORD  VM\_off\_24;  
  DWORD  VM\_ecx\_value;  
  DWORD  VM\_off\_2c;    
  DWORD  VM\_busy\_flag;  
  DWORD  VM\_off\_34;  
  DWORD  VM\_off\_38;  
  DWORD  VM\_off\_3c;  
  DWORD  VM\_off\_40;  
  DWORD  VM\_off\_44;  
  DWORD  VM\_off\_48;  
  DWORD  VM\_dispatch\_xxx1;  
  DWORD  VM\_dispatch\_xxx2;  
  DWORD  VM\_dispatch\_xxx3;  
  ...  
      
}  
  
cv 虚拟机把宿主的寄存器值分别赋值到自身的 vm\_context 结构的寄存器中  
  
// 下一条解析指令 ds:\[004033E8\]=0040640A    
对比一下执行前后寄存器：  
  
前：   
EDX 00403210 CV\_TES~3.00403210  
EBX 00407022 CV\_TES~3.00407022  
  
后：      
EDX 0013FFC0  
EBX 00407075 CV\_TES~3.00407075  
  
可知该解析例程是完成 mov edx，esp   
  
// 下一条解析指令 ds:\[0040334C\]=0040584D   
  
对寄存器无变化，堆栈中压入了 edx 值  
可知该解析例程是完成 push edx  
  
// 下一条解析指令 ds:\[00403350\]=00406AB1    
  
对寄存器无变化，堆栈中压入了 0x00000004 值  
可知该解析例程是完成 push 00000004 ，但这个数字是依赖 vm\_data 计算出来的，所以该例程  
完成的是 push imm32 功能  
  
// 下一条解析指令 ds:\[004033FC\]=00403E01    
寄存器和堆栈都有变化。  
这个例程有两个操作，pop eax， add\[esp\],eax  
   
   
// 下一条解析指令 ds:\[00403390\]=00405B41    
寄存器和堆栈都有变化  
这个例程完成的指令是 mov esp,\[esp\]  
  
// 下一条解析指令 ds:\[00403350\]=00406AB1,这条上面分析过，是 push imm32 ，imm32 是计算出来的。  
该指令完成 push 0x01234567  
  
// 下一条解析指令 ds:\[004034BC\]=004063C3 ，该例程不是 vm\_dispatch 中的解析指令  
该指令完成压入 403000 进栈，这是 cv 虚拟机进入是保持的 eax 值，所以该功能完成 push vm\_eax   
也就是 t\_buff 的值  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F，前面已经分析过  
该指令完成 pop edx  ，把 t\_buff 值给 edx  
  
  
// 下一条解析指令 ds:\[00403324\]=00405F70 ，前面分析过这  
该指令完成 pop \[edx\] ，将 0x1234567 写入到 403000 地址中  
此时我们加载虚拟机里面的指令，  
  
mov dword ptr\[eax\],01234567h ，已经被处理完毕了。下面是恢复虚拟机堆栈  
  
// 下一条解析指令 ds:\[00403350\]=00406AB1    
push 00401015  ； 401015 地址对应的是原 push 0,也就是加密的语句的下体指令的地址  
所以该解析完成的压入一个退出 cv 虚拟机时的返回地址  
  
// 下一条解析指令 ds:\[00403490\]=004056D9   
add ebx,imm32 (不是完全肯定) nop ，感觉混入的一条无效指令， ebx = ebx + 运算结果中的 eax 所以这条指令不是很肯定  
  
  
// 下一条解析指令 ds:\[00403284\]=00404610    
add esi,6 (不完全肯定） nop ，同样刚进是混入的无效指令  
  
// 注意下面的指令是恢复 vm\_context 中的寄存器到实际运行环境的堆栈中，因为原程序中要加密的指令已经执行完毕了。  
现在要恢复环境了。  
  
// 恢复处理 vm\_eflag  
// 下一条解析指令 ds:\[00403468\]=00403FFE (CV\_TES~3.00403FFE)  
该指令完成 push vm\_eflag  
   
// 下一条解析指令 ds:\[00403374\]=00403F8F   
pop edx  
  
// 下一条解析指令 ds:\[00403450\]=004040BD   
该指令完成 push \[edx\]  
  
// 恢复处理 vm\_eax  
// 下一条解析指令 ds:\[00403468\]=00403FFE   
push vm\_eax\_addr (由此看出 403FFE 是根据 eax 参数信息，获得 vm\_data 中的索引)  
  
// 下一条解析指令 ds:\[00403374\]=00403F8F   
pop edx ,此时 edx 里面存放的是 vm\_context 中 vm\_eax 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD （dis\_no -> 0x94）  
push \[edx\]  
  
   
// 恢复处理 vm\_ecx   
// 下一条解析指令  ds:\[00403468\]=00403FFE    
push vm\_ecx\_addr  
  
// 下一条解析指令  ds:\[00403374\]=00403F8F    
pop edx ，此时 edx 里面存放的是 vm\_context 中 vm\_ecx 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD （dis\_no -> 0x94）  
push \[edx\] , 将 vm\_context 中的 vm\_ecx 压入栈  
  
  
// 恢复处理 vm\_edx  
// 下一条解析指令  ds:\[00403468\]=00403FFE  （dis\_no -> 0x9a）  
push vm\_edx\_addr  
  
// 下一条解析指令  ds:\[00403374\]=00403F8F    
pop edx ，此时 edx 里面存放的是 vm\_context 中 vm\_edx 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD （dis\_no -> 0x94）  
push \[edx\] ,将 vm\_context 中的 vm\_edx 压入栈  
  
// 恢复处理 vm\_bex  
// 下一条解析指令  ds:\[00403468\]=00403FFE    
push vm\_ebx\_addr   
  
// 下一条解析指令  ds:\[00403374\]=00403F8F    
pop edx ,此时 edx 里面存放的是 vm\_context 中 vm\_ebx 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD    
push \[edx\] ,将 vm\_context 中的 vm\_ebx 压入栈  
  
又压入了一遍 vm\_ebx  
// 下一条解析指令  ds:\[00403468\]=00403FFE    
push vm\_ebx\_addr   
  
// 下一条解析指令  ds:\[00403374\]=00403F8F    
pop edx ,此时 edx 里面存放的是 vm\_context 中 vm\_ebx 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD    
push \[edx\] ,将 vm\_context 中的 vm\_ebx 压入栈  
  
// 恢复处理 vm\_ebp  
// 下一条解析指令  ds:\[00403468\]=00403FFE    
push vm\_ebp\_addr   
  
// 下一条解析指令  ds:\[00403374\]=00403F8F    
pop edx ,此时 edx 里面存放的是 vm\_context 中 vm\_ebp 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD    
push \[edx\] ,将 vm\_context 中的 vm\_ebp 压入栈  
  
  
// 恢复处理 vm\_esi  
// 下一条解析指令  ds:\[00403468\]=00403FFE    
push vm\_esi\_addr   
  
// 下一条解析指令  ds:\[00403374\]=00403F8F    
pop edx ,此时 edx 里面存放的是 vm\_context 中 vm\_esi 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD    
push \[edx\] ,将 vm\_context 中的 vm\_esi 压入栈  
  
  
// 恢复处理 vm\_edi  
// 下一条解析指令  ds:\[00403468\]=00403FFE    
push vm\_edi\_addr   
  
// 下一条解析指令  ds:\[00403374\]=00403F8F    
pop edx ,此时 edx 里面存放的是 vm\_context 中 vm\_edi 的地址  
  
// 下一条解析指令  ds:\[00403450\]=004040BD    
push \[edx\] ,将 vm\_context 中的 vm\_edi 压入栈  
  
至此原来的宿主寄存器已经全部恢复完毕。此时 cv 的虚拟机要退出。  
  
// 下一条解析指令  ds:\[004033F8\]=004066AC  ，我们详细分析一下  
  
004066AC     FF77 38         push dword ptr ds:\[edi+38\]   ；vm\_context.busyflag 是否忙的标志  
004066AF     FF3424          push dword ptr ss:\[esp\]  
004066B2     59              pop ecx                      ；该标志给 busyflag  
004066B3     81C4 04000000   add esp,4  
004066B9   ^\\E9 CDF5FFFF     jmp CV\_TES~3.00405C8B  
  
// 混淆的垃圾指令  
00405C8B     57              push edi  
00405C8C     56              push esi  
00405C8D     BE 0113AB7B     mov esi,7BAB1301  
00405C92     017424 04       add dword ptr ss:\[esp+4\],esi  
00405C96     5E              pop esi  
00405C97     8B1424          mov edx,dword ptr ss:\[esp\]  
00405C9A     E9 74030000     jmp CV\_TES~3.00406013  
// 混淆的垃圾指令  
00406013     81C4 04000000   add esp,4  
00406019     81EA 0113AB7B   sub edx,7BAB1301  
0040601F   ^ E9 96E1FFFF     jmp CV\_TES~3.004041BA  
004041BA     09C9            or ecx,ecx  
004041BC     E9 7E050000     jmp CV\_TES~3.0040473F  
0040473F   ^\\0F84 FBF7FFFF   je CV\_TES~3.00403F40  
  
  
00403F40     50              push eax      ；把 eax 保存起来，继续折腾  
00403F41     B8 30000000     mov eax,30  
00403F46     01D0            add eax,edx  
00403F48     56              push esi  
00403F49     BE 53496F70     mov esi,706F4953  
00403F4E     F7D6            not esi  
00403F50     E9 2E2D0000     jmp CV\_TES~3.00406C83  
  
00406C83     81EE ACB6908F   sub esi,8F90B6AC    
00406C89     8930            mov dword ptr ds:\[eax\],esi  
00406C8B     5E              pop esi  
00406C8C     8B0424          mov eax,dword ptr ss:\[esp\]  
00406C8F     83C4 04         add esp,4  
00406C92     61              popad        ；恢复宿主的寄存器  
00406C93     9D              popfd           
00406C94     C3              retn         ；这里从虚拟机返回到宿主  
  
至此 cv 虚拟机将控制权交给宿主程序，加密执行的过程完毕。  
  
最后说一下关于欢迎的一点看法，首先一定要解析出每一个 dispatch 的含义。静态的分析倒也可以，但可能操作起来更麻烦。  
  
不如用虚拟机对付虚拟机，大致看 cv 的加密方式后，更多的是指令的混淆和单纯的执行，这样我们实现一个小的堆栈虚拟机来仿真执行一下即可，在 dispatch 入口处断下，每一个 dispatch 含义我们都知道这样直接还原为我们知道的 x86 指令反而更好，当然这只是一个想法不知是否可行，毕竟分析 vm 这东西很是麻烦的，未知的东西太多。

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || cv\_test\_1.38.rar  
 |