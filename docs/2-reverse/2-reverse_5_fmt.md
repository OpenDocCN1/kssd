
# PEID 的识别机制

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-332.htm`](https://www.pediy.com/kssd/pediy08/pediy8-332.htm)

看雪出的软件加密一书写的不错，对程序员开发水平的提高很有帮助。

此书对软件开发环境的伪装有涉及，大致讨论了一下 `FI` 的识别机制，在此讨论基础上，我深入了解了一下一些主流识别软件的机制，对识别方法也有一些补充，很希望和大家讨论交流一下，路过高手请不吝指点一二。

我以 `PEID` 识别某个 `Delphi` 程序为例，跟踪其识别过程。

检查 `PE` 合法性和使用用户定义数据文件暂不讨论。

`PEID` 判断一个应用程序的开发环境主要依据 3 个地方：

1. 代码入口
2. `PE` 结构中的链接器版本
    ```c
    BYTE MajorLinkerVersion;
    BYTE MinorLinkerVersion;
    ```
3. 特征码，对于 `Delphi`，把特征码定位在 `CODE` 节里，这里是 `Delphi` 的支持库代码。不同的 `Delphi` 版本，对应特征码不同。

我们来看看代码。

先自查一下，代码有加密，`PECompact 2.x`，`esp` 定理秒杀之。

```asm
0043FBF0    .   81EC 80050000           sub esp,580
0043FBF6    .   B8 00AB0000             mov eax,0AB00
0043FBFB    .   53                      push ebx
0043FBFC    .   55                      push ebp
0043FBFD    .   56                      push esi
0043FBFE    .   B9 E8000000             mov ecx,0E8
0043FC03    .   BD FF000000             mov ebp,0FF
0043FC08    .   BA 89000000             mov edx,89
0043FC0D    .   BE 42000000             mov esi,42
0043FC12    .   57                      push edi
0043FC13    .   BF 05000000             mov edi,5
0043FC18    .   BB C0000000             mov ebx,0C0
0043FC1D    .   66:C74424 14 5000       mov word ptr ss:[esp+14],50    ; 定义特征码
0043FC24    .   66:C74424 16 6A00       mov word ptr ss:[esp+16],6A
0043FC2B    .   66:C74424 18 0000       mov word ptr ss:[esp+18],0
0043FC32    .   66:894C24 1A            mov word ptr ss:[esp+1A],cx
0043FC37    .   66:894424 1C            mov word ptr ss:[esp+1C],ax
0043FC3C    .   66:894424 1E            mov word ptr ss:[esp+1E],ax
0043FC41    .   66:896C24 20            mov word ptr ss:[esp+20],bp
0043FC46    .   66:896C24 22            mov word ptr ss:[esp+22],bp
0043FC4B    .   66:C74424 24 BA00       mov word ptr ss:[esp+24],0BA
0043FC52    .   66:894424 26            mov word ptr ss:[esp+26],ax
0043FC57    .   66:894424 28            mov word ptr ss:[esp+28],ax
0043FC5C    .   66:894424 2A            mov word ptr ss:[esp+2A],ax
0043FC61    .   66:894424 2C            mov word ptr ss:[esp+2C],ax
0043FC66    .   66:C74424 2E 5200       mov word ptr ss:[esp+2E],52
0043FC6D    .   66:895424 30            mov word ptr ss:[esp+30],dx
0043FC72    .   66:897C24 32            mov word ptr ss:[esp+32],di
0043FC77    .   66:894424 34            mov word ptr ss:[esp+34],ax
0043FC7C    .   66:894424 36            mov word ptr ss:[esp+36],ax
0043FC81    .   66:894424 38            mov word ptr ss:[esp+38],ax
0043FC86    .   66:894424 3A            mov word ptr ss:[esp+3A],ax
0043FC8B    .   66:895424 3C            mov word ptr ss:[esp+3C],dx
0043FC90    .   66:897424 3E            mov word ptr ss:[esp+3E],si
0043FC95    .   66:C74424 40 0400       mov word ptr ss:[esp+40],4
0043FC9C    .   66:894C24 42            mov word ptr ss:[esp+42],cx
0043FCA1    .   66:894424 44            mov word ptr ss:[esp+44],ax
0043FCA6    .   66:894424 46            mov word ptr ss:[esp+46],ax
0043FCAB    .   66:894424 48            mov word ptr ss:[esp+48],ax
0043FCB0    .   66:894424 4A            mov word ptr ss:[esp+4A],ax
0043FCB5    .   66:C74424 4C 5A00       mov word ptr ss:[esp+4C],5A
0043FCBC    .   66:C74424 4E 5800       mov word ptr ss:[esp+4E],58
0043FCC3    .   66:894C24 50            mov word ptr ss:[esp+50],cx
0043FCC8    .   66:894424 52            mov word ptr ss:[esp+52],ax
0043FCCD    .   66:C74424 54 000A       mov word ptr ss:[esp+54],0A00
0043FCD4    .   66:894424 56            mov word ptr ss:[esp+56],ax
0043FCD9    .   66:894424 58            mov word ptr ss:[esp+58],ax
0043FCDE    .   66:C74424 5A C300       mov word ptr ss:[esp+5A],0C3
0043FCE5    .   66:C74424 5C 5500       mov word ptr ss:[esp+5C],55
0043FCEC    .   66:C74424 5E 8B00       mov word ptr ss:[esp+5E],8B
0043FCF3    .   66:C74424 60 EC00       mov word ptr ss:[esp+60],0EC
0043FCFA    .   66:C74424 62 3300       mov word ptr ss:[esp+62],33
0043FD01    .   66:895C24 64            mov word ptr ss:[esp+64],bx
0043FD06    .   66:C78424 C4000000 5000 mov word ptr ss:[esp+C4],50
0043FD10    .   66:C78424 C6000000 6A00 mov word ptr ss:[esp+C6],6A
0043FD1A    .   66:898424 C8000000      mov word ptr ss:[esp+C8],ax
0043FD22    .   66:898C24 CA000000      mov word ptr ss:[esp+CA],cx
0043FD2A    .   66:898424 CC000000      mov word ptr ss:[esp+CC],ax
0043FD32    .   66:898424 CE000000      mov word ptr ss:[esp+CE],ax
0043FD3A    .   66:89AC24 D0000000      mov word ptr ss:[esp+D0],bp
0043FD42    .   66:89AC24 D2000000      mov word ptr ss:[esp+D2],bp
0043FD4A    .   66:C78424 D4000000 BA00 mov word ptr ss:[esp+D4],0BA
0043FD54    .   66:898424 D6000000      mov word ptr ss:[esp+D6],ax
0043FD5C    .   66:898424 D8000000      mov word ptr ss:[esp+D8],ax
0043FD64    .   66:898424 DA000000      mov word ptr ss:[esp+DA],ax
0043FD6C    .   66:898424 DC000000      mov word ptr ss:[esp+DC],ax
0043FD74    .   66:C78424 DE000000 5200 mov word ptr ss:[esp+DE],52
0043FD7E    .   66:899424 E0000000      mov word ptr ss:[esp+E0],dx
0043FD86    .   66:89BC24 E2000000      mov word ptr ss:[esp+E2],di
0043FD8E    .   66:898424 E4000000      mov word ptr ss:[esp+E4],ax
0043FD96    .   66:898424 E6000000      mov word ptr ss:[esp+E6],ax
0043FD9E    .   66:898424 E8000000      mov word ptr ss:[esp+E8],ax
0043FDA6    .   66:898424 EA000000      mov word ptr ss:[esp+EA],ax
0043FDAE    .   66:899424 EC000000      mov word ptr ss:[esp+EC],dx
0043FDB6    .   66:89B424 EE000000      mov word ptr ss:[esp+EE],si
0043FDBE    .   66:C78424 F0000000 0400 mov word ptr ss:[esp+F0],4
0043FDC8    .   66:C78424 F2000000 C700 mov word ptr ss:[esp+F2],0C7
0043FDD2    .   66:89B424 F4000000      mov word ptr ss:[esp+F4],si
0043FDDA    .   66:C78424 F6000000 0800 mov word ptr ss:[esp+F6],8
0043FDE4    .   66:898424 F8000000      mov word ptr ss:[esp+F8],ax
0043FDEC    .   66:898424 FA000000      mov word ptr ss:[esp+FA],ax
0043FDF4    .   66:898424 FC000000      mov word ptr ss:[esp+FC],ax
0043FDFC    .   66:89B424 02010000      mov word ptr ss:[esp+102],si
0043FE04    .   BE A3000000             mov esi,0A3
0043FE09    .   66:89AC24 48010000      mov word ptr ss:[esp+148],bp
0043FE11    .   BD 33000000             mov ebp,33
0043FE16    .   66:898424 FE000000      mov word ptr ss:[esp+FE],ax
0043FE1E    .   66:C78424 00010000 C700 mov word ptr ss:[esp+100],0C7
0043FE28    .   66:C78424 04010000 0C00 mov word ptr ss:[esp+104],0C
0043FE32    .   66:898424 06010000      mov word ptr ss:[esp+106],ax
0043FE3A    .   66:898424 08010000      mov word ptr ss:[esp+108],ax
0043FE42    .   66:898424 0A010000      mov word ptr ss:[esp+10A],ax
0043FE4A    .   66:898424 0C010000      mov word ptr ss:[esp+10C],ax
0043FE52    .   66:898C24 0E010000      mov word ptr ss:[esp+10E],cx
0043FE5A    .   66:898424 10010000      mov word ptr ss:[esp+110],ax
0043FE62    .   66:898424 12010000      mov word ptr ss:[esp+112],ax
```



```
0043FE6A       .  66:898424 14010000         mov word ptr ss:[esp+114],ax  

