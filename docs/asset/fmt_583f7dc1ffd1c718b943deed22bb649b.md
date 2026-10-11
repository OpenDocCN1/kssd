
```
10600A37  |.  F3:A4                ||rep     movs byte ptr es:\[edi\], byte p>
10600A39  |.  8B7C24 10            ||mov     edi, dword ptr \[esp+10\]
10600A3D  |.  8B4C24 14            ||mov     ecx, dword ptr \[esp+14\]
10600A41  |.  83C7 28              ||add     edi, 28
10600A44  |.  83C0 04              ||add     eax, 4
10600A47  |.  83C2 28              ||add     edx, 28
10600A4A  |.  49                   ||dec     ecx
10600A4B  |.  897C24 10            ||mov     dword ptr \[esp+10\], edi
10600A4F  |.  894C24 14            ||mov     dword ptr \[esp+14\], ecx
10600A53  |.^ 75 D2                |\jnz     short 10600A27
10600A55  |.  8B4C24 18            |mov     ecx, dword ptr \[esp+18\]
10600A59  |.  81C3 90000000        |add     ebx, 90
10600A5F  |.  49                   |dec     ecx
10600A60  |.  894C24 18            |mov     dword ptr \[esp+18\], ecx
10600A64  |.^ 75 88                |\jnz     short 106009EE
10600A66  |.  8B5C24 20            mov     ebx, dword ptr \[esp+20\]
10600A6A  |.  5D                   pop     ebp
10600A6B  |>   8D43 04             lea     eax, dword ptr \[ebx+4\]
10600A6E  |.  50                   push    eax                       ; /Arg4
10600A6F  |.  8D43 08             lea     eax, dword ptr \[ebx+8\]         ; |
10600A72  |.  50                   push    eax                       ; |Arg3
10600A73  |.  68 1CE07210         push    1072E01C                  ; |Arg2 = 1072E01C ASCII "nilm"
10600A78  |.  6A 00               push    0                           ; |Arg1 = 00000000
10600A7A  |.  E8 C0640100         call    10616F3F                  ; \cvi.10616F3F
10600A7F  |.  83C4 10             add     esp, 10
10600A82  |.  5F                   pop     edi
10600A83  |.  5E                   pop     esi
10600A84  |.  8BC3                 mov     eax, ebx
10600A86  |.  5B                   pop     ebx
10600A87  |.  83C4 14             add     esp, 14
10600A8A  \\.  C3                  retn
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*这段代码属于 `cvi.dll`，不是 `flexlm` 的函数，不具有通用性，\*\*\*\*\*\*\*\*\*\*\*\*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
106267A1  |> \\83BD 7CFEFFFF 00    |cmp     dword ptr \[ebp-184\], 0
106267A8  |.  0F85 50040000        |jnz     10626BFE
106267AE  |.  83BD 90FEFFFF 00     |cmp     dword ptr \[ebp-170\], 0
106267B5  |.  0F85 47030000        |jnz     10626B02
106267BB  |.  837D FC 00           |cmp     dword ptr \[ebp-4\], 0
106267BF  |.  0F85 3D030000        |jnz     10626B02
106267C5  |.  8B55 08             |mov     edx, dword ptr \[ebp+8\]
106267C8  |.  8B82 50030000       |mov     eax, dword ptr \[edx+350\]
106267CE  |.  25 00001000         |and     eax, 100000       ;特征码从这儿开始
106267D3  |.  85C0                |test    eax, eax          ;250000100085C00F84
106267D5  |.  0F84 27030000       |je      10626B02                          ;修改 jne  ,0f85
106267DB  |.  C785 4CFEFFFF 4874>|mov     dword ptr \[ebp-1B4\], 10937448
106267E5  |.  8B4D F8             |mov     ecx, dword ptr \[ebp-8\]
106267E8  |.  C1E1 04             |shl     ecx, 4
106267EB  |.  8B940D B4FEFFFF      |mov     edx, dword ptr \[ebp+ecx-14C\]
106267F2  |.  8B45 08             |mov     eax, dword ptr \[ebp+8\]
106267F5  |.  33C9                |xor     ecx, ecx
106267F7  |.  8A4C10 08           |mov     cl, byte ptr \[eax+edx+8\]
106267FB  |.  8B55 14             |mov     edx, dword ptr \[ebp+14\]
106267FE  |.  8B42 04             |mov     eax, dword ptr \[edx+4\]
10626801  |.  33C1                |xor     eax, ecx
10626803  |.  8B4D F8             |mov     ecx, dword ptr \[ebp-8\]
10626806  |.  C1E1 04             |shl     ecx, 4
10626809  |.  8B940D B8FEFFFF      |mov     edx, dword ptr \[ebp+ecx-148\]
10626810  |.  8B4D 08             |mov     ecx, dword ptr \[ebp+8\]
10626813  |.  33DB                |xor     ebx, ebx
10626815  |.  8A5C11 08           |mov     bl, byte ptr \[ecx+edx+8\]
10626819  |.  C1E3 08             |shl     ebx, 8
1062681C  |.  33C3                |xor     eax, ebx
1062681E  |.  8B55 F8             |mov     edx, dword ptr \[ebp-8\]
10626821  |.  C1E2 04             |shl     edx, 4
10626824  |.  8B8C15 BCFEFFFF      |mov     ecx, dword ptr \[ebp+edx-144\]
1062682B  |.  8B55 08             |mov     edx, dword ptr \[ebp+8\]
1062682E  |.  33DB                |xor     ebx, ebx
10626830  |.  8A5C0A 08           |mov     bl, byte ptr \[edx+ecx+8\]
10626834  |.  C1E3 10             |shl     ebx, 10
10626837  |.  33C3                |xor     eax, ebx
10626839  |.  8B4D F8             |mov     ecx, dword ptr \[ebp-8\]
1062683C  |.  C1E1 04             |shl     ecx, 4
1062683F  |.  8B940D C0FEFFFF      |mov     edx, dword ptr \[ebp+ecx-140\]
10626846  |.  8B4D 08             |mov     ecx, dword ptr \[ebp+8\]
10626849  |.  33DB                |xor     ebx, ebx
1062684B  |.  8A5C11 08           |mov     bl, byte ptr \[ecx+edx+8\]
1062684F  |.  C1E3 18             |shl     ebx, 18
10626852  |.  33C3                |xor     eax, ebx                  ;eax 即为 seed1
10626854  |.  50                   |push    eax                       ; /Arg1 = A81B372E
10626855  |.  E8 D0060000           |call    10626F2A                  ; \\cvi.10626F2A
1062685A  |.  83C4 04                |add     esp, 4
1062685D  |.  8985 48FEFFFF          |mov     dword ptr \[ebp-1B8\], eax
10626863  |.  8B95 48FEFFFF          |mov     edx, dword ptr \[ebp-1B8\]
10626869  |.  81E2 FF000000          |and     edx, 0FF
1062686F  |.  8B85 4CFEFFFF          |mov     eax, dword ptr \[ebp-1B4\]
10626875  |.  8A08                    |mov     cl, byte ptr \[eax\]
10626877  |.  32CA                    |xor     cl, dl
10626879  |.  8B95 4CFEFFFF          |mov     edx, dword ptr \[ebp-1B4\]
1062687F  |.  880A                    |mov     byte ptr \[edx\], cl
10626881  |.  8B85 4CFEFFFF          |mov     eax, dword ptr \[ebp-1B4\]
10626887  |.  83C0 01                |add     eax, 1
1062688A  |.  8985 4CFEFFFF          |mov     dword ptr \[ebp-1B4\], eax
10626890  |.  81BD 48FEFFFF FF000000   |cmp     dword ptr \[ebp-1B8\], 0FF
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

附件有一个脚本，直接出 `seed`，欢迎测试，

上传的附件

| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) || `Ollydbg_flexlm7.2_annother.txt`  
|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `lmcryptgui.rar`  
|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `nilm.rar`[论坛下载该文件：[`bbs.pediy.com/showthread.php?t=76340`](http://bbs.pediy.com/showthread.php?t=76340%20) ]  
|

没有想到还有人关注，

`nilm.exe` 是 ni 公司所有软件的 daemon，呵呵，这个里有 `seed1` 和 `seed2`，

`nilm` 已经上传了，仅有 `cvi.dll` 是无法执行的，最好到 ni 的网站上下一个安装文件，免费的，

调试 `nilm` 的方法有两种，

（1）`od` 打开，在调试参数里输入 `nilm -T 主机名 11.4 -1 -c "c:\flexlm\LabVIEW.lic"`

就可以里了，

（2）将 `nilm.exe` 的 `oep` 处的 `55` 修改成 `cc`，然后运行 `lmgrd -c "c:\flexlm\LabVIEW.lic"`，

让异常呼出 `od`，或者将 `nilm.exe` 的 `oep` 处的 `558b` 修改成 `ebfe`，然后运行 `lmgrd -c "c:\flexlm\LabVIEW.lic"`，打开 `od`，`attach` 之，就停在 `oep` 处了，改回即可。
