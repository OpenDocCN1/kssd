

# EXE ReFactor 0.2 详细分析(一)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-591.htm`](https://www.pediy.com/kssd/pediy07/pediy7-591.htm)

一. 壳的特点  
  
这个壳是我见过的比较有特点的壳, 很多地方值的我学习,  希望作者多交流.  
当然这个壳有些代码效率有点低, 把一些数据在 heap 里搬来搬去, 可以改进一下.  
  
第一个特点是原程序的 Section 完全消失, Loader 动态创建并解压, 给 Dump 带来一点麻烦, 可能需要手动组装 PE  
  
第二个特点是所有的 API 地址在内存中都是反的, 调用前先 NOT  
  
第三个特点是用全局变量保存 ESP,  JMP 都是用 Call 实现的.  
用这种方式来实现花指令倒是第一次见到.  
  
053BC006    8925 CFEF3B05   MOV DWORD PTR DS:\[53BEFCF\],ESP  
053BC00C    E8 F4350000     CALL EXERe.053BF605  
053BF605    BE 80963B05     MOV ESI,EXERe.053B9680  
053BF60A    E8 BAFEFFFF     CALL EXERe.053BF4C9  
053BF4C9    8B25 CFEF3B05   MOV ESP,DWORD PTR DS:\[53BEFCF\]   // 恢复 ESP  
  
第四个特点是壳会把 API 整个搬到 Heap 空间, Stole 非常厉害,  为了实现这个目的, 作者花了很多工夫, 导致程序启动有点慢.  
  
第五个特点是 IAT 加密方式也比较特别.   
  
第六个特点是壳的字符串加密方法  
  
我们先写一段程序, 解密 53B0554 - 53B0870 的密文, 熟悉一下壳所用的加解密字符串方法, 以后就不再描述了.  
  
053A34EF >    60            PUSHAD  
053A34F0      BE 54053B05   MOV ESI,EXERe.053B0554  
053A34F5      33C0          XOR EAX,EAX  
053A34F7      66:8B06       MOV AX,WORD PTR DS:\[ESI\]  
053A34FA      85C0          TEST EAX,EAX  
053A34FC      74 0E         JE SHORT EXERe.053A350C  
053A34FE      66:25 0F0F    AND AX,0F0F                    // AH, AL 的 高 4 位清零  
053A3502      C0E4 04       SHL AH,4  
053A3505      08E0          OR AL,AH                       // 把 AH, AL 的低 4 位组合在一起  
053A3507      32E4          XOR AH,AH  
053A3509      66:8906       MOV WORD PTR DS:\[ESI\],AX  
053A350C      83C6 02       ADD ESI,2  
053A350F      81FE 70083B05 CMP ESI,EXERe.053B0870  
053A3515    ^ 7C E0         JL SHORT EXERe.053A34F7  
053A3517      61            POPAD  
  
  
60 BE 54 05 3B 05 33 C0 66 8B 06 85 C0 74 0E 66 25 0F 0F C0 E4 04 08 E0 32 E4 66 89 06 83 C6 02  
81 FE 70 08 3B 05 7C E0 61  
  
  
明文, 看看壳所用的 API,  还有 6 个 Section Name  
  
053B0554  57 00 69 00 64 00 65 00 43 00 68 00 61 00 72 00  W.i.d.e.C.h.a.r.  
053B0564  54 00 6F 00 4D 00 75 00 6C 00 74 00 69 00 42 00  T.o.M.u.l.t.i.B.  
053B0574  79 00 74 00 65 00 00 00 48 00 65 00 61 00 70 00  y.t.e...H.e.a.p.  
053B0584  41 00 6C 00 6C 00 6F 00 63 00 00 00 48 00 65 00  A.l.l.o.c...H.e.  
053B0594  61 00 70 00 46 00 72 00 65 00 65 00 00 00 00 00  a.p.F.r.e.e.....  
053B05A4  53 00 6C 00 65 00 65 00 70 00 00 00 43 00 6C 00  S.l.e.e.p...C.l.  
053B05B4  6F 00 73 00 65 00 48 00 61 00 6E 00 64 00 6C 00  o.s.e.H.a.n.d.l.  
053B05C4  65 00 00 00 4C 00 6F 00 61 00 64 00 4C 00 69 00  e...L.o.a.d.L.i.  
053B05D4  62 00 72 00 61 00 72 00 79 00 41 00 00 00 00 00  b.r.a.r.y.A.....  
053B05E4  52 00 65 00 61 00 64 00 46 00 69 00 6C 00 65 00  R.e.a.d.F.i.l.e.  
053B05F4  00 00 00 00 53 00 65 00 74 00 46 00 69 00 6C 00  ....S.e.t.F.i.l.  
053B0604  65 00 50 00 6F 00 69 00 6E 00 74 00 65 00 72 00  e.P.o.i.n.t.e.r.  
053B0614  00 00 00 00 43 00 72 00 65 00 61 00 74 00 65 00  ....C.r.e.a.t.e.  
053B0624  46 00 69 00 6C 00 65 00 57 00 00 00 43 00 72 00  F.i.l.e.W...C.r.  
053B0634  65 00 61 00 74 00 65 00 54 00 68 00 72 00 65 00  e.a.t.e.T.h.r.e.  
053B0644  61 00 64 00 00 00 00 00 56 00 69 00 72 00 74 00  a.d.....V.i.r.t.  
053B0654  75 00 61 00 6C 00 46 00 72 00 65 00 65 00 00 00  u.a.l.F.r.e.e...  
053B0664  56 00 69 00 72 00 74 00 75 00 61 00 6C 00 41 00  V.i.r.t.u.a.l.A.  
053B0674  6C 00 6C 00 6F 00 63 00 00 00 00 00 56 00 69 00  l.l.o.c.....V.i.  
053B0684  72 00 74 00 75 00 61 00 6C 00 50 00 72 00 6F 00  r.t.u.a.l.P.r.o.  
053B0694  74 00 65 00 63 00 74 00 00 00 00 00 52 00 65 00  t.e.c.t.....R.e.  
053B06A4  61 00 64 00 50 00 72 00 6F 00 63 00 65 00 73 00  a.d.P.r.o.c.e.s.  
053B06B4  73 00 4D 00 65 00 6D 00 6F 00 72 00 79 00 00 00  s.M.e.m.o.r.y...  
053B06C4  57 00 72 00 69 00 74 00 65 00 50 00 72 00 6F 00  W.r.i.t.e.P.r.o.  
053B06D4  63 00 65 00 73 00 73 00 4D 00 65 00 6D 00 6F 00  c.e.s.s.M.e.m.o.  
053B06E4  72 00 79 00 00 00 00 00 4F 00 70 00 65 00 6E 00  r.y.....O.p.e.n.  
053B06F4  50 00 72 00 6F 00 63 00 65 00 73 00 73 00 00 00  P.r.o.c.e.s.s...  
053B0704  47 00 65 00 74 00 4D 00 6F 00 64 00 75 00 6C 00  G.e.t.M.o.d.u.l.  
053B0714  65 00 48 00 61 00 6E 00 64 00 6C 00 65 00 41 00  e.H.a.n.d.l.e.A.  
053B0724  00 00 00 00 47 00 65 00 74 00 43 00 75 00 72 00  ....G.e.t.C.u.r.  
053B0734  72 00 65 00 6E 00 74 00 50 00 72 00 6F 00 63 00  r.e.n.t.P.r.o.c.  
053B0744  65 00 73 00 73 00 49 00 64 00 00 00 47 00 65 00  e.s.s.I.d...G.e.  
053B0754  74 00 50 00 72 00 6F 00 63 00 41 00 64 00 64 00  t.P.r.o.c.A.d.d.  
053B0764  72 00 65 00 73 00 73 00 00 00 00 00 47 00 65 00  r.e.s.s.....G.e.  
053B0774  74 00 54 00 69 00 63 00 6B 00 43 00 6F 00 75 00  t.T.i.c.k.C.o.u.  
053B0784  6E 00 74 00 00 00 00 00 47 00 65 00 74 00 50 00  n.t.....G.e.t.P.  
053B0794  72 00 6F 00 63 00 65 00 73 00 73 00 48 00 65 00  r.o.c.e.s.s.H.e.  
053B07A4  61 00 70 00 00 00 00 00 49 00 73 00 42 00 61 00  a.p.....I.s.B.a.  
053B07B4  64 00 52 00 65 00 61 00 64 00 50 00 74 00 72 00  d.R.e.a.d.P.t.r.  
053B07C4  00 00 00 00 2E 00 74 00 65 00 78 00 74 00 00 00  ......t.e.x.t...  
053B07D4  65 00 63 00 6F 00 64 00 65 00 00 00 70 00 61 00  e.c.o.d.e...p.a.  
053B07E4  67 00 65 00 00 00 00 00 2E 00 72 00 73 00 72 00  g.e.......r.s.r.  
053B07F4  63 00 00 00 2E 00 74 00 6C 00 73 00 00 00 00 00  c.....t.l.s.....  
053B0804  2E 00 6F 00 72 00 70 00 63 00 00 00 52 00 65 00  ..o.r.p.c...R.e.  
053B0814  73 00 75 00 6D 00 65 00 54 00 68 00 72 00 65 00  s.u.m.e.T.h.r.e.  
053B0824  61 00 64 00 00 00 00 00 54 00 65 00 72 00 6D 00  a.d.....T.e.r.m.  
053B0834  69 00 6E 00 61 00 74 00 65 00 50 00 72 00 6F 00  i.n.a.t.e.P.r.o.  
053B0844  63 00 65 00 73 00 73 00 00 00 00 00 4D 00 65 00  c.e.s.s.....M.e.  
053B0854  73 00 73 00 61 00 67 00 65 00 42 00 6F 00 78 00  s.s.a.g.e.B.o.x.  
053B0864  45 00 78 00 57 00 00 00 00 00 00 00 00 00 00 00  E.x.W.  
  
  
  
  
二.  
重新来过  
  
