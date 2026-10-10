

# EXE ReFactor 0.2 详细分析(二)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-593.htm`](https://www.pediy.com/kssd/pediy07/pediy7-593.htm)

上次时间紧, 第八部分没有分析仔细. 不好意思.  
实际上是第八部分是处理资源的.  壳对资源特别照顾, 去掉了资源的头, 只保留了数据.  
然后 HOOK 了处理资源的函数, 为主程序提供资源.  
目前好象没有工具可以自动建立资源头, 我是手动组装的, 如果熟悉资源格式的话, 花一个小时就够了.  
  
接下来, 就到了比较有特色的 IAT 处理部分.  
大致思路是 为每一个 DLL 建立一个线程, 为每一个 API 建立一段代码, 但并不求出地址.  
每个线程每过 2ms 扫描一次 53B8C48 的对列.  
  
到了主程序, 每遇到一个 API,  API 的代码就往 53B8C48 队列插入一个成员(APIAddress=0)  
然后就等待 2ms.   这时线程扫描队列, 根据 DLLNameHash 决定是否由本线程处理,  
如果是, 对 APIaddress=0 的成员计算地址.  
  
２ms 后, 主程序得到了 API 地址, 继续执行.  
  
Structure 成员  
{    
  DWORD  DLLNameHash;  
  DWORD  APINameHash;  
  DWORD  APIAddress( 0 表示没计算好)  
}  
  
  
从上面的分析知道, 每遇到一个新 API, 程序就要等待 2ms, 所以程序启动有点慢.  
如果缩小这个时间, 扫描线程又会和主线程抢 CPU 资源.    
  
并且知道了原理, IAT 的修复是非常容易的, 见脱壳文件. 附件中还有全文.  
  
  
九. 解出部分加壳前的 PE 信息 ( 原来的 ImageBase, OEP, baseofcode )   
  
053BECBD    8B45 D8         MOV EAX,DWORD PTR SS:\[EBP-28\]                                 ; StolenOEP + code 压缩的大小  
053BECEB    56              PUSH ESI  
053BED71    6A 04           PUSH 4                                                        ; 读 4 字节  
053BECD8    8D3C08          LEA EDI,DWORD PTR DS:\[EAX+ECX\]  
053BED91    8D45 E4         LEA EAX,DWORD PTR SS:\[EBP-1C\]  
053BED54    50              PUSH EAX  
053BEDA2    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BED31    57              PUSH EDI                                                      ; EXERe.053D2736  
053BE5F4    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BED22    F7D0            NOT EAX  
053BEBA5    FFD0            CALL EAX                                                      ; ReadProcessMemory ( 4 byte)  
  
053BED84    56              PUSH ESI  
053BEBD2    8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
053BEBC4    6A 04           PUSH 4  
053BEC0E    50              PUSH EAX  
053BEC24    8D47 04         LEA EAX,DWORD PTR DS:\[EDI+4\]  
053BEA05    50              PUSH EAX                                                      ; EXERe.053D273A  
053BEBF8    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BEB5C    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BEBE0    F7D0            NOT EAX  
053BEB90    FFD0            CALL EAX                                                       ; ReadProcessMemory  
  
  
053BEB3A    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]                                     ; uncompressed size  
053BEB73    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BEB81    F7D0            NOT EAX  
053BEA5E    6A 08           PUSH 8  
053BEA36    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BEA1E    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
  
053BEAA4    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                    ; compressed size  
053BEA4E    8BF0            MOV ESI,EAX  
053BEA83    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BEAD7    6A 08           PUSH 8  
053BEABF    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BEA6C    F7D0            NOT EAX  
053BEAF2    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
  
053BEB14    6A 00           PUSH 0  
053BEA74    8BD8            MOV EBX,EAX  
053BE798    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]  
053BEB28    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BE636    83C7 08         ADD EDI,8                                                     ; 跳过 8 字节  
053BE628    53              PUSH EBX  
053BE60D    57              PUSH EDI                                                      ; EXERe.053D273E  
053BE6FA    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BE71C    F7D0            NOT EAX  
053BE730    FFD0            CALL EAX                                                      ; ReadProcessMemory  
  
  
050BE70D    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]                                 ; 压缩的大小  
053BE760    33FF            XOR EDI,EDI  
053BE76D    0145 D8         ADD DWORD PTR SS:\[EBP-28\],EAX                                 ; 累加已读入的大小, 为下一个处理做准备  
053BE780    83C0 F8         ADD EAX,-8  
053BE744    85C0            TEST EAX,EAX  
053BE789    897D EC         MOV DWORD PTR SS:\[EBP-14\],EDI  
053BE6A6   /0F86 9F010000   JBE EXERe.053BE84B                                            ; 小于 8 字节不用处理  
  
053BE751    8B041F          MOV EAX,DWORD PTR DS:\[EDI+EBX\]                                ; 循环解压  
053BE6BD    8B543B 04       MOV EDX,DWORD PTR DS:\[EBX+EDI+4\]  
053BE6B1    83C7 08         ADD EDI,8  
053BE6E6    3BC2            CMP EAX,EDX  
053BE6D4    8945 E8         MOV DWORD PTR SS:\[EBP-18\],EAX  
053BE6CA    8955 FC         MOV DWORD PTR SS:\[EBP-4\],EDX  
053BE673   /0F85 B3450000   JNZ EXERe.053C2C2C  
053C2C2C    8B45 EC         MOV EAX,DWORD PTR SS:\[EBP-14\]  
053C2C40    FF75 E8         PUSH DWORD PTR SS:\[EBP-18\]  
053C2C4E    03C6            ADD EAX,ESI  
053C2C12    B9 70083B05     MOV ECX,EXERe.053B0870  
053C2BF2    50              PUSH EAX  
053C2C04    8D041F          LEA EAX,DWORD PTR DS:\[EDI+EBX\]  
053C2CB2    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]  
053C2C76    50              PUSH EAX  
053C2C8A    FF15 262C3C05   CALL DWORD PTR DS:\[53C2C26\]                                   ; EXERe.053A53E1 ( 解压函数 )  
053BE820    8B45 E8         MOV EAX,DWORD PTR SS:\[EBP-18\]  
053BE88E    037D FC         ADD EDI,DWORD PTR SS:\[EBP-4\]  
053BE867    0145 EC         ADD DWORD PTR SS:\[EBP-14\],EAX  
053BE872    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053BE87D    83C0 F8         ADD EAX,-8  
053BE89D    3BF8            CMP EDI,EAX  
053BE857  ^\\0F82 F4FEFFFF   JB EXERe.053BE751  
  
      00168DE8  00 00 40 00 00 10 00 00 2F BF 40 00 00 90 05 00  ..@...../.@.....          ; imgagebase = 400000, baseofcode=1000, oep=40bf2f,  59000 是 ImageSize ?  
      00168DF8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................          ; 0 是 TLS 大小  
      ....  
      00168ED8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  
  
  
053BE84B    8B06            MOV EAX,DWORD PTR DS:\[ESI\]  
053BE840    A3 E4A03B05     MOV DWORD PTR DS:\[53BA0E4\],EAX                                 ; imagebase  
053BE939    8B46 04         MOV EAX,DWORD PTR DS:\[ESI+4\]  
053BE8D0    8945 C0         MOV DWORD PTR SS:\[EBP-40\],EAX                                  ; baseofcode  
  
053BE8F7    8B46 10         MOV EAX,DWORD PTR DS:\[ESI+10\]                                  ; tls size  
053BE922    50              PUSH EAX  
053BE909    A3 DCA03B05     MOV DWORD PTR DS:\[53BA0DC\],EAX  
053BE8E1    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BE94D    6A 08           PUSH 8  
053BE9AE    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BE9E6    F7D0            NOT EAX  
053BE9D7    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap (0 byte)  
  
053BE976    6A 04           PUSH 4                                                        ; 4 byte  
053BE98F    8BF8            MOV EDI,EAX                                                   ; tls 空间  
053BE9F3    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053BE960    6A 08           PUSH 8  
053BE2E7    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BE9C8    F7D0            NOT EAX  
053BE4EC    FFD0            CALL EAX                                                      ; ntdll.RtlAllocateHeap  
053BE544    8945 D0         MOV DWORD PTR SS:\[EBP-30\],EAX  
  
053BE526    8938            MOV DWORD PTR DS:\[EAX\],EDI  
053BE50E    50              PUSH EAX  
053BE913    8B45 D0         MOV EAX,DWORD PTR SS:\[EBP-30\]  
053BE536    64:A3 2C000000  MOV DWORD PTR FS:\[2C\],EAX                                     ; ThreadLocalStoragePointer   \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053BE57F    58              POP EAX  
  
053BE4FA    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053BE8D9    33FF            XOR EDI,EDI  
053BE55E    56              PUSH ESI  
053BE5A6    57              PUSH EDI  
053BE5D3    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BE571    F7D0            NOT EAX  
053BE5BE    FFD0            CALL EAX                                                      ; ntdll.RtlFreeHeap  
  
053BE58F    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053BE366    53              PUSH EBX  
053BE318    57              PUSH EDI  
053BE2FE    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053BE340    F7D0            NOT EAX  
053BE32A    FFD0            CALL EAX                                                      ; ntdll.RtlFreeHeap  
  
  
053BE357    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BE431    6A 01           PUSH 1  
053BE45C    50              PUSH EAX  
053BE442    A1 74A03B05     MOV EAX,DWORD PTR DS:\[53BA074\]  
053BE44C    F7D0            NOT EAX  
053BE4C6    50              PUSH EAX                                                      ; KERNEL32.WriteProcessMemory  
053BE478    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BE4D9    F7D0            NOT EAX  
053BE496    50              PUSH EAX                                                      ; KERNEL32.7C570000  
053BE4AE    FF15 6FE43B05   CALL DWORD PTR DS:\[53BE46F\]                                   ; EXERe.053A102C, ret= 52ec08  
053BE3EC    F7D0            NOT EAX  
053BE3FB    A3 74A03B05     MOV DWORD PTR DS:\[53BA074\],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053BE41B    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BE387    6A 01           PUSH 1  
053BE3A5    50              PUSH EAX  
053BE40C    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053BE3F3    F7D0            NOT EAX  
053BE3C2    50              PUSH EAX                                                      ; KERNEL32.VirtualProtect  
053BE3B1    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BE397    F7D0            NOT EAX  
053BE012    50              PUSH EAX                                                      ; KERNEL32.7C570000  
053BDF74    FF15 E3E33B05   CALL DWORD PTR DS:\[53BE3E3\]                                   ; EXERe.053A102C, ret= 1477f0  
053BE3D5    F7D0            NOT EAX  
053BDF9F    83C4 20         ADD ESP,20  
053BDFB6    A3 78A03B05     MOV DWORD PTR DS:\[53BA078\],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
  
十.  
开始 Hook 20 个 API  
  
053AB5C5  46 69 6E 64 52 65 73 6F 75 72 63 65 45 78 57     FindResourceExW                 ; Jmp 53A31EA  
053AB6E5  46 69 6E 64 52 65 73 6F 75 72 63 65 45 78 41     FindResourceExA                 ; Jmp 53A3264  
053AB805  46 69 6E 64 52 65 73 6F 75 72 63 65 57           FindResourceW  
053AB925  46 69 6E 64 52 65 73 6F 75 72 63 65 41           FindResourceA  
053ABA45  4C 6F 61 64 52 65 73 6F 75 72 63 65              LoadResource  
053ABB65  53 69 7A 65 6F 66 52 65 73 6F 75 72 63 65        SizeofResource  
053ABC85  4C 6F 61 64 49 63 6F 6E 41                       LoadIconA  
053ABDA5  4C 6F 61 64 49 63 6F 6E 57                       LoadIconW  
053ABEC5  4C 6F 61 64 41 63 63 65 6C 65 72 61 74 6F 72 73  LoadAcceleratorsA  
053ABFE5  4C 6F 61 64 41 63 63 65 6C 65 72 61 74 6F 72 73  LoadAcceleratorsW  
053AC105  4C 6F 61 64 42 69 74 6D 61 70 41                 LoadBitmapA  
053AC225  4C 6F 61 64 42 69 74 6D 61 70 57                 LoadBitmapW  
053AC345  4C 6F 61 64 43 75 72 73 6F 72 41                 LoadCursorA  
053AC465  4C 6F 61 64 43 75 72 73 6F 72 57                 LoadCursorW  
053AC585  4C 6F 61 64 4D 65 6E 75 41                       LoadMenuA  
053AC6A5  4C 6F 61 64 4D 65 6E 75 57                       LoadMenuW  
053AC7C5  4C 6F 61 64 53 74 72 69 6E 67 41                 LoadStringA  
053AC8E5  4C 6F 61 64 53 74 72 69 6E 67 57                 LoadStringW  
053ACA05  45 78 69 74 50 72 6F 63 65 73 73                 ExitProcess                     ; Jmp 53A33D8  
053ACB25  43 72 65 61 74 65 54 68 72 65 61 64              CreateThread                    ; Jmp 53A33E8  
  
  
  
  
053BDFAD    33DB            XOR EBX,EBX  
053BDFF1    8D83 C5B53A05   LEA EAX,DWORD PTR DS:\[EBX+53AB5C5\]  
053BDFCF    50              PUSH EAX                                  ; EXERe.053AB5C5  
053BE000    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BDFE5    F7D0            NOT EAX  
053BDE94    50              PUSH EAX                                  ; KERNEL32.7C570000  
053BDF4E    A1 B0A03B05     MOV EAX,DWORD PTR DS:\[53BA0B0\]  
053BDF62    F7D0            NOT EAX  
053BDEFF    FFD0            CALL EAX                                  ; KERNEL32.GetProcAddress  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053BDF0F    8BF0            MOV ESI,EAX                               ; KERNEL32.FindResourceExW  
053BDF44    8D45 D0         LEA EAX,DWORD PTR SS:\[EBP-30\]  
053BDF35    6A 02           PUSH 2                                    ; 2,  HOOK 标志  
053BDF1C    50              PUSH EAX  
053BDEAD    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053BDEBE    56              PUSH ESI                                  ; KERNEL32.FindResourceExW  
053BDEA1    F7D0            NOT EAX  
053BDED9    50              PUSH EAX                                  ; KERNEL32.7C570000  
053BDEEC    8975 F4         MOV DWORD PTR SS:\[EBP-C\],ESI              ; KERNEL32.FindResourceExW  
053BE0AF    FF15 F4DE3B05   CALL DWORD PTR DS:\[53BDEF4\]               ; EXERe.053A102C  
053BE074    8B45 D0         MOV EAX,DWORD PTR SS:\[EBP-30\]             ; size = 5  
053BE08E    83C4 10         ADD ESP,10  
  
053BE09D    8983 DCB63A05   MOV DWORD PTR DS:\[EBX+53AB6DC\],EAX  
053BE05F    8B83 C0B53A05   MOV EAX,DWORD PTR DS:\[EBX+53AB5C0\]        ; EXERe.053A31EA  
053BE06C    2BC6            SUB EAX,ESI                               ; KERNEL32.FindResourceExW  
053BE029    89B3 D8B63A05   MOV DWORD PTR DS:\[EBX+53AB6D8\],ESI        ; KERNEL32.FindResourceExW  
053BDECA    83E8 05         SUB EAX,5  
053BE055    C645 B4 E9      MOV BYTE PTR SS:\[EBP-4C\],0E9  
053BE03E    8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX  
053BE04C    8D45 B5         LEA EAX,DWORD PTR SS:\[EBP-4B\]  
053BE036    85C0            TEST EAX,EAX  
053BE0FB   /0F84 99010000   JE EXERe.053BE29A  
053BE0F2    8D45 F8         LEA EAX,DWORD PTR SS:\[EBP-8\]  
053BE0C0    85C0            TEST EAX,EAX  
053BE0DF   /0F84 B5010000   JE EXERe.053BE29A  
053BE0CF    8D75 F8         LEA ESI,DWORD PTR SS:\[EBP-8\]  
053BE26C    8D7D B5         LEA EDI,DWORD PTR SS:\[EBP-4B\]  
053BE0EA    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES>  
053BE2D3    8B75 F4         MOV ESI,DWORD PTR SS:\[EBP-C\]             ; KERNEL32.FindResourceExW  
053BE0C7    33FF            XOR EDI,EDI  
053BE29A    8D45 E8         LEA EAX,DWORD PTR SS:\[EBP-18\]  
053BE2AE    50              PUSH EAX  
053BE2C8    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053BE287    6A 04           PUSH 4  
053BE228    FFB3 DCB63A05   PUSH DWORD PTR DS:\[EBX+53AB6DC\]  
053BE2BD    F7D0            NOT EAX  
053BE247    56              PUSH ESI                                 ; KERNEL32.FindResourceExW  
053BE1BA    FFD0            CALL EAX                                 ; VirtualProtect  
053BE25F    57              PUSH EDI  
  
053BE1F4    8D83 C4B63A05   LEA EAX,DWORD PTR DS:\[EBX+53AB6C4\]  
053BE210    FFB3 DCB63A05   PUSH DWORD PTR DS:\[EBX+53AB6DC\]          ; size = 5  
053BE1E7    50              PUSH EAX                                 ; EXERe.053AB6C4  
053BE1D4    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053BE123    56              PUSH ESI                                 ; KERNEL32.FindResourceExW  
053BE183    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053BE239    F7D0            NOT EAX  
053BE1A2    FFD0            CALL EAX                                 ; ReadProcessMemory  
053BE108    8B83 DCB63A05   MOV EAX,DWORD PTR DS:\[EBX+53AB6DC\]  
053BE12F    C68403 C4B63A05>MOV BYTE PTR DS:\[EBX+EAX+53AB6C4\],0E9  
053BE156    8BC6            MOV EAX,ESI                              ; KERNEL32.FindResourceExW  
053BE163    2BC3            SUB EAX,EBX  
053BE172    2D C4B63A05     SUB EAX,EXERe.053AB6C4  
053BE13E    83E8 05         SUB EAX,5  
053C2DFE    8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX  
053C2E0C    8B83 DCB63A05   MOV EAX,DWORD PTR DS:\[EBX+53AB6DC\]  
053C2F9D    8D8403 C5B63A05 LEA EAX,DWORD PTR DS:\[EBX+EAX+53AB6C5\]  
053C2FAC    3BC7            CMP EAX,EDI  
053C3043  ^\\0F84 C8FFFFFF   JE EXERe.053C3011  
053C3052    8D4D F8         LEA ECX,DWORD PTR SS:\[EBP-8\]  
053C305A    85C9            TEST ECX,ECX  
053C3029  ^\\0F84 E2FFFFFF   JE EXERe.053C3011  
053C303A    8D75 F8         LEA ESI,DWORD PTR SS:\[EBP-8\]  
053C2FD4    8BF8            MOV EDI,EAX                              ; EXERe.053AB6CA  
053C2FC2    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES>  
053C2FCA    8B75 F4         MOV ESI,DWORD PTR SS:\[EBP-C\]             ; KERNEL32.FindResourceExW  
053C2FB5    33FF            XOR EDI,EDI                              ; EXERe.053AB6CE  
053C3017    57              PUSH EDI  
053C2FF5    8D45 B4         LEA EAX,DWORD PTR SS:\[EBP-4C\]  
053C3003    6A 05           PUSH 5  
053C2FE3    50              PUSH EAX  
053C2E29    A1 74A03B05     MOV EAX,DWORD PTR DS:\[53BA074\]  
053C2F73    56              PUSH ESI                                 ; KERNEL32.FindResourceExW  
053C2F8B    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C2E21    F7D0            NOT EAX  
053C2F3C    FFD0            CALL EAX                                 ; WriteProcessMemory  
                                                                       
                                       7C591786 >- E9 5F1AE188     JMP EXERe.053A31EA  ,  将 FindResourceExW 开头 5 字节改掉  
  
053C2F4F    8D45 E8         LEA EAX,DWORD PTR SS:\[EBP-18\]  
053C2F5F    50              PUSH EAX  
053C2ECC    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053C2F06    FF75 E8         PUSH DWORD PTR SS:\[EBP-18\]  
053C2F23    F7D0            NOT EAX  
053C2EDF    6A 05           PUSH 5  
053C2EF2    56              PUSH ESI                                 ; KERNEL32.FindResourceExW  
053C2E53    FFD0            CALL EAX                                 ; VirtualProtect  
  
  
053C2F2A    81C3 20010000   ADD EBX,120  
053C2F16    81FB 80160000   CMP EBX,1680                             ; 1680/120=14h 个 API  
053C2E3C  ^\\0F82 AFB1FFFF   JB EXERe.053BDFF1  
  
  
  
  
  
十一. 解压代码到原来的 400000 段  
  
053C2E33    8B45 DC         MOV EAX,DWORD PTR SS:\[EBP-24\]                             ; EXERe.053C4100  
053C2E96    8B4D D8         MOV ECX,DWORD PTR SS:\[EBP-28\]                             ; 前面已处理的字节长度  
053C2EAA    57              PUSH EDI  
053C2EBC    6A 04           PUSH 4  
053C2E7A    8D3401          LEA ESI,DWORD PTR DS:\[ECX+EAX\]  
053C2E88    8D45 E4         LEA EAX,DWORD PTR SS:\[EBP-1C\]  
053C2E68    50              PUSH EAX  
053C348D    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053C33D3    56              PUSH ESI                                                  ; EXERe.053D2762  
053C3460    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C3477    F7D0            NOT EAX  
053C3437    FFD0            CALL EAX                                                  ; ReadProcessMemory ( 4 byte )  
  