0043FE72       .  66:898424 16010000         mov word ptr ss:[esp+116],ax  

0043FE7A       .  66:C78424 18010000 5A00     mov word ptr ss:[esp+118],5A  

0043FE84       .  66:C78424 1A010000 5800     mov word ptr ss:[esp+11A],58  

0043FE8E       .  66:898C24 1C010000         mov word ptr ss:[esp+11C],cx  

0043FE96       .  66:898424 1E010000         mov word ptr ss:[esp+11E],ax  

0043FE9E       .  66:898424 20010000         mov word ptr ss:[esp+120],ax  

0043FEA6       .  66:898424 22010000         mov word ptr ss:[esp+122],ax  

0043FEAE       .  66:898424 24010000         mov word ptr ss:[esp+124],ax  

0043FEB6       .  66:C78424 26010000 C300     mov word ptr ss:[esp+126],0C3  

0043FEC0       .  66:C78424 28010000 5300     mov word ptr ss:[esp+128],53  

0043FECA       .  66:C78424 2A010000 8B00     mov word ptr ss:[esp+12A],8B  

0043FED4       .  66:C78424 2C010000 D800     mov word ptr ss:[esp+12C],0D8  

0043FEDE       .  66:C78424 2E010000 3300     mov word ptr ss:[esp+12E],33  