053A34EF > $  55            PUSH EBP  
053A34F0   .  8BEC          MOV EBP,ESP  
053A34F2   .  81EC 7C0B0000 SUB ESP,0B7C  
053A34F8   .  53            PUSH EBX  
053A34F9   .  56            PUSH ESI  
053A34FA   .  57            PUSH EDI  
053A34FB   .- E9 008B0100   JMP EXERe.053BC000  
  
  
  
053BC000    FF15 00103A05   CALL DWORD PTR DS:\[<&KERNEL32.GetCommandLine>  
053BF605    BE 80963B05     MOV ESI,EXERe.053B9680                       ; 基址  
053BF4CF    6A FF           PUSH -1  
053BF517    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680( ECX 也是 Proc 53A67D0 的一个参数)   
053BF505    FF15 E3F43B05   CALL DWORD PTR DS:\[53BF4E3\]                  ; EXERe.053A67D0  
  
  
  
proc 53A67D0(arg)   获取 DLL, EXE 的 Section 信息  
  
ECX = buffer  
arg = -1               处理 kernel32.dll  
arg = -2               处理 EXE ImageBase  
arg = -3               处理 ntdll.dll  
arg = xxxxxxxx(DLL BaseAddress)  
  
对这个 DLL 的每个 Section 判断是不是下面的 SectionName  
  
.text   code, 累加 Law Size  
ecode   code, 累加 Law Size  
page    code, 累加 Law Size  
.rsrc   做标志  
.tls    做标志  
.orpc   code, 累加 Law Size  
  
最后的结果  
  
以 arg = -1 为例  
  
053BA050  04 00 00 00    Section Num  
053BA054  02 00 00 00    .rsrc 的序号(第三个区段)  
053BA058  00 00 00 00    .tls  的序号(没有)  
053BA05C  00 90 05 00    code 段 lawsize  
053BA060  FF FF A8 83    Not BaseAddress  
  
  
  
  
三.    
回到主程序  
  
053BF4EE    68 7C053B05     PUSH EXERe.053B057C  
053BF5AD    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF5BB    FF15 DEF43B05   CALL DWORD PTR DS:\[53BF4DE\]                  ; EXERe.053A7017  
  
  
proc 53A7017(arg)   取 API　address 的一个函数  
  
arg = 加密的 API Name  
ret = Not API Address  
  
053A7017  /$  55            PUSH EBP  
053A7018  |.  8BEC          MOV EBP,ESP  
053A701A  |.  81EC 00020000 SUB ESP,200  
053A7020  |.  56            PUSH ESI  
053A7021  |.  8BF1          MOV ESI,ECX  
053A7023  |.  57            PUSH EDI  
053A7024  |.  B9 80000000   MOV ECX,80  
053A7029  |.  33C0          XOR EAX,EAX  
053A702B  |.  8DBD 00FEFFFF LEA EDI,DWORD PTR SS:\[EBP-200\]                    // zero buffer  
053A7031  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
053A7033  |.  8B4D 08       MOV ECX,DWORD PTR SS:\[EBP+8\]                      // arg  
053A7036  |.  85C9          TEST ECX,ECX  
053A7038  |.  75 25         JNZ SHORT EXERefac.053A705F  
053A703A  |>  50            PUSH EAX                                          ; /Arg3 长度  
053A703B  |.  8D85 00FEFFFF LEA EAX,DWORD PTR SS:\[EBP-200\]                    ; |  
053A7041  |.  51            PUSH ECX                                          ; |Arg2 密文  
053A7042  |.  50            PUSH EAX                                          ; |Arg1 buffer  
053A7043  |.  E8 B4FCFFFF   CALL EXERefac.053A6CFC                            ; \\EXERefac.053A6CFC, 把密文复制到 buffer  
053A7048  |.  83C4 0C       ADD ESP,0C  
053A704B  |.  8D85 00FEFFFF LEA EAX,DWORD PTR SS:\[EBP-200\]  
053A7051  |.  8BCE          MOV ECX,ESI  
053A7053  |.  50            PUSH EAX                                          ; /Arg1  
053A7054  |.  E8 2AFDFFFF   CALL EXERefac.053A6D83                            ; \\EXERefac.053A6D83, 返回 Not GetProcAddress  
053A7059  |.  5F            POP EDI  
053A705A  |.  5E            POP ESI  
053A705B  |.  C9            LEAVE  
053A705C  |.  C2 0400       RETN 4  
053A705F  |>  33C0          XOR EAX,EAX  
053A7061  |>  803C08 00     CMP BYTE PTR DS:\[EAX+ECX\],0                       // 确定密文长度  
053A7065  |.^ 74 D3         JE SHORT EXERefac.053A703A                        //  
053A7067  |.  40            INC EAX                                           //  
053A7068  \\.^ EB F7         JMP SHORT EXERefac.053A7061                       //  
  
  
其中 53A6D83 也特别复杂,  递归调用, 不跟了.  
  
  
  
四.  
接下来一段花指令调用  53A7017, 求出壳要用的 API   
  
053BF5EA    68 90053B05     PUSH EXERe.053B0590  
053BF5FD    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF5CF    A3 B4A03B05     MOV DWORD PTR DS:\[53BA0B4\],EAX              // Not HeapAlloc  
053BF59B    FF15 DEF53B05   CALL DWORD PTR DS:\[53BF5DE\]                  ; EXERe.053A7017  
053BF535    68 8C073B05     PUSH EXERe.053B078C  
053BF527    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF580    A3 90A03B05     MOV DWORD PTR DS:\[53BA090\],EAX              // Not HeapFree  
053BF56E    FF15 D9F53B05   CALL DWORD PTR DS:\[53BF5D9\]                  ; EXERe.053A7017  
053BF58A    A3 80A03B05     MOV DWORD PTR DS:\[53BA080\],EAX              // Not GetProcessHeap  
053BF51F    F7D0            NOT EAX  
053BF55A    FFD0            CALL EAX                                    // KERNEL32.GetProcessHeap  
053BF549    A3 D4A03B05     MOV DWORD PTR DS:\[53BA0D4\],EAX              // Heap=130000  
053BF3D1    FF15 17F43B05   CALL DWORD PTR DS:\[53BF417\]                 // EXERe.053A54A5, (?????, F8)  
  
053BF3E5    33FF            XOR EDI,EDI  
053BF403    68 04073B05     PUSH EXERe.053B0704  
053BF3F5    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF3EC    897D 90         MOV DWORD PTR SS:\[EBP-70\],EDI  
053BF494    897D 94         MOV DWORD PTR SS:\[EBP-6C\],EDI  
053BF4B9    897D 98         MOV DWORD PTR SS:\[EBP-68\],EDI  
053BF4AD    897D E0         MOV DWORD PTR SS:\[EBP-20\],EDI  
053BF478    FF15 C4F43B05   CALL DWORD PTR DS:\[53BF4C4\]                  ; EXERe.053A7017  
053BF464    6A FE           PUSH -2  
053BF49F    B9 908C3B05     MOV ECX,EXERe.053B8C90  
053BF489    A3 7CA03B05     MOV DWORD PTR DS:\[53BA07C\],EAX               // Not GetModuleHandleA  
053BF428    FF15 1DF43B05   CALL DWORD PTR DS:\[53BF41D\]                  // EXERe.053A67D0  (arg = -2)  获取主程序的区块信息  
053BF44D    68 A0063B05     PUSH EXERe.053B06A0  
053BF43C    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF11C    FF15 48F03B05   CALL DWORD PTR DS:\[53BF048\]                  ; EXERe.053A7017  
053BF028    68 28073B05     PUSH EXERe.053B0728  
053BF040    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BEFED    A3 88A03B05     MOV DWORD PTR DS:\[53BA088\],EAX               // Not ReadProcessMemory  
053BF010    FF15 3BF03B05   CALL DWORD PTR DS:\[53BF03B\]                  ; EXERe.053A7017  
053BEFDC    68 EC063B05     PUSH EXERe.053B06EC  
053BF002    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BEFF7    A3 70A03B05     MOV DWORD PTR DS:\[53BA070\],EAX               // Not GetCurrentProcessId  
053BF095    FF15 51F03B05   CALL DWORD PTR DS:\[53BF051\]                  ; EXERe.053A7017  
053BF066    A3 BCA03B05     MOV DWORD PTR DS:\[53BA0BC\],EAX               // Not OpenProcess  
053BF0E4    FFD0            CALL EAX                                     // KERNEL32.GetCurrentProcessId  
053BF0FA    50              PUSH EAX  
053BF0AF    57              PUSH EDI  
053BF0C7    68 FF0F1F00     PUSH 1F0FFF  
053BF33C    FFD0            CALL EAX                                     // KERNEL32.OpenProcess  
053BF056    3BC7            CMP EAX,EDI  
053BF3C0    A3 D8A03B05     MOV DWORD PTR DS:\[53BA0D8\],EAX               // 保存 hProcess   
053BF36C   /0F84 1A1F0000   JE EXERe.053C128C                            // OpenProcess Error 则 Over  
053BF355    68 80063B05     PUSH EXERe.053B0680  
053BF394    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF3A4    FF15 67F33B05   CALL DWORD PTR DS:\[53BF367\]                  ; EXERe.053A7017  
053BF380    68 C4063B05     PUSH EXERe.053B06C4  
053BF3B8    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF313    A3 78A03B05     MOV DWORD PTR DS:\[53BA078\],EAX               // Not VirtualProtect     
053BF324    FF15 4AF33B05   CALL DWORD PTR DS:\[53BF34A\]                  ; EXERe.053A7017  
053BF1E2    68 E4053B05     PUSH EXERe.053B05E4  
053BF1BC    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF1D1    A3 74A03B05     MOV DWORD PTR DS:\[53BA074\],EAX               // Not WriteProcessMemory  
053BF1AA    FF15 06F13B05   CALL DWORD PTR DS:\[53BF106\]                  ; EXERe.053A7017  
053BF144    68 F8053B05     PUSH EXERe.053B05F8  
053BF1C6    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF130    A3 B8A03B05     MOV DWORD PTR DS:\[53BA0B8\],EAX               // Not ReadFile  
053BF183    FF15 D9F03B05   CALL DWORD PTR DS:\[53BF0D9\]                  ; EXERe.053A7017  
053BF15D    68 B0053B05     PUSH EXERe.053B05B0  
053BF19C    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF2D5    A3 A4A03B05     MOV DWORD PTR DS:\[53BA0A4\],EAX               // Not SetFilePointer  
053BF2EE    FF15 BBF03B05   CALL DWORD PTR DS:\[53BF0BB\]                  ; EXERe.053A7017  
053BF288    68 C8053B05     PUSH EXERe.053B05C8  
053BF305    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF172    A3 C4A03B05     MOV DWORD PTR DS:\[53BA0C4\],EAX               // Not CloseHandle  
053BF2A1    FF15 0EF33B05   CALL DWORD PTR DS:\[53BF30E\]                  ; EXERe.053A7017  
053BF2BE    68 30063B05     PUSH EXERe.053B0630  
053BF220    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF215    A3 84A03B05     MOV DWORD PTR DS:\[53BA084\],EAX               // Not LoadLibraryA  
053BF203    FF15 00F33B05   CALL DWORD PTR DS:\[53BF300\]                  ; EXERe.053A7017  
053BF247    68 A4053B05     PUSH EXERe.053B05A4  
053BF1F5    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF236    A3 ACA03B05     MOV DWORD PTR DS:\[53BA0AC\],EAX               // Not CreateThread  
053BF270    FF15 D0F23B05   CALL DWORD PTR DS:\[53BF2D0\]                  ; EXERe.053A7017  
053BF85A    68 50073B05     PUSH EXERe.053B0750  
053BF25C    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF22B    A3 A0A03B05     MOV DWORD PTR DS:\[53BA0A0\],EAX               // Not Sleep  
053BF81E    FF15 57F23B05   CALL DWORD PTR DS:\[53BF257\]                  ; EXERe.053A7017  
053BF843    68 54053B05     PUSH EXERe.053B0554  
053BF832    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF7F5    A3 B0A03B05     MOV DWORD PTR DS:\[53BA0B0\],EAX               // Not GetProcAddress  
053BF806    FF15 6DF13B05   CALL DWORD PTR DS:\[53BF16D\]                  ; EXERe.053A7017  
053BF6C4    68 AC073B05     PUSH EXERe.053B07AC  
053BF69E    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF6B3    A3 CCA03B05     MOV DWORD PTR DS:\[53BA0CC\],EAX               // Not WideCharToMultiByte  
053BF68C    FF15 AEF63B05   CALL DWORD PTR DS:\[53BF6AE\]                  ; EXERe.053A7017  
053BF626    68 64063B05     PUSH EXERe.053B0664  
053BF612    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF671    A3 D0A03B05     MOV DWORD PTR DS:\[53BA0D0\],EAX               // Not IsBadReadPtr  
053BF65F    FF15 80F63B05   CALL DWORD PTR DS:\[53BF680\]                  ; EXERe.053A7017  
053BF642    68 10083B05     PUSH EXERe.053B0810   
053BF745    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF729    A3 C0A03B05     MOV DWORD PTR DS:\[53BA0C0\],EAX               // Not VirtualAlloc  
053BF6F3    FF15 7BF63B05   CALL DWORD PTR DS:\[53BF67B\]                  ; EXERe.053A7017  
053BF718    68 2C083B05     PUSH EXERe.053B082C  
053BF73A    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF6DC    A3 9CA03B05     MOV DWORD PTR DS:\[53BA09C\],EAX               // Not ResumeThread  
053BF7CB    FF15 D6F63B05   CALL DWORD PTR DS:\[53BF6D6\]                  ; EXERe.053A7017  
053BF7E4    68 18063B05     PUSH EXERe.053B0618  
053BF707    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680  
053BF769    A3 A8A03B05     MOV DWORD PTR DS:\[53BA0A8\],EAX               // Not TerminateProcess  
053BF757    FF15 4CF73B05   CALL DWORD PTR DS:\[53BF74C\]                  ; EXERe.053A7017  
053BF78E    8B0D 60963B05   MOV ECX,DWORD PTR DS:\[53B9660\]               // EXE 文件的区段数  
053BF783    A3 98A03B05     MOV DWORD PTR DS:\[53BA098\],EAX               // Not CreateFileW  
  
  
五.  Stolen API 到 Heap  
  
053BF7A2    6A 40           PUSH 40                                      // 分配原程序空间到 400000, 先抢好位置, 哈  
053BF923    68 00300000     PUSH 3000  
053BF7BC    8D0C89          LEA ECX,DWORD PTR DS:\[ECX+ECX\*4\]  
053BF90B    C1E1 03         SHL ECX,3  
053BF777    8B81 FC8D3B05   MOV EAX,DWORD PTR DS:\[ECX+53B8DFC\]  
053BF8F9    8B89 F88D3B05   MOV ECX,DWORD PTR DS:\[ECX+53B8DF8\]  
053BF8F1    F7D0            NOT EAX  
053BF8DC    05 00004000     ADD EAX,400000  
053BF8C9    51              PUSH ECX  
053BF8E6    A3 E4A03B05     MOV DWORD PTR DS:\[53BA0E4\],EAX  
053BF88B    50              PUSH EAX  
053BF87A    A1 C0A03B05     MOV EAX,DWORD PTR DS:\[53BA0C0\]  
053BF86E    890D E8A03B05   MOV DWORD PTR DS:\[53BA0E8\],ECX  
053BF8A6    FFD0            CALL EAX                                     ; KERNEL32.VirtualAlloc(分配原程序的空间) \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053BFCC6    68 80963B05     PUSH EXERe.053B9680                          ; 前面取得的 kernel32 的 PE 信息  
053BFD95    FF15 9BF83B05   CALL DWORD PTR DS:\[53BF89B\]                  ; EXERe.053A706A   (分析 Kernel32 的输出表, F8)  
053BFDB4    85C0            TEST EAX,EAX  
053BFCE5    59              POP ECX  
053BFDA8   /0F84 DE140000   JE EXERe.053C128C  
053BFD17    8D8D 84F4FFFF   LEA ECX,DWORD PTR SS:\[EBP-B7C\]  
053BFCFF    FF15 F3FC3B05   CALL DWORD PTR DS:\[53BFCF3\]                  ; EXERe.053A225D   (Zero buffer, F8)  
  
053BFD81    6A FD           PUSH -3  
053BFD69    8D8D 84F4FFFF   LEA ECX,DWORD PTR SS:\[EBP-B7C\]               ; 12F444 做 Buffer 保存 ntdll.dll 的区块信息  
053BFD57    FF15 76FD3B05   CALL DWORD PTR DS:\[53BFD76\]                  ; EXERe.053A67D0   (获取 ntdll.dll 的区块信息 )  
053BFD45    8D85 84F4FFFF   LEA EAX,DWORD PTR SS:\[EBP-B7C\]  
053BFD2F    50              PUSH EAX  
053BFC0A    FF15 A0FC3B05   CALL DWORD PTR DS:\[53BFCA0\]                 ; EXERe.053A706A    (分析 ntdll.dll 的输出表, F8)  
053BFCB5    85C0            TEST EAX,EAX  
053BFC52    59              POP ECX  
053BFCA9   /0F84 DD150000   JE EXERe.053C128C  
  
053BFC79    6A 01           PUSH 1                                      ; 是否 Stolen 的标志  
053BFC97    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]               ; 保存 Stolen Size  
053BFC66    5F              POP EDI  
053BFC3A    57              PUSH EDI  
053BFC24    50              PUSH EAX  
053BFC8C    A1 98A03B05     MOV EAX,DWORD PTR DS:\[53BA098\]  
053BF9FE    F7D0            NOT EAX  
053BF956    50              PUSH EAX                                    ; KERNEL32.CreateFileW  
053BF936    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BF948    F7D0            NOT EAX  
053BF9C7    50              PUSH EAX                                    ; KERNEL32.7C570000  
053BF9DC    FF15 F3F93B05   CALL DWORD PTR DS:\[53BF9F3\]                 ; EXERe.053A102C,  ret = 1359c0    
  
  
53A102C 这个太牛了, 把 整个 API 搬到 Heap 空间,  返回 新的入口.   以后一定要好好研究一下\*\*\*\*  
  
