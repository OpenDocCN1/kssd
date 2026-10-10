

# 运用 SMC 打补 Ace ScreenSaver 2.21

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-99.htm`](https://www.pediy.com/kssd/pediy07/pediy7-99.htm)

【文章标题】运用 SMC 打补 Ace ScreenSaver 2.21  
【文章作者】WR-911  
【作者邮箱】WR-911@sohu.com  
【软件名称】Ace ScreenSaver 2.21  
【软件限制】15 天时间限制  
【破解平台】Windows XP  
【保护方式】加壳,注册码  
【开发语言】Borland C++  
【本文写于】2005-2-17  
—————————————————————————————————  
【破解分析】  
  
破解分析  
    到程序目录里用 PEiD 查程序 AceLaunch.exe（注意才 19.5KB），发现无壳（作者的压缩功力可真深？），用 Ollydbg 载入后运行时发现 AceLaunch.exe 进程终止了，但程序却运行了！原来 AceLaunch.exe 只是一个 Loader（可能是为了防止内存补丁），它负责执行一些功能，然后通过 GetSystemDirectoryA 定位系统目录，调用 CreateProcessA 加命令的调用 C:\\windows\\system32\\AceSaver.scr（这个才是主程序）。好现在把 C:\\windows\\system32\\AceSaver.scr 复制到程序安装目录里并重命名为 AceSaver.exe，再用 PEiD 查看发现加的是 ASPack 2.1 -> Alexey Solodovnikov（老壳~好说好说 :-） ，但用 PEid 的脱壳器、AspackDie V1.41 汉化版、ASPRStripper V2.07 Final 都不能正常脱壳并运行~！我狂晕~~~难道是自动脱壳机的版本太新了？！  
    事到如今只能单干了。我们先把时间调到一个月以后，执行 AceLaunch.exe，程序提示过期要注册。先把时间调回来再用 OD 的"附加"的方式载入程序，在"Command"中输入"BP DestroyWindow"，最后按下 "取消"，程序中断在 OD 中，运用 Ctrl+F9 和 F8 慢慢走回到目标程序的天空，如下：  
  
004ED5EF     53                 push ebx  
004ED5F0     E8 2F9DF1FF        call acesaver.00407324                       ; jmp to kernel32.ReleaseMutex  
004ED5F5     46                 inc esi  
004ED5F6     75 67              jnz short acesaver.004ED65F                  <==========①重点记住  
004ED5F8     6A 21              push 21  
004ED5FA     68 3CD74E00        push acesaver.004ED73C                       ; ASCII "Ace ScreenSaver"  
004ED5FF     A1 4C9F4F00        mov eax,dword ptr ds:\[4F9F4C\]  
004ED604     E8 536CF1FF        call acesaver.0040425C  
004ED609     50                 push eax  
004ED60A     6A 00              push 0  
004ED60C     E8 E3A3F1FF        call acesaver.004079F4                       ; jmp to user32.MessageBoxA  
004ED611     83F8 02            cmp eax,2                                               <==========我们停在这，这里是判断你按下的是"确定" 还是 "取消"，向上我们可以看到  
004ED5F6     75 67              jnz short acesaver.004ED65F 这里判断是否过期？过期则运行过期让你注册的窗口。那现在还不快下断点在 004ED5F6 处，然后重新运行程序...然后断在这里...然后改掉 FL （标志为的 Z）...然后程序正常运行。错！那么简单就不用写这"破文"了！别忘了程序有壳，你又没本事脱掉！别急，慢慢来，先退出 OD。  
  
再次执行 AceLaunch.exe，程序提示过期要注册，那我们就来注册！填好名字和注册码，按下"确定"，弹出注册码错误窗口。用 OD 的"附加"的方式载入程序，在"Command"中输入"BP DestroyWindow"，最后按下 "确定"，程序中断在 OD 中，运用 Ctrl+F9 和 F8 慢慢走回到目标程序的天空，如下：  
004ED614     0F84 D2000000      je acesaver.004ED6EC  
004ED61A     33C9               xor ecx,ecx  
004ED61C     B2 01              mov dl,1  
004ED61E     A1 2C764E00        mov eax,dword ptr ds:\[4E762C\]  
  
004E79B4     8B45 F8            mov eax,dword ptr ss:\[ebp-8\]  
004E79B7     5A                 pop edx  
004E79B8     E8 37C2FEFF        call acesaver.004D3BF4  
004E79BD     84C0               test al,al  
004E79BF     74 65              je short acesaver.004E7A26                      <==========②重点记住  
004E79C1     8D55 F4            lea edx,dword ptr ss:\[ebp-C\]  
004E79C4     8B83 E0020000      mov eax,dword ptr ds:\[ebx+2E0\]  
004E79CA     E8 E1A9F4FF        call acesaver.004323B0  
004E79CF     8B45 F4            mov eax,dword ptr ss:\[ebp-C\]  
004E79D2     50                 push eax  
004E79D3     8D55 F0            lea edx,dword ptr ss:\[ebp-10\]  
004E79D6     8B83 DC020000      mov eax,dword ptr ds:\[ebx+2DC\]  
004E79DC     E8 CFA9F4FF        call acesaver.004323B0  
004E79E1     8B45 F0            mov eax,dword ptr ss:\[ebp-10\]  
004E79E4     5A                 pop edx  
004E79E5     E8 CEC0FEFF        call acesaver.004D3AB8  
004E79EA     6A 40              push 40  
004E79EC     68 8C7A4E00        push acesaver.004E7A8C                       ; ASCII "Ace ScreenSaver"  
004E79F1     8D4D EC            lea ecx,dword ptr ss:\[ebp-14\]  
004E79F4     BA A47A4E00        mov edx,acesaver.004E7AA4                    ; ASCII "RegisterForm.str\_0"  
004E79F9     8B83 00030000      mov eax,dword ptr ds:\[ebx+300\]  
004E79FF     E8 6C5FF7FF        call acesaver.0045D970  
004E7A04     8B45 EC            mov eax,dword ptr ss:\[ebp-14\]  
004E7A07     E8 50C8F1FF        call acesaver.0040425C  
004E7A0C     50                 push eax  
004E7A0D     8BC3               mov eax,ebx  
004E7A0F     E8 300DF5FF        call acesaver.00438744  
004E7A14     50                 push eax  
004E7A15     E8 DAFFF1FF        call acesaver.004079F4                       ; jmp to user32.MessageBoxA  
004E7A1A     C783 34020000 0100>mov dword ptr ds:\[ebx+234\],1  
004E7A24     EB 30              jmp short acesaver.004E7A56  
004E7A26     6A 40              push 40  
004E7A28     68 8C7A4E00        push acesaver.004E7A8C                       ; ASCII "Ace ScreenSaver"  
004E7A2D     8D4D E8            lea ecx,dword ptr ss:\[ebp-18\]  
004E7A30     BA C07A4E00        mov edx,acesaver.004E7AC0                    ; ASCII "RegisterForm.str\_1"  
004E7A35     8B83 00030000      mov eax,dword ptr ds:\[ebx+300\]  
004E7A3B     E8 305FF7FF        call acesaver.0045D970  
004E7A40     8B45 E8            mov eax,dword ptr ss:\[ebp-18\]  
004E7A43     E8 14C8F1FF        call acesaver.0040425C  
004E7A48     50                 push eax  
004E7A49     8BC3               mov eax,ebx  
004E7A4B     E8 F40CF5FF        call acesaver.00438744  
004E7A50     50                 push eax  
004E7A51     E8 9EFFF1FF        call acesaver.004079F4                       ; jmp to user32.MessageBoxA  
004E7A56     33C0               xor eax,eax                                            <=========我们停在这，往上看可以看到 004E79BF     74 65              je short acesaver.004E7A26 这里判断注册码是否正确，在这行下断点，再 BC DestroyWindow 清除断点。再次注册，就停在我们想要的地方了，试试将 je 改为 jmp ，F9 运行程序，注册成功^\_^!，程序正常进入了。但我一看到帮助下的"注册"就禁不住要去按一下，怎么又要注册了？刚刚是爆破，注册码不对当然还要注册啦！那我就让"注册"按扭失效，免得你手闲！  
    让程序已经正常进入，下断②，按下"帮助"--->"注册"，填好名字和注册码，按下"确定"，改标志后，一直按 F8（你也可以 F9 与 F8 结合起来按）  
  
004EB469     B2 01              mov dl,1  
004EB46B     A1 2C764E00        mov eax,dword ptr ds:\[4E762C\]  
004EB470     E8 2FF0F5FF        call acesaver.0044A4A4  
004EB475     8B15 D4164F00      mov edx,dword ptr ds:\[4F16D4\]  
004EB47B     8902               mov dword ptr ds:\[edx\],eax  
004EB47D     A1 D4164F00        mov eax,dword ptr ds:\[4F16D4\]  
004EB482     8B00               mov eax,dword ptr ds:\[eax\]  
004EB484     8B10               mov edx,dword ptr ds:\[eax\]  
004EB486     FF92 D8000000      call dword ptr ds:\[edx+D8\]  
004EB48C     A1 D4164F00        mov eax,dword ptr ds:\[4F16D4\]                <========③重点记住，光标返回到这里就该停了  
004EB491     8B00               mov eax,dword ptr ds:\[eax\]  
004EB493     E8 347CF1FF        call acesaver.004030CC  
004EB498     E8 5388FEFF        call acesaver.004D3CF0  
004EB49D     84C0               test al,al  
  
我们试着在 004EB482 下个断点，再按下"帮助"--->"注册"，注册的窗口还没打开就马上就被 OD 停住了（看来找对了），把 004EB486     FF92 D8000000      call dword ptr ds:\[edx+D8\]  "使用 NOP 填充"，一切就都搞定了。  
  
具备以上 3 个破解点，我们就要开始用 SMC 补丁程序了。但首先呢，我们还要找到（加壳）程序的在内存中解压完后的入口 OEP，又该 OD 上场了（说实话我并不太喜欢 OD，还是 TRW 和 S-ICE 好呀！可惜操作系统变了，而且 OD 各项功能确实强！）用 OD 打开 AceSaver.exe，不选分析代码，我们停在  
00538000     90                 nop  
00538001 A>  60                 pushad          <=======停在这  
00538002     E8 72050000        call AceSaver.00538579  
00538007     EB 33              jmp short AceSaver.0053803C  
00538009     87DB               xchg ebx,ebx  
0053800B     90                 nop  
0053800C     00D0               add al,dl  
0053800E     4F                 dec edi  
0053800F     0010               add byte ptr ds:\[eax\],dl  
00538011     D04F 00            ror byte ptr ds:\[edi\],1  
00538014     D4 24              aam 24  
00538016     4F                 dec edi  
00538017     0010               add byte ptr ds:\[eax\],dl  
00538019     E0 4F              loopdne short AceSaver.0053806A  
0053801B     0000               add byte ptr ds:\[eax\],al  
0053801D     0000               add byte ptr ds:\[eax\],al  
0053801F     0000               add byte ptr ds:\[eax\],al  
00538021     0000               add byte ptr ds:\[eax\],al  
00538023     0000               add byte ptr ds:\[eax\],al  
00538025     8013 00            adc byte ptr ds:\[ebx\],0  
00538028     0000               add byte ptr ds:\[eax\],al  
0053802A     40                 inc eax  
0053802B     0000               add byte ptr ds:\[eax\],al  
0053802D     A0 0F000000        mov al,byte ptr ds:\[F\]  
00538032     0000               add byte ptr ds:\[eax\],al  
00538034     0000               add byte ptr ds:\[eax\],al  
00538036     0000               add byte ptr ds:\[eax\],al  
00538038     00F0               add al,dh  
0053803A     0F00         ???                                          ; 未知命令  
0053803C     BB 3C394400        mov ebx,AceSaver.0044393C  
00538041     03DD               add ebx,ebp  
00538043     2B9D 60394400      sub ebx,dword ptr ss:\[ebp+443960\]  
00538049     83BD 98474400 00   cmp dword ptr ss:\[ebp+444798\],0  
00538050     899D 98474400      mov dword ptr ss:\[ebp+444798\],ebx  
00538056     0F85 81040000      jnz AceSaver.005384DD  
0053805C     8D85 A0474400      lea eax,dword ptr ss:\[ebp+4447A0\]  
00538062     50                 push eax  
00538063     FF95 AC484400      call dword ptr ss:\[ebp+4448AC\]  
00538069     8985 9C474400      mov dword ptr ss:\[ebp+44479C\],eax  
0053806F     8BF8               mov edi,eax  
00538071     8D9D AD474400      lea ebx,dword ptr ss:\[ebp+4447AD\]  
00538077     53                 push ebx  
00538078     50                 push eax  
00538079     FF95 A8484400      call dword ptr ss:\[ebp+4448A8\]  
0053807F     8985 F5394400      mov dword ptr ss:\[ebp+4439F5\],eax  
00538085     8D9D BA474400      lea ebx,dword ptr ss:\[ebp+4447BA\]  
0053808B     53                 push ebx  
0053808C     57                 push edi  
0053808D     FF95 A8484400      call dword ptr ss:\[ebp+4448A8\]  
00538093     8985 F9394400      mov dword ptr ss:\[ebp+4439F9\],eax  
00538099     8D85 0A3B4400      lea eax,dword ptr ss:\[ebp+443B0A\]  
0053809F     FFE0               jmp eax  
  
很明显是 ASPack 加的壳，按下“Ctrl + S”调出 “查找命令序列”输入    mov eax,1  
retn 0C  
第一次找到的地方如下：  
005384ED     8985 3B3E4400      mov dword ptr ss:\[ebp+443E3B\],eax  
005384F3     61                 popad  
005384F4     75 08              jnz short AceSaver.005384FE  
005384F6     B8 01000000        mov eax,1  
005384FB     C2 0C00            retn 0C      <======光标停在这  
005384FE     68 00000000        push 0       <======这里在程序正常解压完毕后压入 OEP （而不是现在看到的 0）  
00538503     C3                 retn         <======相当于 JMP OEP  
  
因此，在 005384FE 下断点，Ctrl + F2（重载程序）然后 F9（运行），被拦下 005384F6     B8 01000000        mov eax,1  
005384FB     C2 0C00            retn 0C  
005384FE     68 F0D14E00        push AceSaver.004ED1F0  <====停在这，这才是真正的 OEP = 004ED1F0  
00538503     C3                 retn  
  
\*\*\*\*\*我们要先找一块全 0 区域的任意位置把要修改的代码写进去，我找的是 00539F01 做如下修改  
00538503     E9F9190000                   jmp AceSaver.00539F01  
                                 <==========跳到要执行修补的地方  
00539F00     36C605F6D54E0074           mov byte ptr ss:\[4ED5F6\],74  
                                 <==========对应 ①  
00539F08     C70586B44E0090909090        mov dword ptr ds:\[4EB486\],90909090             <==========对应 ③  
00539F12     66C7058AB44E009090         mov word ptr ds:\[4EB48A\],9090                 <==========对应 ③  
00539F1B     66C705BF794E009090         mov word ptr ds:\[4E79BF\],9090                 <==========对应 ②  
00539F24   - E9C732FBFF                   jmp AceSaver.004ED1F0  
                                 <==========跳回程序入口  
  
   按下 F9，运行试试看，出个窗口“Missing Config Files”？？？ 我都说过了嘛！AceLaunch.exe 是一个 Loader，它负责执行一些功能(也可以 SMC 跳过检测 ConfigFiles 和检测另一个文件的地方，程序可以启动但不太正常。)，退出 OD。现在找你顺手的工具如：WinHEX、UltraEdit、C32Asm 等进行文件修改  
  我用的是 C32Asm，用 C32Asm 载入 AceSaver.exe “Ctrl + G”输入 00538503 在这一行右击鼠标选择 “对应 HEX 编辑” 输入“E9F9190000” 保存后跳回汇编代码窗 再“Ctrl + G”输入 00539F00 在这一行右击鼠标选择 “对应 HEX 编辑” 输入 “36C605F6D54E0074C70586B44E009090909066C7058AB44E00909066C705BF794E009090E9C732FBFF” 千万要仔细哦！保存完后将 SMC 修补好的 AceSaver.exe 重命名 AceSaver.scr 并复制到 system32 中，运行 AceLaunch.exe，一切都按我们的计划进行啦！  
  
  
—————————————————————————————————  
【总结】  
其实要脱这个软件的壳也不难 Pe-Scan3.31 或手脱（脱出来的文件很大，俗话说的好“原配的才是最好的嘛！”）。最可惜的是没完全搞懂 AceLaunch.exe 这个 Loader 的具体作用，本想将两个文件（把 AceLaunch 的功能）与 AceSaver 合并为一个文件，且不必复制到 System32 目录中运行的。遗憾呀！  
  
—————————————————————————————————  
【版权说明】Cracked By WR-911.     
         Enjoy It !!!   凸-\_-凸  
  
本文由 <流行时代破文写手> 生成  
http://www.popbase.net 

 4dd466d0e1e0cc123afbe504da2462ca