053C344C    57              PUSH EDI  
053C3484    8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
053C33EB    6A 04           PUSH 4  
053C341C    50              PUSH EAX  
053C3428    8D46 04         LEA EAX,DWORD PTR DS:\[ESI+4\]  
053C3404    50              PUSH EAX                                                  ; EXERe.053D2766  
053C31A1    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053C30F1    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C30CF    F7D0            NOT EAX  
053C30DC    FFD0            CALL EAX                                                   ; ReadProcessMemory ( 4 byte)  
  
053C3073    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]                                ; uncompressed size = 03c000  
053C30AF    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053C30C2    F7D0            NOT EAX  
053C3096    6A 08           PUSH 8  
053C317A    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C3191    FFD0            CALL EAX                                                  ; ntdll.RtlAllocateHeap  
  
053C3130    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                ; compressed size = 0188b2  
053C30B9    8945 FC         MOV DWORD PTR SS:\[EBP-4\],EAX                              ; umcompressed buffer  
053C30A3    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053C3166    6A 08           PUSH 8  
053C3145    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C3156    F7D0            NOT EAX  
053C310E    FFD0            CALL EAX                                                  ; ntdll.RtlAllocateHeap  
  
  
053C3283    57              PUSH EDI  
053C311D    8BD8            MOV EBX,EAX  
053C31B5    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                ; compressed size = 0188B2  
053C3085    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053C320B    83C6 08         ADD ESI,8                                                 ; 跳过 8 字节  
053C31F3    53              PUSH EBX  
053C31CF    56              PUSH ESI                                                  ; EXERe.053D276A  
053C325F    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C3270    F7D0            NOT EAX  
053C3225    FFD0            CALL EAX                                                  ; ReadProcessMemory  
  
053C3215    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053C3203    33F6            XOR ESI,ESI                                               ; EXERe.053D276A  
053C31E3    0145 D8         ADD DWORD PTR SS:\[EBP-28\],EAX                             ; 累加已读入的大小, 为下一个处理做准备  
053C3242    83C0 F8         ADD EAX,-8  
053C324C    85C0            TEST EAX,EAX  
053C3234    897D EC         MOV DWORD PTR SS:\[EBP-14\],EDI  
053C32D9   /0F86 E5010000   JBE EXERe.053C34C4                                        ; 小于 8 字节不用处理  
  
  
053C329B    8B041E          MOV EAX,DWORD PTR DS:\[ESI+EBX\]                            ; 循环解压  
053C328F    8B4C33 04       MOV ECX,DWORD PTR DS:\[EBX+ESI+4\]  
053C32BD    83C6 08         ADD ESI,8  
053C32C9    3BC1            CMP EAX,ECX  
053C32D0    8945 E8         MOV DWORD PTR SS:\[EBP-18\],EAX  
053C32A9    894D F8         MOV DWORD PTR SS:\[EBP-8\],ECX  
053C32B1  ^\\0F85 C0F8FFFF   JNZ EXERe.053C2B77  
053C2B77    8B45 EC         MOV EAX,DWORD PTR SS:\[EBP-14\]  
053C2B47    8B4D FC         MOV ECX,DWORD PTR SS:\[EBP-4\]  
053C2B55    FF75 E8         PUSH DWORD PTR SS:\[EBP-18\]  
053C2B20    03C1            ADD EAX,ECX  
053C2B33    B9 70083B05     MOV ECX,EXERe.053B0870  
053C2AFC    50              PUSH EAX  
053C2B3D    8D041E          LEA EAX,DWORD PTR DS:\[ESI+EBX\]  
053C2AE5    FF75 F8         PUSH DWORD PTR SS:\[EBP-8\]  
053C2C5E    50              PUSH EAX  
053C2BCD    FF15 0E2B3C05   CALL DWORD PTR DS:\[53C2B0E\]                               ; EXERe.053A53E1 (解压代码) \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053C34EA    8B45 E8         MOV EAX,DWORD PTR SS:\[EBP-18\]  
053C34CD    0375 F8         ADD ESI,DWORD PTR SS:\[EBP-8\]  
053C34D9    0145 EC         ADD DWORD PTR SS:\[EBP-14\],EAX  
053C34E1    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053C34B1    83C0 F8         ADD EAX,-8  
053C34BD    3BF0            CMP ESI,EAX  
053C34A0  ^\\0F82 F5FDFFFF   JB EXERe.053C329B  
  
053C34C4    8B75 C0         MOV ESI,DWORD PTR SS:\[EBP-40\]                            ; 修改 401000 读写属性  
053C3497    8D45 D4         LEA EAX,DWORD PTR SS:\[EBP-2C\]  
053C380E    50              PUSH EAX  
053C3780    A1 E4A03B05     MOV EAX,DWORD PTR DS:\[53BA0E4\]  
053C37B0    6A 04           PUSH 4  
053C378E    03C6            ADD EAX,ESI  
053C379B    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]  
053C37D5    50              PUSH EAX  
053C37C3    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053C37E3    F7D0            NOT EAX  
053C37F1    FFD0            CALL EAX                                                ; VirtualProtect(401000, 3c000, 4, ...)   Page\_ReadWrite  
  
  
053C3568    A1 E4A03B05     MOV EAX,DWORD PTR DS:\[53BA0E4\]                          ; 还原代码  
053C352E    57              PUSH EDI  
053C3544    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]                              ; 原来的代码空间 401000  
053C37FE    03C6            ADD EAX,ESI  
053C34FE    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]                               ; uncompressed buffer  
053C355A    50              PUSH EAX  
053C3518    A1 74A03B05     MOV EAX,DWORD PTR DS:\[53BA074\]  
053C3722    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C373B    F7D0            NOT EAX  
053C3755    FFD0            CALL EAX                                                ; WriteProcessMemory(530008->401000, size=3c000) \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053C3746    8D45 D4         LEA EAX,DWORD PTR SS:\[EBP-2C\]                           ; 恢复 401000 属性  
053C376E    50              PUSH EAX  
053C3694    A1 E4A03B05     MOV EAX,DWORD PTR DS:\[53BA0E4\]  
053C36DE    FF75 D4         PUSH DWORD PTR SS:\[EBP-2C\]  
053C36F9    03F0            ADD ESI,EAX  
053C36EC    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053C3706    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]  
053C36AA    F7D0            NOT EAX  
053C36BD    56              PUSH ESI  
053C3617    FFD0            CALL EAX                                                ; VirtualProtect(Page\_Execute\_ReadWrite Page\_guard)  
  
053C3644    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]  
053C36CD    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053C36A2    F7D0            NOT EAX  
053C362C    57              PUSH EDI  
053C3679    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C365E    FFD0            CALL EAX                                                ; ntdll.RtlFreeHeap  
053C35FA    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053C35CA    53              PUSH EBX  
053C35E2    57              PUSH EDI  
053C358F    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C366B    F7D0            NOT EAX  
053C35A7    FFD0            CALL EAX                                                 ; ntdll.RtlFreeHeap  
  
  
  
十二. 读入 IAT 信息  
  
053C3608    8B45 D8         MOV EAX,DWORD PTR SS:\[EBP-28\]                             ; 已处理的长度  
053C35F0    8B4D DC         MOV ECX,DWORD PTR SS:\[EBP-24\]                             ; EXERe.053C4100  
053C357C    57              PUSH EDI  
053C3A48    6A 04           PUSH 4  
053C35BA    8D3408          LEA ESI,DWORD PTR DS:\[EAX+ECX\]  
053C3B0E    8D45 E4         LEA EAX,DWORD PTR SS:\[EBP-1C\]  
053C3B62    50              PUSH EAX  
053C3B4C    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053C3B22    56              PUSH ESI                                                  ; EXERe.053EB014  
053C3B3A    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C3AE3    F7D0            NOT EAX  
053C3AF7    FFD0            CALL EAX                                                  ; ReadProcessMemory(4 byte)  
  
053C3AA9    57              PUSH EDI  
053C3B04    8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
053C3ACA    6A 04           PUSH 4  
053C3A91    50              PUSH EAX  
053C3AB5    8D46 04         LEA EAX,DWORD PTR DS:\[ESI+4\]  
053C3A61    50              PUSH EAX                                                  ; EXERe.053EB018  
053C3AD7    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053C3A79    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C3986    F7D0            NOT EAX  
053C39B1    FFD0            CALL EAX                                                  ; ReadProcessMemory (4 byte)  
  
053C3997    FF75 E4         PUSH DWORD PTR SS:\[EBP-1C\]                                ; uncompressed size = 0F54  
053C39FE    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053C3A14    F7D0            NOT EAX  
053C3A2F    6A 08           PUSH 8  
053C39CA    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C39EB    FFD0            CALL EAX                                                  ; ntdll.RtlAllocateHeap  
  
053C395A    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                ; compressed size = 095F  
053C3A1B    8945 EC         MOV DWORD PTR SS:\[EBP-14\],EAX  
053C3A08    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053C3972    6A 08           PUSH 8  
053C38A4    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C3882    F7D0            NOT EAX  
053C388F    FFD0            CALL EAX                                                  ; ntdll.RtlAllocateHeap  
  
053C383D    8BD8            MOV EBX,EAX  
053C382C    57              PUSH EDI  
053C381C    895D B4         MOV DWORD PTR SS:\[EBP-4C\],EBX  
053C3850    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]                                ; 095F size  
053C3862    A1 88A03B05     MOV EAX,DWORD PTR DS:\[53BA088\]  
053C386E    83C6 08         ADD ESI,8                                                 ; 跳过 8 字节  
053C3912    53              PUSH EBX  
053C392A    56              PUSH ESI                                                  ; EXERe.053EB01C  
053C393C    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C38D7    F7D0            NOT EAX  
053C38F3    FFD0            CALL EAX                                                  ; ReadProcessMemory  
  
053C3902    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053C38E0    33F6            XOR ESI,ESI                                               ; EXERe.053EB01C  
053C38B9    83C0 F8         ADD EAX,-8  
053C38C3    897D DC         MOV DWORD PTR SS:\[EBP-24\],EDI  
053BFDC3    85C0            TEST EAX,EAX  
053BFDE3   /0F86 31010000   JBE EXERe.053BFF1A  
  
053BFDCB    8B041E          MOV EAX,DWORD PTR DS:\[ESI+EBX\]                            ; 循环解压  
053BFE2F    8B4C33 04       MOV ECX,DWORD PTR DS:\[EBX+ESI+4\]  
053BFDD9    83C6 08         ADD ESI,8  
053BFDEE    3BC1            CMP EAX,ECX  
053BFDF9    8945 E8         MOV DWORD PTR SS:\[EBP-18\],EAX  
053BFE05    894D F8         MOV DWORD PTR SS:\[EBP-8\],ECX  
053BFE13   /0F85 6F2B0000   JNZ EXERe.053C2988  
053C2988    8B45 DC         MOV EAX,DWORD PTR SS:\[EBP-24\]  
053C2953    8B4D EC         MOV ECX,DWORD PTR SS:\[EBP-14\]  
053C2D9C    FF75 E8         PUSH DWORD PTR SS:\[EBP-18\]  
053C2DD0    03C1            ADD EAX,ECX  
053C2DDA    B9 70083B05     MOV ECX,EXERe.053B0870  
053C2DEC    50              PUSH EAX  
053C2DB2    8D041E          LEA EAX,DWORD PTR DS:\[ESI+EBX\]  
053C2DC0    FF75 F8         PUSH DWORD PTR SS:\[EBP-8\]  
053C2BA7    50              PUSH EAX  
053C2B8C    FF15 A0283C05   CALL DWORD PTR DS:\[53C28A0\]                               ; EXERe.053A53E1  解压( compressed, size1,  uncompress, size2) \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053BFFC7    8B45 E8         MOV EAX,DWORD PTR SS:\[EBP-18\]  
053BFFBE    0375 F8         ADD ESI,DWORD PTR SS:\[EBP-8\]  
053BFF0C    0145 DC         ADD DWORD PTR SS:\[EBP-24\],EAX  
053BFEE3    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]  
053BFED8    83C0 F8         ADD EAX,-8  
053BFF02    3BF0            CMP ESI,EAX  
053BFF3F  ^\\0F82 86FEFFFF   JB EXERe.053BFDCB  
  
  
  
十三.  
 处理 IAT  
  
053BFF20    FF75 EC         PUSH DWORD PTR SS:\[EBP-14\]                                ; 解压后的 IAT 信息  
053BFF2E    8D4D 8C         LEA ECX,DWORD PTR SS:\[EBP-74\]                             ; 12FF4C  
053BFF70    FF15 F9333C05   CALL DWORD PTR DS:\[53C33F9\]                               ; EXERe.053A486D, 初步解密  
  
                0012FF4C   00000007  ....  ; 7 个 DLL  
                0012FF50   00146240  .)..  ; 开始地址, 每个 DLL 4 + 4 + 40h\*4 + 4 + 4 + 4 byte { A, B, C, D, E, F }  
                0012FF54   001469cc  T1..  ; 结束地址  
                  
                其中 A = 该 DLL 中 API　个数  
                　　 B = 加壳前 FirstThunk 位置 (RVA)  
                     C = DLL Name  
                     D = 指向 A 个 API　信息(每个 API８字节)  
                     E = D+8  
                     F = A  
  
053BFF36    8D45 E0         LEA EAX,DWORD PTR SS:\[EBP-20\]  
053BFF96    6A 01           PUSH 1  
053BFF57    50              PUSH EAX  
053BFEF3    A1 ACA03B05     MOV EAX,DWORD PTR DS:\[53BA0AC\]  
053C0266    F7D0            NOT EAX  
053C002D    50              PUSH EAX                                                  ; KERNEL32.CreateThread  
053C001A    A1 60A03B05     MOV EAX,DWORD PTR DS:\[53BA060\]  
053C000A    F7D0            NOT EAX  
053C0232    50              PUSH EAX                                                  ; KERNEL32.7C570000  
053C0253    FF15 E4323C05   CALL DWORD PTR DS:\[53C32E4\]                               ; EXERe.053A102C,   ret = 510db0  
053C023F    F7D0            NOT EAX  
053C00BD    A3 ACA03B05     MOV DWORD PTR DS:\[53BA0AC\],EAX                            ;替换原来的入口,  SKIP, \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053C005C    A1 94A03B05     MOV EAX,DWORD PTR DS:\[53BA094\]  
053C003F    83C4 10         ADD ESP,10  
053C0054    F7D0            NOT EAX  
053C009D    FFD0            CALL EAX                                                  ; GetTickCount \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053C00AC    397D 8C         CMP DWORD PTR SS:\[EBP-74\],EDI                             ; DLL 个数 == 0 ?  
053C0087    A3 E0A03B05     MOV DWORD PTR DS:\[53BA0E0\],EAX                            ; 时间标志,     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053BFF87    897D F4         MOV DWORD PTR SS:\[EBP-C\],EDI                              ; 外循环变量  
053C006B   /0F8E E4250000   JLE EXERe.053C2655  
053C007C    897D E8         MOV DWORD PTR SS:\[EBP-18\],EDI  
  
  
                                                                                       ; 处理 IAT 的外循环 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053C0155    8B45 90         MOV EAX,DWORD PTR SS:\[EBP-70\]                              ; 开始地址                            
053C010C    8B4D E8         MOV ECX,DWORD PTR SS:\[EBP-18\]  
053C013B    03C1            ADD EAX,ECX  
053C0129    6A 40           PUSH 40  
053C0143    8DBD 7CFEFFFF   LEA EDI,DWORD PTR SS:\[EBP-184\]                             ; buffer1 in stack(100h byte)  
053C011B    8B08            MOV ECX,DWORD PTR DS:\[EAX\]  
053C00EA    8D70 08         LEA ESI,DWORD PTR DS:\[EAX+8\]  
053C00F2    898D 74FEFFFF   MOV DWORD PTR SS:\[EBP-18C\],ECX                             ; A                                
053C0103    8B48 04         MOV ECX,DWORD PTR DS:\[EAX+4\]  
053C00DC    898D 78FEFFFF   MOV DWORD PTR SS:\[EBP-188\],ECX                             ; B  
053C01C3    59              POP ECX                                                    ; 40h  
053C00D0    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]             ; C,  DLL name 复制到 buffer1  
  
053C01FA    05 08010000     ADD EAX,108                                                ; D  
053C020A    8D8D 7CFFFFFF   LEA ECX,DWORD PTR SS:\[EBP-84\]                              ; buffer2 in stack  
053C01E2    50              PUSH EAX                                                     
053C0177    FF15 34303C05   CALL DWORD PTR DS:\[53C3034\]                                ; EXERe.053A4A58,  把 D 所指内容复制到 buffer2  
  
053C021B    8D45 D0         LEA EAX,DWORD PTR SS:\[EBP-30\]  
053C01EE    8B8D 78FEFFFF   MOV ECX,DWORD PTR SS:\[EBP-188\]                             ; B, FirstThunk  
053C0164    50              PUSH EAX  
053C0191    8B85 74FEFFFF   MOV EAX,DWORD PTR SS:\[EBP-18C\]                             ; A, API 个数  
053C01AA    C1E0 02         SHL EAX,2                                                  ; \*4  
053C074C    6A 04           PUSH 4  
053C0334    50              PUSH EAX  
053C0303    A1 E4A03B05     MOV EAX,DWORD PTR DS:\[53BA0E4\]                             ; ImageBase(未加壳的)  
053C0224    03C8            ADD ECX,EAX  
053C0313    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053C0282    51              PUSH ECX  
053C031E    F7D0            NOT EAX  
053C029D    FFD0            CALL EAX                                                   ; VirtualProtect( 420080, size=1F4, ReadWrite)  
  
053C02CB    8D85 7CFEFFFF   LEA EAX,DWORD PTR SS:\[EBP-184\]                            ; buffer1, DLL name  
053C02ED    6A 00           PUSH 0  
  
053C02BC    50              PUSH EAX  
053C049F    FF15 4A2F3C05   CALL DWORD PTR DS:\[53C2F4A\]                               ; EXERe.053A46A7, uppercase  
053C0448    59              POP ECX                                                   ;   
053C0469    50              PUSH EAX  
053C0482    FF15 902E3C05   CALL DWORD PTR DS:\[53C2E90\]                               ; EXERe.053A7509(arg1, arg2)   
                                                                                      ; 对 string 做 Hash 变换  arg2=0 or 4  
                                                                                      ; arg2=4 则只取 arg1 前 4 字节, 否则 arg1 null 结束  
                                                                                        
053C03BE    59              POP ECX                                                     
053C0459    8945 D4         MOV DWORD PTR SS:\[EBP-2C\],EAX                             ; Hash 变换的结果  
053C03E2    59              POP ECX  
  