Proc 53A102c( arg1, arg2, arg3, arg4)  
arg1 = DLL base  
arg2 = API address  
arg3 = pDWORD,   返回 size, 如果 HOOK 一般是 5  
arg4 = 标志,   1 stolen,  2 Hook  
  
ret = New entry address in heap, stolen  
或者  
ret = 0, HOOK  
  
  
053BF940    F7D0            NOT EAX  
053BF9B6    A3 98A03B05     MOV DWORD PTR DS:\[53BA098\],EAX          ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053BF965    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BF999    57              PUSH EDI  
053BF986    50              PUSH EAX  
053BF975    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BF948    F7D0            NOT EAX  
053BFA51    50              PUSH EAX                                 ; ntdll.RtlAllocateHeap  
053BFA3A    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BFA32    F7D0            NOT EAX  
053BFA22    50              PUSH EAX                                 ; KERNEL32.7C570000  
053BFB52    FF15 15FA3B05   CALL DWORD PTR DS:\[53BFA15\]              ; EXERe.053A102C, 好象对 HeapAlloc 不成功  
053BFBCA    F7D0            NOT EAX  
053BFBEF    A3 B4A03B05     MOV DWORD PTR DS:\[53BA0B4\],EAX           ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053BFA0C    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BFBE2    57              PUSH EDI  
053BFBB2    50              PUSH EAX  
053BFB74    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053BFB6C    F7D0            NOT EAX  
053BFB87    50              PUSH EAX                                 ; ntdll.RtlFreeHeap  
053BFB9B    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BFB93    F7D0            NOT EAX  
053BFAB7    50              PUSH EAX                                 ; KERNEL32.7C570000  
053BFA8A    FF15 C5FB3B05   CALL DWORD PTR DS:\[53BFBC5\]              ; EXERe.053A102C, 也不成功  
053BFA69    83C4 30         ADD ESP,30  
  
