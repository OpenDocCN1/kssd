

# ImTOO DVD Audio Ripper 4.0.50.0522

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-393.htm`](https://www.pediy.com/kssd/pediy08/pediy8-393.htm)

ImTOO DVD Audio Ripper 4.0.50.0522  
  
旧版本已经有人分析过了，新版的算法差不多，只是分析的更细，供新手学习 MD5。  
  
Microsoft Visual C++ 7.0 Method2 \[Debug\]  
  
1.断点：  
bp MessageBoxExA ：  
00123F9C   77E180BC    /CALL 到 MessageBoxExA 来自 USER32.77E180B7  
00123FA0   00100122    |hOwner = 00100122 ('注册',class='#32770',parent=000302DE)  
00123FA4   01069858    |Text = "注册信息不正确!"  
00123FA8   0106FE20    |Title = "DVD Audio Ripper"  
00123FAC   00000030    |Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
00123FB0   00000000    \\LanguageID = 0 (LANG\_NEUTRAL)  
  
返回到 UILib71.dll 的领空：  
1001B773      E8 4C820000       call <jmp.&MFC71.#1123>  
1001B778      8D4C24 04         lea ecx,dword ptr ss:\[esp+4\]             ; 返回处  
  
往上找到关键：  
1001B738      56                push esi  
1001B739      8BF1              mov esi,ecx  
1001B73B      E8 D0F4FFFF       call UILib71.ImRegDlg::SaveRegInfo  
1001B740      E8 BBF8FFFF       call UILib71.ImRegDlg::IsValidRegInfo  
1001B745      85C0              test eax,eax  
1001B747      75 49             jnz short UILib71.1001B792  
  
2.其中 call UILib71.ImRegDlg::SaveRegInfo，将注册信息写入到注册表：  
先加密：1001AD2E      E8 2D7A0000    call UILib71.10022760      
1002279E      885424 18       mov byte ptr ss:\[esp+18\],dl  
100227A2      FF50 10         call dword ptr ds:\[eax+10\]         ; 加密注册码  
是这样加密的：先将用户名的 ASCII 码作为初始值进行繁琐的计算，得到值为 al，然后与注册码异或：  
1002274B      32D0           xor dl,al     
根据注册码长度多少位进行循环，如果用户名为 cyto，al 依次为：  
17,61,C9,CF,34,8D,0D,21,D4,16,8B,0D,22,96,09,7A,DB,3D,27,19,79,C0,14,3C,2B,10,C0,35,CC,08,43,02,12,1A,E8,3A,16,0A,0A...  
  
然后转换：1001AD4F      E8 DCF8FFFF    call UILib71.ImRegDlg::String2HexA  
  
再然后保存：  
1001AD9F      8B35 18700210     mov esi,dword ptr ds:\[<&ADVAPI32.RegSetValueExA>\]      
HKCU\\Software\\ImTOO\\DVD Audio Ripper 4\\RegInfo\\Name  SUCCESS  "cyto"    
HKCU\\Software\\ImTOO\\DVD Audio Ripper 4\\RegInfo\\Serial  SUCCESS  ""    
HKCU\\Software\\ImTOO\\DVD Audio Ripper 4\\RegInfo\\Code  SUCCESS  "2F 56 FF FA 34 BE 3F 10 E5 24 B8 39 17 A0 3E 42 "    
  
3.其中 call UILib71.ImRegDlg::IsValidRegInfo 是判断注册码的地方：  
从注册表取值，转换：  
1001B15C      E8 CFF3FFFF    call UILib71.ImRegDlg::Hex2StringA  
  
解密注册码：1001B198      E8 C3750000    call UILib71.10022760  
1002279E      885424 18       mov byte ptr ss:\[esp+18\],dl  
100227A2      FF50 10         call dword ptr ds:\[eax+10\]         ; 解密注册码  
解密刚好与加密一样，al 值没变。  
  
取前 20 位：  
1001B1DA      6A 14             push 14  
1001B1DC      8D4424 0C         lea eax,dword ptr ss:\[esp+C\]  
1001B1E0      50                push eax  
1001B1E1      8D4C24 1C         lea ecx,dword ptr ss:\[esp+1C\]  
1001B1E5      FF15 8C770210     call dword ptr ds:\[<&MFC71.#3997>\]          ; MFC71.7C188E36  
00124054   01A6BA38    ASCII "87654321123456788765"  
  
对注册码的要求：  
1001B234      8D4C24 14         lea ecx,dword ptr ss:\[esp+14\]  
1001B238      FF15 C8770210     call dword ptr ds:\[<&MFC71.#2902>\]         ; MFC71.7C146AB0  
1001B23E      83F8 27           cmp eax,27  
1001B241      0F85 AE030000     jnz UILib71.1001B5F5  
注册码要为 39 位。  
  
取内置的字符串：  
1001B257      E8 04A5FEFF    call UILib71.ImAppPref::GetAppInfo         ; ?  
1001B25C      83C0 38        add eax,38  
1001B25F      50             push eax  
1001B260      8D4C24 14      lea ecx,dword ptr ss:\[esp+14\]  
1001B264      FF15 70770210  call dword ptr ds:\[<&MFC71.#781>\]          ; ?  
堆栈  
0012405C   01065F00    ASCII "ImTOOdvdaudioripper4"  
  
根据要求取单数：  
1001B290      8BCE             mov ecx,esi  
1001B292      81E1 01000080    and ecx,80000001                       ; 取值的条件  
1001B298      79 05            jns short UILib71.1001B29F  
1001B29A      49               dec ecx  
1001B29B      83C9 FE          or ecx,FFFFFFFE  
1001B29E      41               inc ecx  
1001B29F      75 38            jnz short UILib71.1001B2D9  
1001B2A1      56               push esi                               ; 循环指针  
1001B2A2      8D4C24 14        lea ecx,dword ptr ss:\[esp+14\]          ; 内置字符串？  
1001B2A6      FF15 B8770210    call dword ptr ds:\[<&MFC71.#865>\]      ; 取值  
1001B2AC      8D4C24 0C        lea ecx,dword ptr ss:\[esp+C\]  
1001B2B0      50               push eax  
1001B2B1      FF15 6C750210    call dword ptr ds:\[<&MFC71.#908>\]      ; 保存取值  
1001B2B7      8D46 01          lea eax,dword ptr ds:\[esi+1\]           ; 指针＋1  
1001B2BA      99               cdq  
1001B2BB      B9 FF000000      mov ecx,0FF  
1001B2C0      F7F9             idiv ecx  
1001B2C2      84D2             test dl,dl  
1001B2C4      885424 08        mov byte ptr ss:\[esp+8\],dl  
1001B2C8      74 0F            je short UILib71.1001B2D9  
1001B2CA      8B5424 08        mov edx,dword ptr ss:\[esp+8\]  
1001B2CE      52               push edx  
1001B2CF      8D4C24 10        lea ecx,dword ptr ss:\[esp+10\]  
1001B2D3      FF15 6C750210    call dword ptr ds:\[<&MFC71.#908>\]      ; 保存指针＋1  
1001B2D9      8D4C24 10        lea ecx,dword ptr ss:\[esp+10\]  
1001B2DD      46               inc esi  
1001B2DE      FF15 C8770210    call dword ptr ds:\[<&MFC71.#2902>\]     ; MFC71.7C146AB0  
1001B2E4      3BF0             cmp esi,eax  
1001B2E6    ^ 7C A8            jl short UILib71.1001B290  
  
参数：ImTOOdvdaudioripper4  
根据循环指针 and 80000001 的值决定取值，也就是取单数并指针保存，最后值保存在 edx：  
01A6C888  49 01 54 03 4F 05 76 07  ITOv  
01A6C890  61 09 64 0B 6F 0D 69 0F  a.do.i  
01A6C898  70 11 72 13 00           pr.  
  
再同样的取双数并指针保存到后面：  
01A6C888  49 01 54 03 4F 05 76 07  ITOv  
01A6C890  61 09 64 0B 6F 0D 69 0F  a.do.i  
01A6C898  70 11 72 13 6D 02 4F 04  prmO  
01A6C8A0  64 06 64 08 75 0A 69 0C  ddu.i.  
01A6C8A8  72 0E 70 10 65 12 34 14  rpe4  
  
然后在最前面加了一个字节 31：  
1001B38F      FF15 84750210    call dword ptr ds:\[<&MFC71.#3850>\]    ; MFC71.7C189CDC  
D ECX:  
01A6C530  31 49 01 54 03 4F 05 76  1ITOv  
01A6C538  07 61 09 64 0B 6F 0D 69  a.d o.i  
01A6C540  0F 70 11 72 13 6D 02 4F  prmO  
01A6C548  04 64 06 64 08 75 0A 69  ddu.i  
01A6C550  0C 72 0E 70 10 65 12 34  .rpe4  
01A6C558  14 00                    .  
  
第一次连接：1001B47D      E8 4E6DFEFF      call UILib71.100021D0  
10002236      FF15 D0770210    call dword ptr ds:\[<&MFC71.#1489>\]     ; MFC71.7C15A7BD  
堆栈：  
00124014   01A6B630    ASCII "87654321123456788765"  
0012401C   01065F00    ASCII "ImTOOdvdaudioripper4"  
0012402C   01A6C970    ASCII "87654321123456788765ImTOOdvdaudioripper4"  
就是将输入的注册码前 20 位与内置的字符串 ImTOOdvdaudioripper4 连接。  
  
第二次连接：  
1001B492      FF15 80770210    call dword ptr ds:\[<&MFC71.#907>\]     ; MFC71.7C14E599  
D EDX;  
01A6CB00  31 49 01 54 03 4F 05 76 07 61 09 64 0B 6F 0D 69  1ITOva.d o.i  
01A6CB10  0F 70 11 72 13 6D 02 4F 04 64 06 64 08 75 0A 69  prmOddu.i  
01A6CB20  0C 72 0E 70 10 65 12 34 14 30 30 38 37 36 35 34  .rpe40087654  
01A6CB30  33 32 31 31 32 33 34 35 36 37 38 38 37 36 35 49  321123456788765I  
01A6CB40  6D 54 4F 4F 64 76 64 61 75 64 69 6F 72 69 70 70  mTOOdvdaudioripp  
01A6CB50  65 72 34                                         er4  
这次连接把对内置字符串的 2 次取值结果与第一次连接值连接起来，得到了值作为 MD5 的待加密值。  
  
MD5 加密：       
1001B4B3      50               push eax                              ; 待加密值？  
1001B4B4      8D4C24 70        lea ecx,dword ptr ss:\[esp+70\]  
1001B4B8      E8 03740000      call UILib71.100228C0                 ; MD5?  
  
因为待加密值太大，分批加密：  
10023327      E8 D4FEFFFF      call UILib71.10023200                 ; MD5  
10023334      E8 C7FEFFFF      call UILib71.10023200                 ; MD5  
常数：  
1002295D      8B70 04        mov esi,dword ptr ds:\[eax+4\]      ; EFCDAB89  
10022960      8B78 08        mov edi,dword ptr ds:\[eax+8\]      ; 98BADCFE  
10022963      8B50 0C        mov edx,dword ptr ds:\[eax+C\]      ; 10325476  
10022966      8B00           mov eax,dword ptr ds:\[eax\]        ; 67452301  
  
待加密值：  
第一批：  
01A6CB00  31 49 01 54 03 4F 05 76 07 61 09 64 0B 6F 0D 69  1ITOva.d o.i  
01A6CB10  0F 70 11 72 13 6D 02 4F 04 64 06 64 08 75 0A 69  prmOddu.i  
01A6CB20  0C 72 0E 70 10 65 12 34 14 30 30 38 37 36 35 34  .rpe40087654  
01A6CB30  33 32 31 31 32 33 34 35 36 37 38 38 37 36 35 49  321123456788765I  
第二批：  
01A6CB40  6D 54 4F 4F 64 76 64 61 75 64 69 6F 72 69 70 70  mTOOdvdaudioripp  
01A6CB50  65 72 34                                         er4  
  
加密值：  
1001B4C9      E8 32730000        call UILib71.10022800         ; 转换为字符  
eax=001240CD, (ASCII "4ad70c3c17240215f69ec551e6f4c020")  
  
对加密值的取值：  
1001B4F0      56                 push esi  
1001B4F1      8D4C24 34          lea ecx,dword ptr ss:\[esp+34\]  
1001B4F5      FF15 B8770210      call dword ptr ds:\[<&MFC71.#865>\]           ; MFC71.7C1894E7  
1001B4FB      8D4C24 0C          lea ecx,dword ptr ss:\[esp+C\]  
1001B4FF      50                 push eax  
1001B500      FF15 6C750210      call dword ptr ds:\[<&MFC71.#908>\]           ; MFC71.7C18B24E  
1001B506      8BC6               mov eax,esi  
1001B508      D1E8               shr eax,1  
1001B50A      40                 inc eax  
1001B50B      25 03000080        and eax,80000003  
1001B510      79 05              jns short UILib71.1001B517  
1001B512      48                 dec eax  
1001B513      83C8 FC            or eax,FFFFFFFC  
1001B516      40                 inc eax  
1001B517      75 0F              jnz short UILib71.1001B528  
1001B519      68 40E10210        push UILib71.1002E140  
1001B51E      8D4C24 10          lea ecx,dword ptr ss:\[esp+10\]  
1001B522      FF15 C4730210      call dword ptr ds:\[<&MFC71.#911>\]           ; MFC71.7C14E587  
1001B528      83C6 02            add esi,2  
1001B52B      83FE 20            cmp esi,20  
1001B52E    ^ 7C C0              jl short UILib71.1001B4F0  
  
取单数，并 4 位 4 位用“-”号连接：  
00124058   01A6C9C0    ASCII "4d03-1201-f9c5-efc2-"  
  
转成大写：  
1001B530      8D4C24 0C          lea ecx,dword ptr ss:\[esp+C\]                ; 值  
1001B534      FF15 80750210      call dword ptr ds:\[<&MFC71.#4085>\]          ; 转成大写  
00124058   01A6C9C0    ASCII "4D03-1201-F9C5-EFC2-"  
  
去除最后一个“-”号，变成 19 位：  
1001B546      48                 dec eax                                     ; 19 位  
1001B547      50                 push eax  
1001B548      8D4C24 14          lea ecx,dword ptr ss:\[esp+14\]  
1001B54C      FF15 7C750210      call dword ptr ds:\[<&MFC71.#1916>\]          ; MFC71.7C189568  
  
00124058   01A6C9C0    ASCII "4D03-1201-F9C5-EFC2"  
  
连接注册码的前 20 位：  
1001B55C      50                 push eax  
1001B55D      6A 00              push 0  
1001B55F      8D4C24 14          lea ecx,dword ptr ss:\[esp+14\]  
1001B563      FF15 84750210      call dword ptr ds:\[<&MFC71.#3850>\]          ; MFC71.7C189CDC  
00124058   01A6C9C0    ASCII "876543211234567887654D03-1201-F9C5-EFC2"  
  
然后比较：  
1001B573      50                 push eax                                    ; 原输入的注册码  
1001B574      8D4C24 10          lea ecx,dword ptr ss:\[esp+10\]               ; 计算后的连接值  
1001B578      FF15 4C740210      call dword ptr ds:\[<&MFC71.#1482>\]          ; MFC71.7C144DAE  
1001B57E      F7D8               neg eax  
1001B580      1AC0               sbb al,al  
1001B582      FEC0               inc al  
1001B584      8D4C24 30          lea ecx,dword ptr ss:\[esp+30\]  
1001B588      0FB6F0             movzx esi,al  
原输入的注册码：00124048   01A6C888    ASCII "876543211234567887654321123456787654321"  
计算后的连接值：00124058   01A6C9C0    ASCII "876543211234567887654D03-1201-F9C5-EFC2"  
比较值返回到 eax，最后作为判断指针返回结果。  
  
然后返回到：  
1001B740      E8 BBF8FFFF       call UILib71.ImRegDlg::IsValidRegInfo  
1001B745      85C0              test eax,eax                             ; 这个就是返回值  
1001B747      75 49             jnz short UILib71.1001B792  
  
4.算法总结：  
程序用到的字符串： ASCII "ImTOOdvdaudioripper4"  
输入的注册码（39 位）：ASCII "876543211234567887654321123456787654321"  
  
取注册码前 20 位，连接字符串得到参数之一：ASCII "87654321123456788765ImTOOdvdaudioripper4"  
取字符串单数并序号，再取字符串双数并序号，连接，前面补一个字节 31，后面补 2 个字节 30 30，得到参数之一：  
01A6CB00  31 49 01 54 03 4F 05 76 07 61 09 64 0B 6F 0D 69  1ITOva.d o.i  
01A6CB10  0F 70 11 72 13 6D 02 4F 04 64 06 64 08 75 0A 69  prmOddu.i  
01A6CB20  0C 72 0E 70 10 65 12 34 14 30 30                 .rpe400  
  
两个参数连接起来作为待加密值：  
01A6CB00  31 49 01 54 03 4F 05 76 07 61 09 64 0B 6F 0D 69  1ITOva.d o.i  
01A6CB10  0F 70 11 72 13 6D 02 4F 04 64 06 64 08 75 0A 69  prmOddu.i  
01A6CB20  0C 72 0E 70 10 65 12 34 14 30 30 38 37 36 35 34  .rpe40087654  
01A6CB30  33 32 31 31 32 33 34 35 36 37 38 38 37 36 35 49  321123456788765I  
01A6CB40  6D 54 4F 4F 64 76 64 61 75 64 69 6F 72 69 70 70  mTOOdvdaudioripp  
01A6CB50  65 72 34                                         er4  
  
MD5 加密得到：(ASCII "4ad70c3c17240215f69ec551e6f4c020")，转换成大写，并取单数，4 位 4 位用“-”号连接，去除最后一个“-”号得到： ASCII "4D03-1201-F9C5-EFC2"  
然后与输入的注册码的后 19 位比较，相等就 ok。  
  
用户名的用途：用来加解密注册码，保存到注册表。  
  
注册信息：  
用户名：cyto  
注册码：876543211234567887654D03-1201-F9C5-EFC2 

 4dd466d0e1e0cc123afbe504da2462ca