053C03CF    8D85 7CFEFFFF   LEA EAX,DWORD PTR SS:\[EBP-184\]                            ; Dll Name  
053C03F4    50              PUSH EAX  
053C02DB    B9 188C3B05     MOV ECX,EXERe.053B8C18                                    ; 关键全局变量  
053C0422    FF15 752E3C05   CALL DWORD PTR DS:\[53C2E75\]                               ; EXERe.053A5514, MyLoadLibrary, 见 十五 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053C040D    8B85 74FEFFFF   MOV EAX,DWORD PTR SS:\[EBP-18C\]                            ; A, API 个数  
053C0439    33C9            XOR ECX,ECX  
053C02AA    85C0            TEST EAX,EAX  
053C0380    894D FC         MOV DWORD PTR SS:\[EBP-4\],ECX                              ; 循环变量  
053C03A4   /0F8E F6210000   JLE EXERe.053C25A0                                        ; =0 不用处理  
  
  
                                                                                      ; 处理 DLL 的内循环开始 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
                                                                                        
             00145150  21 33 1A 5D 00 E1 5D 01 53 96 96 70 00 E1 5D 01                ; buffer2 ( 每个 API ８字节）{ G, H }  
             00145160  6B A8 3C 3E 00 E1 5D 01 B4 11 B1 14 00 E1 5D 01                ; G = 053A7509(API Name, 0)  
                                                                                      ; H = 0  API name 导入  
                                                                                      ; H = 1  API ordinal 导入?  猜测, 没遇到这种情况  
  
  
053C038E    8B85 7CFFFFFF   MOV EAX,DWORD PTR SS:\[EBP-84\]                             ; buffer2                            
053C0370    807CC8 04 00    CMP BYTE PTR DS:\[EAX+ECX\*8+4\],0                           ; API 序号的循环变量  
053C0352    8B1CC8          MOV EBX,DWORD PTR DS:\[EAX+ECX\*8\]                          ; G                           
053C0360   /0F85 F5260000   JNZ EXERe.053C2A5B                                        ; 没遇到这种情况  
053C061C    53              PUSH EBX                                                  ; G  
053C064C    FF15 CA033C05   CALL DWORD PTR DS:\[53C03CA\]                               ; EXERe.053A4848, 查表 x -> y  
  
        查表 根据 x 找 y  
  
        053A4848   .  33C9          XOR ECX,ECX  
        053A484A   .  B8 40CC3A05   MOV EAX,EXERe.053ACC40                             ; 表头  
        053A484F   >  8B5424 04     MOV EDX,DWORD PTR SS:\[ESP+4\]                       ; x  
        053A4853   .  3B10          CMP EDX,DWORD PTR DS:\[EAX\]  
        053A4855   .  74 0E         JE SHORT EXERe.053A4865  
        053A4857   .  83C0 08       ADD EAX,8                                          ; 每项 8 字节 (x, y)   
        053A485A   .  41            INC ECX  
        053A485B   .  3D E0CC3A05   CMP EAX,EXERe.053ACCE0                             ; 表尾  
        053A4860   .^ 72 ED         JB SHORT EXERe.053A484F  
        053A4862   .  33C0          XOR EAX,EAX                                        ; 没找到, 返回 0  
        053A4864   .  C3            RETN  
        053A4865   >  8B04CD 44CC3A>MOV EAX,DWORD PTR DS:\[ECX\*8+53ACC44\]               ; 找到,   返回 y                               
        053A486C   .  C3            RETN  
  
        053ACC40  96 FD 5E AA EA 31 3A 05 6F 4B 11 7D 64 32 3A 05                      ; { M, N }, 20 个被 HOOK 的 API  
        053ACC50  62 A8 23 1F FA 30 3A 05 FF 90 B7 1F 72 31 3A 05  
        053ACC60  E2 EF 3B 16 A7 27 3A 05 FF 42 13 8F E9 28 3A 05  
        053ACC70  33 39 41 0F 72 29 3A 05 FB FE 29 D3 5C 2A 3A 05  
        053ACC80  85 B1 18 A3 46 2B 3A 05 16 F4 1A 95 67 2C 3A 05  
        053ACC90  B8 B4 02 41 88 2D 3A 05 B1 AF 72 41 60 2E 3A 05  
        053ACCA0  11 2F 36 89 38 2F 3A 05 E3 E3 C4 F0 80 2F 3A 05  
        053ACCB0  23 40 A1 5E C8 2F 3A 05 87 61 9A 2E DE 30 3A 05  
        053ACCC0  D0 00 31 EA DE 32 3A 05 3D 4C 8C 15 41 33 3A 05  
        053ACCD0  47 E2 B2 B1 D8 33 3A 05 E2 F9 E6 84 E8 33 3A 05  
  
  
053C0634    59              POP ECX  
053C03AF    85C0            TEST EAX,EAX  
053C028E    8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX  
053C0340   /0F85 38080000   JNZ EXERe.053C0B7E                                         ; 如果找到了, 直接去 53C0B7E ( 这是 20 个完全被偷到壳里的 API) \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053C0273    8B45 D4         MOV EAX,DWORD PTR SS:\[EBP-2C\]                              ; Dll name Hash 变换的结果  
053C072A    99              CDQ                                                        ; 符号扩展  
053C0188    8BC8            MOV ECX,EAX  
053C0736    8BF2            MOV ESI,EDX  
053C00C7    8BC3            MOV EAX,EBX                                                ; G  
053C0697    56              PUSH ESI  
053C01D5    99              CDQ  
053C066F    51              PUSH ECX  
053C06D6    56              PUSH ESI  
053C06AF    51              PUSH ECX  
053C06C6    8945 C8         MOV DWORD PTR SS:\[EBP-38\],EAX  
053C0681    8955 CC         MOV DWORD PTR SS:\[EBP-34\],EDX  
053C06F4    FF15 B1023C05   CALL DWORD PTR DS:\[53C02B1\]                               ; EXERe.053A74D2( 32 位带符号乘法） Hash \* Hash  
053C070C    FF75 CC         PUSH DWORD PTR SS:\[EBP-34\]  
053C071D    8BF8            MOV EDI,EAX                                               ; 结果 ESI:EDI  
053C0552    8BF2            MOV ESI,EDX  
053C0534    FF75 C8         PUSH DWORD PTR SS:\[EBP-38\]  
053C04EA    FF75 CC         PUSH DWORD PTR SS:\[EBP-34\]  
053C0510    FF75 C8         PUSH DWORD PTR SS:\[EBP-38\]  
053C04BD    FF15 B8013C05   CALL DWORD PTR DS:\[53C01B8\]                               ; EXERe.053A74D2( 32 位带符号乘法） G \* G  
                                                                                      ; 结果 EDX:EAX  
053C0595    8365 AC 00      AND DWORD PTR SS:\[EBP-54\],0                               ; \[12FF6C\]=0  
053C06E2    8945 A0         MOV DWORD PTR SS:\[EBP-60\],EAX                             ; G\*G 结果低位  
053C065D    8BC6            MOV EAX,ESI  
053C0577    83C4 20         ADD ESP,20  
053C055F    C1F8 1F         SAR EAX,1F                                                ; 取符号位  
053C0542    8BC2            MOV EAX,EDX  
053C05B4    6A 02           PUSH 2  
053C051E    8975 A4         MOV DWORD PTR SS:\[EBP-5C\],ESI                             ; Hash \* Hash 结果高位  
053C04D4    897D 9C         MOV DWORD PTR SS:\[EBP-64\],EDI                             ; Hash \* Hash 结果低位  
053C05C7    C1F8 1F         SAR EAX,1F  
053C05E5    8365 B0 00      AND DWORD PTR SS:\[EBP-50\],0                               ; \[12FF70\]=0  
053C0606    8955 A8         MOV DWORD PTR SS:\[EBP-58\],EDX                             ; G\*G 结果高位  
053C1126    5E              POP ESI                                                   ; ESI = 2  
  
            0012FF5C   3302ED49    
            0012FF60   66EA2A41  
            0012FF64   1052C68C  
            0012FF68   21DC0BD4  
            0012FF6C   00000000  
            0012FF70   00000000  
  
053C10E3    A1 94A03B05     MOV EAX,DWORD PTR DS:\[53BA094\]                           ; 打乱上面 6 个数顺序  
053C10FB    F7D0            NOT EAX  
053C0896    FFD0            CALL EAX                                                 ; GetTickCount \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053C1108    8945 DC         MOV DWORD PTR SS:\[EBP-24\],EAX                            ; 时间  
053C07ED    8D4E FF         LEA ECX,DWORD PTR DS:\[ESI-1\]                             ; ECX=ESI-1  
053C081D    99              CDQ                                                        
053C080B    F7F9            IDIV ECX  
053C0857    6A 06           PUSH 6  
053C0872    59              POP ECX                                                  ; ECX=6  
053C0833    6A 06           PUSH 6  
053C07C1    5F              POP EDI                                                  ; EDI=6  
053C07FB    99              CDQ  
053C07D4    F7F9            IDIV ECX  
053C0787    8B45 DC         MOV EAX,DWORD PTR SS:\[EBP-24\]                            ; 时间  
053C07A3    8BCA            MOV ECX,EDX                                              ; /ESI-1  /6 余数  
053C0795    99              CDQ  
053C0778    F7FE            IDIV ESI  
053C0760    99              CDQ  
053C0568    F7FF            IDIV EDI  
053C1014    8BFA            MOV EDI,EDX                                              ; /ESI  /6 余数  
053C0502    3BCF            CMP ECX,EDI                                              ; 这两个余数比较  
053C1088  ^\\0F84 FEEDFFFF   JE EXERe.053BFE8C  
  
053C109D    8B548D 9C       MOV EDX,DWORD PTR SS:\[EBP+ECX\*4-64\]                      ; 将 12FF5C 6 个数中 第 ECX 个 和 第 EDI 个 交换顺序  
053C1093    8D448D 9C       LEA EAX,DWORD PTR SS:\[EBP+ECX\*4-64\]  
053C10CE    8D4CBD 9C       LEA ECX,DWORD PTR SS:\[EBP+EDI\*4-64\]  
053C10D8    8B7CBD 9C       MOV EDI,DWORD PTR SS:\[EBP+EDI\*4-64\]  
053C10BC    8938            MOV DWORD PTR DS:\[EAX\],EDI  
053C10C3    8911            MOV DWORD PTR DS:\[ECX\],EDX  
  
053BFE8C    46              INC ESI  
053BFDBC    46              INC ESI  
053C10A8    83FE 16         CMP ESI,16  
053C10B0   /0F8C 2D000000   JL EXERe.053C10E3  
  
   
      0012FF5C   21DC0BD4  ; 最终的结果  
      0012FF60   00000000    
      0012FF64   66EA2A41  
      0012FF68   3302ED49  
      0012FF6C   1052C68C  
      0012FF70   00000000  
  
053C1037    6A 05           PUSH 5                                                   ; 找出 2 个零  
053C1028    33FF            XOR EDI,EDI  
053C106E    33C9            XOR ECX,ECX  
053C107B    58              POP EAX                                                  ; EAX = 5  
053C105A    8D55 B0         LEA EDX,DWORD PTR SS:\[EBP-50\]                            ; 6 个数中最后一个  
053C1062    8D75 9C         LEA ESI,DWORD PTR SS:\[EBP-64\]                            ; 6 个数中最前一个  
053C1048    393E            CMP DWORD PTR DS:\[ESI\],EDI  
053C0CB5   /0F84 13030000   JE EXERe.053C0FCE  
053C1053    41              INC ECX  
053C0EE4    83C6 04         ADD ESI,4  
053C0FCE    393A            CMP DWORD PTR DS:\[EDX\],EDI  
053C0FFD  ^\\0F84 E8FFFFFF   JE EXERe.053C0FEB  
053C1021    48              DEC EAX  
053C1008    83EA 04         SUB EDX,4  
053C0FEB    393E            CMP DWORD PTR DS:\[ESI\],EDI  
053C0FD5   /0F85 78000000   JNZ EXERe.053C1053  
053C0FF2    393A            CMP DWORD PTR DS:\[EDX\],EDI  
053C0F59   /0F85 E9000000   JNZ EXERe.053C1048                                    
                                                                                     ; 最终到这里, \[ESI\]=\[EDX\]=0, ECX=1, EAX=5 表示序号  
053C0FE1    8B55 D4         MOV EDX,DWORD PTR SS:\[EBP-2C\]                            ; DLL name hash 结果                            
053C0F74    6A 20           PUSH 20  
053C0F64    89548D 9C       MOV DWORD PTR SS:\[EBP+ECX\*4-64\],EDX                      ; Hash,  DLL Name Hash 在前, 这个很重要   
053C0FB9    895C85 9C       MOV DWORD PTR SS:\[EBP+EAX\*4-64\],EBX                      ; G  
  
      0012FF5C   21DC0BD4    
      0012FF60   BF5B46E3  Hash  
      0012FF64   66EA2A41    
      0012FF68   3302ED49  
      0012FF6C   1052C68C  
      0012FF70   5D1A3321  G  
  
  
053C0FC2    8B45 DC         MOV EAX,DWORD PTR SS:\[EBP-24\]                            ; 上面最后一次循环的时间  
053C0F98    59              POP ECX                                                  ; ecx = 20h  
053C0FAB    99              CDQ  
053C0FB1    F7F9            IDIV ECX                                                 ; 时间/20h  
053C0F86    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053C0FA4    42              INC EDX                                                  ; 余数+1  
053C0F05    F7D0            NOT EAX  
053C0EF4    C1E2 0A         SHL EDX,0A                                               ; \* 1024  
053C0F36    52              PUSH EDX                                                 ; 字节数 (1-20h) Kbyte  
053C0F4B    6A 08           PUSH 8  
053C0F16    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C0D8E    FFD0            CALL EAX                                                  ; ntdll.RtlAllocateHeap (用时间生产垃圾空间, 确报下面的地址每次都不一样)  
  
053C0D3B    6A 32           PUSH 32                         
053C0F28    8BF0            MOV ESI,EAX                                               ; temp1  
053C0D61    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053C0D7A    6A 08           PUSH 8  
053C0CF5    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C0D55    F7D0            NOT EAX  
053C0D21    FFD0            CALL EAX                                                  ; ntdll.RtlAllocateHeap (32h byte)  
  
053C0CD3    56              PUSH ESI                                                  ; temp1  
053C0D0C    8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX                              ; 有用的空间 buffer1  
053C0CE0    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053C0E70    6A 00           PUSH 0  
053C0E92    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C0ECE    F7D0            NOT EAX  
053C0EB0    FFD0            CALL EAX                                                  ; ntdll.RtlFreeHeap (释放垃圾空间 temp1 )  
  
  
                                                                                      ; 准备工作都做好了, 为这个 API 生成一段代码  
053C0EBE    8B5D F8         MOV EBX,DWORD PTR SS:\[EBP-8\]                              ; buffer1  
053C0E34    6A 02           PUSH 2  
053C0E4F    5E              POP ESI                                                   ; esi = 2  
053C0E61    8D4D 9C         LEA ECX,DWORD PTR SS:\[EBP-64\]                             ; 12FF5C  
053C0DCE    6A 06           PUSH 6  
053C0CC2    C603 60         MOV BYTE PTR DS:\[EBX\],60                                  ; pushad  
053C0DB8    C643 01 9C      MOV BYTE PTR DS:\[EBX+1\],9C                                ; pushfd  
053C0DAA    8D43 02         LEA EAX,DWORD PTR DS:\[EBX+2\]  
053C0E0B    5A              POP EDX                                                   ; edx =6 个数  
  
053C0E18    C600 68         MOV BYTE PTR DS:\[EAX\],68                                  ; push xxxxxxxx                                  
053C0E27    46              INC ESI                                                   ; 目前代码长度  
053C0DE8    40              INC EAX  
053C0DF5    8975 D8         MOV DWORD PTR SS:\[EBP-28\],ESI                               
053C0934  ^\\0F84 83FFFFFF   JE EXERe.053C08BD  
053C0907    85C9            TEST ECX,ECX                                              ; 12FF5C  
053C090F  ^\\0F84 A8FFFFFF   JE EXERe.053C08BD  
053C0927    8BF1            MOV ESI,ECX  
053C08E3    8BF8            MOV EDI,EAX  
053C0920    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                ; 复制  
053C08F1    8B75 D8         MOV ESI,DWORD PTR SS:\[EBP-28\]  
053C08BD    83C6 04         ADD ESI,4  
053C08C7    83C0 04         ADD EAX,4  
053C0A3B    83C1 04         ADD ECX,4  
053C08DC    4A              DEC EDX  
053C09DB   /0F85 37040000   JNZ EXERe.053C0E18                                         ; 循环 6 次  
  
  
053C0A2C    A1 B4A03B05     MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053C0A14    6A 04           PUSH 4  
053C09ED    6A 08           PUSH 8  
053C097E    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C09FB    F7D0            NOT EAX  
053C095D    FFD0            CALL EAX                                                  ; ntdll.RtlAllocateHeap ( 4byte)  
  
053C096A    C60433 68       MOV BYTE PTR DS:\[EBX+ESI\],68  
053C0949    46              INC ESI  
053C09CD    8945 DC         MOV DWORD PTR SS:\[EBP-24\],EAX                             ; buffer2  
   
053C09B1    8975 D8         MOV DWORD PTR SS:\[EBP-28\],ESI  
053C09C3    8D3C33          LEA EDI,DWORD PTR DS:\[EBX+ESI\]  
053C0990    85FF            TEST EDI,EDI  
053C09A1   /0F84 4A010000   JE EXERe.053C0AF1  
053C0B43    8D45 DC         LEA EAX,DWORD PTR SS:\[EBP-24\]  
053C0A77    85C0            TEST EAX,EAX  
053C0A58   /0F84 93000000   JE EXERe.053C0AF1  
053C0A6D    8D75 DC         LEA ESI,DWORD PTR SS:\[EBP-24\]  
053C0A65    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                ; 复制  
053C0A4A    8B75 D8         MOV ESI,DWORD PTR SS:\[EBP-28\]               
053C0AF1    83C6 04         ADD ESI,4                                                 ; 目前代码长度  
  
053C0B09    B8 B7473A05     MOV EAX,EXERe.053A47B7                                    ; 计算 API 真正地址的关键 CALL, 见 十五 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053C0AF9    C60433 E8       MOV BYTE PTR DS:\[EBX+ESI\],0E8  
053C0B38    2B45 F8         SUB EAX,DWORD PTR SS:\[EBP-8\]                              ; 计算到 CALL 的偏移  
053C0B2B    46              INC ESI  
053C0A9B    83E8 2A         SUB EAX,2A  
053C0A8B    8975 D8         MOV DWORD PTR SS:\[EBP-28\],ESI  
053C0B21    8D3C33          LEA EDI,DWORD PTR DS:\[EBX+ESI\]  
053C0ABE    8945 C0         MOV DWORD PTR SS:\[EBP-40\],EAX  
053C0AB4    85FF            TEST EDI,EDI  
053C0ADB   /0F84 47010000   JE EXERe.053C0C28  
053C0AE7    8D45 C0         LEA EAX,DWORD PTR SS:\[EBP-40\]  
053C0AAA    85C0            TEST EAX,EAX  
053C0AC9   /0F84 59010000   JE EXERe.053C0C28  
053C0C12    8D75 C0         LEA ESI,DWORD PTR SS:\[EBP-40\]  
053C0AD4    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                ; 复制成 CALL 53A47B7  
053C0C33    8B75 D8         MOV ESI,DWORD PTR SS:\[EBP-28\]  
053C0C28    83C6 04         ADD ESI,4                                                 ; 目前代码长度  
053C0C1D    C60433 9D       MOV BYTE PTR DS:\[EBX+ESI\],9D                              ; POPFD  
053C0C88    46              INC ESI  
053C0C9B    C60433 61       MOV BYTE PTR DS:\[EBX+ESI\],61                              ; POPAD  
053C0CA8    804C33 01 FF    OR BYTE PTR DS:\[EBX+ESI+1\],0FF                            ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
053C0C92    46              INC ESI  
053C0C7C    8D5C33 01       LEA EBX,DWORD PTR DS:\[EBX+ESI+1\]  
053C0C3F    8D7B 01         LEA EDI,DWORD PTR DS:\[EBX+1\]  
053C0C72    C603 25         MOV BYTE PTR DS:\[EBX\],25                                  ; FF25 xxxx,  jmp \[xxxx\]  
053C0C60    85FF            TEST EDI,EDI  
053C0C52  ^\\0F84 26FFFFFF   JE EXERe.053C0B7E  
053C0C68    8D45 DC         LEA EAX,DWORD PTR SS:\[EBP-24\]  
053C0C48    85C0            TEST EAX,EAX  
053C0C04  ^\\0F84 74FFFFFF   JE EXERe.053C0B7E  
053C0BF4    8D75 DC         LEA ESI,DWORD PTR SS:\[EBP-24\]  
053C0BDF    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]                ; 到这里 buffer1 如下  
  
                        00163DB8    60              PUSHAD                            ; buffer1        
                        00163DB9    9C              PUSHFD  
                        00163DBA    68 49ED0233     PUSH 3302ED49  
                        00163DBF    68 D40BDC21     PUSH 21DC0BD4  
                        00163DC4    68 8CC65210     PUSH 1052C68C  
                        00163DC9    68 412AEA66     PUSH 66EA2A41  
                        00163DCE    68 E3465BBF     PUSH BF5B46E3  
                        00163DD3    68 21331A5D     PUSH 5D1A3321  
                        00163DD8    68 88431400     PUSH 144388                       ; buffer2  
                        00163DDD    E8 D5092405     CALL EXERe.053A47B7  
                        00163DE2    9D              POPFD  
                        00163DE3    61              POPAD  
                        00163DE4    FF25 88431400   JMP DWORD PTR DS:\[144388\]         ; buffer2  
  
  
053C0B7E    8B4D FC         MOV ECX,DWORD PTR SS:\[EBP-4\]                              ; 循环变量, API 序号  
053C0B59    6A 00           PUSH 0  
053C0B88    8D45 F8         LEA EAX,DWORD PTR SS:\[EBP-8\]                              ; buffer1 保存在这  
053C0BC5    6A 04           PUSH 4  
053C0B9F    50              PUSH EAX  
053C0BAD    8B85 78FEFFFF   MOV EAX,DWORD PTR SS:\[EBP-188\]                            ; FirstThunk 位置 (RVA)  
053C24D9    8D0488          LEA EAX,DWORD PTR DS:\[EAX+ECX\*4\]  
053C2822    0305 E4A03B05   ADD EAX,DWORD PTR DS:\[53BA0E4\]                            ; ImageBase (加壳前的)  
053C25DF    50              PUSH EAX  
053C2538    A1 74A03B05     MOV EAX,DWORD PTR DS:\[53BA074\]  
053C2504    FF35 D8A03B05   PUSH DWORD PTR DS:\[53BA0D8\]  
053C2522    F7D0            NOT EAX  
053C2565    FFD0            CALL EAX                                                  ; WriteProcessMemory ( 写 4 字节, 填充 FirstThunk 指向 buffer1 )  
  
  
                                  
053C2517    FF45 FC         INC DWORD PTR SS:\[EBP-4\]                              ; 循环变量 ++  
053C254D    8B85 74FEFFFF   MOV EAX,DWORD PTR SS:\[EBP-18C\]  
053C2544    3945 FC         CMP DWORD PTR SS:\[EBP-4\],EAX                          ; API 个数  
053C24EC   /0F8C 22050000   JL EXERe.053C2A14                                     ; 循环出口  
  
053C2A14    8B4D FC         MOV ECX,DWORD PTR SS:\[EBP-4\]  
053C2A48  ^\\E9 41D9FFFF     JMP EXERe.053C038E                                    ; 继续下一个 API   
  
  
053C257E    8B5D B4         MOV EBX,DWORD PTR SS:\[EBP-4C\]                         ; 内循环结束到这里,  EBX 下面清场用  
053C25A0    8D4D D0         LEA ECX,DWORD PTR SS:\[EBP-30\]  
053C25BC    51              PUSH ECX  
053C2588    8B8D 78FEFFFF   MOV ECX,DWORD PTR SS:\[EBP-188\]  
053C2748    FF75 D0         PUSH DWORD PTR SS:\[EBP-30\]  
053C278D    C1E0 02         SHL EAX,2  
053C275E    50              PUSH EAX  
053C2771    A1 E4A03B05     MOV EAX,DWORD PTR DS:\[53BA0E4\]  
053C25AE    03C8            ADD ECX,EAX  
053C2595    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053C2800    51              PUSH ECX  
053C2811    F7D0            NOT EAX  
053C27E5    FFD0            CALL EAX                                             ; VirtualProtect( 420080 ) 恢复   
  
053C2818    FF45 F4         INC DWORD PTR SS:\[EBP-C\]                             ; 外循环变量 ++  
053C27AF    8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]  
053C279F    8145 E8 1401000>ADD DWORD PTR SS:\[EBP-18\],114                        ; 每个 DLL 114h byte  
053C27D3    3B45 8C         CMP EAX,DWORD PTR SS:\[EBP-74\]                        ; DLL 个数  
053C27BB  ^\\0F8C 94D9FFFF   JL EXERe.053C0155                                    ; NL 外循环结束  
  
  
053C27F2    33FF            XOR EDI,EDI                                          ; 清理现场  
053C265B    FF75 EC         PUSH DWORD PTR SS:\[EBP-14\]  
053C261D    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053C263B    F7D0            NOT EAX  
053C25F5    57              PUSH EDI  
053C270F    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C272E    FFD0            CALL EAX                                             ; ntdll.RtlFreeHeap  
  
053C2609    A1 90A03B05     MOV EAX,DWORD PTR DS:\[53BA090\]  
053C2673    53              PUSH EBX  
053C26CF    57              PUSH EDI  
053C26EC    FF35 D4A03B05   PUSH DWORD PTR DS:\[53BA0D4\]  
053C262D    F7D0            NOT EAX  
053C26B2    FFD0            CALL EAX                                             ; ntdll.RtlFreeHeap  
  
  
  
  
  
十四. OEP  
  
053C2698    8D45 BC         LEA EAX,DWORD PTR SS:\[EBP-44\]  
053C2ACD    50              PUSH EAX  
053C2854    A1 78A03B05     MOV EAX,DWORD PTR DS:\[53BA078\]  
053C283C    68 80000000     PUSH 80  
053C2887    FF35 E8A03B05   PUSH DWORD PTR DS:\[53BA0E8\]  
053C2871    F7D0            NOT EAX  
053C29EF    FF35 E4A03B05   PUSH DWORD PTR DS:\[53BA0E4\]  
053C2A83    FFD0            CALL EAX                                              ; VirtualProtect(400000, 10B000, Page\_Exectue\_writecopy)  
  
053C2AA5    FF75 88         PUSH DWORD PTR SS:\[EBP-78\]                            ; 1461F8    注意这个地址不固定  
053C2A2D    C3              RETN                                                  ; Stolen oep  
  
  
整理好的被偷代码  
  
40BF2F-40C031  
  
  
0040BF2F    55              PUSH EBP  
0040BF30    8BEC            MOV EBP,ESP  
0040BF32    6A FF           PUSH -1  
0040BF34    68 78194200     PUSH 421978  
0040BF39    68 54034100     PUSH 410354  
0040BF3E    64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]  
0040BF44    50              PUSH EAX  
0040BF45    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
0040BF4C    83EC 58         SUB ESP,58  
0040BF4F    53              PUSH EBX  
0040BF50    56              PUSH ESI  
0040BF51    57              PUSH EDI  
0040BF52    8965 E8         MOV DWORD PTR SS:\[EBP-18\],ESP  
0040BF55    FF15 D0014200   CALL DWORD PTR DS:\[4201D0\]        ; KERNEL32.GetVersion  
0040BF5B    33D2            XOR EDX,EDX  
0040BF5D    8AD4            MOV DL,AH  
0040BF5F    8915 4CB54300   MOV DWORD PTR DS:\[43B54C\],EDX  
0040BF65    8BC8            MOV ECX,EAX  
0040BF67    81E1 FF000000   AND ECX,0FF  
0040BF6D    890D 48B54300   MOV DWORD PTR DS:\[43B548\],ECX  
0040BF73    C1E1 08         SHL ECX,8  
0040BF76    03CA            ADD ECX,EDX  
0040BF78    890D 44B54300   MOV DWORD PTR DS:\[43B544\],ECX  
0040BF7E    C1E8 10         SHR EAX,10  
0040BF81    A3 40B54300     MOV DWORD PTR DS:\[43B540\],EAX  
0040BF86    6A 01           PUSH 1  
0040BF88    E8 60430000     CALL 004102ED  
0040BF8D    59              POP ECX  
0040BF8E    85C0            TEST EAX,EAX  
0040BF90    75 08           JNZ SHORT 0040BF9A  
0040BF92    6A 1C           PUSH 1C  
0040BF94    E8 C3000000     CALL 0040C05C  
0040BF99    59              POP ECX  
0040BF9A    E8 ED220000     CALL 0040E28C  
0040BF9F    85C0            TEST EAX,EAX  
0040BFA1    75 08           JNZ SHORT 0040BFAB  
0040BFA3    6A 10           PUSH 10  
0040BFA5    E8 B2000000     CALL 0040C05C  
0040BFAA    59              POP ECX  
0040BFAB    33F6            XOR ESI,ESI  
0040BFAD    8975 FC         MOV DWORD PTR SS:\[EBP-4\],ESI  
0040BFB0    E8 FF390000     CALL 0040F9B4  
0040BFB5    FF15 C0004200   CALL DWORD PTR DS:\[4200C0\]        ; KERNEL32.GetCommandLineA  
0040BFBB    A3 58CC4300     MOV DWORD PTR DS:\[43CC58\],EAX  
0040BFC0    E8 81400000     CALL 00410046  
0040BFC5    A3 30B54300     MOV DWORD PTR DS:\[43B530\],EAX  
0040BFCA    E8 2A3E0000     CALL 0040FDF9  
0040BFCF    E8 6C3D0000     CALL 0040FD40  
0040BFD4    E8 39010000     CALL 0040C112  
0040BFD9    8975 D0         MOV DWORD PTR SS:\[EBP-30\],ESI  
0040BFDC    8D45 A4         LEA EAX,DWORD PTR SS:\[EBP-5C\]  
0040BFDF    50              PUSH EAX  
0040BFE0    FF15 BC004200   CALL DWORD PTR DS:\[4200BC\]        ; KERNEL32.GetStartupInfoA  
0040BFE6    E8 FD3C0000     CALL 0040FCE8  
0040BFEB    8945 9C         MOV DWORD PTR SS:\[EBP-64\],EAX  
0040BFEE    F645 D0 01      TEST BYTE PTR SS:\[EBP-30\],1  
0040BFF2    74 06           JE SHORT 0040BFFA  
0040BFF4    0FB745 D4       MOVZX EAX,WORD PTR SS:\[EBP-2C\]  
0040BFF8    EB 03           JMP SHORT 0040BFFD  
0040BFFA    6A 0A           PUSH 0A  
0040BFFC    58              POP EAX  
0040BFFD    50              PUSH EAX  
0040BFFE    FF75 9C         PUSH DWORD PTR SS:\[EBP-64\]  
0040C001    56              PUSH ESI  
0040C002    56              PUSH ESI  
0040C003    FF15 58024200   CALL DWORD PTR DS:\[420258\]        ; KERNEL32.GetModuleHandleA  
0040C009    50              PUSH EAX  
0040C00A    E8 21920000     CALL 00415230  
0040C00F    8945 A0         MOV DWORD PTR SS:\[EBP-60\],EAX  
0040C012    50              PUSH EAX  
0040C013    E8 27010000     CALL 0040C13F  
0040C018    8B45 EC         MOV EAX,DWORD PTR SS:\[EBP-14\]  
0040C01B    8B08            MOV ECX,DWORD PTR DS:\[EAX\]  
0040C01D    8B09            MOV ECX,DWORD PTR DS:\[ECX\]  
0040C01F    894D 98         MOV DWORD PTR SS:\[EBP-68\],ECX  
0040C022    50              PUSH EAX  
0040C023    51              PUSH ECX  
0040C024    E8 473B0000     CALL 0040FB70  
0040C029    59              POP ECX  
0040C02A    59              POP ECX  
0040C02B    C3              RETN  
  
  
  