053BFB64    8BCE            MOV ECX,ESI                              ; EXERe.053B9680  
053BFAA9    F7D0            NOT EAX                                  ; ntdll.RtlFreeHeap  
053BFAFA    68 70073B05     PUSH EXERe.053B0770  
053BFAE0    A3 90A03B05     MOV DWORD PTR DS:\[53BA090\],EAX           ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
053BFACE    FF15 5DFA3B05   CALL DWORD PTR DS:\[53BFA5D\]              ; EXERe.053A7017  
053BFA9E    A3 94A03B05     MOV DWORD PTR DS:\[53BA094\],EAX           ; Not GetTickCount  
053BFB0B    8D4D E0         LEA ECX,DWORD PTR SS:\[EBP-20\]  
053BFB39    57              PUSH EDI  
053BFB1C    51              PUSH ECX  
053BC047    F7D0            NOT EAX  
053BC03A    50              PUSH EAX                                 ; KERNEL32.GetTickCount  
053BFB28    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BC026    F7D0            NOT EAX  
053BC0C0    50              PUSH EAX                                 ; KERNEL32.7C570000  
053BC0A3    FF15 B5C03B05   CALL DWORD PTR DS:\[53BC0B5\]              ; EXERe.053A102C, ret = 137BB8  
053BC08F    F7D0            NOT EAX  
053BC083    8B0D 048D3B05   MOV ECX,DWORD PTR DS:\[53B8D04\]           ; EXERe.053A0000, EXE image Base  
053BC078    A3 94A03B05     MOV DWORD PTR DS:\[53BA094\],EAX           ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
  
  
六.  解压一段数据 (StolenOEP 信息)  
  
053BC063    A1 60963B05     MOV EAX,DWORD PTR DS:\[53B9660\]           ; 主程序的区段数  
053BC1B0    57              PUSH EDI  
053BC0F5    8D0480          LEA EAX,DWORD PTR DS:\[EAX+EAX\*4\]         ; eax \* 5  
053BC0D3    8B04C5 AC8D3B05 MOV EAX,DWORD PTR DS:\[EAX\*8+53B8DAC\]     ; 24000  ,  主程序最后一个 Section 的 Voffset  
053BC0E8    8DB408 00010000 LEA ESI,DWORD PTR DS:\[EAX+ECX+100\]  
053BC11E    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BC106    50              PUSH EAX  
053BC151    A1 A4A03B05     MOV EAX,DWORD PTR DS:\[53BA0A4\]  
053BC129    F7D0            NOT EAX  
053BC139    50              PUSH EAX                                 ; KERNEL32.SetFilePointer  
053BC179    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BC162    F7D0            NOT EAX  
053BC18C    50              PUSH EAX                                 ; KERNEL32.7C570000  
  
053BC1A1    8975 DC         MOV DWORD PTR SS:\[EBP-24\],ESI            ; EXERe.053C4100     这里保存压缩的 StolenOEP 信息  
  
053BC399    FF15 56C03B05   CALL DWORD PTR DS:\[53BC056\]              ; EXERe.053A102C, ret=135EF8  
053BC169    F7D0            NOT EAX  
053BC2BD    A3 A4A03B05     MOV DWORD PTR DS:\[53BA0A4\],EAX           ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053BC36A    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BC37B    57              PUSH EDI  
053BC342    50              PUSH EAX  
053BC355    A1 B8A03B05     MOV EAX,DWORD PTR DS:\[53BA0B8\]  
053BC312    F7D0            NOT EAX  
053BC325    50              PUSH EAX                                 ; KERNEL32.ReadFile  
053BC331    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BC2E4    F7D0            NOT EAX  
053BC2FA    50              PUSH EAX                                 ; KERNEL32.7C570000  
053BC2D0    FF15 0DC33B05   CALL DWORD PTR DS:\[53BC30D\]              ; EXERe.053A102C, ret=136678  
053BC1C2    83C4 30         ADD ESP,30  
053BC269    F7D0            NOT EAX  
053BC2A1    A3 B8A03B05     MOV DWORD PTR DS:\[53BA0B8\],EAX           ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053BC279    6A 00           PUSH 0  
053BC2AE    8D45 E4         LEA EAX,DWORD PTR SS:\[EBP-1C\]  
053BC293    6A 04           PUSH 4  
053BC21E    50              PUSH EAX  
053BC247    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BC257    56              PUSH ESI                                 ; EXERe.053C4100  
053BC1FD    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BC233    F7D0            NOT EAX  
053BC1E3    FFD0            CALL EAX                                 ; KERNEL32.ReadProcessMemory ( 从 53C4100 读 4 byte)  
  