0043FEE8       .  66:899C24 30010000         mov word ptr ss:[esp+130],bx  

0043FEF0       .  66:89B424 32010000         mov word ptr ss:[esp+132],si  

0043FEF8       .  66:898424 34010000         mov word ptr ss:[esp+134],ax  

0043FF00       .  66:898424 36010000         mov word ptr ss:[esp+136],ax  

0043FF08       .  66:898424 38010000         mov word ptr ss:[esp+138],ax  

0043FF10       .  66:898424 3A010000         mov word ptr ss:[esp+13A],ax  

0043FF18       .  66:C78424 3C010000 6A00     mov word ptr ss:[esp+13C],6A  

0043FF22       .  66:898424 3E010000         mov word ptr ss:[esp+13E],ax  

0043FF2A       .  66:898C24 40010000         mov word ptr ss:[esp+140],cx  

0043FF32       .  66:898424 42010000         mov word ptr ss:[esp+142],ax  

0043FF3A       .  66:898424 44010000         mov word ptr ss:[esp+144],ax  

0043FF42       .  66:898424 46010000         mov word ptr ss:[esp+146],ax  

0043FF4A       .  66:89B424 4A010000         mov word ptr ss:[esp+14A],si  

0043FF52       .  66:898424 4C010000         mov word ptr ss:[esp+14C],ax  

0043FF5A       .  66:898424 4E010000         mov word ptr ss:[esp+14E],ax  

0043FF62       .  66:898424 50010000         mov word ptr ss:[esp+150],ax  

0043FF6A       .  66:898424 52010000         mov word ptr ss:[esp+152],ax  

0043FF72       .  66:C78424 54010000 A100     mov word ptr ss:[esp+154],0A1  

0043FF7C       .  66:898424 56010000         mov word ptr ss:[esp+156],ax  

0043FF84       .  66:898424 58010000         mov word ptr ss:[esp+158],ax  

0043FF8C       .  66:898424 5A010000         mov word ptr ss:[esp+15A],ax  

0043FF94       .  66:898424 5C010000         mov word ptr ss:[esp+15C],ax  

0043FF9C       .  66:89B424 5E010000         mov word ptr ss:[esp+15E],si  

0043FFA4       .  66:898424 60010000         mov word ptr ss:[esp+160],ax  

0043FFAC       .  66:898424 62010000         mov word ptr ss:[esp+162],ax  

0043FFB4       .  66:898424 64010000         mov word ptr ss:[esp+164],ax  

0043FFBC       .  66:898424 66010000         mov word ptr ss:[esp+166],ax  

0043FFC4       .  66:89AC24 68010000         mov word ptr ss:[esp+168],bp  

0043FFCC       .  66:899C24 6A010000         mov word ptr ss:[esp+16A],bx  

0043FFD4       .  66:89B424 6C010000         mov word ptr ss:[esp+16C],si  

0043FFDC       .  66:898424 6E010000         mov word ptr ss:[esp+16E],ax  

0043FFE4       .  66:898424 70010000         mov word ptr ss:[esp+170],ax  

0043FFEC       .  66:898424 72010000         mov word ptr ss:[esp+172],ax  

0043FFF4       .  66:898424 74010000         mov word ptr ss:[esp+174],ax  

0043FFFC       .  66:89AC24 76010000         mov word ptr ss:[esp+176],bp  

00440004       .  66:899C24 78010000         mov word ptr ss:[esp+178],bx  

0044000C       .  66:89B424 7A010000         mov word ptr ss:[esp+17A],si  

00440014       .  66:898424 7C010000         mov word ptr ss:[esp+17C],ax  

0044001C       .  66:898424 7E010000         mov word ptr ss:[esp+17E],ax  

00440024       .  66:898424 80010000         mov word ptr ss:[esp+180],ax  

0044002C       .  66:898424 82010000         mov word ptr ss:[esp+182],ax  

00440034       .  66:898C24 84010000         mov word ptr ss:[esp+184],cx  