十五. 计算 API　Address 的关键 CALL ( 以 kernel32.dll 的 GetVersion 为例  
  
      几个重要的线程函数  
  
  
001487E4    FF15 D0014200   CALL DWORD PTR DS:\[4201D0\]  
  
  
00144A78    60              PUSHAD  
00144A79    9C              PUSHFD  
00144A7A    68 E3465BBF     PUSH BF5B46E3                      ; DLL Name Hash  
00144A7F    68 0089DCBA     PUSH BADC8900  
00144A84    68 BB695C0C     PUSH 0C5C69BB  
00144A89    68 D033BFC7     PUSH C7BF33D0                      ; API Name Hash  
00144A8E    68 8CC65210     PUSH 1052C68C  
00144A93    68 49ED0233     PUSH 3302ED49  
00144A98    68 C84A1400     PUSH 144AC8  
00144A9D    E8 15FD2505     CALL EXERe.053A47B7  
00144AA2    9D              POPFD  
00144AA3    61              POPAD  
00144AA4    FF25 C84A1400   JMP DWORD PTR DS:\[144AC8\]  
  
  
  
EXERe.053A47B7(arg0, arg1, arg2, arg3, arg4, arg5, arg6)  
  
arg1 - arg6 其中两个数的平方是其他四个数  
保存时 DLL Name Hash 在前   
  
053A47B7  /.  55            PUSH EBP  
053A47B8  |.  8BEC          MOV EBP,ESP  
053A47BA  |.  83EC 24       SUB ESP,24                         ; 9 个 var,  var1- var6, 放参数 arg1-arg6  
053A47BD  |.  8B45 0C       MOV EAX,DWORD PTR SS:\[EBP+C\]  
053A47C0  |.  8365 F8 00    AND DWORD PTR SS:\[EBP-8\],0         ; var8=0  
053A47C4  |.  8945 DC       MOV DWORD PTR SS:\[EBP-24\],EAX  
053A47C7  |.  8B45 10       MOV EAX,DWORD PTR SS:\[EBP+10\]  
053A47CA  |.  8945 E0       MOV DWORD PTR SS:\[EBP-20\],EAX  
053A47CD  |.  8B45 14       MOV EAX,DWORD PTR SS:\[EBP+14\]  
053A47D0  |.  8945 E4       MOV DWORD PTR SS:\[EBP-1C\],EAX  
053A47D3  |.  8B45 18       MOV EAX,DWORD PTR SS:\[EBP+18\]  
053A47D6  |.  8365 FC 00    AND DWORD PTR SS:\[EBP-4\],0         ; var9=0  
053A47DA  |.  8945 E8       MOV DWORD PTR SS:\[EBP-18\],EAX  
053A47DD  |.  8B45 1C       MOV EAX,DWORD PTR SS:\[EBP+1C\]  
053A47E0  |.  56            PUSH ESI  
053A47E1  |.  8945 EC       MOV DWORD PTR SS:\[EBP-14\],EAX  
053A47E4  |.  8B45 20       MOV EAX,DWORD PTR SS:\[EBP+20\]  
053A47E7  |.  57            PUSH EDI  
053A47E8  |.  894D F4       MOV DWORD PTR SS:\[EBP-C\],ECX       ; var7 = ecx = 57 ???  
053A47EB  |.  8945 F0       MOV DWORD PTR SS:\[EBP-10\],EAX  
053A47EE  |.  8D75 DC       LEA ESI,DWORD PTR SS:\[EBP-24\]      ; esi -> var1  
053A47F1  |.  C745 0C 06000>MOV DWORD PTR SS:\[EBP+C\],6         ; 循环 6 次, 找出两个原始数  
053A47F8  |>  8B3E          /MOV EDI,DWORD PTR DS:\[ESI\]        ; var1  
053A47FA  |.  8BC7          |MOV EAX,EDI  
053A47FC  |.  99            |CDQ  
053A47FD  |.  52            |PUSH EDX  
053A47FE  |.  50            |PUSH EAX  
053A47FF  |.  52            |PUSH EDX  
053A4800  |.  50            |PUSH EAX  
053A4801  |.  E8 CC2C0000   |CALL EXERe.053A74D2               ; EXERe.053A74D2( 32 位带符号乘法）  
053A4806  |.  83C4 10       |ADD ESP,10  
053A4809  |.  8D4D DC       |LEA ECX,DWORD PTR SS:\[EBP-24\]     ; ecx -> var1  
053A480C  |.  6A 06         |PUSH 6  
053A480E  |.  5A            |POP EDX                           ; edx =6  
053A480F  |>  3B01          |/CMP EAX,DWORD PTR DS:\[ECX\]  
053A4811  |.  75 0E         ||JNZ SHORT EXERe.053A4821  
053A4813  |.  837D FC 00    ||CMP DWORD PTR SS:\[EBP-4\],0  
053A4817  |.  75 05         ||JNZ SHORT EXERe.053A481E  
053A4819  |.  897D FC       ||MOV DWORD PTR SS:\[EBP-4\],EDI  
053A481C  |.  EB 03         ||JMP SHORT EXERe.053A4821  
053A481E  |>  897D F8       ||MOV DWORD PTR SS:\[EBP-8\],EDI  
053A4821  |>  83C1 04       ||ADD ECX,4  
053A4824  |.  4A            ||DEC EDX  
053A4825  |.^ 75 E8         |\\JNZ SHORT EXERe.053A480F  
053A4827  |.  83C6 04       |ADD ESI,4  
053A482A  |.  FF4D 0C       |DEC DWORD PTR SS:\[EBP+C\]  
053A482D  |.^ 75 C9         \\JNZ SHORT EXERe.053A47F8  
053A482F  |.  FF75 FC       PUSH DWORD PTR SS:\[EBP-4\]                     ; /Arg2 原始数 2 (API Name Hash)  
053A4832  |.  8B4D F4       MOV ECX,DWORD PTR SS:\[EBP-C\]                  ; |  
053A4835  |.  FF75 F8       PUSH DWORD PTR SS:\[EBP-8\]                     ; |Arg1 原始数 1 (Dll Name Hash)  
053A4838  |.  E8 F3DFFFFF   CALL EXERe.053A2830                           ; \\EXERe.053A2830  
053A483D  |.  8B4D 08       MOV ECX,DWORD PTR SS:\[EBP+8\]       ; arg0  
053A4840  |.  5F            POP EDI  
053A4841  |.  5E            POP ESI  
053A4842  |.  8901          MOV DWORD PTR DS:\[ECX\],EAX         ; 最终结果  
053A4844  |.  C9            LEAVE  
053A4845  \\.  C2 1C00       RETN 1C  
  
  
  
Structure  // 0c byte                                          ; 这个结构很重要 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
{    
  DWORD  DLLNameHash;  
  DWORD  APINameHash;  
  DWORD  APIAddress( 被 53A102C 处理过的);  
}  
  
  
proc 53A2830(DLL Name Hash,  API Name Hash)  
  
053A2830  /$  55            PUSH EBP  
053A2831  |.  8BEC          MOV EBP,ESP  
053A2833  |.  83EC 30       SUB ESP,30  
053A2836  |.  A1 4C8C3B05   MOV EAX,DWORD PTR DS:\[53B8C4C\]                  ; 全局变量 2  
053A283B  |.  53            PUSH EBX  
053A283C  |.  56            PUSH ESI  
053A283D  |.  8B75 0C       MOV ESI,DWORD PTR SS:\[EBP+C\]                    ; API Name Hash  
053A2840  |.  8D4D F4       LEA ECX,DWORD PTR SS:\[EBP-C\]  
053A2843  |.  57            PUSH EDI  
053A2844  |.  8B7D 08       MOV EDI,DWORD PTR SS:\[EBP+8\]                    ; DLL Name Hash  
053A2847  |.  51            PUSH ECX                                      ; /Arg3 pS1  
053A2848  |.  50            PUSH EAX                                      ; |Arg2 \[53B8C4C\]  全局变量 2, 初始=0  
053A2849  |.  33DB          XOR EBX,EBX                                   ; |  
053A284B  |.  FF35 488C3B05 PUSH DWORD PTR DS:\[53B8C48\]                   ; |Arg1 \[53B8C48\]  全局变量 1, 初始=0  
053A2851  |.  897D F4       MOV DWORD PTR SS:\[EBP-C\],EDI                  ; |  
053A2854  |.  8975 F8       MOV DWORD PTR SS:\[EBP-8\],ESI                  ; |  
053A2857  |.  895D FC       MOV DWORD PTR SS:\[EBP-4\],EBX                  ; |  
053A285A  |.  E8 20240000   CALL EXERe.053A4C7F                           ; \\EXERe.053A4C7F  
053A285F  |.  83C4 0C       ADD ESP,0C  
053A2862  |.  3B05 4C8C3B05 CMP EAX,DWORD PTR DS:\[53B8C4C\]                ; 判断是否成功  
053A2868  |.  74 0D         JE SHORT EXERe.053A2877  
  
053A286A  |.  8BF0          MOV ESI,EAX                                   ; 成功了  
053A286C  |.  8D7D D0       LEA EDI,DWORD PTR SS:\[EBP-30\]  
053A286F  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]    ; 复制到临时空间, 这个多余  
053A2870  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]    ; 这个多余  
053A2871  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]    ; 有这个就够了  
053A2872  |.  8B45 D8       MOV EAX,DWORD PTR SS:\[EBP-28\]                 ; API address ( 被 53A102C 处理过的)  
053A2875  |.  EB 6B         JMP SHORT EXERe.053A28E2  
  
053A2877  |>  8D45 E8       LEA EAX,DWORD PTR SS:\[EBP-18\]                 ; 不成功  
053A287A  |.  B9 488C3B05   MOV ECX,EXERe.053B8C48                        ; proc 53A2301 的一个参数  
053A287F  |.  50            PUSH EAX                                      ; pS2                                        
053A2880  |.  895D F0       MOV DWORD PTR SS:\[EBP-10\],EBX                 ; 填充 S2  
053A2883  |.  897D E8       MOV DWORD PTR SS:\[EBP-18\],EDI  
053A2886  |.  8975 EC       MOV DWORD PTR SS:\[EBP-14\],ESI  
053A2889  |.  E8 73FAFFFF   CALL EXERe.053A2301                           ; 扩充 \[53B8C48\] - \[53B8C50\] 的队列, \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053A288E  |.  895D E4       MOV DWORD PTR SS:\[EBP-1C\],EBX                 ; 填充 S3  
053A2891  |.  897D DC       MOV DWORD PTR SS:\[EBP-24\],EDI  
053A2894  |.  8975 E0       MOV DWORD PTR SS:\[EBP-20\],ESI  
053A2897  |.  BB 000000F0   MOV EBX,F0000000  
  