053BC777    6A 00           PUSH 0  
053BC689    8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
053BC72B    6A 04           PUSH 4  
053BC73E    50              PUSH EAX  
053BC762    8D46 04         LEA EAX,DWORD PTR DS:\[ESI+4\]  
053BC6AE    50              PUSH EAX                                                      ; EXERe.053C4104  
053BC751    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BC6FF    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BC717    F7D0            NOT EAX  
053BC6C7    FFD0            CALL EAX                                                      ; KERNEL32.ReadProcessMemory (53C4104 读 4 byte)  
  
                    053C4100  BE 11 00 00 87 09 00 00 BE 11 00 00 77 09 00 00   
                    053C4110  BE 11 00 00  
  
053BC6E0    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]                                   ; 11BE (压缩前的大小)  
053BC6EE    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BC694    F7D0            NOT EAX  
053BC3F7    6A 08           PUSH 8  
053BC3B3    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BC3CD    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap ( 分配 11BE Heap)  
  
053BC57A    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                    ; 987h ( 压缩后的大小)  
053BC3E3    8BF8            MOV EDI,EAX                                                   ; 11BEh 空间地址保存到 EDI = 2189B0  
053BC649    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BC65C    897D D4         MOV DWORD PTR SS:\[EBP-2C\],EDI  
053BC66D    6A 08           PUSH 8  
053BC591    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BC67D    F7D0            NOT EAX  
053BC5C7    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap ( 分配 987h Heap)  
  
053BC5AB    6A 00           PUSH 0  
053BC5B9    8BD8            MOV EBX,EAX                                                   ; 987h 空间地址保存到 EBX  
053BC5E3    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                    ; 987h  
053BC5FA    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BC625    83C6 08         ADD ESI,8  
053BC63A    895D C0         MOV DWORD PTR SS:\[EBP-40\],EBX  
053BC60D    53              PUSH EBX  
053BC561    56              PUSH ESI                                                      ; EXERe.053C4108  
053BC541    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BC457    F7D0            NOT EAX  
053BC40F    FFD0            CALL EAX                                                      ; KERNEL32.ReadProcessMemory (53C4108 读 987h byte 到分配的 987 空间)  
  
053BC425    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053BC432    33F6            XOR ESI,ESI                                                   ; EXERe.053C4108  
053BC442    8945 D8         MOV DWORD PTR SS:\[EBP-28\],EAX                                 ; 保留 压缩后的 OEP 数据大小  
053BC47C    83C0 F8         ADD EAX,-8  
053BC44D    85C0            TEST EAX,EAX  
053BC461    8975 FC         MOV DWORD PTR SS:\[EBP-4\],ESI  
053BC470   /0F86 E0040000   JBE EXERe.053BC956                                            ; 小于 8 字节不用解压缩了  
  
053BC49C    8B041E          MOV EAX,DWORD PTR DS:\[ESI+EBX\]                                ; 第二个 11BEh  
053BC48D    8B5433 04       MOV EDX,DWORD PTR DS:\[EBX+ESI+4\]                              ; 977h  
053BC4E6    83C6 08         ADD ESI,8  
053BC4AA    3BC2            CMP EAX,EDX  
053BC4D1    8945 E8         MOV DWORD PTR SS:\[EBP-18\],EAX  
053BC4B8    8955 EC         MOV DWORD PTR SS:\[EBP-14\],EDX  
053BC51B   /0F85 28680000   JNZ EXERe.053C2D49  
053C2D49    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
053C1A99    FF75 E8         PUSH DWORD PTR SS:\[EBP-18\]  
053C2D54    03C7            ADD EAX,EDI  
053C130C    B9 70083B05     MOV ECX,EXERe.053B0870  
053C12D3    50              PUSH EAX  
053C12FD    8D041E          LEA EAX,DWORD PTR DS:\[ESI+EBX\]  
053C1247    FF75 EC         PUSH DWORD PTR SS:\[EBP-14\]  
053C12EE    50              PUSH EAX  
053C12A5    FF15 C8123C05   CALL DWORD PTR DS:\[53C12C8\]                                   ; EXERe.053A53E1, F8 (什么压缩算法?)  
                                                                                          ; 解压(compressed buf, size1,  uncompress buf, size2) \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053BC948    8B45 E8         MOV EAX,DWORD PTR SS:\[EBP-18\]  
053BC92B    0375 EC         ADD ESI,DWORD PTR SS:\[EBP-14\]  
053BC916    0145 FC         ADD DWORD PTR SS:\[EBP-4\],EAX  
053BC980    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053BC960    83C0 F8         ADD EAX,-8  
053BC989    3BF0            CMP ESI,EAX  
053BC96E  ^\\0F82 28FBFFFF   JB EXERe.053BC49C  
  
053BC956    8B07            MOV EAX,DWORD PTR DS:\[EDI\]                                    ; 11BEh 空间, \[2189B0\]= 570, 前面 570h 是类似重定位信息,  后面是 Stolen OEP 代码        
  
                        
                        
                      002189B0  70 05 00 00 03 00 00 00 00 00 00 00 7F 11 3A 05           ; 以前三组为例,  每组 0c 字节 { x,  y,  z}    
                      002189C0  31 02 00 00 00 00 00 00 7F 11 3A 05 39 02 00 00           ; x 是需要重定位的相对偏移  
                      002189D0  00 00 00 00 7F 11 3A 05                                   ; y=0,1,2,   
                                                                                          ;   0 表示保存 ESP 的全局变量需要重定位  
                                                                                              1 没发现  
                                                                                              2 表示 Call \[40xxxxxx\] 的情况  
                                                                                                
                                                                                          ; z 好象是不同的基址(这里只偷了一个 Call 都相同)   
                         
  
053BC8D0    6A 0C           PUSH 0C                                                       ; 后面 0ch 字节一组  
053BC8ED    33D2            XOR EDX,EDX  
053BC8B6    59              POP ECX  
053BC8E3    F7F1            DIV ECX  
053BC79F    8BD8            MOV EBX,EAX                                                   ; 74h 组  
053BC794    33C0            XOR EAX,EAX  
053BC7C8    3BD8            CMP EBX,EAX  
053BC7D6    8945 A8         MOV DWORD PTR SS:\[EBP-58\],EAX  
053BC7B2    8945 AC         MOV DWORD PTR SS:\[EBP-54\],EAX  
053BC83C    8945 B0         MOV DWORD PTR SS:\[EBP-50\],EAX  
053BC7FB   /0F8E 7C060000   JLE EXERe.053BCE7D  
  
053BC7E6    8D47 04         LEA EAX,DWORD PTR DS:\[EDI+4\]  
053BC7EF    895D F4         MOV DWORD PTR SS:\[EBP-C\],EBX                                  ; 循环变量  
053BC810    8945 FC         MOV DWORD PTR SS:\[EBP-4\],EAX                                  ; 每组开始的地址  
  
                                                                                          ; 下面这段代码效率很低, 为什么不一次性复制,   
                                                                                          ; 而要用 53A2301 一组一组追加,  迷惑人?  
