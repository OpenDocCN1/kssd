
```
004A09E7  |.  8B4D FC       |mov     ecx, dword ptr [ebp-4]
004A09EA  |.  BA A80A4A00   |mov     edx, 004A0AA8
004A09EF  |.  E8 C43FF6FF   |call    004049B8
004A09F4  |>  8B45 FC        mov     eax, dword ptr [ebp-4]
004A09F7  |.  E8 703FF6FF   |call    0040496C
004A09FC  |.  25 07000080   |and     eax, 80000007
004A0A01  |.  79 05         |jns     short 004A0A08
004A0A03  |.  48            |dec     eax
004A0A04  |.  83C8 F8       |or      eax, FFFFFFF8
004A0A07  |.  40            |inc     eax
004A0A08  |>  85C0          |test    eax, eax
004A0A0A  |.^ 75 D8         \jnz     short 004A09E4   //要求二进制串是8的次方
004A0A0C  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
004A0A0F  |.  E8 583FF6FF   call    0040496C     //取二进制串长度
004A0A14  |.  85C0          test    eax, eax
004A0A16  |.  79 03         jns     short 004A0A1B
004A0A18  |.  83C0 07       add     eax, 7
004A0A1B  |>  C1F8 03       sar     eax, 3
004A0A1E  |.  8BD8          mov     ebx, eax
004A0A20  |.  85DB          test    ebx, ebx
004A0A22  |.  7E 4B         jle     short 004A0A6F
004A0A24  |>  8D45 F4       /lea     eax, dword ptr [ebp-C]
004A0A27  |.  50            |push    eax
004A0A28  |.  B9 08000000   |mov     ecx, 8
004A0A2D  |.  BA 01000000   |mov     edx, 1
004A0A32  |.  8B45 FC       |mov     eax, dword ptr [ebp-4]
004A0A35  |.  E8 9241F6FF   |call    00404BCC               //截取二进制串左边的8位数
004A0A3A  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]
004A0A3D  |.  8D45 FB       |lea     eax, dword ptr [ebp-5]
004A0A40  |.  E8 F3FAFFFF   |call    004A0538               //将8位二进制串依次转换16进制数
004A0A45  |.  8D45 F0       |lea     eax, dword ptr [ebp-10]
004A0A48  |.  8A55 FB       |mov     dl, byte ptr [ebp-5]
004A0A4B  |.  E8 443EF6FF   |call    00404894
004A0A50  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]
004A0A53  |.  8BC6          |mov     eax, esi
004A0A55  |.  E8 1A3FF6FF   |call    00404974                转换的结果在EAX中显示
004A0A5A  |.  8D45 FC       |lea     eax, dword ptr [ebp-4]
004A0A5D  |.  B9 08000000   |mov     ecx, 8
004A0A62  |.  BA 01000000   |mov     edx, 1
004A0A67  |.  E8 A041F6FF   |call    00404C0C          //截取后余下的串
004A0A6C  |.  4B            |dec     ebx    //循环的次数
004A0A6D  |.^ 75 B5         \jnz     short 004A0A24
004A0A6F  |>  33C0          xor     eax, eax
004A0A71  |.  5A            pop     edx
004A0A72  |.  59            pop     ecx
004A0A73  |.  59            pop     ecx
004A0A74  |.  64:8910       mov     dword ptr fs:[eax], edx
004A0A77  |.  68 990A4A00   push    004A0A99
004A0A7C  |>  8D45 F0       lea     eax, dword ptr [ebp-10]
004A0A7F  |.  BA 02000000   mov     edx, 2
004A0A84  |.  E8 473CF6FF   call    004046D0
004A0A89  |.  8D45 FC       lea     eax, dword ptr [ebp-4]
004A0A8C  |.  E8 1B3CF6FF   call    004046AC
004A0A91  \.  C3            retn

//注册机参考happytown大侠的，改编而成
program Sound;
{$APPTYPE CONSOLE}
Uses Windows,Messages,FGInt,FGIntRSA;
  Var
    n,e: TFGInt;
    name,sn: String;
  Begin
    Base10StringToFGInt('296330597038313779621622317537', n);
    Base10StringToFGInt('65537', e);
    Write('User name:');
    ReadLn(name);           //name
    RSAEncrypt(name, e, n, name);
    ConvertBase256to64(name, sn);
    FGIntdestroy(n);
    FGIntdestroy(e);
    WriteLn('Registration code:', sn);
    MessageBox(0, PChar(sn), 'Registration code:', MB\_OK);//用MessageBox显示出来，
  End.
```

```
//注册码信息存放在安装目录的Setup.ini中，

[general]
info=aKi+u=bjzkBp+1NENi   //我们写入的注册码
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