053A289C  |>  A1 A0A03B05   /MOV EAX,DWORD PTR DS:\[53BA0A0\]  
053A28A1  |.  6A 02         |PUSH 2  
053A28A3  |.  F7D0          |NOT EAX  
053A28A5  |.  FFD0          |CALL EAX                                     ; Sleep(2ms), 等待 2 ms  
053A28A7  |.  A1 4C8C3B05   |MOV EAX,DWORD PTR DS:\[53B8C4C\]  
053A28AC  |.  8D4D DC       |LEA ECX,DWORD PTR SS:\[EBP-24\]  
053A28AF  |.  51            |PUSH ECX                                     ; /Arg3 pS3  
053A28B0  |.  50            |PUSH EAX                                     ; |Arg2 \[53B8C4C\]  
053A28B1  |.  FF35 488C3B05 |PUSH DWORD PTR DS:\[53B8C48\]                  ; |Arg1 \[53B8C48\]  
053A28B7  |.  E8 C3230000   |CALL EXERe.053A4C7F                          ; \\EXERe.053A4C7F, 再找一次  
053A28BC  |.  83C4 0C       |ADD ESP,0C  
053A28BF  |.  3B05 4C8C3B05 |CMP EAX,DWORD PTR DS:\[53B8C4C\]               ; 再次判断是否匹配  
053A28C5  |.  74 19         |JE SHORT EXERe.053A28E0                      ; 如果还没成功, 就放弃, 返回 0 , 这样程序就结束了 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*                     
053A28C7  |.  8BF0          |MOV ESI,EAX                                  ; 成功  
053A28C9  |.  8D7D D0       |LEA EDI,DWORD PTR SS:\[EBP-30\]  
053A28CC  |.  A5            |MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A28CD  |.  A5            |MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A28CE  |.  A5            |MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A28CF  |.  8B45 D8       |MOV EAX,DWORD PTR SS:\[EBP-28\]                ; API address ( 被 53A102C 处理过的)  
053A28D2  |.  85C0          |TEST EAX,EAX  
053A28D4  |.^ 74 C6         |JE SHORT EXERe.053A289C                      ; APIaddress=0, 再去等 2 ms                       
053A28D6  |.  8BC8          |MOV ECX,EAX  
053A28D8  |.  23CB          |AND ECX,EBX  
053A28DA  |.  3BCB          |CMP ECX,EBX  
053A28DC  |.  75 04         |JNZ SHORT EXERe.053A28E2                     ; API address 不能是 Fxxxxxxx     
053A28DE  |.^ EB BC         \\JMP SHORT EXERe.053A289C  
053A28E0  |>  33C0          XOR EAX,EAX  
053A28E2  |>  5F            POP EDI  
053A28E3  |.  5E            POP ESI  
053A28E4  |.  5B            POP EBX  
053A28E5  |.  C9            LEAVE  
053A28E6  \\.  C2 0800       RETN 8  
  
  
  
proc 53A4C7F( X, Y, Z)   
  
X pStructure (第一个)  
Y pStructure (最后一个)  
Z pStructure  
  
从 X 到 Y 的队列中找出与 Z 匹配的 (比较 DLL hash, API hash)   
  
  
  
053A4C7F  /$  55            PUSH EBP  
053A4C80  |.  8BEC          MOV EBP,ESP  
053A4C82  |.  8B4D 0C       MOV ECX,DWORD PTR SS:\[EBP+C\]                  ; ECX = Y  
053A4C85  |.  53            PUSH EBX  
053A4C86  |.  8B5D 08       MOV EBX,DWORD PTR SS:\[EBP+8\]                  ; EBX = X  
053A4C89  |.  8BC1          MOV EAX,ECX  
053A4C8B  |.  56            PUSH ESI  
053A4C8C  |.  57            PUSH EDI  
053A4C8D  |.  2BC3          SUB EAX,EBX                                   ; EAX = Y-X  
053A4C8F  |.  6A 0C         PUSH 0C  
053A4C91  |.  99            CDQ  
053A4C92  |.  5E            POP ESI                                       ; ESI = 0c  
053A4C93  |.  894D 0C       MOV DWORD PTR SS:\[EBP+C\],ECX                  ; Y 不变  
053A4C96  |.  F7FE          IDIV ESI  
053A4C98  |.  33C9          XOR ECX,ECX                                   ; 循环变量  
053A4C9A  |.  85C0          TEST EAX,EAX  
053A4C9C  |.  8945 08       MOV DWORD PTR SS:\[EBP+8\],EAX                  ; 下面循环 (Y-X)/0c 次  
053A4C9F  |.  7E 27         JLE SHORT EXERe.053A4CC8  
          |.  
053A4CA1  |.  8B75 10       MOV ESI,DWORD PTR SS:\[EBP+10\]                 ; Z  
053A4CA4  |.  8BD3          MOV EDX,EBX                                   ; EDX = X  
053A4CA6  |.  8B7E 04       MOV EDI,DWORD PTR DS:\[ESI+4\]                  ; Z.APINameHash  
          |.  
053A4CA9  |>  397A 04       /CMP DWORD PTR DS:\[EDX+4\],EDI                 ; 先比较 API Hash Name  
053A4CAC  |.  75 06         |JNZ SHORT EXERe.053A4CB4  
053A4CAE  |.  8B02          |MOV EAX,DWORD PTR DS:\[EDX\]  
053A4CB0  |.  3B06          |CMP EAX,DWORD PTR DS:\[ESI\]                   ; 再比较 DLL Name Hash  
053A4CB2  |.  74 0B         |JE SHORT EXERe.053A4CBF                      ; 两个都 OK 则找到  
053A4CB4  |>  41            |INC ECX  
053A4CB5  |.  83C2 0C       |ADD EDX,0C  
053A4CB8  |.  3B4D 08       |CMP ECX,DWORD PTR SS:\[EBP+8\]                 ; 是否超过最后一个  
053A4CBB  |.  7D 0B         |JGE SHORT EXERe.053A4CC8                     ; 找不到返回 Y  
053A4CBD  |.^ EB EA         \\JMP SHORT EXERe.053A4CA9  
053A4CBF  |>  8D0449        LEA EAX,DWORD PTR DS:\[ECX+ECX\*2\]  
053A4CC2  |.  8D0483        LEA EAX,DWORD PTR DS:\[EBX+EAX\*4\]              ; EAX = X + 0ch\*ECX  
053A4CC5  |.  8945 0C       MOV DWORD PTR SS:\[EBP+C\],EAX                  ; 匹配的结构地址  
053A4CC8  |>  8B45 0C       MOV EAX,DWORD PTR SS:\[EBP+C\]  
053A4CCB  |.  5F            POP EDI  
053A4CCC  |.  5E            POP ESI  
053A4CCD  |.  5B            POP EBX  
053A4CCE  |.  5D            POP EBP  
053A4CCF  \\.  C3            RETN  
  
  
  
Proc 53A2301(pStructure)  
  
ECX = 53B8C48, 很重要的一个全局变量  
  
053B8C48  ;第一个结构地址  
053B8C4C  ;最后一个结构地址  
053B8C50  ;结构个数           
  
Structure  // 0c byte                                          ; 这个结构很重要 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
{    
  DWORD  DLLNameHash;  
  DWORD  APINameHash;  
  DWORD  APIAddress( 被 53A102C 处理过的);  
}  
  
053A2301  /$  55            PUSH EBP  
053A2302  |.  8BEC          MOV EBP,ESP  
053A2304  |.  51            PUSH ECX  
053A2305  |.  53            PUSH EBX  
053A2306  |.  8BD9          MOV EBX,ECX                                          ; 53B8C48                                           
053A2308  |.  56            PUSH ESI  
053A2309  |.  57            PUSH EDI  
053A230A  |.  8B43 08       MOV EAX,DWORD PTR DS:\[EBX+8\]                         ; 结构个数  
053A230D  |.  85C0          TEST EAX,EAX  
053A230F  |.  75 30         JNZ SHORT EXERe.053A2341  
  
053A2311  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]                       ; 原来为 0, 新生成一个  
053A2316  |.  6A 0C         PUSH 0C  
053A2318  |.  6A 08         PUSH 8  
053A231A  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A2320  |.  F7D0          NOT EAX  
053A2322  |.  FFD0          CALL EAX                                             ; HeapAlloc(0ch byte)  
053A2324  |.  85C0          TEST EAX,EAX  
053A2326  |.  0F84 80000000 JE EXERe.053A23AC  
053A232C  |.  8B75 08       MOV ESI,DWORD PTR SS:\[EBP+8\]                         ; pStructure  
053A232F  |.  8BF8          MOV EDI,EAX  
053A2331  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]           ; 复制结构   
053A2332  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A2333  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A2334  |.  8903          MOV DWORD PTR DS:\[EBX\],EAX                           ; \[53B8C48\]  
053A2336  |.  83C0 0C       ADD EAX,0C  
053A2339  |.  FF43 08       INC DWORD PTR DS:\[EBX+8\]                             ; 结构个数 ++  
053A233C  |.  8943 04       MOV DWORD PTR DS:\[EBX+4\],EAX  
053A233F  |.  EB 6B         JMP SHORT EXERe.053A23AC  
  
053A2341  |>  8D4440 03     LEA EAX,DWORD PTR DS:\[EAX+EAX\*2+3\]                   ; 原来不为 0, 扩展一个  
053A2345  |.  C1E0 02       SHL EAX,2                                            ; EAX = (EAX+1)\*12  
053A2348  |.  50            PUSH EAX  
053A2349  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053A234E  |.  6A 08         PUSH 8  
053A2350  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A2356  |.  F7D0          NOT EAX  
053A2358  |.  FFD0          CALL EAX                                            ; HeapAlloc(新分配空间, 比原来大 0c 字节)  
053A235A  |.  33C9          XOR ECX,ECX  
053A235C  |.  8945 FC       MOV DWORD PTR SS:\[EBP-4\],EAX  
053A235F  |.  394B 08       CMP DWORD PTR DS:\[EBX+8\],ECX                         
053A2362  |.  7E 15         JLE SHORT EXERe.053A2379  
053A2364  |.  33D2          XOR EDX,EDX                                         ; 把老的内容复制到新空间  
053A2366  |>  8BF2          /MOV ESI,EDX   
053A2368  |.  8D3C02        |LEA EDI,DWORD PTR DS:\[EDX+EAX\]  
053A236B  |.  0333          |ADD ESI,DWORD PTR DS:\[EBX\]  
053A236D  |.  41            |INC ECX  
053A236E  |.  83C2 0C       |ADD EDX,0C  
053A2371  |.  A5            |MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A2372  |.  A5            |MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A2373  |.  A5            |MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A2374  |.  3B4B 08       |CMP ECX,DWORD PTR DS:\[EBX+8\]  
053A2377  |.^ 7C ED         \\JL SHORT EXERe.053A2366  
053A2379  |>  8B75 08       MOV ESI,DWORD PTR SS:\[EBP+8\]                        ; pStructure    
053A237C  |.  8D0C49        LEA ECX,DWORD PTR DS:\[ECX+ECX\*2\]  
053A237F  |.  8D3C88        LEA EDI,DWORD PTR DS:\[EAX+ECX\*4\]  
053A2382  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]          ; append 到最后  
053A2383  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A2384  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
053A2385  |.  FF33          PUSH DWORD PTR DS:\[EBX\]                             ; 释放老的空间  
053A2387  |.  A1 90A03B05   MOV EAX,DWORD PTR DS:\[53BA090\]  
053A238C  |.  F7D0          NOT EAX  
053A238E  |.  6A 00         PUSH 0  
053A2390  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A2396  |.  FFD0          CALL EAX                                           ; HeapFree  
053A2398  |.  FF43 08       INC DWORD PTR DS:\[EBX+8\]                           ; 结构个数 ++  
053A239B  |.  8B43 08       MOV EAX,DWORD PTR DS:\[EBX+8\]  
053A239E  |.  8B4D FC       MOV ECX,DWORD PTR SS:\[EBP-4\]  
053A23A1  |.  8D0440        LEA EAX,DWORD PTR DS:\[EAX+EAX\*2\]  
053A23A4  |.  890B          MOV DWORD PTR DS:\[EBX\],ECX                         ; 新空间的开始地址  
053A23A6  |.  8D0481        LEA EAX,DWORD PTR DS:\[ECX+EAX\*4\]  
053A23A9  |.  8943 04       MOV DWORD PTR DS:\[EBX+4\],EAX                       ; 新空间的结束地址  
053A23AC  |>  5F            POP EDI  
053A23AD  |.  5E            POP ESI  
053A23AE  |.  5B            POP EBX  
053A23AF  |.  C9            LEAVE  
053A23B0  \\.  C2 0400       RETN 4  
  
  
  
053B8C18 ; 开始结构地址  
053B8C1C ; 结束结构地址  
053B8C20 ; hThread 结构个数  
structure hThread  
{  
  Dword ThreadHandle;  
}  
  
  
053B8C24 ; 开始结构地址   
053B8C28 ; 结束结构地址   
053B8C2C ; DllNameHash 结构个数  
structure DllNameHash  
{   
   Dword DllNameHash;  
}  
  
  
053B8C38    ; 开始结构地址  
053B8C3C    ; 结束结构地址  
053B8C40    ; Dll UniName Path 结构个数  
structure UniName  
{   
   Dword BaseAddress;  
   Dword UnicodeString; 指向 Unicode Name  
}  
  
  
053B8C58    ; 开始结构地址       
053B8C5C    ; 结束结构地址  
053B8C60    ; DLL API Address 结构个数  
structure Export  
{   
   Dword BaseAddress;  
   Dword pFirstThunk;  (Dll 所有的 API address, 按原始循序, 0 结束)  
   Dword TlsSize  
}  
  
  
MyLoadLibrary(pDllName)  
  
ECX = 053B8C18  ;关键全局变量  
  
053A5514  /.  55            PUSH EBP  
053A5515  |.  8BEC          MOV EBP,ESP  
053A5517  |.  81EC 180A0000 SUB ESP,0A18  
053A551D  |.  53            PUSH EBX  
053A551E  |.  56            PUSH ESI  
053A551F  |.  57            PUSH EDI  
053A5520  |.  8B7D 08       MOV EDI,DWORD PTR SS:\[EBP+8\]                  ; pDllName  
053A5523  |.  33DB          XOR EBX,EBX  
053A5525  |.  8BF1          MOV ESI,ECX                                   ; 053B8C18  
053A5527  |.  3BFB          CMP EDI,EBX  
053A5529  |.  8975 F0       MOV DWORD PTR SS:\[EBP-10\],ESI  
053A552C  |.  0F84 75010000 JE EXERe.053A56A7                             ; pDllName == 0 则结束  
053A5532  |.  53            PUSH EBX  
053A5533  |.  57            PUSH EDI  
053A5534  |.  E8 6EF1FFFF   CALL EXERe.053A46A7                           ; uppercase  
053A5539  |.  59            POP ECX  
053A553A  |.  50            PUSH EAX  
053A553B  |.  E8 C91F0000   CALL EXERe.053A7509                           ; 对 Dll Name 做 Hash 变换  
053A5540  |.  59            POP ECX  
053A5541  |.  33D2          XOR EDX,EDX  
053A5543  |.  59            POP ECX  
053A5544  |.  8945 F4       MOV DWORD PTR SS:\[EBP-C\],EAX                  ; 保存 Dll Name Hash  
053A5547  |.  8B4E 14       MOV ECX,DWORD PTR DS:\[ESI+14\]                 ; 已处理的 DLL 个数  
053A554A  |.  3BCB          CMP ECX,EBX  
053A554C  |.  7E 13         JLE SHORT EXERe.053A5561  
053A554E  |.  8B76 0C       MOV ESI,DWORD PTR DS:\[ESI+C\]  
053A5551  |>  3B06          /CMP EAX,DWORD PTR DS:\[ESI\]                   ; 搜索数组  
053A5553  |.  0F84 4E010000 |JE EXERe.053A56A7                            ; 相等就表示已经处理过了, 返回  
053A5559  |.  42            |INC EDX  
053A555A  |.  83C6 04       |ADD ESI,4                                    ; 下一个  
053A555D  |.  3BD1          |CMP EDX,ECX  
053A555F  |.^ 7C F0         \\JL SHORT EXERe.053A5551  
053A5561  |>  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053A5566  |.  6A 08         PUSH 8                                        ; 8 字节  
053A5568  |.  6A 08         PUSH 8  
053A556A  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A5570  |.  F7D0          NOT EAX  
053A5572  |.  FFD0          CALL EAX                                      ; ntdll.RtlAllocateHeap  
053A5574  |.  8BF0          MOV ESI,EAX  
053A5576  |.  A1 7CA03B05   MOV EAX,DWORD PTR DS:\[53BA07C\]  
053A557B  |.  57            PUSH EDI  
053A557C  |.  F7D0          NOT EAX  
053A557E  |.  FFD0          CALL EAX                                      ; GetModuleHandleA  
053A5580  |.  3BC3          CMP EAX,EBX  
053A5582  |.  8906          MOV DWORD PTR DS:\[ESI\],EAX                    ; 包存 DLL Base Address  
053A5584  |.  75 11         JNZ SHORT EXERe.053A5597                      ; 不等已经加载了  
  
053A5586  |.  A1 84A03B05   MOV EAX,DWORD PTR DS:\[53BA084\]  
053A558B  |.  57            PUSH EDI  
053A558C  |.  F7D0          NOT EAX  
053A558E  |.  FFD0          CALL EAX                                      ; LoadLibraryA  
053A5590  |.  8906          MOV DWORD PTR DS:\[ESI\],EAX  
053A5592  |.  E8 0EFFFFFF   CALL EXERe.053A54A5                           ; 修正 53B8C38-53B8C40 所指的 UniName 列表  
  
053A5597  |>  E8 12010000   CALL EXERe.053A56AE                           ; 判断是否加载了 User32.dll,  0 没有, 否则返回 Not User32 BaseAddress  
053A559C  |.  F7D0          NOT EAX  
053A559E  |.  3906          CMP DWORD PTR DS:\[ESI\],EAX  
053A55A0  |.  75 16         JNZ SHORT EXERe.053A55B8                      ; 改成 JMP, 跳过未注册提示 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
053A55A2  |.  8D45 F8       LEA EAX,DWORD PTR SS:\[EBP-8\]                  ; 当前处理的是 User32.dll  
053A55A5  |.  50            PUSH EAX  
053A55A6  |.  A1 ACA03B05   MOV EAX,DWORD PTR DS:\[53BA0AC\]  
053A55AB  |.  53            PUSH EBX  
053A55AC  |.  53            PUSH EBX  
053A55AD  |.  68 76573A05   PUSH EXERe.053A5776                           ; ThreadFunction = 53A5776  
053A55B2  |.  53            PUSH EBX  
053A55B3  |.  53            PUSH EBX  
053A55B4  |.  F7D0          NOT EAX  
053A55B6  |.  FFD0          CALL EAX                                      ; CreateThread ( 显示未注册对话框)  
  
053A55B8  |>  8D8D E8F5FFFF LEA ECX,DWORD PTR SS:\[EBP-A18\]  
053A55BE  |.  E8 9ACCFFFF   CALL EXERe.053A225D                           ; zero buffer  
053A55C3  |.  FF36          PUSH DWORD PTR DS:\[ESI\]                             ; /Arg1 = dll base  
053A55C5  |.  8D8D E8F5FFFF LEA ECX,DWORD PTR SS:\[EBP-A18\]                      ; |  
053A55CB  |.  E8 00120000   CALL EXERe.053A67D0                                 ; \\EXERe.053A67D0 ( 获取 DLL 的区块信息)  
  
053A55D0  |.  8B06          MOV EAX,DWORD PTR DS:\[ESI\]                    ; dll base  
053A55D2  |.  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]                 ; pExport  
053A55D5  |.  8945 E4       MOV DWORD PTR SS:\[EBP-1C\],EAX  
053A55D8  |.  A1 5C8C3B05   MOV EAX,DWORD PTR DS:\[53B8C5C\]  
053A55DD  |.  51            PUSH ECX  
053A55DE  |.  50            PUSH EAX  
053A55DF  |.  FF35 588C3B05 PUSH DWORD PTR DS:\[53B8C58\]  
053A55E5  |.  895D E8       MOV DWORD PTR SS:\[EBP-18\],EBX  
053A55E8  |.  895D EC       MOV DWORD PTR SS:\[EBP-14\],EBX  
053A55EB  |.  E8 C7D0FFFF   CALL EXERe.053A26B7                          ; 搜索 53B8C58-53B8C60 所指的 Export 列表  
053A55F0  |.  83C4 0C       ADD ESP,0C  
053A55F3  |.  3B05 5C8C3B05 CMP EAX,DWORD PTR DS:\[53B8C5C\]  
053A55F9  |.  75 0D         JNZ SHORT EXERe.053A5608                     ; 是否已有 Export 表  
  
053A55FB  |.  8D85 E8F5FFFF LEA EAX,DWORD PTR SS:\[EBP-A18\]               ; 没有  
053A5601  |.  50            PUSH EAX  
053A5602  |.  E8 631A0000   CALL EXERe.053A706A                          ; 修正 53B8C58-53B8C60 所指的 Export 列表  
053A5607  |.  59            POP ECX  
  
053A5608  |>  395D C0       CMP DWORD PTR SS:\[EBP-40\],EBX                ; 这个 DLL 是否有 .tls 段  
053A560B  |.  7C 62         JL SHORT EXERe.053A566F  
  
