
# GLCD Font Creator v1.0 算法注册机

> 原文：[`www.pediy.com/kssd/pediy12/131352.html`](https://www.pediy.com/kssd/pediy12/131352.html)

目标：`GLCD Font Creator v1.0.1.0`

文件: `C:\\GLCDFontCreator.exe`

大小: 2208768 字节

文件版本: 1.0.1.0

修改时间: 2007 年 8 月 6 日 星期一, 21:26:38

MD5: `4A5AA90C09D2BC96CDD6B42C62A89314`

SHA1: `1DCC4396228F4226DD2C54236EA3BDE77D39DFF4`

CRC32: `706692EE`

关键代码：

代码:

```
004D3AC0   push ebp
004D3AC1   mov ebp,esp
004D3AC3   add esp,-18
004D3AC6   push ebx
004D3AC7   push esi
004D3AC8   push edi
004D3AC9   xor ebx,ebx
004D3ACB   mov dword ptr ss:\[ebp-18\],ebx
004D3ACE   mov dword ptr ss:\[ebp-14\],ebx
004D3AD1   mov dword ptr ss:\[ebp-10\],ebx
004D3AD4   mov esi,ecx
004D3AD6   mov dword ptr ss:\[ebp-8\],edx
004D3AD9   mov dword ptr ss:\[ebp-4\],eax
004D3ADC   mov eax,dword ptr ss:\[ebp-4\]
004D3ADF   call <System::\_\_linkproc\_\_ LStrAddRef(void \*)>
004D3AE4   xor eax,eax
004D3AE6   push ebp
004D3AE7   push 004D3C0F
004D3AEC   push dword ptr fs:\[eax\]
004D3AEF   mov dword ptr fs:\[eax\],esp
004D3AF2   lea ecx,dword ptr ss:\[ebp-10\]
004D3AF5   mov edx,dword ptr ss:\[ebp-8\]
004D3AF8   mov eax,dword ptr ss:\[ebp-4\]
004D3AFB   call 004D38C8
004D3B00   mov edx,dword ptr ss:\[ebp-10\]
004D3B03   lea eax,dword ptr ss:\[ebp-4\]
004D3B06   call <System::\_\_linkproc\_\_ LStrLAsg(void \*,void \*)>
004D3B0B   mov dword ptr ss:\[ebp-C\],3FF1
004D3B12   mov eax,esi
004D3B14   call <System::\_\_linkproc\_\_ LStrClr(void \*)>
004D3B19   mov eax,dword ptr ss:\[ebp-4\]
004D3B1C   call 004045E8
004D3B21   mov edi,eax
004D3B23   test edi,edi
004D3B25   jle short 004D3B69
004D3B27   mov ebx,1
004D3B2C   /lea eax,dword ptr ss:\[ebp-14\]
004D3B2F   |mov edx,dword ptr ss:\[ebp-4\]
004D3B32   |mov dl,byte ptr ds:\[edx+ebx-1\]
004D3B36   |mov ecx,dword ptr ss:\[ebp-C\]
004D3B39   |shr ecx,8
004D3B3C   |xor dl,cl
004D3B3E   |call <unknown\_libname\_73>
004D3B43   |mov edx,dword ptr ss:\[ebp-14\]
004D3B46   |mov eax,esi
004D3B48   |call <System::\_\_linkproc\_\_ LStrCat(void)>
004D3B4D   |mov eax,dword ptr ds:\[esi\]
004D3B4F   |movzx eax,byte ptr ds:\[eax+ebx-1\]
004D3B54   |add eax,dword ptr ss:\[ebp-C\]
004D3B57   imul eax,eax,0CE6D
004D3B5D   |add eax,58BF
004D3B62   |mov dword ptr ss:\[ebp-C\],eax
004D3B65   |inc ebx
004D3B66   |dec edi
004D3B67   \\jnz short 004D3B2C
004D3B69   mov ebx,dword ptr ss:\[ebp-8\]
004D3B6C   mov eax,dword ptr ss:\[ebp-4\]
004D3B6F   call 004045E8
004D3B74   mov edi,eax
004D3B76   sub edi,ebx
004D3B78   jl short 004D3BA5
004D3B7A   inc edi
004D3B7B   /mov eax,esi
004D3B7D   |call <j\_unknown\_libname\_79\_0>
004D3B82   |push eax
004D3B83   |mov eax,ebx
004D3B85   |cdq
004D3B86   |idiv dword ptr ss:\[ebp-8\]
004D3B89   |pop eax
004D3B8A   |add eax,edx
004D3B8C   |push eax
004D3B8D   |mov eax,ebx
004D3B8F   |cdq
004D3B90   |idiv dword ptr ss:\[ebp-8\]
004D3B93   |mov eax,dword ptr ds:\[esi\]
004D3B95   |mov al,byte ptr ds:\[eax+edx\]
004D3B98   |mov edx,dword ptr ds:\[esi\]
004D3B9A   |xor al,byte ptr ds:\[edx+ebx-1\]
004D3B9E   |pop edx
004D3B9F   |mov byte ptr ds:\[edx\],al
004D3BA1   |inc ebx
004D3BA2   |dec edi
004D3BA3   \\jnz short 004D3B7B
004D3BA5   lea ecx,dword ptr ss:\[ebp-18\]
004D3BA8   mov eax,dword ptr ds:\[esi\]
004D3BAA   mov edx,dword ptr ss:\[ebp-8\]
004D3BAD   call 004D3828
004D3BB2   mov edx,dword ptr ss:\[ebp-18\]
004D3BB5   mov eax,esi
004D3BB7   call <System::\_\_linkproc\_\_ LStrAsg(void \*,void \*)>
004D3BBC   mov edi,dword ptr ss:\[ebp-8\]
004D3BBF   test edi,edi
004D3BC1   jle short 004D3BEC
004D3BC3   mov ebx,1
004D3BC8   /mov eax,esi
004D3BCA   |call <j\_unknown\_libname\_79\_0>
004D3BCF   |mov edx,dword ptr ds:\[esi\]
004D3BD1   |mov dl,byte ptr ds:\[edx+ebx-1\]
004D3BD5   |and dl,3F
004D3BD8   |and edx,0FF
004D3BDE   |mov dl,byte ptr ds:\[edx+5B2B18\]
004D3BE4   |mov byte ptr ds:\[eax+ebx-1\],dl
004D3BE8   |inc ebx
004D3BE9   |dec edi
004D3BEA   \\jnz short 004D3BC8
004D3BEC   xor eax,eax
004D3BEE   pop edx
004D3BEF   pop ecx
004D3BF0   pop ecx
004D3BF1   mov dword ptr fs:\[eax\],edx
004D3BF4   push 004D3C16
004D3BF9   lea eax,dword ptr ss:\[ebp-18\]
004D3BFC   mov edx,3
004D3C01   call <System::\_\_linkproc\_\_ LStrArrayClr(void \*,int>
004D3C06   lea eax,dword ptr ss:\[ebp-4\]
004D3C09   call <System::\_\_linkproc\_\_ LStrClr(void \*)>
004D3C0E   retn
004D3C0F   jmp <unknown\_libname\_62>
004D3C14   jmp short 004D3BF9
004D3C16   pop edi
004D3C17   pop esi
004D3C18   pop ebx
004D3C19   mov esp,ebp
004D3C1B   pop ebp
004D3C1C   retn
```

算法函数：

代码:

```
void GetSerial(OUT LPSTR pszSerial, IN LPSTR pszName, IN LPSTR pszKeyCode)
{
  char  sz\[144\];
  UCHAR  data\[144\];
  LPCSTR  table = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxzMOZ";
  int    val = 0x3FF1;
  int    len = sprintf(sz, "%s%s%s", pszName, pszKeyCode, "gdi32.dll");
  int    i, j;

  if (len < 19)
  {
    for (i=len; i<19; i++)
      sz\[i\] = ' ';
    sz\[19\] = '\\0';
    len = 19;
  }

  for (i=0; i<len; i++)
  {
    data\[i\] = sz\[i\] ^ ((UINT)val >> 8);
    val = (val + data\[i\]) * 0xCE6D + 0x58BF;
  }

  for (i=0, j=19; i<(len - 19 + 1); i++, j++)
    data\[j % 19\] ^= data\[j - 1\];

  for (i=0; i<19; i++)
  {
    sz\[i\] = table\[data\[i\] % 64\];
  }

  sz\[19\] = '\\0';

  strcpy(pszSerial, sz);
}
```

注册信息：

\---------------------------------------------------------------------------------

Windows Registry Editor Version 5.00

\[HKEY\_CURRENT\_USER\\Software\\PocketMicroTechnics\\GlcdFontCreator\]

"USER"="rocktx"

"ORGANISATION"="..."

"KEYCODE"="1234567890"

"SERIAL"="FUeXVjsBtTs4OjbSgz2"

\---------------------------------------------------------------------------------

如有疏漏，望请海涵。。。

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `FCKeygen.zip` |
|:------------------------------------------:|:--------------:|
|                                          |                |