053BC806    8D45 C4         LEA EAX,DWORD PTR SS:\[EBP-3C\]                                 ; \[ebp-3c\] 0ch buffer in stack  
053BC820    85C0            TEST EAX,EAX  
053BC82D   /0F84 51000000   JE EXERe.053BC884  
053BC89A    837D FC 00      CMP DWORD PTR SS:\[EBP-4\],0                                    ;   
053BC859   /0F84 25000000   JE EXERe.053BC884  
053BC8A3    8B75 FC         MOV ESI,DWORD PTR SS:\[EBP-4\]                                  ; 每组开始的地址  
053BC844    8D7D C4         LEA EDI,DWORD PTR SS:\[EBP-3C\]  
053BC84F    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                    ; 复制 0ch 字节 到 stack buffer  
053BC818    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BC86A    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BC876    8B7D D4         MOV EDI,DWORD PTR SS:\[EBP-2C\]  
053BC884    8D45 C4         LEA EAX,DWORD PTR SS:\[EBP-3C\]  
053BC88C    8D4D A8         LEA ECX,DWORD PTR SS:\[EBP-58\]                                 ; 新空间的信息(开始地址, 结束地址, 个数)  
053BCCE6    50              PUSH EAX  
053BCE50    FF15 94C83B05   CALL DWORD PTR DS:\[53BC894\]                                   ; EXERe.053A2301  (arg=stack buffer),  ecx 也是参数, 结果 buffer  
                                                                                          ; 追加 \[ebp-3c\] 的 12 字节, 这个函数比较简单  
053BCE95    8345 FC 0C      ADD DWORD PTR SS:\[EBP-4\],0C                                   ; + 0ch  
053BCE6F    FF4D F4         DEC DWORD PTR SS:\[EBP-C\]  
053BCE61  ^\\0F85 9FF9FFFF   JNZ EXERe.053BC806                                            ; 循环结束, 570h 字节全部搬到新的空间  
  
  
053BCE7D    A1 94A03B05     MOV EAX,DWORD PTR DS:\[53BA094\]                                ; GetTickCount  
053BCE8A    F7D0            NOT EAX  
053BCECE    FFD0            CALL EAX  
053BCEA4    25 FFFF0000     AND EAX,0FFFF  
053BCEBA    50              PUSH EAX                                                      ; 用时间分配空间, 确保下面的伪 OEP 每次运行都不一样  
053BCEF5    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BCF0B    6A 08           PUSH 8  
053BCEE1    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BCF1B    F7D0            NOT EAX  
053BCDD8    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
  
053BCE27    8D345B          LEA ESI,DWORD PTR DS:\[EBX+EBX\*2\]                              ; ESI = EBX \* 3  
053BCE3C    8945 D0         MOV DWORD PTR SS:\[EBP-30\],EAX  
053BCE33    8B45 E4         MOV EAX,DWORD PTR SS:\[EBP-1C\]                                 ; 11BE 大小  
053BCDF8    C1E6 02         SHL ESI,2                                                     ; ESI \* = 4 ( 570h )  
053BCDE8    2BC6            SUB EAX,ESI                                                   ; 剩下的就是 StolenOEP 代码大小  
053BCE1E    83E8 04         SUB EAX,4  
  
053BCE0F    50              PUSH EAX                                                      ; 为 StolenOEP 分配空间  
053BCD25    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BCD11    6A 08           PUSH 8  
053BCCF9    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BCD80    F7D0            NOT EAX  
053BCDB6    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
053BCDC4    8BD8            MOV EBX,EAX  
  
053BCD8D    8B45 E4         MOV EAX,DWORD PTR SS:\[EBP-1C\]                                 ; 准备复制  
053BCD9E    2BC6            SUB EAX,ESI  
053BCDA5    895D 88         MOV DWORD PTR SS:\[EBP-78\],EBX                                 ; 保存最终的伪 OEP 入口 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053BCD95    83E8 04         SUB EAX,4  
053BCD38    8D743E 04       LEA ESI,DWORD PTR DS:\[ESI+EDI+4\]  
053BCD30    85DB            TEST EBX,EBX  
053BCD5E  ^\\0F84 DFFCFFFF   JE EXERe.053BCA43  
053BCD72    85F6            TEST ESI,ESI  
053BCD4A  ^\\0F84 F3FCFFFF   JE EXERe.053BCA43  
053BCD6A    85C0            TEST EAX,EAX  
053BCB0A  ^\\0F84 33FFFFFF   JE EXERe.053BCA43  
053BCAE8  ^\\0F86 55FFFFFF   JBE EXERe.053BCA43  
053BCD56    8BCB            MOV ECX,EBX  
053BCD42    2BF3            SUB ESI,EBX  
053BCAFC    8BF8            MOV EDI,EAX  
  
053BCAF3    8A040E          MOV AL,BYTE PTR DS:\[ESI+ECX\]                                  ;Stole OEP 从 218F24 复制到 Heap 空间 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053BCA72    8801            MOV BYTE PTR DS:\[ECX\],AL  
053BCDCB    41              INC ECX  
053BCD79    4F              DEC EDI  
053BCA58   /0F85 95000000   JNZ EXERe.053BCAF3  
  
053BCA49    FF75 D0         PUSH DWORD PTR SS:\[EBP-30\]                                    ; 释放前面用时间分配的垃圾空间  
053BCA67    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053BCA32    F7D0            NOT EAX  
053BCA22    6A 00           PUSH 0  
053BCA90    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BCAC4    FFD0            CALL EAX                                                      ; ntdll.RtlFreeHeap  
  
  
  
七. 处理 StolenOEP 的重定位  
  
053BCAD8    8365 F8 00      AND DWORD PTR SS:\[EBP-8\],0                                     
053BCA7A    837D B0 00      CMP DWORD PTR SS:\[EBP-50\],0                                  ; 循环变量( 开始=74h)  
053BCAAA   /0F84 68110000   JE EXERe.053BDC18                                            ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  处理完毕, 循环出口  
  
053BCAA1    8B75 A8         MOV ESI,DWORD PTR SS:\[EBP-58\]                                ; 570h 的新空间开始地址                                ;                                 
053BCA3A    8D7D C4         LEA EDI,DWORD PTR SS:\[EBP-3C\]                                ; 还用刚才的 buffer in stack 做临时空间  
053BCA13    8975 FC         MOV DWORD PTR SS:\[EBP-4\],ESI                                 ; \*\*\*\* 包存新空间的开始地址  
053BCD1E    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                   ; 取 0ch 到临时空间  
053BCB03    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BCA83    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BCB78    837D C8 02      CMP DWORD PTR SS:\[EBP-38\],2                                  ; y 共有三种情况 0, 1, 2  
053BCB34   /0F85 EC000000   JNZ EXERe.053BCC26  
                                                                                         ; 对应 2 的情况 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 2  
053BCB1B    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BCB4E    6A 04           PUSH 4  
053BCB68    6A 08           PUSH 8  
053BCBF8    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BCB26    F7D0            NOT EAX  
053BCB98    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap (分配 4 字节)  
  
053BCB81    8B4D CC         MOV ECX,DWORD PTR SS:\[EBP-34\]                                 ; 40xxxxx  
053BCBE1    8945 E8         MOV DWORD PTR SS:\[EBP-18\],EAX                                 ; 分配的空间  
053BCB89    85C0            TEST EAX,EAX  
053BCBB7    894D F4         MOV DWORD PTR SS:\[EBP-C\],ECX  
053BCBAB   /0F84 58000000   JE EXERe.053BCC09  
053BCBC7    8D4D F4         LEA ECX,DWORD PTR SS:\[EBP-C\]  
053BCBD7    85C9            TEST ECX,ECX  
053BCCB7  ^\\0F84 4CFFFFFF   JE EXERe.053BCC09  
053BCCD4    8D75 F4         LEA ESI,DWORD PTR SS:\[EBP-C\]  
053BCC19    8BF8            MOV EDI,EAX  
053BCCC8    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                    ; 把 40xxxx 放入分配的空间  
053BCC09    8B45 C4         MOV EAX,DWORD PTR SS:\[EBP-3C\]                                 ; 相对偏移  
053BCC61    8D3C18          LEA EDI,DWORD PTR DS:\[EAX+EBX\]                                ; EBX 是最终的伪 OEP  
053BCC46    85FF            TEST EDI,EDI  
053BCC4D   /0F84 16000000   JE EXERe.053BCC69  
053BCC32    8D45 E8         LEA EAX,DWORD PTR SS:\[EBP-18\]  
053BCC89    85C0            TEST EAX,EAX  
053BCC3A   /0F84 29000000   JE EXERe.053BCC69  
053BCCA6    8D75 E8         LEA ESI,DWORD PTR SS:\[EBP-18\]  
053BCC98    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BCC6F    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]  
053BDE7D    FF45 F8         INC DWORD PTR SS:\[EBP-8\]  
053BD153    8D4D A8         LEA ECX,DWORD PTR SS:\[EBP-58\]  
053BD016    FF15 7DCC3B05   CALL DWORD PTR DS:\[53BCC7D\]                                    ; EXERe.053A4B78,  减小新空间 0ch 字节,  效率很低的移动, 循环变量--  
053BCF58  ^\\E9 1DFBFFFF     JMP EXERe.053BCA7A  
                                                                                     
  
  
  
053BCC26    837D C8 01      CMP DWORD PTR SS:\[EBP-38\],1  
053BCF46   /0F85 21010000   JNZ EXERe.053BD06D                                            ; 对应 1 的情况没发现 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 1  
  
  
  
  
053BD06D    837D C8 00      CMP DWORD PTR SS:\[EBP-38\],0  
053BDA0E  ^\\0F85 66F0FFFF   JNZ EXERe.053BCA7A  
                                                                                          ; 对应 0 的情况 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 0  