053A560D  |.  53            PUSH EBX                                            ; 有 tls  
053A560E  |.  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]                       ; 放 tls 的 buffer  
053A5611  |.  6A 18         PUSH 18  
053A5613  |.  50            PUSH EAX  
053A5614  |.  8B06          MOV EAX,DWORD PTR DS:\[ESI\]                          ; DLL baseaddress  
053A5616  |.  0385 E8F6FFFF ADD EAX,DWORD PTR SS:\[EBP-918\]                      ; PE 头 Directory 中的 .tls RVA  
053A561C  |.  50            PUSH EAX  
053A561D  |.  A1 88A03B05   MOV EAX,DWORD PTR DS:\[53BA088\]  
053A5622  |.  FF35 D8A03B05 PUSH DWORD PTR DS:\[53BA0D8\]                         ; hProcess  
053A5628  |.  F7D0          NOT EAX  
053A562A  |.  FFD0          CALL EAX                                            ; ReadProcessMemory ( 读 18h byte)  
053A562C  |.  8B45 DC       MOV EAX,DWORD PTR SS:\[EBP-24\]  
053A562F  |.  2B45 D8       SUB EAX,DWORD PTR SS:\[EBP-28\]                       ; tls 空间大小  
053A5632  |.  0105 DCA03B05 ADD DWORD PTR DS:\[53BA0DC\],EAX                      ; 累加 tls 空间  
053A5638  |.  50            PUSH EAX  
053A5639  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053A563E  |.  6A 08         PUSH 8  
053A5640  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A5646  |.  F7D0          NOT EAX  
053A5648  |.  FFD0          CALL EAX                                            ; HeapAlloc( DLL 所需的 tls 空间)  
053A564A  |.  6A 04         PUSH 4  
053A564C  |.  8BF8          MOV EDI,EAX  
053A564E  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053A5653  |.  6A 08         PUSH 8  
053A5655  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A565B  |.  F7D0          NOT EAX  
053A565D  |.  FFD0          CALL EAX                                            ; HeapAlloc ( 4 byte)  
053A565F  |.  8945 F8       MOV DWORD PTR SS:\[EBP-8\],EAX  
053A5662  |.  8938          MOV DWORD PTR DS:\[EAX\],EDI  
053A5664  |.  50            PUSH EAX  
053A5665  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  
053A5668  |.  64:A3 2C00000>MOV DWORD PTR FS:\[2C\],EAX                           ; ThreadLocalStoragePointer                            
053A566E  |.  58            POP EAX  
  
053A566F  |>  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]                         ; DLL Name Hash  
053A5672  |.  8D7E 04       LEA EDI,DWORD PTR DS:\[ESI+4\]  
053A5675  |.  8907          MOV DWORD PTR DS:\[EDI\],EAX  
053A5677  |.  8D45 FC       LEA EAX,DWORD PTR SS:\[EBP-4\]  
053A567A  |.  50            PUSH EAX  
053A567B  |.  A1 ACA03B05   MOV EAX,DWORD PTR DS:\[53BA0AC\]  
053A5680  |.  53            PUSH EBX  
053A5681  |.  56            PUSH ESI                                            ; Param = { DLLBase, DllHash}  
053A5682  |.  68 40583A05   PUSH EXERe.053A5840                                 ; ThreadFunction = 53A5840  关键 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053A5687  |.  53            PUSH EBX  
053A5688  |.  53            PUSH EBX  
053A5689  |.  F7D0          NOT EAX  
053A568B  |.  FFD0          CALL EAX                                            ; CreateThread  
053A568D  |.  8B75 F0       MOV ESI,DWORD PTR SS:\[EBP-10\]                       ; 53b8C18  
053A5690  |.  8945 08       MOV DWORD PTR SS:\[EBP+8\],EAX                        ; hThread  
053A5693  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]  
053A5696  |.  8BCE          MOV ECX,ESI  
053A5698  |.  50            PUSH EAX  
053A5699  |.  E8 30070000   CALL EXERe.053A5DCE                                 ; 修正 53B8C18-53B8C20 所指的 hThread 列表  
053A569E  |.  57            PUSH EDI  
053A569F  |.  8D4E 0C       LEA ECX,DWORD PTR DS:\[ESI+C\]  
053A56A2  |.  E8 27070000   CALL EXERe.053A5DCE                                 ; 修正 53B8C24-53B8C2C 所指的 DllNameHash 列表  
053A56A7  |>  5F            POP EDI  
053A56A8  |.  5E            POP ESI  
053A56A9  |.  5B            POP EBX  
053A56AA  |.  C9            LEAVE  
053A56AB  \\.  C2 0400       RETN 4  
  
  
  
  
053A5840  /.  55            PUSH EBP  
053A5841  |.  8BEC          MOV EBP,ESP  
053A5843  |.  81EC 940B0000 SUB ESP,0B94  
053A5849  |.  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]             ; Parameter  
053A584C  |.  53            PUSH EBX  
053A584D  |.  56            PUSH ESI  
053A584E  |.  57            PUSH EDI  
053A584F  |.  8B18          MOV EBX,DWORD PTR DS:\[EAX\]               ; DLL base  
053A5851  |.  8B48 04       MOV ECX,DWORD PTR DS:\[EAX+4\]             ; DLL Name Hash  
053A5854  |.  33F6          XOR ESI,ESI  
053A5856  |.  50            PUSH EAX  
053A5857  |.  A1 90A03B05   MOV EAX,DWORD PTR DS:\[53BA090\]  
053A585C  |.  56            PUSH ESI  
053A585D  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A5863  |.  894D A8       MOV DWORD PTR SS:\[EBP-58\],ECX            ; DLL Name Hash  
053A5866  |.  F7D0          NOT EAX  
053A5868  |.  FFD0          CALL EAX                                 ; HeapFree  
053A586A  |.  8D8D 6CF4FFFF LEA ECX,DWORD PTR SS:\[EBP-B94\]  
053A5870  |.  E8 E8C9FFFF   CALL EXERe.053A225D                      ; Zero buffer  
053A5875  |.  53            PUSH EBX                                 ; /Arg1=DLL Base  
053A5876  |.  8D8D 6CF4FFFF LEA ECX,DWORD PTR SS:\[EBP-B94\]           ; |  
053A587C  |.  E8 4F0F0000   CALL EXERe.053A67D0                      ; \\EXERe.053A67D0( 获取 DLL 的区段信息)  
053A5881  |.  56            PUSH ESI  
053A5882  |.  8D85 6CFFFFFF LEA EAX,DWORD PTR SS:\[EBP-94\]  
053A5888  |.  6A 28         PUSH 28  
053A588A  |.  50            PUSH EAX  
053A588B  |.  8B85 24F5FFFF MOV EAX,DWORD PTR SS:\[EBP-ADC\]           ; Export table 的 RVA  
053A5891  |.  03C3          ADD EAX,EBX  
053A5893  |.  50            PUSH EAX  
053A5894  |.  A1 88A03B05   MOV EAX,DWORD PTR DS:\[53BA088\]  
053A5899  |.  FF35 D8A03B05 PUSH DWORD PTR DS:\[53BA0D8\]  
053A589F  |.  F7D0          NOT EAX  
053A58A1  |.  FFD0          CALL EAX                                 ; ReadProcessMemory (28h) Export Table Header  
053A58A3  |.  8B45 80       MOV EAX,DWORD PTR SS:\[EBP-80\]            ; NumberOfFunctions  
053A58A6  |.  C1E0 02       SHL EAX,2                                ; \* 4  
053A58A9  |.  50            PUSH EAX  
053A58AA  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053A58AF  |.  6A 08         PUSH 8  
053A58B1  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A58B7  |.  F7D0          NOT EAX  
053A58B9  |.  FFD0          CALL EAX                                ; HeapAlloc  
053A58BB  |.  8945 F4       MOV DWORD PTR SS:\[EBP-C\],EAX            ; 空间指针 1 放 API 地址  
053A58BE  |.  8B45 84       MOV EAX,DWORD PTR SS:\[EBP-7C\]           ; NumberOfNames  
053A58C1  |.  03C0          ADD EAX,EAX                             ; \* 2  
053A58C3  |.  8975 B8       MOV DWORD PTR SS:\[EBP-48\],ESI  
053A58C6  |.  50            PUSH EAX  
053A58C7  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053A58CC  |.  6A 08         PUSH 8  
053A58CE  |.  8975 BC       MOV DWORD PTR SS:\[EBP-44\],ESI  
053A58D1  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A58D7  |.  8975 C0       MOV DWORD PTR SS:\[EBP-40\],ESI  
053A58DA  |.  F7D0          NOT EAX  
053A58DC  |.  FFD0          CALL EAX                               ; HeapAlloc  
053A58DE  |.  8945 D8       MOV DWORD PTR SS:\[EBP-28\],EAX          ; 空间指针 2 放 NameOrdinal  
053A58E1  |.  8B45 84       MOV EAX,DWORD PTR SS:\[EBP-7C\]          ; NumberOfNames  
053A58E4  |.  C1E0 08       SHL EAX,8                              ; \* 256  
053A58E7  |.  50            PUSH EAX  
053A58E8  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:\[53BA0B4\]  
053A58ED  |.  6A 08         PUSH 8  
053A58EF  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A58F5  |.  F7D0          NOT EAX  
053A58F7  |.  FFD0          CALL EAX                               ; HeapAlloc  
053A58F9  |.  8945 DC       MOV DWORD PTR SS:\[EBP-24\],EAX          ; 空间指针 3 放 NameString  
053A58FC  |.  8B45 80       MOV EAX,DWORD PTR SS:\[EBP-80\]          ; NumberOfFunctions  
053A58FF  |.  C1E0 02       SHL EAX,2                              ; \* 4  
053A5902  |.  56            PUSH ESI  
053A5903  |.  50            PUSH EAX  
053A5904  |.  8B45 88       MOV EAX,DWORD PTR SS:\[EBP-78\]          ; AddressOfFunctions  
053A5907  |.  FF75 F4       PUSH DWORD PTR SS:\[EBP-C\]              ; 空间指针 1  
053A590A  |.  03C3          ADD EAX,EBX  
053A590C  |.  50            PUSH EAX  
053A590D  |.  A1 88A03B05   MOV EAX,DWORD PTR DS:\[53BA088\]  
053A5912  |.  FF35 D8A03B05 PUSH DWORD PTR DS:\[53BA0D8\]  
053A5918  |.  F7D0          NOT EAX  
053A591A  |.  FFD0          CALL EAX                               ; ReadProcessMemory 读入所有的地址  
053A591C  |.  8B45 84       MOV EAX,DWORD PTR SS:\[EBP-7C\]          ; NumberOfNames  
053A591F  |.  56            PUSH ESI  
053A5920  |.  03C0          ADD EAX,EAX  
053A5922  |.  50            PUSH EAX  
053A5923  |.  8B45 90       MOV EAX,DWORD PTR SS:\[EBP-70\]          ; AddressOfNameOrdinals  
053A5926  |.  FF75 D8       PUSH DWORD PTR SS:\[EBP-28\]             ; 空间指针 2  
053A5929  |.  03C3          ADD EAX,EBX  
053A592B  |.  50            PUSH EAX  
053A592C  |.  FF35 D8A03B05 PUSH DWORD PTR DS:\[53BA0D8\]  
053A5932  |.  A1 88A03B05   MOV EAX,DWORD PTR DS:\[53BA088\]  
053A5937  |.  F7D0          NOT EAX  
053A5939  |.  FFD0          CALL EAX                               ; ReadProcessMemory 读入所有的 NameOrdinal  
053A593B  |.  33FF          XOR EDI,EDI  
053A593D  |.  3975 84       CMP DWORD PTR SS:\[EBP-7C\],ESI          ; NumberOfNames > 0 ?  
053A5940  |.  7E 6B         JLE SHORT EXERe.053A59AD  
053A5942  |.  8B45 DC       MOV EAX,DWORD PTR SS:\[EBP-24\]          ; 空间指针 3  
053A5945  |.  8945 E8       MOV DWORD PTR SS:\[EBP-18\],EAX  
  
053A5948  |>  56            /PUSH ESI                              ; 循环读入所有的 NameString  
053A5949  |.  8D45 E0       |LEA EAX,DWORD PTR SS:\[EBP-20\]  
053A594C  |.  6A 04         |PUSH 4  
053A594E  |.  50            |PUSH EAX  
053A594F  |.  8B45 8C       |MOV EAX,DWORD PTR SS:\[EBP-74\]         ; AddressOfNames  
053A5952  |.  8975 EC       |MOV DWORD PTR SS:\[EBP-14\],ESI  
053A5955  |.  8D04B8        |LEA EAX,DWORD PTR DS:\[EAX+EDI\*4\]      ; EDI 循环变量  
053A5958  |.  03C3          |ADD EAX,EBX                           ; RVA -> VA  
053A595A  |.  50            |PUSH EAX  
053A595B  |.  A1 88A03B05   |MOV EAX,DWORD PTR DS:\[53BA088\]  
053A5960  |.  FF35 D8A03B05 |PUSH DWORD PTR DS:\[53BA0D8\]  
053A5966  |.  F7D0          |NOT EAX  
053A5968  |.  FFD0          |CALL EAX                              ; 读 4 字节地址  
053A596A  |.  8B45 E8       |MOV EAX,DWORD PTR SS:\[EBP-18\]  
053A596D  |.  8945 08       |MOV DWORD PTR SS:\[EBP+8\],EAX  
053A5970  |>  8B45 E0       |/MOV EAX,DWORD PTR SS:\[EBP-20\]  
053A5973  |.  8B4D EC       ||MOV ECX,DWORD PTR SS:\[EBP-14\]  
053A5976  |.  56            ||PUSH ESI  
053A5977  |.  6A 01         ||PUSH 1  
053A5979  |.  FF75 08       ||PUSH DWORD PTR SS:\[EBP+8\]  
053A597C  |.  03C1          ||ADD EAX,ECX  
053A597E  |.  03C3          ||ADD EAX,EBX  
053A5980  |.  50            ||PUSH EAX  
053A5981  |.  A1 88A03B05   ||MOV EAX,DWORD PTR DS:\[53BA088\]  
053A5986  |.  FF35 D8A03B05 ||PUSH DWORD PTR DS:\[53BA0D8\]  
053A598C  |.  F7D0          ||NOT EAX  
053A598E  |.  FFD0          ||CALL EAX                            ; ReadProcessMemory (1 byte)  
053A5990  |.  8B45 08       ||MOV EAX,DWORD PTR SS:\[EBP+8\]  
053A5993  |.  8038 00       ||CMP BYTE PTR DS:\[EAX\],0             ; 是否 0  
053A5996  |.  74 08         ||JE SHORT EXERe.053A59A0             ; 是 0 一个 Name 结束  
053A5998  |.  FF45 EC       ||INC DWORD PTR SS:\[EBP-14\]  
053A599B  |.  FF45 08       ||INC DWORD PTR SS:\[EBP+8\]  
053A599E  |.^ EB D0         |\\JMP SHORT EXERe.053A5970  
053A59A0  |>  8145 E8 00010>|ADD DWORD PTR SS:\[EBP-18\],100        ; 每个 Name 256 byte  
053A59A7  |.  47            |INC EDI  
053A59A8  |.  3B7D 84       |CMP EDI,DWORD PTR SS:\[EBP-7C\]        ; 所有的 Name 读完?  
053A59AB  |.^ 7C 9B         \\JL SHORT EXERe.053A5948  
  
