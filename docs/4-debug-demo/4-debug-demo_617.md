

# W32DASM v8.93 的小小改动

> 原文：[`www.pediy.com/kssd/pediy03/forum572.htm`](https://www.pediy.com/kssd/pediy03/forum572.htm)

对 W32DASM v8.93 的小小改动    
作者：TiANWEi   
仅以此篇献给喜用 W32dasm 的 Cai Jie 兄    
  
W32dasm 是个很好的反汇编工具,不用多说,   
但它有个好臭屁的毛病,就是每次启动都要   
在它的背景上贴上那个深绿色的图片,告诉   
你这是 W32DASM 的 XXX 版本,是怎么怎么样的.   
(我用的 8.93 正式版)   
那么我们来搞掉它吧!   
  
用 SICE 的 SYMLOADER 装上 USER32.DLL,   
KERNEL32.DLL,GDI32.DLL,然后   
:bpx bitblt   
因为这个是标准的 WINDOWS 用来显示 BITMAP 的   
函数,一直没变过.   
程序会在启动过程中被 SICE 拦截好多次,最后   
注意有一次用 F5 带过后就会显示出来那幅图片,   
那么可以从头来,到那时用 F12 来带.   
会发现程序停在此处:   
  
\* Reference To: GDI32.BitBlt, Ord:0000h   
                                  |   
:00478B34 E805620300              Call 004AED3E   
:00478B39 FF75E0                  push \[ebp\-20\]    ;\*\*\*此处\*\*\*   
:00478B3C FF75E4                  push \[ebp\-1C\]   
  
原始的想法可能是改成让 BITBLT 不显示图片,将此 CALL   
略过,但仔细往上看代码,会发现有更好的办法:   
(以下是用 W32DASM 反汇编自己的结果)   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00478A54(U)   
|   
:00478A58 52                      push edx   
:00478A59 53                      push ebx   
:00478A5A E8CAF20200              call 004A7D29   
:00478A5F 83C414                  add esp, 00000014   
:00478A62 8BD8                    mov ebx, eax   
:00478A64 895B77                  mov dword ptr \[ebx+77\], ebx   
:00478A67 8B8350126F00            mov eax, dword ptr \[ebx+006F1250\]   
:00478A6D 85C0                    test eax, eax        ;\*\*\*注意!!\*\*\*   
:00478A6F 0F85DF000000            jne 00478B54          ;\*\*\*\*\*\*\*\*\*\*\*\*    
:00478A75 C745C84D010000          mov \[ebp\-38\], 0000014D   
:00478A7C FF75C8                  push \[ebp\-38\]   
:00478A7F FF730C                  push \[ebx+0C\]   
  
\* Reference To: USER32.LoadBitmapA, Ord:0000h   
                                  |   
:00478A82 E83D670300              Call 004AF1C4   
:00478A87 8945DC                  mov dword ptr \[ebp\-24\], eax   
:00478A8A FF37                    push dword ptr \[edi\]   
  
\* Reference To: GDI32.CreateCompatibleDC, Ord:0000h   
                                  |   
:00478A8C E883620300              Call 004AED14   
:00478A91 8945E4                  mov dword ptr \[ebp\-1C\], eax   
:00478A94 8D8DD8FCFFFF            lea ecx, dword ptr \[ebp+FFFFFCD8\]   
:00478A9A 51                      push ecx   
:00478A9B FFB38C116F00            push dword ptr \[ebx+006F118C\]   
:00478AA1 E8C59E0100              call 0049296B   
:00478AA6 83C408                  add esp, 00000008   
:00478AA9 56                      push esi   
:00478AAA 57                      push edi   
:00478AAB 8DB5D8FCFFFF            lea esi, dword ptr \[ebp+FFFFFCD8\]   
:00478AB1 8DBDC8FCFFFF            lea edi, dword ptr \[ebp+FFFFFCC8\]   
:00478AB7 B904000000              mov ecx, 00000004   
:00478ABC F3                      repz   
:00478ABD A5                      movsd   
:00478ABE 5F                      pop edi   
:00478ABF 5E                      pop esi   
:00478AC0 56                      push esi   
:00478AC1 57                      push edi   
:00478AC2 8DB5C8FCFFFF            lea esi, dword ptr \[ebp+FFFFFCC8\]   
:00478AC8 8DBDE8FCFFFF            lea edi, dword ptr \[ebp+FFFFFCE8\]   
:00478ACE B904000000              mov ecx, 00000004   
:00478AD3 F3                      repz   
:00478AD4 A5                      movsd   
:00478AD5 5F                      pop edi   
:00478AD6 5E                      pop esi   
:00478AD7 FF75DC                  push \[ebp\-24\]   
:00478ADA FF75E4                  push \[ebp\-1C\]   
  
\* Reference To: GDI32.SelectObject, Ord:0000h   
                                  |   
:00478ADD E8F2620300              Call 004AEDD4   
:00478AE2 8945E0                  mov dword ptr \[ebp\-20\], eax   
:00478AE5 8B85F0FCFFFF            mov eax, dword ptr \[ebp+FFFFFCF0\]   
:00478AEB 8B95E8FCFFFF            mov edx, dword ptr \[ebp+FFFFFCE8\]   
:00478AF1 2BC2                    sub eax, edx   
:00478AF3 05C0FEFFFF              add eax, FFFFFEC0   
:00478AF8 D1F8                    sar eax, 1   
:00478AFA 7903                    jns 00478AFF   
:00478AFC 83D000                  adc eax, 00000000   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00478AFA(C)   
|   
:00478AFF 8B95F4FCFFFF            mov edx, dword ptr \[ebp+FFFFFCF4\]   
:00478B05 8B8DECFCFFFF            mov ecx, dword ptr \[ebp+FFFFFCEC\]   
:00478B0B 2BD1                    sub edx, ecx   
:00478B0D 81C210FFFFFF            add edx, FFFFFF10   
:00478B13 D1FA                    sar edx, 1   
:00478B15 7903                    jns 00478B1A   
:00478B17 83D200                  adc edx, 00000000   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00478B15(C)   
|   
:00478B1A 682000CC00              push 00CC0020   
:00478B1F 6A00                    push 00000000   
:00478B21 6A00                    push 00000000   
:00478B23 FF75E4                  push \[ebp\-1C\]   
:00478B26 68F0000000              push 000000F0   
:00478B2B 6840010000              push 00000140   
:00478B30 52                      push edx   
:00478B31 50                      push eax   
:00478B32 FF37                    push dword ptr \[edi\]   
  
\* Reference To: GDI32.BitBlt, Ord:0000h   
                                  |   
:00478B34 E805620300              Call 004AED3E   
:00478B39 FF75E0                  push \[ebp\-20\]   
:00478B3C FF75E4                  push \[ebp\-1C\]   
  
\* Reference To: GDI32.SelectObject, Ord:0000h   
                                  |   
:00478B3F E890620300              Call 004AEDD4   
:00478B44 FF75DC                  push \[ebp\-24\]   
  
\* Reference To: GDI32.DeleteObject, Ord:0000h   
                                  |   
:00478B47 E878630300              Call 004AEEC4   
:00478B4C FF75E4                  push \[ebp\-1C\]   
  
\* Reference To: GDI32.DeleteDC, Ord:0000h   
                                  |   
:00478B4F E89A630300              Call 004AEEEE   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00478A6F(C)   
|   
:00478B54 85DB                    test ebx, ebx   
:00478B56 0F84050A0000            je 00479561   
:00478B5C 8B8393000000            mov eax, dword ptr \[ebx+00000093\]   
:00478B62 8B4008                  mov eax, dword ptr \[eax+08\]   
:00478B65 8945C4                  mov dword ptr \[ebp\-3C\], eax   
:00478B68 8B9397000000            mov edx, dword ptr \[ebx+00000097\]   
:00478B6E 85D2                    test edx, edx   
:00478B70 0F840B020000            je 00478D81   
  
  
以上的代码是 windows 程序标准的显示一个 BITMAP 的过程,   
先 LOAD,再 SELECT,再 BITBLT,最后 RELEASE   
在这之前会有一个判断,注意我加\*号的地方,这个判断实际   
上是看你现在是否正在显示程序原代码,如果 W32DASM 已经反   
汇编了一个程序,背景不可能再显示 BITMAP 了.   
所以,将下面的那个 JNE 改为 JMP 就行了!   
自己会改吧,很容易的!   
改过后,啊,天空一片纯净,就象<<大话西游>>中齐天大圣将   
那个爱唠叨的唐僧一棍子打死过后,世界顿时安静下来了...  4dd466d0e1e0cc123afbe504da2462ca