0044003C       .  66:894C24 68             mov word ptr ss:[esp+68],cx  

00440041       .  66:894424 6A             mov word ptr ss:[esp+6A],ax  

00440046       .  66:894424 6C             mov word ptr ss:[esp+6C],ax  

0044004B       .  66:894424 6E             mov word ptr ss:[esp+6E],ax  

00440050       .  66:894424 70             mov word ptr ss:[esp+70],ax  

00440055       .  66:C74424 72 6A00         mov word ptr ss:[esp+72],6A  

0044005C       .  66:894424 74             mov word ptr ss:[esp+74],ax  

00440061       .  66:894C24 76             mov word ptr ss:[esp+76],cx  

00440066       .  66:894424 78             mov word ptr ss:[esp+78],ax  

0044006B       .  66:894424 7A             mov word ptr ss:[esp+7A],ax  

00440070       .  66:894424 7C             mov word ptr ss:[esp+7C],ax  

00440075       .  8BB424 98050000           mov esi,dword ptr ss:[esp+598]  

0044007C       .  66:894424 7E             mov word ptr ss:[esp+7E],ax  

00440081       .  66:898424 84000000         mov word ptr ss:[esp+84],ax  

00440089       .  66:898424 86000000         mov word ptr ss:[esp+86],ax  

00440091       .  66:898424 88000000         mov word ptr ss:[esp+88],ax  

00440099       .  66:898424 8A000000         mov word ptr ss:[esp+8A],ax  

004400A1       .  66:898424 8E000000         mov word ptr ss:[esp+8E],ax  

004400A9       .  66:898424 90000000         mov word ptr ss:[esp+90],ax  

004400B1       .  66:898424 92000000         mov word ptr ss:[esp+92],ax  

004400B9       .  66:898424 94000000         mov word ptr ss:[esp+94],ax  

004400C1       .  66:898424 9A000000         mov word ptr ss:[esp+9A],ax  

004400C9       .  66:898424 9C000000         mov word ptr ss:[esp+9C],ax  

004400D1       .  66:898424 9E000000         mov word ptr ss:[esp+9E],ax  

004400D9       .  66:898424 A0000000         mov word ptr ss:[esp+A0],ax  

004400E1       .  66:898424 A6000000         mov word ptr ss:[esp+A6],ax  

004400E9       .  66:898424 A8000000         mov word ptr ss:[esp+A8],ax  

004400F1       .  66:898424 AA000000         mov word ptr ss:[esp+AA],ax  

004400F9       .  66:898424 AC000000         mov word ptr ss:[esp+AC],ax  

00440101       .  66:898424 B0000000         mov word ptr ss:[esp+B0],ax  

00440109       .  66:898424 B2000000         mov word ptr ss:[esp+B2],ax  

00440111       .  66:898424 B4000000         mov word ptr ss:[esp+B4],ax  

00440119       .  66:898424 B8000000         mov word ptr ss:[esp+B8],ax  

00440121       .  66:898424 BA000000         mov word ptr ss:[esp+BA],ax  

00440129       .  66:898424 BC000000         mov word ptr ss:[esp+BC],ax  

00440131       .  66:898424 BE000000         mov word ptr ss:[esp+BE],ax  

00440139       .  8B46 0C                   mov eax,dword ptr ds:[esi+C]  

0044013C       .  66:898C24 8C000000         mov word ptr ss:[esp+8C],cx  

00440144       .  66:899424 80000000         mov word ptr ss:[esp+80],dx  

0044014C       .  66:89BC24 82000000         mov word ptr ss:[esp+82],di  

00440154       .  66:899424 96000000         mov word ptr ss:[esp+96],dx  

0044015C       .  66:89BC24 98000000         mov word ptr ss:[esp+98],di  

00440164       .  66:C78424 A2000000 C700     mov word ptr ss:[esp+A2],0C7  