053A59AD  |>  3975 80       CMP DWORD PTR SS:\[EBP-80\],ESI         ; NumberOfFunctions > 0 ?  
053A59B0  |.  8975 AC       MOV DWORD PTR SS:\[EBP-54\],ESI  
053A59B3  |.  8975 B0       MOV DWORD PTR SS:\[EBP-50\],ESI  
053A59B6  |.  8975 B4       MOV DWORD PTR SS:\[EBP-4C\],ESI  
053A59B9  |.  8975 F8       MOV DWORD PTR SS:\[EBP-8\],ESI  
053A59BC  |.  0F8E EA000000 JLE EXERe.053A5AAC  
053A59C2  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]          ; 空间指针 1  
053A59C5  |.  8945 08       MOV DWORD PTR SS:\[EBP+8\],EAX            
053A59C8  |>  8B45 08       /MOV EAX,DWORD PTR SS:\[EBP+8\]         ; 循环修正 API 地址, 入口 RVA 在输出表中，指向另一 DLL 的某函数 ( "DLL2.API2" )  
053A59CB  |.  8B8D 24F5FFFF |MOV ECX,DWORD PTR SS:\[EBP-ADC\]       ; Export table 的 RVA  
053A59D1  |.  8B00          |MOV EAX,DWORD PTR DS:\[EAX\]           ; API 相对地址  
053A59D3  |.  3BC1          |CMP EAX,ECX  
053A59D5  |.  0F8C BE000000 |JL EXERe.053A5A99                    ; API 地址< Export 就不用处理  
053A59DB  |.  8B95 28F5FFFF |MOV EDX,DWORD PTR SS:\[EBP-AD8\]       ; Export Table Size  
053A59E1  |.  03D1          |ADD EDX,ECX  
053A59E3  |.  3BC2          |CMP EAX,EDX  
053A59E5  |.  0F83 AE000000 |JNB EXERe.053A5A99                   ; API 地址>= Export+Size 也不用处理  
053A59EB  |.  8D8D 5CFEFFFF |LEA ECX,DWORD PTR SS:\[EBP-1A4\]       ; 开始修正  
053A59F1  |.  03C3          |ADD EAX,EBX  
053A59F3  |.  85C9          |TEST ECX,ECX  
053A59F5  |.  74 25         |JE SHORT EXERe.053A5A1C  
053A59F7  |.  3BC6          |CMP EAX,ESI  
053A59F9  |.  74 21         |JE SHORT EXERe.053A5A1C  
053A59FB  |.  8BC8          |MOV ECX,EAX  
053A59FD  |.  8D85 5CFEFFFF |LEA EAX,DWORD PTR SS:\[EBP-1A4\]  
053A5A03  |.  33FF          |XOR EDI,EDI  
053A5A05  |.  2BC8          |SUB ECX,EAX  
053A5A07  |>  8D843D 5CFEFF>|/LEA EAX,DWORD PTR SS:\[EBP+EDI-1A4\]  ; 读入 API 地址所指的字符 256 byte  
053A5A0E  |.  47            ||INC EDI  
053A5A0F  |.  81FF 00010000 ||CMP EDI,100  
053A5A15  |.  8A1401        ||MOV DL,BYTE PTR DS:\[ECX+EAX\]  
053A5A18  |.  8810          ||MOV BYTE PTR DS:\[EAX\],DL  
053A5A1A  |.^ 72 EB         |\\JB SHORT EXERe.053A5A07  
053A5A1C  |>  33C0          |XOR EAX,EAX  
053A5A1E  |>  80BC05 5CFEFF>|/CMP BYTE PTR SS:\[EBP+EAX-1A4\],2E    ; 找到 "."  
053A5A26  |.  8945 F0       ||MOV DWORD PTR SS:\[EBP-10\],EAX  
053A5A29  |.  74 0A         ||JE SHORT EXERe.053A5A35  
053A5A2B  |.  3D 00010000   ||CMP EAX,100  
053A5A30  |.  77 0B         ||JA SHORT EXERe.053A5A3D  
053A5A32  |.  40            ||INC EAX  
053A5A33  |.^ EB E9         |\\JMP SHORT EXERe.053A5A1E  
053A5A35  |>  80A405 5CFEFF>|AND BYTE PTR SS:\[EBP+EAX-1A4\],0      ; "." 变成 0  
053A5A3D  |>  8D85 5CFEFFFF |LEA EAX,DWORD PTR SS:\[EBP-1A4\]  
053A5A43  |.  50            |PUSH EAX  
053A5A44  |.  A1 7CA03B05   |MOV EAX,DWORD PTR DS:\[53BA07C\]  
053A5A49  |.  F7D0          |NOT EAX  
053A5A4B  |.  FFD0          |CALL EAX                             ; GetModuleHandleA(DLL2)  
053A5A4D  |.  8BF8          |MOV EDI,EAX  
053A5A4F  |.  3BFE          |CMP EDI,ESI  
053A5A51  |.  75 15         |JNZ SHORT EXERe.053A5A68  
053A5A53  |.  8D85 5CFEFFFF |LEA EAX,DWORD PTR SS:\[EBP-1A4\]  
053A5A59  |.  50            |PUSH EAX  
053A5A5A  |.  A1 84A03B05   |MOV EAX,DWORD PTR DS:\[53BA084\]  
053A5A5F  |.  F7D0          |NOT EAX  
053A5A61  |.  FFD0          |CALL EAX                             ; LoadLibraryA  
053A5A63  |.  E8 3DFAFFFF   |CALL EXERe.053A54A5                  ; 修正 53B8C38-53B8C40 所指的 UniName 列表  
053A5A68  |>  8B45 F0       |MOV EAX,DWORD PTR SS:\[EBP-10\]          
053A5A6B  |.  8D8405 5DFEFF>|LEA EAX,DWORD PTR SS:\[EBP+EAX-1A3\]   ; 指向 API2  
053A5A72  |.  50            |PUSH EAX  
053A5A73  |.  A1 B0A03B05   |MOV EAX,DWORD PTR DS:\[53BA0B0\]  
053A5A78  |.  57            |PUSH EDI  
053A5A79  |.  F7D0          |NOT EAX  
053A5A7B  |.  FFD0          |CALL EAX                             ; KERNEL32.GetProcAddress  
053A5A7D  |.  8B4D 08       |MOV ECX,DWORD PTR SS:\[EBP+8\]         ; 已移动到的空间指针 1   
053A5A80  |.  2BC7          |SUB EAX,EDI                          ; DLL2 base  
053A5A82  |.  897D C8       |MOV DWORD PTR SS:\[EBP-38\],EDI        ; DLL2 base  
053A5A85  |.  8901          |MOV DWORD PTR DS:\[ECX\],EAX  
053A5A87  |.  8B45 F8       |MOV EAX,DWORD PTR SS:\[EBP-8\]         ; 已处理的 API 个数  
053A5A8A  |.  8945 C4       |MOV DWORD PTR SS:\[EBP-3C\],EAX  
053A5A8D  |.  8D45 C4       |LEA EAX,DWORD PTR SS:\[EBP-3C\]  
053A5A90  |.  50            |PUSH EAX  
053A5A91  |.  8D4D AC       |LEA ECX,DWORD PTR SS:\[EBP-54\]        ; 70FF60  
053A5A94  |.  E8 09EFFFFF   |CALL EXERe.053A49A2                  ; 修正 70FF60-70FF68 所指的 dll2 列表    structure dll2 { dword apiorder;   API 在 Dll1 中的序号          
053A5A99  |>  FF45 F8       |INC DWORD PTR SS:\[EBP-8\]                                                                      dword dll2base;   DLL2 基址 }  
053A5A9C  |.  8345 08 04    |ADD DWORD PTR SS:\[EBP+8\],4  
053A5AA0  |.  8B45 F8       |MOV EAX,DWORD PTR SS:\[EBP-8\]  
053A5AA3  |.  3B45 80       |CMP EAX,DWORD PTR SS:\[EBP-80\]  
053A5AA6  |.^ 0F8C 1CFFFFFF \\JL EXERe.053A59C8  
  
  
053A5AAC  |>  33FF          XOR EDI,EDI                           ; 循环变量  
053A5AAE  |.  3975 80       CMP DWORD PTR SS:\[EBP-80\],ESI         ; NumberOfFunctions > 0 ?  
053A5AB1  |.  0F8E 86000000 JLE EXERe.053A5B3D  
053A5AB7  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]          ; 空间指针 1  
053A5ABA  |.  8945 08       MOV DWORD PTR SS:\[EBP+8\],EAX  
053A5ABD  |>  8B45 08       /MOV EAX,DWORD PTR SS:\[EBP+8\]  
053A5AC0  |.  3930          |CMP DWORD PTR DS:\[EAX\],ESI           ; API RVA  
053A5AC2  |.  74 6F         |JE SHORT EXERe.053A5B33  
053A5AC4  |.  834D F0 FF    |OR DWORD PTR SS:\[EBP-10\],FFFFFFFF  
053A5AC8  |.  33C0          |XOR EAX,EAX  
053A5ACA  |.  3975 84       |CMP DWORD PTR SS:\[EBP-7C\],ESI        ; NumberOfNames > 0 ?  
053A5ACD  |.  7E 34         |JLE SHORT EXERe.053A5B03  
053A5ACF  |.  8B4D D8       |MOV ECX,DWORD PTR SS:\[EBP-28\]        ; 空间指针 2  
053A5AD2  |>  0FBF11        |/MOVSX EDX,WORD PTR DS:\[ECX\]         ; 循环找出 EDI  
053A5AD5  |.  3BD7          ||CMP EDX,EDI  
053A5AD7  |.  74 0A         ||JE SHORT EXERe.053A5AE3  
053A5AD9  |.  40            ||INC EAX  
053A5ADA  |.  41            ||INC ECX  
053A5ADB  |.  41            ||INC ECX  
053A5ADC  |.  3B45 84       ||CMP EAX,DWORD PTR SS:\[EBP-7C\]  
053A5ADF  |.^ 7C F1         |\\JL SHORT EXERe.053A5AD2  
053A5AE1  |.  EB 20         |JMP SHORT EXERe.053A5B03             ; 到这里说明这个 API 没有名字, 直接去 053A5B03  
053A5AE3  |>  3BC6          |CMP EAX,ESI  
053A5AE5  |.  8945 F0       |MOV DWORD PTR SS:\[EBP-10\],EAX        ; AddressOfNameOrdinals 中第 EAX 个为 AddressOfFunctions 中第 EDI 个  
053A5AE8  |.  7C 19         |JL SHORT EXERe.053A5B03  
053A5AEA  |.  C1E0 08       |SHL EAX,8                            ; \* 256  
053A5AED  |.  0345 DC       |ADD EAX,DWORD PTR SS:\[EBP-24\]        ; 空间指针 3  
053A5AF0  |.  56            |PUSH ESI  
053A5AF1  |.  50            |PUSH EAX  
053A5AF2  |.  E8 B0EBFFFF   |CALL EXERe.053A46A7                  ; uppercase  
053A5AF7  |.  59            |POP ECX  
053A5AF8  |.  50            |PUSH EAX  
053A5AF9  |.  E8 0B1A0000   |CALL EXERe.053A7509                  ; Hash 变换  
053A5AFE  |.  59            |POP ECX  
053A5AFF  |.  8945 9C       |MOV DWORD PTR SS:\[EBP-64\],EAX        ; API Name Hash        
053A5B02  |.  59            |POP ECX  
053A5B03  |>  8B85 7CFFFFFF |MOV EAX,DWORD PTR SS:\[EBP-84\]        ; Export Table 中的 base  
053A5B09  |.  6A 04         |PUSH 4                               ; Hash 变换只取 4 字节  
053A5B0B  |.  03C7          |ADD EAX,EDI                          ; API 对外的 Ordinal  
053A5B0D  |.  8945 E0       |MOV DWORD PTR SS:\[EBP-20\],EAX  
053A5B10  |.  8D45 E0       |LEA EAX,DWORD PTR SS:\[EBP-20\]  
053A5B13  |.  50            |PUSH EAX  
053A5B14  |.  E8 F0190000   |CALL EXERe.053A7509                  ; 对 Ordinal 做 Hash 变换  
053A5B19  |.  8945 98       |MOV DWORD PTR SS:\[EBP-68\],EAX        ; API Ordinal Hash  
053A5B1C  |.  8B45 F0       |MOV EAX,DWORD PTR SS:\[EBP-10\]        ;   
053A5B1F  |.  59            |POP ECX  
053A5B20  |.  8945 A0       |MOV DWORD PTR SS:\[EBP-60\],EAX          
053A5B23  |.  59            |POP ECX  
053A5B24  |.  8D45 94       |LEA EAX,DWORD PTR SS:\[EBP-6C\]  
053A5B27  |.  50            |PUSH EAX  
053A5B28  |.  8D4D B8       |LEA ECX,DWORD PTR SS:\[EBP-48\]        ; 70FF6C  
053A5B2B  |.  897D 94       |MOV DWORD PTR SS:\[EBP-6C\],EDI  
053A5B2E  |.  E8 31030000   |CALL EXERe.053A5E64                  ; 修正 70FF6C-70FF74 所指的 NameOrdinalHash 列表  
053A5B33  |>  8345 08 04    |ADD DWORD PTR SS:\[EBP+8\],4  
053A5B37  |.  47            |INC EDI  
053A5B38  |.  3B7D 80       |CMP EDI,DWORD PTR SS:\[EBP-80\]  
053A5B3B  |.^ 7C 80         \\JL SHORT EXERe.053A5ABD  
  
structure NameOrdinalHash  
{  
    dword order\_in\_AddressofFunctions;  
    dword OrdinalHash;  
    dword NameHash;  
    dword order\_in\_AddressofNameOrdinal;  
}  
      
  
053A5B3D  |>  A1 A0A03B05   /MOV EAX,DWORD PTR DS:\[53BA0A0\]      ; 都处理好了, 开始等待主线程的命令  
053A5B42  |.  6A 02         |PUSH 2  
053A5B44  |.  F7D0          |NOT EAX  
053A5B46  |.   FFD0         |CALL EAX                            ; Sleep(2ms)  
053A5B48  |.  803D 308C3B05>|CMP BYTE PTR DS:\[53B8C30\],0         ; 程序结束命令送到这  
053A5B4F  |.  0F85 E1010000 |JNZ EXERe.053A5D36  
053A5B55  |.  A1 94A03B05   |MOV EAX,DWORD PTR DS:\[53BA094\]      ; 程序没结束  
053A5B5A  |.  F7D0          |NOT EAX  
053A5B5C  |.  FFD0          |CALL EAX                            ; GetTickCount  
053A5B5E  |.  8B0D E0A03B05 |MOV ECX,DWORD PTR DS:\[53BA0E0\]      ; 053C0087 做的时间标志       
053A5B64  |.  BF E8030000   |MOV EDI,3E8                         ; 1000  
053A5B69  |.  2BC1          |SUB EAX,ECX                         ; 时间差  
053A5B6B  |.  99            |CDQ  
053A5B6C  |.  F7FF          |IDIV EDI  
053A5B6E  |.  33D2          |XOR EDX,EDX  
053A5B70  |.  8BF8          |MOV EDI,EAX                         ; 过了几秒  
053A5B72  |.  8BC1          |MOV EAX,ECX  
053A5B74  |.  B9 2C010000   |MOV ECX,12C                         ; 300  
053A5B79  |.  F7F1          |DIV ECX  
053A5B7B  |.  03D1          |ADD EDX,ECX  
053A5B7D  |.  3BFA          |CMP EDI,EDX                         ; 唯一的一个 Anti  
053A5B7F  |.  7E 0C         |JLE SHORT EXERe.053A5B8D            ; 改成 JMP  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
053A5B81  |.  A1 A8A03B05   |MOV EAX,DWORD PTR DS:\[53BA0A8\]  
053A5B86  |.  56            |PUSH ESI  
053A5B87  |.  6A FF         |PUSH -1  
053A5B89  |.  F7D0          |NOT EAX  
053A5B8B  |.  FFD0          |CALL EAX                            ; TerminateProcess  
053A5B8D  |>  A1 508C3B05   |MOV EAX,DWORD PTR DS:\[53B8C50\]      ; 主线程的命令送到这  
053A5B92  |.  3BC6          |CMP EAX,ESI  
053A5B94  |.^ 7E A7         |JLE SHORT EXERe.053A5B3D  
053A5B96  |.  8975 F0       |MOV DWORD PTR SS:\[EBP-10\],ESI  
053A5B99  |.  8945 E0       |MOV DWORD PTR SS:\[EBP-20\],EAX       ; 队列中需要处理的 API 个数  
053A5B9C  |>  A1 488C3B05   |/MOV EAX,DWORD PTR DS:\[53B8C48\]     ; 开始地址  
053A5BA1  |.  8B4D F0       ||MOV ECX,DWORD PTR SS:\[EBP-10\]      ; 已处理过  
053A5BA4  |.  03C1          ||ADD EAX,ECX  
053A5BA6  |.  8B4D A8       ||MOV ECX,DWORD PTR SS:\[EBP-58\]      ; Dll Name Hash(线程的局部变量)  
053A5BA9  |.  8945 F8       ||MOV DWORD PTR SS:\[EBP-8\],EAX  
053A5BAC  |.  3908          ||CMP DWORD PTR DS:\[EAX\],ECX         ; 不相等说明不是本 DLL 的, 队列中下一个 API  
053A5BAE  |.  0F85 6E010000 ||JNZ EXERe.053A5D22  
053A5BB4  |.  8065 0B 00    ||AND BYTE PTR SS:\[EBP+B\],0  
053A5BB8  |.  8B40 08       ||MOV EAX,DWORD PTR DS:\[EAX+8\]       ; 0 或 API Address  
053A5BBB  |.  8B55 C0       ||MOV EDX,DWORD PTR SS:\[EBP-40\]      ; NameOrdinalHash 列表中个数  
053A5BBE  |.  8BF0          ||MOV ESI,EAX  
053A5BC0  |.  B9 000000F0   ||MOV ECX,F0000000  
053A5BC5  |.  8955 C8       ||MOV DWORD PTR SS:\[EBP-38\],EDX  
053A5BC8  |.  23F1          ||AND ESI,ECX  
053A5BCA  |.  3BF1          ||CMP ESI,ECX  
053A5BCC  |.  74 08         ||JE SHORT EXERe.053A5BD6            ; Fxxxxxxx 重新来过  
053A5BCE  |.  85C0          ||TEST EAX,EAX                       ; 0 or API address  
053A5BD0  |.  0F85 4C010000 ||JNZ EXERe.053A5D22                 ; 不等于 0 的话就跳过  
053A5BD6  |>  8365 EC 00    ||AND DWORD PTR SS:\[EBP-14\],0  
053A5BDA  |.  85D2          ||TEST EDX,EDX  
053A5BDC  |.  0F8E 40010000 ||JLE EXERe.053A5D22  
053A5BE2  |>  8B45 EC       ||/MOV EAX,DWORD PTR SS:\[EBP-14\]  
053A5BE5  |.  8365 E8 00    |||AND DWORD PTR SS:\[EBP-18\],0       ; 0, 1 两种情况  
053A5BE9  |.  C1E0 04       |||SHL EAX,4  
053A5BEC  |.  8945 A4       |||MOV DWORD PTR SS:\[EBP-5C\],EAX  
053A5BEF  |>  8B45 B8       |||/MOV EAX,DWORD PTR SS:\[EBP-48\]    ; NameOrdianlHash 列表开始地址  
053A5BF2  |.  8B4D A4       ||||MOV ECX,DWORD PTR SS:\[EBP-5C\]  
053A5BF5  |.  8DBD 5CFFFFFF ||||LEA EDI,DWORD PTR SS:\[EBP-A4\]  
053A5BFB  |.  8D3401        ||||LEA ESI,DWORD PTR DS:\[ECX+EAX\]  
053A5BFE  |.  8B45 E8       ||||MOV EAX,DWORD PTR SS:\[EBP-18\]  
053A5C01  |.  A5            ||||MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  
053A5C02  |.  A5            ||||MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  
053A5C03  |.  A5            ||||MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  
053A5C04  |.  83E8 00       ||||SUB EAX,0                            ;  Switch (cases 0..1)  
053A5C07  |.  A5            ||||MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  
053A5C08  |.  74 0B         ||||JE SHORT EXERe.053A5C15  
053A5C0A  |.  48            ||||DEC EAX  
053A5C0B  |.  75 11         ||||JNZ SHORT EXERe.053A5C1E  
053A5C0D  |.  8B85 60FFFFFF ||||MOV EAX,DWORD PTR SS:\[EBP-A0\]        ;  Case 1 of switch 053A5C04, 对应 OridnalHash, 没有名字  
053A5C13  |.  EB 06         ||||JMP SHORT EXERe.053A5C1B  
053A5C15  |>  8B85 64FFFFFF ||||MOV EAX,DWORD PTR SS:\[EBP-9C\]        ;  Case 0 of switch 053A5C04, 对应 NameHash  
053A5C1B  |>  8945 D4       ||||MOV DWORD PTR SS:\[EBP-2C\],EAX  
053A5C1E  |>  8B55 B4       ||||MOV EDX,DWORD PTR SS:\[EBP-4C\]        ;  Default case of switch 053A5C04, 是否是指向另一 DLL 的 API  
053A5C21  |.  8065 FF 00    ||||AND BYTE PTR SS:\[EBP-1\],0  
053A5C25  |.  8365 E4 00    ||||AND DWORD PTR SS:\[EBP-1C\],0  
053A5C29  |.  8B85 5CFFFFFF ||||MOV EAX,DWORD PTR SS:\[EBP-A4\]  
053A5C2F  |.  85D2          ||||TEST EDX,EDX  
053A5C31  |.  7E 20         ||||JLE SHORT EXERe.053A5C53  
053A5C33  |.  8B4D AC       ||||MOV ECX,DWORD PTR SS:\[EBP-54\]  
053A5C36  |>  8B31          ||||/MOV ESI,DWORD PTR DS:\[ECX\]  
053A5C38  |.  8B79 04       |||||MOV EDI,DWORD PTR DS:\[ECX+4\]  
053A5C3B  |.  3BF0          |||||CMP ESI,EAX  
053A5C3D  |.  897D A0       |||||MOV DWORD PTR SS:\[EBP-60\],EDI       ; 另一 DLL 的 BaseAddress  
053A5C40  |.  74 0D         |||||JE SHORT EXERe.053A5C4F  
053A5C42  |.  FF45 E4       |||||INC DWORD PTR SS:\[EBP-1C\]  
053A5C45  |.  83C1 08       |||||ADD ECX,8  
053A5C48  |.  3955 E4       |||||CMP DWORD PTR SS:\[EBP-1C\],EDX  
053A5C4B  |.^ 7C E9         ||||\\JL SHORT EXERe.053A5C36  
053A5C4D  |.  EB 04         ||||JMP SHORT EXERe.053A5C53  
053A5C4F  |>  C645 FF 01    ||||MOV BYTE PTR SS:\[EBP-1\],1            ; 指向另一 DLL 的标志  
053A5C53  |>  8B7D F8       ||||MOV EDI,DWORD PTR SS:\[EBP-8\]  
053A5C56  |.  8B4F 04       ||||MOV ECX,DWORD PTR DS:\[EDI+4\]         ; 队列中 API　Hash  
053A5C59  |.  3B4D D4       ||||CMP ECX,DWORD PTR SS:\[EBP-2C\]        ; 运气非常好，第一个就成功？  
053A5C5C  |.  894D CC       ||||MOV DWORD PTR SS:\[EBP-34\],ECX  
053A5C5F  |.  75 55         ||||JNZ SHORT EXERe.053A5CB6   
053A5C61  |.  8365 D0 00    ||||AND DWORD PTR SS:\[EBP-30\],0  
053A5C65  |.  83BD 3CFEFFFF>||||CMP DWORD PTR SS:\[EBP-1C4\],0  
053A5C6C  |.  0F8E 91000000 ||||JLE EXERe.053A5D03  
053A5C72  |.  8B75 F4       ||||MOV ESI,DWORD PTR SS:\[EBP-C\]  
053A5C75  |.  8D95 74FDFFFF ||||LEA EDX,DWORD PTR SS:\[EBP-28C\]  
053A5C7B  |.  8D8D B0F5FFFF ||||LEA ECX,DWORD PTR SS:\[EBP-A50\]  
053A5C81  |.  8955 E4       ||||MOV DWORD PTR SS:\[EBP-1C\],EDX  
053A5C84  |>  8B55 E4       ||||/MOV EDX,DWORD PTR SS:\[EBP-1C\]  
053A5C87  |.  833A 00       |||||CMP DWORD PTR DS:\[EDX\],0  
053A5C8A  |.  74 13         |||||JE SHORT EXERe.053A5C9F  
053A5C8C  |.  8B3C86        |||||MOV EDI,DWORD PTR DS:\[ESI+EAX\*4\]  
053A5C8F  |.  8B11          |||||MOV EDX,DWORD PTR DS:\[ECX\]  
053A5C91  |.  3BFA          |||||CMP EDI,EDX  
053A5C93  |.  72 0A         |||||JB SHORT EXERe.053A5C9F  
053A5C95  |.  8B79 FC       |||||MOV EDI,DWORD PTR DS:\[ECX-4\]  
053A5C98  |.  03FA          |||||ADD EDI,EDX  
053A5C9A  |.  393C86        |||||CMP DWORD PTR DS:\[ESI+EAX\*4\],EDI  
053A5C9D  |.  72 17         |||||JB SHORT EXERe.053A5CB6  
053A5C9F  |>  FF45 D0       |||||INC DWORD PTR SS:\[EBP-30\]  
053A5CA2  |.  8345 E4 04    |||||ADD DWORD PTR SS:\[EBP-1C\],4  
053A5CA6  |.  8B55 D0       |||||MOV EDX,DWORD PTR SS:\[EBP-30\]  
053A5CA9  |.  83C1 28       |||||ADD ECX,28  
053A5CAC  |.  3B95 3CFEFFFF |||||CMP EDX,DWORD PTR SS:\[EBP-1C4\]  
053A5CB2  |.  7D 54         |||||JGE SHORT EXERe.053A5D08  
053A5CB4  |.^ EB CE         ||||\\JMP SHORT EXERe.053A5C84  
053A5CB6  |>  807D FF 00    ||||CMP BYTE PTR SS:\[EBP-1\],0           ；是否指向另一 DLL 的标志  
053A5CBA  |.  8D4D 0B       ||||LEA ECX,DWORD PTR SS:\[EBP+B\]      
053A5CBD  |.  51            ||||PUSH ECX                             ; Arg5 返回是否 stolen 成功的标志  
053A5CBE  |.  8B4D F4       ||||MOV ECX,DWORD PTR SS:\[EBP-C\]  
053A5CC1  |.  FF3481        ||||PUSH DWORD PTR DS:\[ECX+EAX\*4\]        ; Arg4   
053A5CC4  |.  74 05         ||||JE SHORT EXERe.053A5CCB  
053A5CC6  |.  FF75 A0       ||||PUSH DWORD PTR SS:\[EBP-60\]           ;  Arg3( Dll2 base)  
053A5CC9  |.  EB 01         ||||JMP SHORT EXERe.053A5CCC  
053A5CCB  |>  53            ||||PUSH EBX                                Arg3( DLL1 base)  
053A5CCC  |>  FF75 D4       ||||PUSH DWORD PTR SS:\[EBP-2C\]           ; |Arg2  
053A5CCF  |.  FF75 CC       ||||PUSH DWORD PTR SS:\[EBP-34\]           ; |Arg1  
053A5CD2  |.  E8 B0000000   ||||CALL EXERe.053A5D87                  ; \\EXERe.053A5D87  
053A5CD7  |.  8B4D F8       ||||MOV ECX,DWORD PTR SS:\[EBP-8\]  
053A5CDA  |.  83C4 14       ||||ADD ESP,14  
053A5CDD  |.  8941 08       ||||MOV DWORD PTR DS:\[ECX+8\],EAX         ; 为主线程返回 API 地址  
053A5CE0  |.  807D 0B 00    ||||CMP BYTE PTR SS:\[EBP+B\],0            ; 是否 Stolen 成功  
053A5CE4  |.  75 3C         ||||JNZ SHORT EXERe.053A5D22  
053A5CE6  |.  FF45 E8       ||||INC DWORD PTR SS:\[EBP-18\]  
053A5CE9  |.  837D E8 02    ||||CMP DWORD PTR SS:\[EBP-18\],2  
053A5CED  |.^ 0F8C FCFEFFFF |||\\JL EXERe.053A5BEF  
053A5CF3  |.  FF45 EC       |||INC DWORD PTR SS:\[EBP-14\]  
053A5CF6  |.  8B45 EC       |||MOV EAX,DWORD PTR SS:\[EBP-14\]  
053A5CF9  |.  3B45 C8       |||CMP EAX,DWORD PTR SS:\[EBP-38\]  
053A5CFC  |.  7D 24         |||JGE SHORT EXERe.053A5D22  
053A5CFE  |.^ E9 DFFEFFFF   ||\\JMP EXERe.053A5BE2  
053A5D03  |>  8B75 F4       ||MOV ESI,DWORD PTR SS:\[EBP-C\]  
053A5D06  |.  EB 03         ||JMP SHORT EXERe.053A5D0B  
053A5D08  |>  8B7D F8       ||MOV EDI,DWORD PTR SS:\[EBP-8\]  
053A5D0B  |>  807D FF 00    ||CMP BYTE PTR SS:\[EBP-1\],0  
053A5D0F  |.  8B0486        ||MOV EAX,DWORD PTR DS:\[ESI+EAX\*4\]  
053A5D12  |.  74 05         ||JE SHORT EXERe.053A5D19  
053A5D14  |.  0345 A0       ||ADD EAX,DWORD PTR SS:\[EBP-60\]  
053A5D17  |.  EB 02         ||JMP SHORT EXERe.053A5D1B  
053A5D19  |>  03C3          ||ADD EAX,EBX  
053A5D1B  |>  8947 08       ||MOV DWORD PTR DS:\[EDI+8\],EAX  
053A5D1E  |.  C645 0B 01    ||MOV BYTE PTR SS:\[EBP+B\],1  
053A5D22  |>  8345 F0 0C    ||ADD DWORD PTR SS:\[EBP-10\],0C  
053A5D26  |.  FF4D E0       ||DEC DWORD PTR SS:\[EBP-20\]  
053A5D29  |.^ 0F85 6DFEFFFF |\\JNZ EXERe.053A5B9C  
053A5D2F  |.  33F6          |XOR ESI,ESI  
053A5D31  |.^ E9 07FEFFFF   \\JMP EXERe.053A5B3D  
053A5D36  |>  3975 F4       CMP DWORD PTR SS:\[EBP-C\],ESI  
053A5D39  |.  74 13         JE SHORT EXERe.053A5D4E  
053A5D3B  |.  FF75 F4       PUSH DWORD PTR SS:\[EBP-C\]               ; 空间指针 1  
053A5D3E  |.  A1 90A03B05   MOV EAX,DWORD PTR DS:\[53BA090\]  
053A5D43  |.  F7D0          NOT EAX  
053A5D45  |.  56            PUSH ESI  
053A5D46  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A5D4C  |.  FFD0          CALL EAX                                ; HeapFree  
053A5D4E  |>  3975 D8       CMP DWORD PTR SS:\[EBP-28\],ESI  
053A5D51  |.  74 13         JE SHORT EXERe.053A5D66  
053A5D53  |.  FF75 D8       PUSH DWORD PTR SS:\[EBP-28\]              ; 空间指针 2  
053A5D56  |.  A1 90A03B05   MOV EAX,DWORD PTR DS:\[53BA090\]  
053A5D5B  |.  F7D0          NOT EAX  
053A5D5D  |.  56            PUSH ESI  
053A5D5E  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A5D64  |.  FFD0          CALL EAX                                ; HeapFree  
053A5D66  |>  3975 DC       CMP DWORD PTR SS:\[EBP-24\],ESI  
053A5D69  |.  74 13         JE SHORT EXERe.053A5D7E  
053A5D6B  |.  FF75 DC       PUSH DWORD PTR SS:\[EBP-24\]              ; 空间指针 3  
053A5D6E  |.  A1 90A03B05   MOV EAX,DWORD PTR DS:\[53BA090\]  
053A5D73  |.  F7D0          NOT EAX  
053A5D75  |.  56            PUSH ESI  
053A5D76  |.  FF35 D4A03B05 PUSH DWORD PTR DS:\[53BA0D4\]  
053A5D7C  |.  FFD0          CALL EAX                                ; HeapFree  
053A5D7E  |>  5F            POP EDI  
053A5D7F  |.  5E            POP ESI  
053A5D80  |.  33C0          XOR EAX,EAX  
053A5D82  |.  5B            POP EBX  
053A5D83  |.  C9            LEAVE  
053A5D84  \\.  C2 0400       RETN 4  
  
  
  
  
Proc 53A5D87(arg1, arg2, arg3, arg4, arg5)  
arg1 = 要求的 API Hash  
arg2 = NameOrdinalHash 列表中 API Hash  
arg3 = DLL BASE  
arg4 = NameOrdinalHash 列表中 API address RVA  
arg5 = pDword, 返回是否成功的标志  
  