053BDA1B    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BDB56    6A 04           PUSH 4                                                        ; 分配 4 byte 做全局变量保存 ESP (到 OEP 后用, 这个只分配一次)  
053BDAF3    6A 08           PUSH 8  
053BDB1C    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BDB0D    F7D0            NOT EAX  
053BDB33    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
053BDB40    8945 EC         MOV DWORD PTR SS:\[EBP-14\],EAX  
  
053BDACE    8B45 C4         MOV EAX,DWORD PTR SS:\[EBP-3C\]                                 ; 相对偏移  
053BDADD    8D3C18          LEA EDI,DWORD PTR DS:\[EAX+EBX\]                                ; EBX 是最终的伪 OEP  
053BDA92    85FF            TEST EDI,EDI  
053BDAAC  ^\\0F84 92FFFFFF   JE EXERe.053BDA44  
053BDABE    8D45 EC         LEA EAX,DWORD PTR SS:\[EBP-14\]  
053BDAA4    85C0            TEST EAX,EAX  
053BDA86  ^\\0F84 B8FFFFFF   JE EXERe.053BDA44  
053BDA33    8D75 EC         LEA ESI,DWORD PTR SS:\[EBP-14\]  
053BDA26    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                    ; 保存 ESP 全局变量  
053BDA44    FF45 F8         INC DWORD PTR SS:\[EBP-8\]  
053BDA64    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]                                     ; 开始地址  
  
053BDA75    8D4D A8         LEA ECX,DWORD PTR SS:\[EBP-58\]  
053BDB82    FF15 7DDA3B05   CALL DWORD PTR DS:\[53BDA7D\]                                   ; EXERe.053A4B78,  减小新空间 0ch 字节,  效率很低的移动, 循环变量--  
  
053BDA55    8D45 C4         LEA EAX,DWORD PTR SS:\[EBP-3C\]                                 ; 临时空间地址  
053BDE5C    50              PUSH EAX  
053BDDDD    FF75 AC         PUSH DWORD PTR SS:\[EBP-54\]                                    ; 新空间的结束地址 (这个一直在减少)  
053BDE40    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]                                     ; 新空间的开始地址 (这个不变)  
053BDE24    FF15 78DE3B05   CALL DWORD PTR DS:\[53BDE78\]                                   ; EXERe.053A4CD0, 返回开始地址  
053BDDFF    83C4 0C         ADD ESP,0C  
053BDE0E    3B45 AC         CMP EAX,DWORD PTR SS:\[EBP-54\]                                 ; 开始地址 == 结束地址?  
053BDB6A  ^\\0F84 0AEFFFFF   JE EXERe.053BCA7A  
  
053BDE4E    8BF0            MOV ESI,EAX                                                   ; 再取 0ch 到临时空间  
053BDDEC    8D7D C4         LEA EDI,DWORD PTR SS:\[EBP-3C\]  
053BDCDE    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BDD57    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BDCEC    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053BDD3D    8B4D C4         MOV ECX,DWORD PTR SS:\[EBP-3C\]                                 ; 相对偏移  
053BDD47    8D3C19          LEA EDI,DWORD PTR DS:\[ECX+EBX\]                                ; + OEP  
053BDD28    85FF            TEST EDI,EDI  
053BDD0B   /0F84 AA000000   JE EXERe.053BDDBB  
053BDD79    8D4D EC         LEA ECX,DWORD PTR SS:\[EBP-14\]  
053BDCF7    85C9            TEST ECX,ECX  
053BDD6B   /0F84 4A000000   JE EXERe.053BDDBB  
053BDD9D    8D75 EC         LEA ESI,DWORD PTR SS:\[EBP-14\]  
053BDD89    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                    ; 保存 ESP 全局变量  
053BDDBB    FF45 F8         INC DWORD PTR SS:\[EBP-8\]  
053BDDAD    50              PUSH EAX                                                      ; 开始地址  
053BDDCB  ^\\E9 A5FCFFFF     JMP EXERe.053BDA75  
  
  
  
053BDC18                                                                                  ;  Stolen OEP 处理完, 这里继续  
053BDC1E    FF75 D4         PUSH DWORD PTR SS:\[EBP-2C\]                                    ; 11BEh 的 Heap 空间  
053BDBF1    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053BDC03    33FF            XOR EDI,EDI  
053BDBB3    57              PUSH EDI  
053BDB9A    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BDBC6    F7D0            NOT EAX  
053BDBD8    FFD0            CALL EAX                                                      ; ntdll.RtlFreeHeap  
  
053BDC92    FF75 C0         PUSH DWORD PTR SS:\[EBP-40\]                                    ; 987h 的 Heap 空间  
053BDBE5    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053BDCAC    F7D0            NOT EAX  
053BDCC2    57              PUSH EDI  
053BDC33    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BDC53    FFD0            CALL EAX                                                      ; ntdll.RtlFreeHeap  
  
  
  
  
八. 解压 Code 了,  dump PE 头的好时机  
  
053BDC68    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BD66D    6A 01           PUSH 1  
053BD972    50              PUSH EAX  
053BDC7A    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BD986    F7D0            NOT EAX  
053BD9CE    50              PUSH EAX                                                      ; KERNEL32.ReadProcessMemory  
053BD9AC    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BD9F7    F7D0            NOT EAX  
053BD930    50              PUSH EAX                                                      ; KERNEL32.7C570000  
053BD953    FF15 C0D93B05   CALL DWORD PTR DS:\[53BD9C0\]                                   ; EXERe.053A102C, ret = 1369f0  
003BD9E6    F7D0            NOT EAX  
053BD796    A3 88A03B05     MOV DWORD PTR DS:\[53BA088\],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
      
053BD6FA    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BD68F    6A 01           PUSH 1  
053BD6C0    50              PUSH EAX  
053BD6A2    A1 7CA03B05     MOV EAX,DWORD PTR DS:\[53BA07C\]  
053BD6AE    F7D0            NOT EAX  
053BD6D4    50              PUSH EAX                                                      ; KERNEL32.GetModuleHandleA  
053BD6EA    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BD6E2    F7D0            NOT EAX  
053BD781    50              PUSH EAX                                                      ; KERNEL32.7C570000  
053BD747    FF15 B5D63B05   CALL DWORD PTR DS:\[53BD6B5\]                                   ; EXERe.053A102C, ret = 136200  
053BD76C    8B4D D8         MOV ECX,DWORD PTR SS:\[EBP-28\]  
053BD724    83C4 20         ADD ESP,20  
053BD75E    F7D0            NOT EAX  
053BD736    A3 7CA03B05     MOV DWORD PTR DS:\[53BA07C\],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*    
  
  
053BD714    8B45 DC         MOV EAX,DWORD PTR SS:\[EBP-24\]                                 ; EXERe.053C4100  
053BD7E7    57              PUSH EDI  
053BD7C9    6A 04           PUSH 4                                                        ; 4 字节  
053BD7D8    8D3401          LEA ESI,DWORD PTR DS:\[ECX+EAX\]                                ; ECX = 987h, OEP 信息长度  
053BD7AB    8D45 E4         LEA EAX,DWORD PTR SS:\[EBP-1C\]  
053BD8F9    50              PUSH EAX                                                      ; 53C4A87 放的是 Code 压缩前长度   
053BD7B8    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BD919    56              PUSH ESI                                                      ; EXERe.053C4A87  
053BD892    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BD905    F7D0            NOT EAX  
053BD8B1    FFD0            CALL EAX                                                      ; ReadProcessMemory  
  
053BD8D7    57              PUSH EDI  
053BD8E8    8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
053BD844    6A 04           PUSH 4                                                        ; 再读 4 字节  Code 压缩后长度  
053BD8C4    50              PUSH EAX  
053BD86E    8D46 04         LEA EAX,DWORD PTR DS:\[ESI+4\]  
053BD85B    50              PUSH EAX                                                      ; EXERe.053C4A8B  
053BD87B    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BD7F9    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BD8A3    F7D0            NOT EAX  
053BD81E    FFD0            CALL EAX                                                      ; ReadProcessMemory  
  
