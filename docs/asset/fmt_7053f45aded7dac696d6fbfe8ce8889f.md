
```
  0040F381  |.  BA 0A000000   mov     edx, 0A  
  0040F386  |.  E8 814DFFFF   call    0040410C  
  0040F38B  \.  C3           retn  
```

这里很奇怪调用了 `winexe` 来运行 `screenshot2email.dll`，我们知道 `dll` 文件不可能单独运行，这里就十分古怪

一看 `screenshot2email.dll` 近 600KB 的体积比 `exe` 文件大多了，猜想这个 `exe` 像个 `loder` 来加载实际为可执行文件的 `dll`

`screenshot2email.dll` 改名为 `screenshot2email.dll.exe` 果然可以顺利运行

`peid` 查看 `screenshot2email.dll`，也是 `Delphi` 写的东西，用 `KANAL` 插件一看

`MD5 :: 0006DFAB :: 0046EBAB`

原来是用到 `MD5` 加密算法

`dede` 加载 `screenshot2email.dll.exe`，在过程里面的 `uRegistration` 里面的 `btnOKClick` 找到地址 `0046F744`

找到这里我们就成功一半了

`od` 里面 `ctrl+g` 填入 `0046F744`，`F2` 下内存断点，`F9` 运行

填入注册名 `xss517` 和假的注册码 `123456`，确定，稳稳的中断在 `0046F744`,`F8` 大致跟一下