053A5D87  /$  55            PUSH EBP                                          
053A5D88  |.  8BEC          MOV EBP,ESP  
053A5D8A  |.  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]  
053A5D8D  |.  3B45 0C       CMP EAX,DWORD PTR SS:\[EBP+C\]  
053A5D90  |.  75 2D         JNZ SHORT EXERe.053A5DBF  
053A5D92  |.  8B4D 14       MOV ECX,DWORD PTR SS:\[EBP+14\]  
053A5D95  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]  
053A5D98  |.  8365 08 00    AND DWORD PTR SS:\[EBP+8\],0  
053A5D9C  |.  6A 00         PUSH 0  
053A5D9E  |.  50            PUSH EAX  
053A5D9F  |.  8B45 10       MOV EAX,DWORD PTR SS:\[EBP+10\]  
053A5DA2  |.  03C8          ADD ECX,EAX  
053A5DA4  |.  51            PUSH ECX  
053A5DA5  |.  50            PUSH EAX  
053A5DA6  |.  E8 81B2FFFF   CALL EXERe.053A102C                              ;  偷 API 的 CALL  
053A5DAB  |.  8B4D 18       MOV ECX,DWORD PTR SS:\[EBP+18\]  
053A5DAE  |.  83C4 10       ADD ESP,10  
053A5DB1  |.  837D 08 00    CMP DWORD PTR SS:\[EBP+8\],0                       ; StolenSize>0 ?  
053A5DB5  |.  C601 01       MOV BYTE PTR DS:\[ECX\],1  
053A5DB8  |.  75 12         JNZ SHORT EXERe.053A5DCC  
053A5DBA  |.  8021 00       AND BYTE PTR DS:\[ECX\],0                            
053A5DBD  |.  5D            POP EBP  
053A5DBE  |.  C3            RETN  
053A5DBF  |>  8B45 14       MOV EAX,DWORD PTR SS:\[EBP+14\]  
053A5DC2  |.  8B4D 10       MOV ECX,DWORD PTR SS:\[EBP+10\]  
053A5DC5  |.  03C1          ADD EAX,ECX  
053A5DC7  |.  0D 000000F0   OR EAX,F0000000  
053A5DCC  |>  5D            POP EBP  
053A5DCD  \\.  C3            RETN  
  
  
  
  
  
053A5776  /.  55            PUSH EBP  
053A5777  |.  8BEC          MOV EBP,ESP  
053A5779  |.  81EC 540A0000 SUB ESP,0A54  
053A577F  |.  A1 A0A03B05   MOV EAX,DWORD PTR DS:\[53BA0A0\]           
053A5784  |.  F7D0          NOT EAX  
053A5786  |.  85C0          TEST EAX,EAX  
053A5788  |.  0F84 AC000000 JE EXERe.053A583A  
053A578E  |.  68 60EA0000   PUSH 0EA60        
053A5793  |.  FFD0          CALL EAX                                 ; Sleep (60s)  
053A5795  |>  A1 A0A03B05   /MOV EAX,DWORD PTR DS:\[53BA0A0\]  
053A579A  |.  6A 02         |PUSH 2  
053A579C  |.  F7D0          |NOT EAX  
053A579E  |.  FFD0          |CALL EAX  
053A57A0  |.  E8 09FFFFFF   |CALL EXERe.053A56AE                     ; 判断 User32 是否已加载  
053A57A5  |.  85C0          |TEST EAX,EAX  
053A57A7  |.^ 74 EC         \\JE SHORT EXERe.053A5795                 ; 未加载, 等待 2 ms  
053A57A9  |.  8D8D ACF5FFFF LEA ECX,DWORD PTR SS:\[EBP-A54\]  
053A57AF  |.  E8 A9CAFFFF   CALL EXERe.053A225D                      ; zero buffer  
053A57B4  |.  E8 F5FEFFFF   CALL EXERe.053A56AE                      ; 取 user32 base address  
053A57B9  |.  F7D0          NOT EAX  
053A57BB  |.  50            PUSH EAX                                 ; /Arg1  
053A57BC  |.  8D8D ACF5FFFF LEA ECX,DWORD PTR SS:\[EBP-A54\]           ; |  
053A57C2  |.  E8 09100000   CALL EXERe.053A67D0                      ; \\EXERe.053A67D0 ( 分析区块信息)  
053A57C7  |.  85C0          TEST EAX,EAX  
053A57C9  |.  74 6F         JE SHORT EXERe.053A583A  
053A57CB  |.  68 50083B05   PUSH EXERe.053B0850                      ; /Arg1 = 053B0850  
053A57D0  |.  8D8D ACF5FFFF LEA ECX,DWORD PTR SS:\[EBP-A54\]           ; |  
053A57D6  |.  E8 3C180000   CALL EXERe.053A7017                      ; \\EXERe.053A7017 (  Stolen MessageBoxExW)  
053A57DB  |.  85C0          TEST EAX,EAX  
053A57DD  |.  A3 8CA03B05   MOV DWORD PTR DS:\[53BA08C\],EAX  
053A57E2  |.  74 56         JE SHORT EXERe.053A583A  
053A57E4  |.  56            PUSH ESI  
053A57E5  |.  57            PUSH EDI  
053A57E6  |.  8D4D FC       LEA ECX,DWORD PTR SS:\[EBP-4\]  
053A57E9  |.  6A 00         PUSH 0  
053A57EB  |.  F7D0          NOT EAX  
053A57ED  |.  51            PUSH ECX  
053A57EE  |.  50            PUSH EAX  
053A57EF  |.  8B45 8C       MOV EAX,DWORD PTR SS:\[EBP-74\]  
053A57F2  |.  F7D0          NOT EAX  
053A57F4  |.  50            PUSH EAX  
053A57F5  |.  E8 32B8FFFF   CALL EXERe.053A102C  
053A57FA  |.  83C4 10       ADD ESP,10  
053A57FD  |.  BE 9CCF3A05   MOV ESI,EXERe.053ACF9C  
053A5802  |.  8D7D 9C       LEA EDI,DWORD PTR SS:\[EBP-64\]  
053A5805  |.  6A 0C         PUSH 0C  
053A5807  |.  59            POP ECX  
053A5808  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS> ; 提示信息  
053A580A  |.  66:A5         MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  
053A580C  |.  6A 0A         PUSH 0A  
053A580E  |.  BE 70CF3A05   MOV ESI,EXERe.053ACF70  
053A5813  |.  59            POP ECX  
053A5814  |.  8D7D D0       LEA EDI,DWORD PTR SS:\[EBP-30\]  
053A5817  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS> ; 提示信息  
053A5819  |.  F7D0          NOT EAX  
053A581B  |.  66:A5         MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  
053A581D  |.  5F            POP EDI  
053A581E  |.  A3 8CA03B05   MOV DWORD PTR DS:\[53BA08C\],EAX  
053A5823  |.  85C0          TEST EAX,EAX  
053A5825  |.  5E            POP ESI  
053A5826  |.  74 12         JE SHORT EXERe.053A583A  
053A5828  |.  6A 00         PUSH 0  
053A582A  |.  8D4D D2       LEA ECX,DWORD PTR SS:\[EBP-2E\]  
053A582D  |.  6A 00         PUSH 0  
053A582F  |.  51            PUSH ECX  
053A5830  |.  8D4D 9E       LEA ECX,DWORD PTR SS:\[EBP-62\]  
053A5833  |.  51            PUSH ECX  
053A5834  |.  6A 00         PUSH 0  
053A5836  |.  F7D0          NOT EAX  
053A5838  |.  FFD0          CALL EAX                                ; 显示未注册           
053A583A  |>  33C0          XOR EAX,EAX  
053A583C  |.  C9            LEAVE  
053A583D  \\.  C2 0400       RETN 4  
  
  
  
  
十六. 壳处理好的资源, 没有好的办法自动变成标准格式, 我是手动建立的.  
  
053B8C68  00 00 00 00   
053B8C6C  00 00 00 00   
053B8C70  C8 BB 15 00      ;Rsrc1 结构队列开始地址    
053B8C74  2C BC 15 00      ;Rsrc1 结构队列结束地址  
053B8C78  19 00 00 00      ;Rsrc1 结构队列个数                                     
  
Structure Rsrc1  
{  
   Rsrc2\*  address;  
}  
  
Structure Rsrc2  
{  
   dword  ResType;         ; 如果 ResType=-1, 名字导入,   
   byte   dup 208h (0);    ; 名字在这里  
   dword  ResID;  
   dword  ??  
   dword  ??  
   dword  size;  
   dword  0;  
   dword  0;  
   dword  RsrcAddress;  
}  
  
  
壳用下面的函数来找资源  
  
proc53A6437(ResType, ResID, pSize)  
  
ret = 地址  
  
053A6437  /$  55            PUSH EBP  
053A6438  |.  8BEC          MOV EBP,ESP  
053A643A  |.  51            PUSH ECX  
053A643B  |.  51            PUSH ECX  
053A643C  |.  A1 788C3B05   MOV EAX,DWORD PTR DS:\[53B8C78\]           ; Rsrc 个数  
053A6441  |.  53            PUSH EBX  
053A6442  |.  33DB          XOR EBX,EBX  
053A6444  |.  56            PUSH ESI  
053A6445  |.  3BC3          CMP EAX,EBX  
053A6447  |.  57            PUSH EDI  
053A6448  |.  8945 F8       MOV DWORD PTR SS:\[EBP-8\],EAX  
053A644B  |.  895D FC       MOV DWORD PTR SS:\[EBP-4\],EBX             ; 结果先清 0  
053A644E  |.  0F8E A7000000 JLE EXERe.053A64FB  
053A6454  |.  83CF FF       OR EDI,FFFFFFFF                          ; EDI = -1  
053A6457  |>  A1 708C3B05   /MOV EAX,DWORD PTR DS:\[53B8C70\]  
053A645C  |.  8B3498        |MOV ESI,DWORD PTR DS:\[EAX+EBX\*4\]        ; ESI 每一块资源的地址   
053A645F  |.  8B06          |MOV EAX,DWORD PTR DS:\[ESI\]              ; EAX 表示 资源 Type                
053A6461  |.  3BC7          |CMP EAX,EDI  
053A6463  |.  74 18         |JE SHORT EXERe.053A647D                 ; \[ESI\] == -1 ? 053A647D : 053A6465  
  
053A6465  |.  8B8E 0C020000 |MOV ECX,DWORD PTR DS:\[ESI+20C\]          ; ECX 表示 资源 ID  
053A646B  |.  3BCF          |CMP ECX,EDI  
053A646D  |.  74 0A         |JE SHORT EXERe.053A6479                 ; \[ESI + 20c\] == -1 ?  
  
053A646F  |.  3B45 08       |CMP EAX,DWORD PTR SS:\[EBP+8\]            ; EAX == ResType?  
053A6472  |.  75 67         |JNZ SHORT EXERe.053A64DB  
053A6474  |.  3B4D 0C       |CMP ECX,DWORD PTR SS:\[EBP+C\]            ; ECX == ResID?  
053A6477  |.  EB 60         |JMP SHORT EXERe.053A64D9  
  
053A6479  |>  3BC7          |CMP EAX,EDI  
053A647B  |.  75 1E         |JNZ SHORT EXERe.053A649B  
  
053A647D  |>  8B8E 0C020000 |MOV ECX,DWORD PTR DS:\[ESI+20C\]         ; ECX 表示 资源 ID  
053A6483  |.  3BCF          |CMP ECX,EDI  
053A6485  |.  74 10         |JE SHORT EXERe.053A6497                ; \[ESI + 20c\] == -1 ?  
053A6487  |.  3B4D 0C       |CMP ECX,DWORD PTR SS:\[EBP+C\]           ; == ResID?  
053A648A  |.  75 4F         |JNZ SHORT EXERe.053A64DB                 
053A648C  |.  FF75 08       |PUSH DWORD PTR SS:\[EBP+8\]              ; ResType  
053A648F  |.  8D86 0C010000 |LEA EAX,DWORD PTR DS:\[ESI+10C\]         ; \[ESI + 10c\]  
053A6495  |.  EB 38         |JMP SHORT EXERe.053A64CF  
  
053A6497  |>  3BC7          |CMP EAX,EDI  
053A6499  |.  74 11         |JE SHORT EXERe.053A64AC  
053A649B  |>  39BE 0C020000 |CMP DWORD PTR DS:\[ESI+20C\],EDI  
053A64A1  |.  75 05         |JNZ SHORT EXERe.053A64A8  
053A64A3  |.  3B45 08       |CMP EAX,DWORD PTR SS:\[EBP+8\]           ; == ResType?  
053A64A6  |.  EB 1F         |JMP SHORT EXERe.053A64C7  
053A64A8  |>  3BC7          |CMP EAX,EDI  
053A64AA  |.  75 2F         |JNZ SHORT EXERe.053A64DB  
  
053A64AC  |>  39BE 0C020000 |CMP DWORD PTR DS:\[ESI+20C\],EDI  
053A64B2  |.  75 27         |JNZ SHORT EXERe.053A64DB  
053A64B4  |.  FF75 08       |PUSH DWORD PTR SS:\[EBP+8\]              ; == ResType?  
053A64B7  |.  8D86 0C010000 |LEA EAX,DWORD PTR DS:\[ESI+10C\]  
053A64BD  |.  50            |PUSH EAX  
053A64BE  |.  E8 42000000   |CALL EXERe.053A6505  
053A64C3  |.  59            |POP ECX  
053A64C4  |.  59            |POP ECX  
053A64C5  |.  85C0          |TEST EAX,EAX  
053A64C7  |>  75 12         |JNZ SHORT EXERe.053A64DB  
053A64C9  |.  FF75 0C       |PUSH DWORD PTR SS:\[EBP+C\]              ; == ResID/16+1  
053A64CC  |.  8D46 0C       |LEA EAX,DWORD PTR DS:\[ESI+C\]  
053A64CF  |>  50            |PUSH EAX  
053A64D0  |.  E8 30000000   |CALL EXERe.053A6505  
053A64D5  |.  59            |POP ECX  
053A64D6  |.  59            |POP ECX  
053A64D7  |.  85C0          |TEST EAX,EAX  
053A64D9  |>  74 0C         |JE SHORT EXERe.053A64E7              ; 如果相等就找到了  
053A64DB  |>  43            |INC EBX                              ; 到下一块去找  
053A64DC  |.  3B5D F8       |CMP EBX,DWORD PTR SS:\[EBP-8\]         ; 是否最后一块了?  
053A64DF  |.^ 0F8C 72FFFFFF \\JL EXERe.053A6457  
053A64E5  |.  EB 14         JMP SHORT EXERe.053A64FB              ; 资源找不到, 到这里  
053A64E7  |>  8B4D 10       MOV ECX,DWORD PTR SS:\[EBP+10\]  
053A64EA  |.  8B86 18020000 MOV EAX,DWORD PTR DS:\[ESI+218\]        ; size  
053A64F0  |.  8901          MOV DWORD PTR DS:\[ECX\],EAX  
053A64F2  |.  8B86 24020000 MOV EAX,DWORD PTR DS:\[ESI+224\]  
053A64F8  |.  8945 FC       MOV DWORD PTR SS:\[EBP-4\],EAX          ; 资源数据的地址  
053A64FB  |>  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
053A64FE  |.  5F            POP EDI  
053A64FF  |.  5E            POP ESI  
053A6500  |.  5B            POP EBX  
053A6501  |.  C9            LEAVE  
053A6502  \\.  C2 0C00       RETN 0C 

 4dd466d0e1e0cc123afbe504da2462ca