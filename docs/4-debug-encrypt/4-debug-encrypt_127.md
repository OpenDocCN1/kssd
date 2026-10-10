

# RM to AVI MPEG WMV VCD SVCD DVD Converter 2.7

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-163.htm`](https://www.pediy.com/kssd/pediy08/pediy8-163.htm)

RM to AVI MPEG WMV VCD SVCD DVD Converter 2.7  
  
说明：程序使用 RSA 加密的，N 有 82 位十六进制，太难分解了，于是想到捏造一个 N 替换它，然后逆推注册码就完美的爆破它了。  
终于在除夕夜完成爆破，以此文辞旧迎新！祝大家春节快乐，身体健康，合家平安，万事如意！  
  
1.重启验证，注册后生成一个文件：rmtoall.ini  
\[Options\]  
User=cyto  
Pass=87654321012345678  
  
2.查壳脱壳：  
PEID:ASPack 2.12 -> Alexey Solodovnikov  
OD 载入：  
00595001 r>  60             pushad                          ; 入口点  
00595002     E8 03000000    call rmtoall.0059500A  
...  
005953AF     61             popad  
005953B0     75 08          jnz short rmtoall.005953BA  
005953B2     B8 01000000    mov eax,1  
005953B7     C2 0C00        retn 0C  
005953BA     68 3C4E5000    push rmtoall.00504E3C  
005953BF     C3             retn  
dump，Borland Delphi 6.0 - 7.0  
运行 ok。  
  
3.注册码存在 ini 文件里，下断 kernel.GetPrivateProfileStringA  
断下返回，上下翻翻，发现是读 ini 文件的内容，比如 user,pass：  
0050481F   mov ecx,up－rmto.005048E4                        ;  ASCII "User"  
00504851   mov ecx,up－rmto.00504904                        ;  ASCII "Pass"  
然后返回到：  
00504FC3   |.  E8 E8F7FFFF     call up－rmto.005047B0               ;  取得 ini 内容  
00504FC8   |.  A1 58815000     mov eax,dword ptr ds:\[508158\]  
00504FCD   |.  8338 00         cmp dword ptr ds:\[eax\],0            ;  用户名是否为空？  
00504FD0   |.  0F84 EE000000   je up－rmto.005050C4  
00504FD6   |.  A1 18825000     mov eax,dword ptr ds:\[508218\]  
00504FDB   |.  8338 00         cmp dword ptr ds:\[eax\],0            ;  注册码是否为空？  
00504FDE   |.  0F84 E0000000   je up－rmto.005050C4  
00504FE4   |.  B8 60A05000     mov eax,up－rmto.0050A060  
00504FE9   |.  8B15 18825000   mov edx,dword ptr ds:\[508218\]       ;  up－rmto.00509FA8  
00504FEF   |.  8B12            mov edx,dword ptr ds:\[edx\]  
00504FF1   |.  E8 B2F4EFFF     call up－rmto.004044A8  
00504FF6   |.  BA 44A05000     mov edx,up－rmto.0050A044  
00504FFB   |.  A1 60A05000     mov eax,dword ptr ds:\[50A060\]  
00505000   |.  E8 23CBFFFF     call up－rmto.00501B28               ;  输入的注册码处理  
00505005   |.  B8 60A05000     mov eax,up－rmto.0050A060  
0050500A   |.  E8 45F4EFFF     call up－rmto.00404454  
0050500F   |.  8D55 E8         lea edx,dword ptr ss:\[ebp-18\]  
00505012   |.  A1 AC815000     mov eax,dword ptr ds:\[5081AC\]  
00505017   |.  8B00            mov eax,dword ptr ds:\[eax\]  
00505019   |.  8B80 50040000   mov eax,dword ptr ds:\[eax+450\]  
0050501F   |.  E8 8CF5F5FF     call up－rmto.004645B0              ; 取字符串 A＝RSA（E）  
00505024   |.  8B45 E8         mov eax,dword ptr ss:\[ebp-18\]       ; E=65537=10001(H)  
00505027   |.  8D55 EC         lea edx,dword ptr ss:\[ebp-14\]  
0050502A   |.  E8 B53AF0FF     call up－rmto.00408AE4  
0050502F   |.  8B45 EC         mov eax,dword ptr ss:\[ebp-14\]  
00505032   |.  BA 48A05000     mov edx,up－rmto.0050A048  
00505037   |.  E8 E0CEFFFF     call up－rmto.00501F1C              ;  对 E 的处理  
0050503C   |.  8D55 E0         lea edx,dword ptr ss:\[ebp-20\]  
0050503F   |.  A1 AC815000     mov eax,dword ptr ds:\[5081AC\]  
00505044   |.  8B00            mov eax,dword ptr ds:\[eax\]  
00505046   |.  8B80 54040000   mov eax,dword ptr ds:\[eax+454\]  
0050504C   |.  E8 5FF5F5FF     call up－rmto.004645B0              ;取字符串 B＝RSA（N）  
00505051   |.  8B45 E0         mov eax,dword ptr ss:\[ebp-20\]       ; N  
00505054   |.  8D55 E4         lea edx,dword ptr ss:\[ebp-1C\]  
00505057   |.  E8 883AF0FF     call up－rmto.00408AE4  
0050505C   |.  8B45 E4         mov eax,dword ptr ss:\[ebp-1C\]  
0050505F   |.  BA 50A05000     mov edx,up－rmto.0050A050  
00505064   |.  E8 B3CEFFFF     call up－rmto.00501F1C              ;  对 N 的处理  
00505069   |.  68 58A05000     push up－rmto.0050A058  
0050506E   |.  68 58A05000     push up－rmto.0050A058  
00505073   |.  68 58A05000     push up－rmto.0050A058  
00505078   |.  68 58A05000     push up－rmto.0050A058  
0050507D   |.  68 60A05000     push up－rmto.0050A060  
00505082   |.  B9 50A05000     mov ecx,up－rmto.0050A050  
00505087   |.  BA 48A05000     mov edx,up－rmto.0050A048  
0050508C   |.  A1 44A05000     mov eax,dword ptr ds:\[50A044\]       ;  转换后的十六进制  
00505091   |.  E8 EAF2FFFF     call up－rmto.00504380              ;  RSA 加密  
00505096   |.  A1 18825000     mov eax,dword ptr ds:\[508218\]  
0050509B   |.  8B15 60A05000   mov edx,dword ptr ds:\[50A060\]  
005050A1   |.  E8 02F4EFFF     call up－rmto.004044A8  
005050A6   |.  B8 48A05000     mov eax,up－rmto.0050A048  
005050AB   |.  E8 E4D0FFFF     call up－rmto.00502194  
005050B0   |.  B8 50A05000     mov eax,up－rmto.0050A050  
005050B5   |.  E8 DAD0FFFF     call up－rmto.00502194  
005050BA   |.  B8 58A05000     mov eax,up－rmto.0050A058  
005050BF   |.  E8 D0D0FFFF     call up－rmto.00502194  
005050C4   |>  A1 58815000     mov eax,dword ptr ds:\[508158\]  
005050C9   |.  8B00            mov eax,dword ptr ds:\[eax\]  
005050CB   |.  8B15 18825000   mov edx,dword ptr ds:\[508218\]       ;  up－rmto.00509FA8  
005050D1   |.  8B12            mov edx,dword ptr ds:\[edx\]  
005050D3   |.  E8 88F7EFFF     call up－rmto.00404860              ;  比较  
005050D8   |.  75 2D           jnz short up－rmto.00505107         ;  关键跳转  
  
以上就是注册码计算判断的主线。经过计算后的注册码最后与用户名比较，相等就成功了。  
  
4.从关键比较入手，分析比较值与存放地址：  
005050C4   |>  A1 58815000     mov eax,dword ptr ds:\[508158\]  
005050C9   |.  8B00            mov eax,dword ptr ds:\[eax\]  
005050CB   |.  8B15 18825000   mov edx,dword ptr ds:\[508218\]       ;  up－rmto.00509FA8  
005050D1   |.  8B12            mov edx,dword ptr ds:\[edx\]  
005050D3   |.  E8 88F7EFFF     call up－rmto.00404860              ;  比较  
005050D8   |.  75 2D           jnz short up－rmto.00505107         ;  关键跳转  
  
比较的长度：  
00404877   |.  8B46 FC    mov eax,dword ptr ds:\[esi-4\]    ;  用户名长度＝4  
0040487A   |.  8B57 FC    mov edx,dword ptr ds:\[edi-4\]    ;  计算值长度＝26  
  
比较的东西：  
00508158  A4 9F 50 00              P.0  
00509FA4  8C 3F EF 00              ??(帮  
00EF3F8C  63 79 74 6F                cyto  
  
00508218  A8 9F 50 00              P.皉  
00509FA8  28 B0 EF 00              帮..  
00EFB028  03 90 19 BD 6A D3 87 3F E4 B3 D9 C9 50 79 9F B8  ?絡訃?涑偕 Py 煾  
00EFB038  11 77 77 C9 E6 C7 8F 56 4B FB 7C 35 2D 62 D5 86  ww 涉菑 VK 鹼 5-b 諉  
00EFB048  6C 8A 10 D9 16 57                                  l??W..埉?埉?  
  
通过修改 ini 的注册码那如发现：只要注册码不变上面比较的地址都是固定的。如果注册码有变的话，存放地址也会跟着变，值也会变，比较的长度也变。  
  
5.定位加密算法  
从上面的比较结果可以看出，太像 RSA 运算了，于是上 cryptosearcher 0.97 分析，发现 FGIntRSA，查找 google：  
RSA 加密一共需要三个单元 FGInt, FGIntPrimeGeneration, FGIntRSA：  
FGint 是一个大型数运算库；GIntPrimeGeneration 是一个找大质数的类库；FGIntRSA 是实现加密和解密以及验证的单元。  
而且主线对输入的注册码处理后紧接着出现两个字符串：  
65537＝10001（H）  
63790510521550840388844862178357891512889443215420175471639593023445165917945890380913786330112957＝1DDD59DD5BE4EB02D560C324464193E0369A5ADAA8776B560E6ECA51F550EFD32B3FAE992B478963BD（H）  
估计第一个就是 RSA 的 E，第二个应该就是传说中的 N。  
  
6.算法过程分析：  
对注册码下硬件访问断点，分析程序对注册码的处理：  
从 ini 中取得注册码，存放地址：  
00EFB1CC  38 37 36 35 34 33 32 31 30 31 32 33 34 35 36 37  38  87654321012345678  
  
6.1 00505000  call up－rmto.00501B28：对输入的注册码进行处理        
首先根据注册码的 ASCII 码取出表中的值：  
00501B99   |> /8D45 F8          /lea eax,dword ptr ss:\[ebp-8\]  
00501B9C   |. |8B55 FC          |mov edx,dword ptr ss:\[ebp-4\]  
00501B9F   |. |0FB65432 FF      |movzx edx,byte ptr ds:\[edx+esi-1\]     ;edx＝注册码 ASCII 码  
00501BA4   |. |8B9495 F4FBFFFF  |mov edx,dword ptr ss:\[ebp+edx\*4-40C\]  ;取值，ebp=0012FF4C  
00501BAB   |. |E8 6C2BF0FF      |call up－rmto.0040471C  
00501BB0   |. |46               |inc esi  
00501BB1   |. |4B               |dec ebx  
00501BB2   |.^\\75 E5            \\jnz short up－rmto.00501B99  
在 00501BA4 处先根据顺取到的注册码 ASCII 码计算得到存放表单的地址。  
  
对应关系：  
0＝110100；1＝110101；2＝110110；3＝110111；4＝111000；5＝111001；  
6＝111010；7＝111011；8＝111100；9＝111101。  
  
a＝000000 ；b＝000010 ；c＝000100 ；d＝000110 ；e＝001000 ；f＝001010 ；  
g＝001100 ；h＝001110 ；i＝010000 ；j＝010010 ；k＝010100 ；l＝010110 ；  
m＝011000 ；n＝011010 ；o＝011100 ；p＝011110 ；q＝100000 ；r＝100010 ；  
s＝100100 ；t＝100110 ；u＝101000 ；v＝101010 ；w＝101100 ；x＝101110 ；  
y＝110000 ；z＝110010 。  
  
A＝000001 ；B＝000011 ；C＝000101 ；D＝000111 ；E＝001001 ；F＝001011 ；  
G＝001101 ；H＝001111 ；I＝010001 ；J＝010011 ；K＝010101 ；L＝010111 ；  
M＝011001 ；N＝011011 ；O＝011101 ；P＝011111 ；Q＝100001 ；R＝100011 ；  
S＝100101 ；T＝100111 ；U＝101001 ；V＝101011 ；W＝101101 ；X＝101111 ；  
Y＝110001 ；Z＝110011  
  
注册码：87654321012345678；38 37 36 35 34 33 32 31 30 31 32 33 34 35 36 37 38  
堆栈 ss:\[0012FF44\]=00EFAED4, (ASCII "111100 111011 111010 111001 111000 110111 110110 110101 110100 110101 110110 110111 111000 111001 111010 111011 111100")  
  
然后转换成十六进制：  
顺取 8 位转换成十六进制，最后剩余的 6 位不够就丢弃了：  
11110011 10111110 10111001 11100011 01111101 10110101 11010011 01011101 10110111 11100011 10011110 10111011 111100＝F3 BE B9 E3 7D B5 D3 5D B7 E3 9E BB  
即：111100111011111010111001111000110111110110110101110100110101110110110111111000111001111010111011＝F3BEB9E37DB5D35DB7E39EBB  
在内存搜索到： 00EFAEB8  F3 BE B9 E3 7D B5 D3 5D B7 E3 9E BB           
  
6.2 取得 RSA 的 E＆N：  
0050501F   |.  E8 8CF5F5FF     call up－rmto.004645B0              ; 取字符串 A＝RSA（E）  
00505024   |.  8B45 E8         mov eax,dword ptr ss:\[ebp-18\]       ; E=65537=10001(H)  
00505027   |.  8D55 EC         lea edx,dword ptr ss:\[ebp-14\]  
0050502A   |.  E8 B53AF0FF     call up－rmto.00408AE4  
0050502F   |.  8B45 EC         mov eax,dword ptr ss:\[ebp-14\]  
00505032   |.  BA 48A05000     mov edx,up－rmto.0050A048  
00505037   |.  E8 E0CEFFFF     call up－rmto.00501F1C              ;  对 E 的处理  
0050503C   |.  8D55 E0         lea edx,dword ptr ss:\[ebp-20\]  
0050503F   |.  A1 AC815000     mov eax,dword ptr ds:\[5081AC\]  
00505044   |.  8B00            mov eax,dword ptr ds:\[eax\]  
00505046   |.  8B80 54040000   mov eax,dword ptr ds:\[eax+454\]  
0050504C   |.  E8 5FF5F5FF     call up－rmto.004645B0              ;取字符串 B＝RSA（N）  
00505051   |.  8B45 E0         mov eax,dword ptr ss:\[ebp-20\]       ; N  
00505054   |.  8D55 E4         lea edx,dword ptr ss:\[ebp-1C\]  
00505057   |.  E8 883AF0FF     call up－rmto.00408AE4  
0050505C   |.  8B45 E4         mov eax,dword ptr ss:\[ebp-1C\]  
0050505F   |.  BA 50A05000     mov edx,up－rmto.0050A050  
00505064   |.  E8 B3CEFFFF     call up－rmto.00501F1C              ;  对 N 的处理  
这两个计算对应两个字符串：  
E=65537(D)=10001(H)  
N=63790510521550840388844862178357891512889443215420175471639593023445165917945890380913786330112957(D)=1DDD59DD5BE4EB02D560C324464193E0369A5ADAA8776B560E6ECA51F550EFD32B3FAE992B478963BD(H)  
  
6.3 加密并处理得到最后结果：00505091    call up－rmto.00504380    
加密：  
M：处理后的注册码=F3BEB9E37DB5D35DB7E39EBB（H）  
111100111011111010111001111000110111110110110101110100110101110110110111111000111001111010111011  
  
E：10001（H）  
N：325 个字符  
63790510521550840388844862178357891512889443215420175471639593023445165917945890380913786330112957（D）  
1DDD59DD5BE4EB02D560C324464193E0369A5ADAA8776B560E6ECA51F550EFD32B3FAE992B478963BD（H）  
1110111011101010110011101110101011011111001001110101100000010110101010110000011000011001001000100011001000001100100111110000000110110100110100101101011011010101010000111011101101011010101100000111001101110110010100101000111110101010100001110111111010011001010110011111110101110100110010010101101000111100010010110001110111101（2）  
  
C=ME(mod N)=F3BEB9E37DB5D35DB7E39EBB¹⁰⁰⁰¹(mod 1DDD59DD5BE4EB02D560C324464193E0369A5ADAA8776B560E6ECA51F550EFD32B3FAE992B478963BD)=ACA353B9019BD6AD3873FE4B3D9C950799FB8117777C9E6C78F564BFB7C352D62D5866C8A10D91657  
  
加密后得到密文：ACA353B9019BD6AD3873FE4B3D9C950799FB8117777C9E6C78F564BFB7C352D62D5866C8A10D91657（H）  
  
密文转换成 2 进制：00504654   call up－rmto.00502DCC  
00502E00   |.  8945 F8       mov dword ptr ss:\[ebp-8\],eax  
00502E03   |.  BF 01000000   mov edi,1  
00502E08   |>  33DB          /xor ebx,ebx  
00502E0A   |>  8B45 FC       |/mov eax,dword ptr ss:\[ebp-4\]  
00502E0D   |.  8B40 04       ||mov eax,dword ptr ds:\[eax+4\]  
00502E10   |.  8B54F8 04     ||mov edx,dword ptr ds:\[eax+edi\*8+4\]  
00502E14   |.  8B04F8        ||mov eax,dword ptr ds:\[eax+edi\*8\]  
00502E17   |.  8BCB          ||mov ecx,ebx  
00502E19   |.  E8 D626F0FF   ||call up－rmto.004054F4  
00502E1E   |.  81E0 01000000 ||and eax,1  
00502E24   |.  33D2          ||xor edx,edx  
00502E26   |.  52            ||push edx                              ; /Arg2 => 00000000  
00502E27   |.  50            ||push eax                              ; |Arg1  
00502E28   |.  8D45 F4       ||lea eax,dword ptr ss:\[ebp-C\]          ; |  
00502E2B   |.  E8 A05EF0FF   ||call up－rmto.00408CD0                 ; \\up－rmto.00408CD0  
00502E30   |.  8B55 F4       ||mov edx,dword ptr ss:\[ebp-C\]  
00502E33   |.  8B0E          ||mov ecx,dword ptr ds:\[esi\]            ;  存于 n 个地址，最后存于 00EF4330&00EFAED4  
00502E35   |.  8BC6          ||mov eax,esi  
00502E37   |.  E8 2419F0FF   ||call up－rmto.00404760  
00502E3C   |.  43            ||inc ebx  
00502E3D   |.  83FB 1F       ||cmp ebx,1F  
00502E40   |.^ 75 C8         |\\jnz short up－rmto.00502E0A  
00502E42   |.  47            |inc edi  
00502E43   |.  FF4D F8       |dec dword ptr ss:\[ebp-8\]  
00502E46   |.^ 75 C0         \\jnz short up－rmto.00502E08  
101011001010001101010011101110010000000110011011110101101010110100111000011100111111111001001011001111011001110010010101000001111001100111111011100000010001011101110111011111001001111001101100011110001111010101100100101111111011011111000011010100101101011000101101010110000110011011001000101000010000110110010001011001010111  
  
2 进制里寻找第一个 111，去除前面的数  
005046AF   |. /EB 12         jmp short up－rmto.005046C3  
005046B1   |> |8D45 B0       /lea eax,dword ptr ss:\[ebp-50\]  
005046B4   |. |B9 01000000   |mov ecx,1  
005046B9   |. |BA 01000000   |mov edx,1  
005046BE   |. |E8 F102F0FF   |call up－rmto.004049B4                ;  下一位开始  
005046C3   |> \\8D45 A0        lea eax,dword ptr ss:\[ebp-60\]  
005046C6   |.  50            |push eax  
005046C7   |.  B9 03000000   |mov ecx,3  
005046CC   |.  BA 01000000   |mov edx,1  
005046D1   |.  8B45 B0       |mov eax,dword ptr ss:\[ebp-50\]        ;  初始值为 00504654 计算的 2 进制，存在 00EFAED4  
005046D4   |.  E8 9B02F0FF   |call up－rmto.00404974                ;  取出 3 位  
005046D9   |.  8B45 A0       |mov eax,dword ptr ss:\[ebp-60\]  
005046DC   |.  BA 74475000   |mov edx,up－rmto.00504774             ;  ASCII "111"  
005046E1   |.  E8 7A01F0FF   |call up－rmto.00404860                ;  比较  
005046E6   |.  74 0D         |je short up－rmto.005046F5            ;  相等就跳走  
005046E8   |.  8B45 B0       |mov eax,dword ptr ss:\[ebp-50\]  
005046EB   |.  E8 2400F0FF   |call up－rmto.00404714  
005046F0   |.  83F8 03       |cmp eax,3  
005046F3   |.^ 7F BC         \\jg short up－rmto.005046B1  
数据放置的初始位置为 00EFAED4,然后转移到 00EF4330，找第一个 111，找到后就跳走，去除其前面的字符。  
11101110010000000110011011110101101010110100111000011100111111111001001011001111011001110010010101000001111001100111111011100000010001011101110111011111001001111001101100011110001111010101100100101111111011011111000011010100101101011000101101010110000110011011001000101000010000110110010001011001010111  
  
又去掉值的前面 3 位：  
005046F5   |> \\8D45 B0       lea eax,dword ptr ss:\[ebp-50\]  
005046F8   |.  B9 03000000   mov ecx,3  
005046FD   |.  BA 01000000   mov edx,1  
00504702   |.  E8 AD02F0FF   call up－rmto.004049B4                 ;  又去掉 3 位？  
01110010000000110011011110101101010110100111000011100111111111001001011001111011001110010010101000001111001100111111011100000010001011101110111011111001001111001101100011110001111010101100100101111111011011111000011010100101101011000101101010110000110011011001000101000010000110110010001011001010111  
  
转换成十六进制：得到最终的结果  
00EFB028  03 90 19 BD 6A D3 87 3F E4 B3 D9 C9 50 79 9F B8  ?絡訃?涑偕 Py 煾  
00EFB038  11 77 77 C9 E6 C7 8F 56 4B FB 7C 35 2D 62 D5 86  ww 涉菑 VK 鹼 5-b 諉  
00EFB048  6C 8A 10 D9 16 57                                l??W..  
即：39019BD6AD3873FE4B3D9C950799FB8117777C9E6C78F564BFB7C352D62D5866C8A10D91657  
  
6.4 算法流程：  
处理输入的注册码，查表得 2 进制再转换成十六进制，加密，然后加密后的数据转换成 2 进制，去除第一个 111 前面的数，然后再去除前面 3 位，然后再转换成十六进制与用户名比较，相等就注册成功。  
  
7.爆破  
RSA 加密，N 太长，无法运算，只好爆破。  
爆破之一：启动程序后判断注册码  
005050C4   |> \\A1 58815000     mov eax,dword ptr ds:\[508158\]  
005050C9   |.  8B00            mov eax,dword ptr ds:\[eax\]         ;  用户名  
005050CB   |.  8B15 18825000   mov edx,dword ptr ds:\[508218\]      ;  up－rmto.00509FA8  
005050D1   |.  8B12            mov edx,dword ptr ds:\[edx\]         ;  注册码  
005050D3   |.  E8 88F7EFFF     call up－rmto.00404860              ;  bj  
005050D8       75 2D           jnz short up－rmto.00505107  
  
005050C4       A1 58815000     mov eax,dword ptr ds:\[508158\]  
改为：  
005050C4       A1 18825000     mov eax,dword ptr ds:\[508218\]  
  
修改后输入注册码用户名的地方变灰色的。  
  
爆破之二：退出程序判断  
可是退出的时候还是提示：This is a unregistered version。  
地址=004FCD98  反汇编=mov edx,up－rmto.004FCDEC  
 文本字符串=This is a unregistered version, Don't forget to register it.\\nTo register software,please click 'OK' button.  
  
004FCD6F   |.  A1 A49F5000        mov eax,dword ptr ds:\[509FA4\]  
004FCD74   |.  8B15 A89F5000      mov edx,dword ptr ds:\[509FA8\]  
004FCD7A   |.  E8 E17AF0FF        call up－rmto.00404860  
004FCD7F       75 12              jnz short up－rmto.004FCD93  
004FCD81   |.  833D A49F5000 00   cmp dword ptr ds:\[509FA4\],0  
004FCD88   |.  74 09              je short up－rmto.004FCD93  
004FCD8A   |.  833D A89F5000 00   cmp dword ptr ds:\[509FA8\],0  
004FCD91   |.  75 37              jnz short up－rmto.004FCDCA  
004FCD93   |>  6A 00              push 0  
004FCD95   |.  8D45 FC            lea eax,dword ptr ss:\[ebp-4\]  
004FCD98   |.  BA ECCD4F00        mov edx,up－rmto.004FCDEC       ;  ASCII "This is a unregistered version, Don't forget to register it.  
To register software,please click 'OK' button."  
  
004FCD6F   |.  A1 A49F5000        mov eax,dword ptr ds:\[509FA4\]  
改为：  
004FCD6F       A1 A89F5000        mov eax,dword ptr ds:\[509FA8\]  
  
意外发现：爆破之二的比较入栈 2 个参数又跟爆破之一的不一样，可能退出的时候算法又不一样？还是只是挑取其中的一部分判断？  
  
爆破之三：  
通过了上面两个爆破后，点击 convert 后跳出注册框。  
其中 About 里面写着：Licence to: Unregister  
好，先搞定这个：  
地址=004FB45D  反汇编=mov edx,baopo.004FB53C  文本字符串=Licence to: Unregister  
004FB433   |.  A1 58815000        mov eax,dword ptr ds:\[508158\]  
004FB438   |.  8B00               mov eax,dword ptr ds:\[eax\]  
004FB43A   |.  8B15 18825000      mov edx,dword ptr ds:\[508218\]          ;  baopo.00509FA8  
004FB440   |.  8B12               mov edx,dword ptr ds:\[edx\]  
004FB442   |.  E8 1994F0FF        call baopo.00404860  
004FB447   |.  75 14              jnz short baopo.004FB45D  
004FB449   |.  A1 58815000        mov eax,dword ptr ds:\[508158\]  
004FB44E   |.  8338 00            cmp dword ptr ds:\[eax\],0  
004FB451   |.  74 0A              je short baopo.004FB45D  
004FB453   |.  A1 18825000        mov eax,dword ptr ds:\[508218\]  
004FB458   |.  8338 00            cmp dword ptr ds:\[eax\],0  
004FB45B   |.  75 12              jnz short baopo.004FB46F  
004FB45D   |>  BA 3CB54F00        mov edx,baopo.004FB53C                 ;  ASCII "Licence to: Unregister"  
  
004FB433   |.  A1 58815000        mov eax,dword ptr ds:\[508158\]  
改为：  
004FB433       A1 18825000        mov eax,dword ptr ds:\[508218\]  
  
点击 convert 还是跳出要注册的框，郁闷！继续爆！  
  
可疑的串参考：  
地址=004CCA03  反汇编=mov eax,baopo.004CCA34  文本字符串=This version of AlphaControls is trial. For purchase of the fully functional version please come to the http://www.alphaskins.com. Thanks!  
地址=00505251  反汇编=mov eax,baopo.00505520  文本字符串=This software can only try 7 day.\\nPlease registration!  
都下断，不管从启动到退出，时间调整 1 年都没能断下，看来无关。  
  
看来来硬的不行了，可能暗桩的缘故。  
  
8.修改程序：  
因为爆破的话不晓得会不会有暗桩，而且程序本身的 N（82 位十六进制）实在太大了，很难分解因子，所以想到将 N 替换掉，利用 RSAtool 随机产生长度一样的 N：  
P=4D52ECB185B842EC6F55395F3F6B9CE826A7F3043  
Q=55E9F05EF5518B1FB4FAEDF2A4B48CF35733455AB  
N=19F331AACB6B3741A5ECA989567BFB8BB8573554BD31E8247BFD2196A907C9746820362DDC44977BC1（H）＝55428813623153400097073003728047268187658784397941308834162197241730534619050749533994595198139329（D）  
D=153C6FE8D58F6828D1205B9D88A8EDB1E9A9747B5BFF968BB520E979F2DE72FB77F7DF3A968F9CFE1  
  
Winhex 打开脱壳后的程序，然后搜索：  
63790510521550840388844862178357891512889443215420175471639593023445165917945890380913786330112957  
替换成产生的 N＝55428813623153400097073003728047268187658784397941308834162197241730534619050749533994595198139329（D）  
好了，这下不用分解大数因子就可以轻松搞定加解密。  
  
9.然后逆推注册码：  
用户名＝otyc＝6F747963＝1101111011101000111100101100011  
加上前面 3 位数 111＝1111101111011101000111100101100011＝3EF747963  
解密 3EF747963＝16D71276E2E7617551A0251C940897907C38097CED12E6472BBF5568A5308C50AA533165922F269223  
其 2 进制为：（328 位，注意前面的 0 不能省略，关系到下面转换的问题）  
0001011011010111000100100111011011100010111001110110000101110101010100011010000000100101000111001001010000001000100101111001000001111100001110000000100101111100111011010001001011100110010001110010101110111111010101010110100010100101001100001000110001010000101010100101001100110001011001011001001000101111001001101001001000100011  
328 位不能为 6 整除，只好在后面（因为注册码处理是从前面开始处理的，每次取 8 位，最后面不够位数的话就放弃）加 2 个 0，使其能被 6 整除：  
000101101101011100010010011101101110001011100111011000010111010101010001101000000010010100011100100101000000100010010111100100000111110000111000000010010111110011101101000100101110011001000111001010111011111101010101011010001010010100110000100011000101000010101010010100110011000101100101100100100010111100100110100100100010001100  
  
分成 6 位 6 位的：  
000101 101101 011100 010010 011101 101110 001011 100111 011000 010111 010101 010001 101000 000010 010100 011100 100101 000000 100010 010111 100100 000111 110000 111000 000010 010111 110011 101101 000100 101110 011001 000111 001010 111011 111101 010101 011010 001010 010100 110000 100011 000101 000010 101010 010100 110011 000101 100101 100100 100010 111100 100110 100100 100010 001100  
  
逆查表得：  
000101 101101 011100 010010 011101 101110 001011 100111 011000 010111 010101 010001 101000 000010   
  C      W      o      j      O      x      F      T       m     L      K       I     u      b                      
010100 011100 100101 000000 100010 010111 100100 000111 110000 111000 000010 010111 110011 101101   
  k      o      S      a      r      L      s      D      y      4       b      L      Z     W     
000100 101110 011001 000111 001010 111011 111101 010101 011010 001010 010100 110000 100011 000101   
  c      x      M      D      f       7     9       K      n      f      k      y     R       C     
000010 101010 010100 110011 000101 100101 100100 100010 111100 100110 100100 100010 001100  
 b       v       k      Z     C      S      s       r      8      t      s     r       g      
  
整理后：  
CWojOxFTmLKIubkoSarLsDy4bLZWcxMDf79KnfkyRCbvkZCSsr8tsrg  
  
用户名：otyc  
注册码：CWojOxFTmLKIubkoSarLsDy4bLZWcxMDf79KnfkyRCbvkZCSsr8tsrg  
这下输入用户名注册码的地方变成灰色的，退出的时候不再提示未注册版本。  
理论上讲应该是完美爆破。  
  

不错，继续努力。  
但是楼主没有仔细看 FGInt，上面的那些转换都是 FGInt 中的 Procedure，如 ConvertBase2to64.  
你可以做出 FGInt 的 sig,然后在 IDA 中 sig 一下，这样就更清晰，不用这么麻烦了。  
另外，这个软件已经有注册机了，N 已经被跑出来了，当然替换也是一种办法。  
RM.to.AVI.MPEG.WMV.VCD.SVCD.DVD.Converter.v2.4.0.WinALL.Incl.Keygen-ViRiLiTY