0044016E       .  66:89BC24 A4000000         mov word ptr ss:[esp+A4],di  
```



00440176       .  66:C78424 AE000000 0A00     `mov word ptr ss:[esp+AE],0A`  

00440180       .  66:C78424 B6000000 B800     `mov word ptr ss:[esp+B6],0B8`  

0044018A       .  66:C78424 C0000000 C300     `mov word ptr ss:[esp+C0],0C3`  

; 特征码定义完成， 如下所示  

01DBF9BC  50 00 6A 00 00 00 E8 00 00 AB 00 AB FF 00 FF 00   P.j...?.??.??.  

01DBF9CC  BA 00 00 AB 00 AB 00 AB 00 AB 52 00 89 00 05 00   ?+????.?.  

01DBF9DC  00 AB 00 AB 00 AB 00 AB 89 00 42 00 04 00 E8 00   .?+??.B..?  

01DBF9EC  00 AB 00 AB 00 AB 00 AB 5A 00 58 00 E8 00 00 AB   .?+?+?X.?.?  

01DBF9FC  00 0A 00 AB 00 AB C3 00 55 00 8B 00 EC 00 33 00   .+?+.U.??3.  

01DBFA0C  C0 00 00 00 E8 00 00 AB 00 AB 00 AB 00 AB 6A 00   ?..?.+?+?+?J.  

01DBFA1C  00 AB E8 00 00 AB 00 AB 00 AB 00 AB 89 00 05 00   .+?00?++?+.  

01DBFA2C  00 AB 00 AB 00 AB 00 AB E8 00 00 AB 00 AB 00 AB   .++?+?+??  

01DBFA3C  00 AB 89 00 05 00 00 AB 00 AB 00 AB 00 AB C7 00   .?0...+?+?  

01DBFA4C  05 00 00 AB 00 AB 00 AB 00 AB 0A 00 00 AB 00 AB   ..???+??  

01DBFA5C  00 AB B8 00 00 AB 00 AB 00 AB 00 AB C3 00 DB 01   .+B00+?++?.  

01DBFA6C  50 00 6A 00 00 AB E8 00 00 AB 00 AB FF 00 FF 00   P.J00++?+FF.  

01DBFA7C  BA 00 00 AB 00 AB 00 AB 00 AB 52 00 89 00 05 00   BA00++?+R.?.  

01DBFA8C  00 AB 00 AB 00 AB 00 AB 89 00 42 00 04 00 C7 00   .++?+.B.?  

01DBFA9C  42 00 08 00 00 AB 00 AB 00 AB 00 AB C7 00 42 00   B0000++?+B  

01DBFAAC  0C 00 00 AB 00 AB 00 AB 00 AB E8 00 00 AB 00 AB   000+++?+?  

01DBFABC  00 AB 00 AB 5A 00 58 00 E8 00 00 AB 00 AB 00 AB   .+?X00?+?  

01DBFACC  00 AB C3 00 53 00 8B 00 D8 00 33 00 C0 00 A3 00   +C3S0?0030?  

01DBFADC  00 AB 00 AB 00 AB 00 AB 6A 00 00 AB E8 00 00 AB   +++++?00?+  

01DBFAEC  00 AB 00 AB FF 00 A3 00 00 AB 00 AB 00 AB 00 AB   +++?000++++  

01DBFAFC  A1 00 00 AB 00 AB 00 AB 00 AB A3 00 00 AB 00 AB   A100+?++?++  

01DBFB0C  00 AB 00 AB 33 00 C0 00 A3 00 00 AB 00 AB 00 AB   +?+30C00A30??  

01DBFB1C  00 AB 33 00 C0 00 A3 00 00 AB 00 AB 00 AB 00 AB   +30C00A30???  

01DBFB2C  E8 00 00 00                                 ?..  

00440197       .  50                              `push eax`  

00440198       .  8BCE                             `mov ecx,esi`                             ;  ecx  MZ 头部  

0044019A       .  E8 E1300100                      `call unpack.00453280`  

0044019F       .  8B0E                             `mov ecx,dword ptr ds:[esi]`  

004401A1       .  8BF8                             `mov edi,eax`  

004401A3       .  8D0439                           `lea eax,dword ptr ds:[ecx+edi]`           ;  eax  OEP  

004401A6       .  8038 55                          `cmp byte ptr ds:[eax],55`                 ;  比较入口， Delphi 的入口应该是 55 8B EC 83(B9)  

004401A9       .  0F85 44010000                   `jnz unpack.004402F3`  

004401AF       .  8078 01 8B                       `cmp byte ptr ds:[eax+1],8B`  

004401B3       .  0F85 3A010000                   `jnz unpack.004402F3`  

004401B9       .  8078 02 EC                       `cmp byte ptr ds:[eax+2],0EC`  

004401BD       .  0F85 30010000                   `jnz unpack.004402F3`  

004401C3       .  8A40 03                          `mov al,byte ptr ds:[eax+3]`  

004401C6       .  3C 83                           `cmp al,83`  

004401C8       .  74 08                           `je short unpack.004401D2`  

004401CA       .  3C B9                           `cmp al,0B9`  

004401CC       .  0F85 21010000                   `jnz unpack.004402F3`  

004401D2       >  8B46 0C                          `mov eax,dword ptr ds:[esi+C]`               ;  eax  PE 头  

004401D5       .  8078 1A 02                       `cmp byte ptr ds:[eax+1A],2`                 ;  检查连接器版本, Delphi 对应的值应该是 02 19  

004401D9       .  0F85 14010000                   `jnz unpack.004402F3`  

004401DF       .  8078 1B 19                       `cmp byte ptr ds:[eax+1B],19`  

004401E3       .  0F85 0A010000                   `jnz unpack.004402F3`  

004401E9       .  6A 29                           `push 29`                                   ;  特征码长度  

004401EB       .  8D5424 18                       `lea edx,dword ptr ss:[esp+18]`              ;  edx  Delphi 特征码库首地址  

004401EF       .  52                              `push edx`  

004401F0       .  8D8C24 90010000                 `lea ecx,dword ptr ss:[esp+190]`             ;  ecx  Delphi 特征码库尾地址  

004401F7       .  E8 4471FFFF                      `call unpack.00437340`  

004401FC       .  8B0E                             `mov ecx,dword ptr ds:[esi]`                 ;  ecx  MZ 头  

004401FE       .  8D4424 10                        `lea eax,dword ptr ss:[esp+10]`  

00440202       .  50                              `push eax`                                   ; /Arg3  

00440203       .  81C1 00040000                    `add ecx,400`                              ; |ecx  CODE 节  

00440209       .  57                              `push edi`                                   ; |Arg2  

0044020A       .  51                              `push ecx`                                   ; |Arg1  

0044020B       .  8D8C24 94010000                  `lea ecx,dword ptr ss:[esp+194]`             ; |ecx  特征码地址  

00440212       .  E8 39F4FFFF                      `call unpack.0043F650`                      ; \\在 CODE 节里搜索特征码  

00440217       .  84C0                             `test al,al`  

00440219       .  74 0A                             `je short unpack.00440225`  

0044021B       .  68 94634000                       `push unpack.00406394`                      ;  ASCII "Borland Delphi 3.0"  

00440220       .  E9 AF000000                      `jmp unpack.004402D4`  

00440225       >  6A 32                             `push 32`  

00440227       .  8D9424 C8000000                  `lea edx,dword ptr ss:[esp+C8]`  

0044022E       .  52                              `push edx`  

0044022F       .  8D8C24 90010000                  `lea ecx,dword ptr ss:[esp+190]`  

00440236       .  E8 0571FFFF                      `call unpack.00437340`  

0044023B       .  8B0E                             `mov ecx,dword ptr ds:[esi]`                 ;  MZ  

0044023D       .  8D4424 10                        `lea eax,dword ptr ss:[esp+10]`  

00440241       .  50                              `push eax`                                   ; /Arg3  

00440242       .  81C1 00040000                    `add ecx,400`                              ; |CODE 节  

00440248       .  57                              `push edi`                                   ; |Arg2  

00440249       .  51                              `push ecx`                                   ; |Arg1  

0044024A       .  8D8C24 94010000                  `lea ecx,dword ptr ss:[esp+194]`             ; |  

00440251       .  E8 FAF3FFFF                      `call unpack.0043F650`                      ; \\在 CODE 节里搜索特征码  

00440256       .  84C0                             `test al,al`  

00440258       .  74 07                             `je short unpack.00440261`  

0044025A       .  68 78634000                       `push unpack.00406378`                      ;  ASCII "Borland Delphi 4.0 - 5.0"  

0044025F       .  EB 73                             `jmp short unpack.004402D4`  

00440261       >  6A 2F                             `push 2F`  

00440263       .  8D9424 2C010000                  `lea edx,dword ptr ss:[esp+12C]`  

0044026A       .  52                              `push edx`  

0044026B       .  8D8C24 90010000                  `lea ecx,dword ptr ss:[esp+190]`  

00440272       .  E8 C970FFFF                      `call unpack.00437340`  

00440277       .  8B0E                             `mov ecx,dword ptr ds:[esi]`  

00440279       .  8D4424 10                        `lea eax,dword ptr ss:[esp+10]`  

0044027D       .  50                              `push eax`                                   ; /Arg3  

0044027E       .  81C1 00040000                    `add ecx,400`                              ; |  

00440284       .  57                              `push edi`                                   ; |Arg2  

00440285       .  51                              `push ecx`                                   ; |Arg1  

00440286       .  8D8C24 94010000                  `lea ecx,dword ptr ss:[esp+194]`             ; |  



0044028D       .  E8 BEF3FFFF              call unpack.0043F650                   ; \在 CODE 节里搜索特征码  

00440292       .  84C0                      test al,al  

00440294        74 07                       je short unpack.0044029D  

00440296       .  68 5C634000               push unpack.0040635C                   ;  ASCII "Borland Delphi 6.0 - 7.0"  

0044029B       .  EB 37                    jmp short unpack.004402D4  

0044029D       >  6A 2D                    push 2D  

0044029F       .  8D5424 6C                lea edx,dword ptr ss:\[esp+6C\]  

004402A3       .  52                        push edx  

004402A4       .  8D8C24 90010000          lea ecx,dword ptr ss:\[esp+190\]  

004402AB       .  E8 9070FFFF              call unpack.00437340  

004402B0       .  8B0E                      mov ecx,dword ptr ds:\[esi\]  

004402B2       .  8D4424 10                lea eax,dword ptr ss:\[esp+10\]  

004402B6       .  50                        push eax                                 ; /Arg3  

004402B7       .  81C1 00040000            add ecx,400                              ; |  

004402BD       .  57                        push edi                                 ; |Arg2  

004402BE       .  51                        push ecx                                 ; |Arg1  

004402BF       .  8D8C24 94010000          lea ecx,dword ptr ss:\[esp+194\]         ; |  

004402C6       .  E8 85F3FFFF              call unpack.0043F650                   ; \在 CODE 节里搜索特征码  

004402CB       .  84C0                      test al,al  

004402CD       .  74 24                    je short unpack.004402F3  

004402CF       .  68 48634000              push unpack.00406348                   ;  ASCII "Borland Delphi 2.0"  

004402D4       >  8BB424 98050000          mov esi,dword ptr ss:\[esp+598\]  

004402DB       .  8D4E 04                 lea ecx,dword ptr ds:\[esi+4\]  

004402DE       .  E8 BD73FFFF              call unpack.004376A0  

004402E3       .  5F                        pop edi  

004402E4       .  C606 01                   mov byte ptr ds:\[esi\],1  

004402E7       .  5E                        pop esi  

004402E8       .  5D                        pop ebp  

004402E9       .  B0 01                     mov al,1  

004402EB       .  5B                        pop ebx  

004402EC       .  81C4 80050000            add esp,580  

004402F2       .  C3                        retn  

; 在 CODE 节里搜索特征码，  

; 特征码库定义为一个 WORD 数组，  

; 按 CODE 节里的数据一一和对应版本的特征码比较。  

; 其中，特征码中的 00 AB 是通配符。  

0043F650     /$  83EC 08                   sub esp,8                              ;  在 CODE 节里搜索特征码  

0043F653     |.  8B5424 10                mov edx,dword ptr ss:\[esp+10\]  

0043F657     |.  57                       push edi  

0043F658     |.  8B79 04                 mov edi,dword ptr ds:\[ecx+4\]  

0043F65B     |.  3BD7                     cmp edx,edi  

0043F65D     |.  894C24 04               mov dword ptr ss:\[esp+4\],ecx  

0043F661     |.  7D 09                    jge short unpack.0043F66C  

0043F663     |.  32C0                     xor al,al  

0043F665     |.  5F                       pop edi  

0043F666     |.  83C4 08                 add esp,8  

0043F669     |.  C2 0C00                  retn 0C  

0043F66C     |>  53                       push ebx  

0043F66D     |.  55                       push ebp  

0043F66E     |.  8B6C24 18               mov ebp,dword ptr ss:\[esp+18\]           ;  ebp = CODE 节首地址  

0043F672     |.  8BC2                     mov eax,edx  

0043F674     |.  2BC7                     sub eax,edi  

0043F676     |.  56                       push esi  

0043F677     |.  33F6                     xor esi,esi  

0043F679     |.  85C0                     test eax,eax  

0043F67B     |.  894424 14               mov dword ptr ss:\[esp+14\],eax  

0043F67F     |.  7E 5F                    jle short unpack.0043F6E0  

0043F681     |>  33C0                     xor eax,eax  

0043F683     |.  85FF                     test edi,edi  

0043F685     |.  7E 26                    jle short unpack.0043F6AD  

0043F687     |.  8B19                     mov ebx,dword ptr ds:\[ecx\]            ;  ebx = 特征码首地址  

0043F689     |.  8DA424 00000000         lea esp,dword ptr ss:\[esp\]  

0043F690     |>  66:8B13                  /mov dx,word ptr ds:\[ebx\]  

0043F693     |.  F6C6 FF                  |test dh,0FF  

0043F696     |.  75 0D                    |jnz short unpack.0043F6A5  

0043F698     |.  8D0C30                  |lea ecx,dword ptr ds:\[eax+esi\]  

0043F69B        66:0FB60C29             movzx cx,byte ptr ds:\[ecx+ebp\]  

0043F6A0        66:3BCA                 cmp cx,dx  

0043F6A3        75 1E                    jnz short unpack.0043F6C3  

0043F6A5     |>  40                       |inc eax  

0043F6A6     |.  83C3 02                |add ebx,2  

0043F6A9     |.  3BC7                    |cmp eax,edi  

0043F6AB     |.^ 7C E3                   \jl short unpack.0043F690  

0043F6AD     |>  8B4424 24               mov eax,dword ptr ss:\[esp+24\]  

0043F6B1     |.  85C0                    test eax,eax  

0043F6B3     |.  74 02                   je short unpack.0043F6B7  

0043F6B5     |.  8930                    mov dword ptr ds:\[eax\],esi  

0043F6B7     |>  5E                       pop esi  

0043F6B8     |.  5D                       pop ebp  

0043F6B9     |.  5B                       pop ebx  

0043F6BA     |.  B0 01                    mov al,1  

0043F6BC     |.  5F                       pop edi  

0043F6BD     |.  83C4 08                 add esp,8  

0043F6C0     |.  C2 0C00                  retn 0C  

0043F6C3     |>  8B4C24 10               mov ecx,dword ptr ss:\[esp+10\]  

0043F6C7     |.  8D1437                  lea edx,dword ptr ds:\[edi+esi\]  

0043F6CA     |.  0FB6042A               movzx eax,byte ptr ds:\[edx+ebp\]  

0043F6CE     |.  8B5C81 08              mov ebx,dword ptr ds:\[ecx+eax\*4+8\]  

0043F6D2     |.  8B4424 14             mov eax,dword ptr ss:\[esp+14\]  

0043F6D6     |.  03F3                   add esi,ebx  

0043F6D8     |.  3BF0                   cmp esi,eax  

0043F6DA     |.^ 7C A5                  jl short unpack.0043F681  

0043F6DC     |.  8B5424 20             mov edx,dword ptr ss:\[esp+20\]  

0043F6E0     |>  3BF0                   cmp esi,eax  

0043F6E2     |.  75 10                  jnz short unpack.0043F6F4  

0043F6E4     |.  2BD6                   sub edx,esi  

0043F6E6     |.  52                      push edx  

0043F6E7     |.  8D142E                lea edx,dword ptr ds:\[esi+ebp\]  

0043F6EA     |.  52                      push edx  

0043F6EB     |.  E8 D07CFFFF           call unpack.004373C0  

0043F6F0     |.  84C0                   test al,al  

0043F6F2     |.^ 75 B9                  jnz short unpack.0043F6AD  

0043F6F4     |>  5E                      pop esi  

0043F6F5     |.  5D                      pop ebp  

0043F6F6     |.  5B                      pop ebx  

0043F6F7     |.  32C0                    xor al,al  

0043F6F9     |.  5F                      pop edi  

0043F6FA     |.  83C4 08                add esp,8  

0043F6FD     \\.  C2 0C00                retn 0C  

Delphi 各个版本的特征码如下所示：  

;Borland  Delphi 3.0  

01BBF9BC  50 00 6A 00 00 00 E8 00 00 AB 00 AB FF 00 FF 00  P.j...?.??.

01BBF9CC  BA 00 00 AB 00 AB 00 AB 00 AB 52 00 89 00 05 00  ?.???玆.?.  

01BBF9DC  00 AB 00 AB 00 AB 00 AB 89 00 42 00 04 00 E8 00  .???珘.B..?  



```
01BBF9EC  00 AB 00 AB 00 AB 00 AB 5A 00 58 00 E8 00 00 AB   .???玓.X.?.?.
01BBF9FC  00 0A 00 AB 00 AB C3 00 55 00 8B 00 EC 00 33 00  .?..U.??.3.
01BBFA0C  C0 00
;Borland  Delphi 2.0
01BBFA10  E8 00 00 AB 00 AB 00 AB 00 AB 6A 00 00 AB E8 00   ?..???玧..
01BBFA20  00 AB 00 AB 00 AB 00 AB 89 00 05 00 00 AB 00 AB  .???珘...??.
01BBFA30  00 AB 00 AB E8 00 00 AB 00 AB 00 AB 00 AB 89 00   .?..???.??.
01BBFA40  05 00 00 AB 00 AB 00 AB 00 AB C7 00 05 00 00 AB  ..???...?.
01BBFA50  00 AB 00 AB 00 AB 0A 00 00 AB 00 AB 00 AB B8 00  .???..??.
01BBFA60  00 AB 00 AB 00 AB 00 AB C3 00                    .???..
;Borland Delphi 4.0 - 5.0"
01BBFA6C  50 00 6A 00 00 AB E8 00 00 AB 00 AB FF 00 FF 00   P.j...??.??.
01BBFA7C  BA 00 00 AB 00 AB 00 AB 00 AB 52 00 89 00 05 00  ?..???玆.??.
01BBFA8C  00 AB 00 AB 00 AB 00 AB 89 00 42 00 04 00 C7 00  .???珘.B..?.
01BBFA9C  42 00 08 00 00 AB 00 AB 00 AB 00 AB C7 00 42 00   B...????.B.
01BBFAAC  0C 00
;Borland Delphi 6.0 - 7.0
01BBFAD0  53 00 8B 00 D8 00 33 00 C0 00 A3 00 00 AB 00 AB   S.??3.??.??.
01BBFAE0  00 AB 00 AB 6A 00 00 AB E8 00 00 AB 00 AB 00 AB  .?玧...??.
01BBFAF0  FF 00 A3 00 00 AB 00 AB 00 AB 00 AB A1 00 00 AB  ?.?.???...?.
01BBFB00  00 AB 00 AB 00 AB A3 00 00 AB 00 AB 00 AB 00 AB  .?..????.
01BBFB10  33 00 C0 00 A3 00 00 AB 00 AB 00 AB 00 AB 33 00   3.??.????.
01BBFB20  C0 00 A3 00 00 AB 00 AB 00 AB 00 AB E8 00
;这是某个 Delphi 6.0 程序对应的特征码
017565E0  53 8B D8 33 C0 A3 F8 A0 51 00 6A 00 E8 2B FF FF   S 嬝 3 溃鴂 Q.j.??.
017565F0  FF A3 64 E6 51 00 A1 64 E6 51 00 A3 04 A1 51 00  鍽.鍽.?.鍗.
01756600  33 C0 A3 08 A1 51 00 33 C0 A3 0C A1 51 00 E8      3 溃.鍗.3 溃.鍗.枇
```

附件里是网上随便找的一个 `VC` 程序，然后手工修改伪装成 `Borland Delphi 3.0`，成功欺骗了 `PEID`。

`4dd466d0e1e0cc123afbe504da2462ca`
