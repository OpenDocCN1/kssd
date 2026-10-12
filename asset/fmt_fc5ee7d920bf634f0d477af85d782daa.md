
# 逆向 Keygen for Matlab R14

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-130.htm`](https://www.pediy.com/kssd/pediy09/pediy09-130.htm)

> 很想知道 Matlab 的注册算法，不过自己去跟 Matlab 难度太大，还是分析现成的 keygen 来得快。

> TEAM ROR 在很久以前曾经出过几个 Matlab 的 keygen，分别对应 Matlab v7.0, v7.01, v7.1。

> 我选择了 Keygen for Matlab R14（for v7.1）动手。废话少说，开始工作。

## 1、查壳

`UPX 0.71 - 0.72 -> Markus & Laszlo`

手动脱壳，轻松搞定。

## 2、脱壳后检查

`Microsoft Visual C++ 6.0`

没有使用任何密码学算法。

## 3、找注册码生成函数

`IDA` 加载，分析完成后搜索字符串“14”，找到一处。该处的函数正是生成注册码的关键函数：

```
UPX0:00407890 GenerateKey     proc near            ; CODE XREF: sub_407470+1E8p
UPX0:00407890                 ; UPX0:00407880j
UPX0:00407890

UPX0:00407890 szTemp          = byte ptr -1020h
UPX0:00407890 szSerial        = dword ptr -1000h
UPX0:00407890

UPX0:00407890                 mov     eax, 1020h
UPX0:00407895                 call    __alloca_probe
UPX0:0040789A                 push    ebx
UPX0:0040789B                 push    ebp
UPX0:0040789C                 push    esi
UPX0:0040789D                 push    edi
UPX0:0040789E                 mov     ebp, ecx
UPX0:004078A0                 mov     szBuf, 62h
UPX0:004078A7                 mov     byte_412215, 9
UPX0:004078AE                 call    GetTickCount
UPX0:004078B4                 mov     esi, eax
UPX0:004078B6                 call    rand
UPX0:004078BC                 imul    esi, eax
UPX0:004078BF                 mov     word_412216, si
UPX0:004078C6                 mov     byte ptr dword_412218, 33h
UPX0:004078CD                 mov     byte ptr dword_412218+1, 0
UPX0:004078D4                 call    func1           ; 这是一个关键 call,比较简单
UPX0:004078D9                 call    func4           ; 这也是一个关键 call，稍微复杂一些
UPX0:004078DE                 lea     eax, [esp+1030h+szSerial]
UPX0:004078E2                 push    offset a14      ; "14"
UPX0:004078E7                 push    eax
UPX0:004078E8                 call    lstrcpy
UPX0:004078EE                 mov     ebx, wsprintfA
UPX0:004078F4                 mov     edi, lstrcat
UPX0:004078FA                 mov     esi, offset szBuf
UPX0:004078FF

UPX0:004078FF loc_4078FF:                       ; CODE XREF: GenerateKey+AEj
UPX0:004078FF                 push    esi
UPX0:00407900                 call    func5           ; 这个函数比较简单
UPX0:00407905                 add     esp, 4
UPX0:00407908                 and     eax, 0FFFFh
UPX0:0040790D                 lea     ecx, [esp+1030h+szTemp]
UPX0:00407911                 push    eax
UPX0:00407912                 push    offset a05lu    ; "%05lu"
UPX0:00407917                 push    ecx           ; LPSTR
UPX0:00407918                 call    ebx ; wsprintfA
UPX0:0040791A                 add     esp, 0Ch
UPX0:0040791D                 lea     edx, [esp+1030h+szSerial]
UPX0:00407921                 push    offset asc_412060 ; "-"
UPX0:00407926                 push    edx
UPX0:00407927                 call    edi ; lstrcat
UPX0:00407929                 lea     eax, [esp+1030h+szTemp]
UPX0:0040792D                 lea     ecx, [esp+1030h+szSerial]
UPX0:00407931                 push    eax
UPX0:00407932                 push    ecx
UPX0:00407933                 call    edi ; lstrcat
UPX0:00407935                 add     esi, 2
UPX0:00407938                 cmp     esi, offset word_41226A
UPX0:0040793E                 jb      short loc_4078FF
UPX0:00407940                 lea     edx, [esp+1030h+szSerial]
UPX0:00407944                 lea     ecx, [ebp+9D8h]
UPX0:0040794A                 push    edx
UPX0:0040794B                 call    CWnd::SetWindowTextA(char const *)
UPX0:00407950                 pop     edi
UPX0:00407951                 pop     esi
UPX0:00407952                 pop     ebp
UPX0:00407953                 pop     ebx
UPX0:00407954                 add     esp, 1020h
UPX0:0040795A                 retn
UPX0:0040795A GenerateKey     endp
```

这个函数不太复杂，其中只有 3 个未知函数需要继续分析。逆向出来的 cpp 代码如下：

```cpp
unsigned char szBuf[87] = {0};