053BD4D5    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]                                    ; uncompress size = 1EE82h  
053BD833    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BD80A    F7D0            NOT EAX  
053BD548    6A 08           PUSH 8  
053BD509    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BD522    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
  
053BD4F1    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                    ; compress size = DCAFh  
053BD537    8BD8            MOV EBX,EAX  
053BD5E7    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BD656    6A 08           PUSH 8  
053BD61D    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BD4E3    F7D0            NOT EAX  
053BD63F    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
  
053BD5FD    6A 00           PUSH 0  
053BD60C    8BF8            MOV EDI,EAX  
053BD590    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]  
053BD569    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BD57B    83C6 08         ADD ESI,8                                                     ; 跳过 8 字节  
053BD55C    57              PUSH EDI  
053BD5AE    56              PUSH ESI                                                      ; EXERe.053C4A8F  
053BD5C3    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BD5DF    F7D0            NOT EAX  
053BD2B4    FFD0            CALL EAX                                                      ; ReadProcessMemory( DCAFh)  
  
053BD59F    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053BD236    33F6            XOR ESI,ESI                                                   ; EXERe.053C4A8F  
053BD29F    0145 D8         ADD DWORD PTR SS:\[EBP-28\],EAX  
053BD260    83C0 F8         ADD EAX,-8  
053BD252    85C0            TEST EAX,EAX  
053BD243    8975 EC         MOV DWORD PTR SS:\[EBP-14\],ESI  
053BD293   /0F86 21010000   JBE EXERe.053BD3BA                                            ; 小于 8 字节不用解压  
  
053BD281    8B043E          MOV EAX,DWORD PTR DS:\[ESI+EDI\]                                ; 每次最多压缩 2000h,   所以解压要分几次  
053BD26D    8B5437 04       MOV EDX,DWORD PTR DS:\[EDI+ESI+4\]  
053BD28A    83C6 08         ADD ESI,8  
053BD1DA    3BC2            CMP EAX,EDX  
053BD220    8945 E8         MOV DWORD PTR SS:\[EBP-18\],EAX  
053BD1EF    8955 FC         MOV DWORD PTR SS:\[EBP-4\],EDX  
053BD206   /0F85 D2590000   JNZ EXERe.053C2BDE  
053C2BDE    8B45 EC         MOV EAX,DWORD PTR SS:\[EBP-14\]  
053C2D71    FF75 E8         PUSH DWORD PTR SS:\[EBP-18\]  
053C2D82    03C3            ADD EAX,EBX  
053C2D5D    B9 70083B05     MOV ECX,EXERe.053B0870  
053C2CDC    50              PUSH EAX  
053C2CC1    8D043E          LEA EAX,DWORD PTR DS:\[ESI+EDI\]  
053C2D38    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]  
053C2D19    50              PUSH EAX  
053C2CFB    FF15 0D2D3C05   CALL DWORD PTR DS:\[53C2D0D\]                                   ; EXERe.053A53E1 (调用解压函数)  
053BD2EC    8B45 E8         MOV EAX,DWORD PTR SS:\[EBP-18\]  
053BD2FE    0375 FC         ADD ESI,DWORD PTR SS:\[EBP-4\]  
053BD2F4    0145 EC         ADD DWORD PTR SS:\[EBP-14\],EAX  
053BD48B    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053BD49D    83C0 F8         ADD EAX,-8  
053BD4B4    3BF0            CMP ESI,EAX  
053BD4C1  ^\\0F82 BAFDFFFF   JB EXERe.053BD281  
  
053BD3C0    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]                                    ; 解压后长度  
053BD380    B9 688C3B05     MOV ECX,EXERe.053B8C68  
053BD3A8    53              PUSH EBX                                                      ; 146278 初步解压后的代码就放在这 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053BD36C    FF15 5ED33B05   CALL DWORD PTR DS:\[53BD35E\]                                   ; EXERe.053A46CA  分离 PE 头等  
  
053BD390    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]                                  
053BD2E4    33F6            XOR ESI,ESI  
053BD425    53              PUSH EBX  
053BD464    56              PUSH ESI  
053BD477    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BD44E    F7D0            NOT EAX  
053BD43C    FFD0            CALL EAX                                                      ; ntdll.RtlFreeHeap  
053BD3D4    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053BD3F6    57              PUSH EDI  
053BD408    56              PUSH ESI  
053BEC34    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BD4A5    F7D0            NOT EAX  
053BEF3D    FFD0            CALL EAX                                                      ; ntdll.RtlFreeHeap  
  
053BD3E3    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BEFA7    6A 01           PUSH 1  
053BEFBA    50              PUSH EAX  
053BEF74    A1 84A03B05     MOV EAX,DWORD PTR DS:\[53BA084\]  
053BEFC7    F7D0            NOT EAX  
053BEF8D    50              PUSH EAX                                                      ; KERNEL32.LoadLibraryA  
053BEF67    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BEF7E    F7D0            NOT EAX  
053BEF53    50              PUSH EAX                                                      ; KERNEL32.7C570000  
053BEE9B    FF15 DED33B05   CALL DWORD PTR DS:\[53BD3DE\]                                   ; EXERe.053A102C,  ret = 155098  
053BEEB8    F7D0            NOT EAX  
053BEECF    A3 84A03B05     MOV DWORD PTR DS:\[53BA084\],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
053BEEBF    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BEEEB    6A 01           PUSH 1  
053BEF01    50              PUSH EAX  
053BEF1D    A1 A8A03B05     MOV EAX,DWORD PTR DS:\[53BA0A8\]  
053BEF2D    F7D0            NOT EAX  
053BEDB3    50              PUSH EAX                                                      ; KERNEL32.TerminateProcess  
053BEF0D    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BEEAF    F7D0            NOT EAX  
053BEE4B    50              PUSH EAX                                                      ; KERNEL32.7C570000  
053BEE60    FF15 8EEE3B05   CALL DWORD PTR DS:\[53BEE8E\]                                   ; EXERe.053A102C, ret = 1558d0  
053BEE75    F7D0            NOT EAX  
053BEE82    A3 A8A03B05     MOV DWORD PTR DS:\[53BA0A8\],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
   
053BEDE1    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BEDD2    6A 01           PUSH 1  
053BEE2B    50              PUSH EAX  
053BEDE9    A1 9CA03B05     MOV EAX,DWORD PTR DS:\[53BA09C\]  
053BEE02    F7D0            NOT EAX  
053BEE0F    50              PUSH EAX                                                      ; KERNEL32.ResumeThread  
053BEC6F    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BEC4C    F7D0            NOT EAX  
053BEC59    50              PUSH EAX                                                      ; KERNEL32.7C570000  
053BECA4    FF15 46EC3B05   CALL DWORD PTR DS:\[53BEC46\]                                   ; EXERe.053A102C, ret = 52EE58  
053BEC92    8B4D DC         MOV ECX,DWORD PTR SS:\[EBP-24\]                                 ; EXERe.053C4100  
053BED0D    83C4 30         ADD ESP,30  
053BEC80    F7D0            NOT EAX  
053BECC8    A3 9CA03B05     MOV DWORD PTR DS:\[53BA09C\],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
等我出差回来, 继续  

第三个特点是用全局变量保存 ESP,  JMP 都是用 Call 实现的.  
用这种方式来实现花指令倒是第一次见到.  
053BC006    8925 CFEF3B05   MOV DWORD PTR DS:\[53BEFCF\],ESP  
053BC00C    E8 F4350000     CALL EXERe.053BF605  
053BF605    BE 80963B05     MOV ESI,EXERe.053B9680  
053BF60A    E8 BAFEFFFF     CALL EXERe.053BF4C9  
053BF4C9    8B25 CFEF3B05   MOV ESP,DWORD PTR DS:\[53BEFCF\]   // 恢复 ESP  
我不知道别人的花指令是怎么实现的。不知道是不是开发的时候就在  
每个汇编指令之间插入花指令。  
我的壳是用 VC 开发的，编译之后是没有什么花指令的。所以只能再写  
一个程序来自动将原来的程序中某个函数处理成这样.这样的的方式不知道是不是可以叫 obfuscation.  
现在这种用全局变量保存 esp 的方式比较原始，很容易去掉这种干扰  
指令，用其他保存 esp 的方式实现起来太复杂，是我以后修改的方  
向。  
虽然觉得花指令没多大用，但是还是按照国际惯例加上一点。 

 4dd466d0e1e0cc123afbe504da2462ca