```
  0046F744  /.  55            push    ebp  
  0046F745  |.  8BEC         mov     ebp, esp  
  0046F747  |.  B9 06000000  mov     ecx, 6  
  0046F74C  |>  6A 00        /push   0  
  0046F74E  |.  6A 00        |push   0  
  0046F750  |.  49           |dec    ecx  
  0046F751  |.^ 75 F9        \jnz    short 0046F74C  
  0046F753  |.  53          push    ebx  
  0046F754  |.  56          push    esi  
  0046F755  |.  8BD8        mov     ebx, eax  
  0046F757  |.  33C0        xor     eax, eax  
  0046F759  |.  55          push    ebp  
  0046F75A  |.  68 18F94600 push    0046F918  
  0046F75F  |.  64:FF30     push    dword ptr fs:[eax]  
  0046F762  |.  64:8920     mov     dword ptr fs:[eax], esp  
  0046F765  |.  8D55 F4     lea     edx, dword ptr [ebp-C]  
  0046F768  |.  8B83 68030000 mov   eax, dword ptr [ebx+368]  
  0046F76E  |.  E8 C599FDFF call    00449138  
  0046F773  |.  8B45 F4     mov     eax, dword ptr [ebp-C]  
  0046F776  |.  8D55 FC     lea     edx, dword ptr [ebp-4]       ;  eax=注册名 xss517  
  0046F779  |.  E8 36FCFFFF call    0046F3B4  
  0046F77E  |.  8D45 F8     lea     eax, dword ptr [ebp-8]  
  0046F781  |.  8B55 FC     mov     edx, dword ptr [ebp-4]       ;  edx=MD5(注册名） F5D8D5D774391CA908D4E0A46E8258EE  
  0046F784  |.  E8 BF51F9FF call    00404948  
  0046F789  |.  8D4D F0     lea     ecx, dword ptr [ebp-10]  
  0046F78C  |.  A1 1CAE4700 mov     eax, dword ptr [47AE1C]  
  0046F791  |.  8B00        mov     eax, dword ptr [eax]  
  0046F793  |.  8B55 F8     mov     edx, dword ptr [ebp-8]  
  0046F796  |.  E8 7D760000 call    00476E18                ;  算法，对 MD5 结果进行转换  
  0046F79B  |.  8B55 F0     mov     edx, dword ptr [ebp-10]  
  0046F79E  |.  8D45 F8     lea     eax, dword ptr [ebp-8]  
  0046F7A1  |.  E8 A251F9FF call    00404948  
  0046F7A6  |.  8D55 EC     lea     edx, dword ptr [ebp-14]  
  0046F7A9  |.  8B83 6C030000 mov   eax, dword ptr [ebx+36C]  
  0046F7AF  |.  E8 8499FDFF call    00449138  
  0046F7B4  |.  8B45 EC     mov     eax, dword ptr [ebp-14]    ;  真注册码   FPBD5JHFDMDLJHE7  
  0046F7B7  |.  8B55 F8     mov     edx, dword ptr [ebp-8]     ;  假注册码   123456  
  0046F7BA  |.  E8 0155F9FF call    00404CC0  
  0046F7BF  |.  0F85 EF000000 jnz    0046F8B4                ;  关键跳  
  0046F7C5  |.  B8 90000000 mov     eax, 90  
  0046F7CA  |.  E8 D934F9FF call    00402CA8  
  0046F7CF  |.  8BF0        mov     esi, eax  
  0046F7D1  |.  68 90000000 push    90                        ; /BufSize = 90 (144.)  
  0046F7D6  |.  56          push    esi                         ; |Buffer  
  0046F7D7  |.  E8 6C72F9FF call    <jmp.&kernel32.GetWindowsDirecto>; \GetWindowsDirectoryA  
  0046F7DC  |.  8D45 E0     lea     eax, dword ptr [ebp-20]  
  0046F7DF  |.  8BD6        mov     edx, esi  
  0046F7E1  |.  E8 C252F9FF call    00404AA8  
  0046F7E6  |.  8B45 E0     mov     eax, dword ptr [ebp-20]  
  0046F7E9  |.  8D55 E4     lea     edx, dword ptr [ebp-1C]  
  0046F7EC  |.  E8 5F86FFFF call    00467E50  
  0046F7F1  |.  8B45 E4     mov     eax, dword ptr [ebp-1C]  
  0046F7F4  |.  8D55 E8     lea     edx, dword ptr [ebp-18]  
  0046F7F7  |.  E8 9CFEFFFF call    0046F698  
  0046F7FC  |.  8B4D E8     mov     ecx, dword ptr [ebp-18]  
  0046F7FF  |.  8D45 F8     lea     eax, dword ptr [ebp-8]  
  0046F802  |.  8B55 FC     mov     edx, dword ptr [ebp-4]  
  0046F805  |.  E8 B653F9FF call    00404BC0  
  0046F80A  |.  8D4D DC     lea     ecx, dword ptr [ebp-24]  
  0046F80D  |.  A1 1CAE4700 mov     eax, dword ptr [47AE1C]  
  0046F812  |.  8B00        mov     eax, dword ptr [eax]  
  0046F814  |.  8B55 F8     mov     edx, dword ptr [ebp-8]  
  0046F817  |.  E8 FC750000 call    00476E18  
  0046F81C  |.  8B55 DC     mov     edx, dword ptr [ebp-24]  
  0046F81F  |.  8D45 F8     lea     eax, dword ptr [ebp-8]  
  0046F822  |.  E8 2151F9FF call    00404948  
  0046F827  |.  8B55 F8     mov     edx, dword ptr [ebp-8]  
  0046F82A  |.  B8 30F94600 mov     eax, 0046F930             ;  ASCII "Value1"  
  0046F82F  |.  E8 F87F0000 call    0047782C  
  0046F834  |.  8D55 D8     lea     edx, dword ptr [ebp-28]  
  0046F837  |.  8B83 68030000 mov   eax, dword ptr [ebx+368]  
  0046F83D  |.  E8 F698FDFF call    00449138  
  0046F842  |.  8B55 D8     mov     edx, dword ptr [ebp-28]  
  0046F845  |.  B8 40F94600 mov     eax, 0046F940             ;  ASCII "Value2"  
  0046F84A  |.  E8 DD7F0000 call    0047782C  
  0046F84F  |.  A1 14AB4700 mov     eax, dword ptr [47AB14]  
  0046F854  |.  FF30        push    dword ptr [eax]  
  0046F856  |.  68 50F94600 push    0046F950  
  0046F85B  |.  8D55 D0     lea     edx, dword ptr [ebp-30]  
  0046F85E  |.  8B83 68030000 mov   eax, dword ptr [ebx+368]  
  0046F864  |.  E8 CF98FDFF call    00449138  
  0046F869  |.  FF75 D0     push    dword ptr [ebp-30]  
  0046F86C  |.  8D45 D4     lea     eax, dword ptr [ebp-2C]  
  0046F86F  |.  BA 03000000 mov     edx, 3  
  0046F874  |.  E8 BB53F9FF call    00404C34  
  0046F879  |.  8B55 D4     mov     edx, dword ptr [ebp-2C]  
  0046F87C  |.  A1 1CAE4700 mov     eax, dword ptr [47AE1C]  
  0046F881  |.  8B00        mov     eax, dword ptr [eax]  
  0046F883  |.  8B80 80040000 mov   eax, dword ptr [eax+480]  
  0046F889  |.  E8 DA98FDFF call    00449168  
  0046F88E  |.  A1 1CAE4700 mov     eax, dword ptr [47AE1C]  
  0046F893  |.  8B00        mov     eax, dword ptr [eax]  
  0046F895  |.  C680 59050000>mov    byte ptr [eax+559], 0  
  0046F89C  |.  A1 C8AA4700 mov     eax, dword ptr [47AAC8]  
  0046F8A1  |.  8B00        mov     eax, dword ptr [eax]  
  0046F8A3  |.  E8 74B9FCFF call    0043B21C  
  0046F8A8  |.  C783 94020000>mov    dword ptr [ebx+294], 1  
  0046F8B2  |.  EB 0C       jmp     short 0046F8C0  
  0046F8B4  |>  A1 BCAB4700 mov     eax, dword ptr [47ABBC]  
  0046F8B9  |.  8B00        mov     eax, dword ptr [eax]  
  0046F8BB  |.  E8 5CB9FCFF call    0043B21C  
  0046F8C0  |>  33C0        xor     eax, eax  
  0046F8C2  |.  5A          pop     edx  
  0046F8C3  |.  59          pop     ecx  
  0046F8C4  |.  59          pop     ecx  
  0046F8C5  |.  64:8910     mov     dword ptr fs:[eax], edx  
  0046F8C8  |.  68 1FF94600 push    0046F91F  
  0046F8CD  |>  8D45 D0     lea     eax, dword ptr [ebp-30]  
  0046F8D0  |.  E8 DB4FF9FF call    004048B0  
  0046F8D5  |.  8D45 D4     lea     eax, dword ptr [ebp-2C]  
  0046F8D8  |.  E8 D34FF9FF call    004048B0  
  0046F8DD  |.  8D45 D8     lea     eax, dword ptr [ebp-28]  
  0046F8E0  |.  E8 CB4FF9FF call    004048B0  
  0046F8E5  |.  8D45 DC     lea     eax, dword ptr [ebp-24]  
  0046F8E8  |.  BA 04000000 mov     edx, 4  
```