void CKeyGenDlg::OnGenerate()
{
  char szSerial[1024] = {0};
  char szTemp[32] = {0};
  unsigned long dw = 0;

  szBuf[0] = 0x62;
  szBuf[1] = 0x09;
  dw = GetTickCount() * rand();
  szBuf[2] = (char)(dw & 0xFF);
  szBuf[3] = (char)((dw >> 8) & 0xFF);
  szBuf[4] = 0x33;
  szBuf[5] = 0x0;

  func1();
  func4();

  lstrcpy( szSerial, "14" );

  int i = 0;
  while( i < 85 )
  {
    wsprintf( szTemp, "%05lu", func5(szBuf+i)&0xFFFF );
    lstrcat( szSerial, "-" );
    lstrcat( szSerial, szTemp );
    i += 2;
  }

  SetDlgItemText( IDC_SERIAL, szSerial );
}
```

下面是 `func1()` 的汇编代码，由 3 个独立的循环组成，也不是很复杂。

```
UPX0:00407170 func1           proc near            ; CODE XREF: GenerateKey+44p
UPX0:00407170

UPX0:00407170 var_A80         = dword ptr -0A80h
UPX0:00407170 var_A00         = dword ptr -0A00h
UPX0:00407170

UPX0:00407170                 sub     esp, 0A80h
UPX0:00407176                 mov     ecx, 4
UPX0:0040717B                 mov     [esp+0A80h+var_A80], 1F1F1F1Fh
UPX0:00407183                 mov     edx, ecx
UPX0:00407185

UPX0:00407185 loc_407185:                       ; CODE XREF: func1+26j
UPX0:00407185                 mov     eax, 1
UPX0:0040718A

UPX0:0040718A loc_40718A:                       ; CODE XREF: func1+23j
UPX0:0040718A                 mov     byte ptr [esp+ecx+0A80h+var_A80], al
UPX0:0040718E                 inc     ecx
UPX0:0040718F                 inc     eax
UPX0:00407190                 cmp     eax, 31
UPX0:00407193                 jle     short loc_40718A
UPX0:00407195                 dec     edx
UPX0:00407196                 jnz     short loc_407185
UPX0:00407198                 push    ebx
UPX0:00407199                 push    ebp
UPX0:0040719A                 push    esi
UPX0:0040719B                 push    edi
UPX0:0040719C                 xor     ebp, ebp
UPX0:0040719E                 lea     edi, [esp+0A90h+var_A00]
UPX0:004071A5

UPX0:004071A5 loc_4071A5:                       ; CODE XREF: func1+60j
UPX0:004071A5                 xor     eax, eax
UPX0:004071A7                 mov     edx, 4
UPX0:004071AC                 mov     al, byte ptr [esp+ebp+0A90h+var_A80]
UPX0:004071B0                 mov     esi, eax
UPX0:004071B2                 mov     eax, edi
UPX0:004071B4                 add     edi, 20
UPX0:004071B7

UPX0:004071B7 loc_4071B7:                       ; CODE XREF: func1+57j
UPX0:004071B7                 mov     ebx, esi
UPX0:004071B9                 mov     cl, dl
UPX0:004071BB                 shr     ebx, cl
UPX0:004071BD                 add     eax, 4
UPX0:004071C0                 and     ebx, 1
UPX0:004071C3                 dec     edx
UPX0:004071C4                 mov     [eax-4], ebx
UPX0:004071C7                 jns     short loc_4071B7
UPX0:004071C9                 inc     ebp
UPX0:004071CA                 cmp     ebp, 128
```
