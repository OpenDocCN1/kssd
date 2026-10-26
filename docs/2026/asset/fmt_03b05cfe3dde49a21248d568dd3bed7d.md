
# [旧帖][原创]第一次写破解文章0.00雪花

我是个菜菜鸟，我第一次写破解文章，所以有很多说不清楚的地方，还希望高手指正.同时想得到邀请码 .能让我学习方便。

`WinASO Registry Optimizer`是一Windows优化工具和高级注册表清理工具。它可以使大家以简单的鼠标来安全的清理及修复注册表故障。不介绍这么多，这是简单介绍这个软件。

用到的东东：

破解软件：`WinASO Registry Optimizer 3.2`

工具：`PEid` , `OllyICE`

我们开始：

1. 用`PEid`检查，结果显示软件使用`Borland Delphi`编写，没壳。这下方便了，不用脱壳。
2. 这个软件以弹出注册提示窗口，收集信息。
3. 用`OD`载入这软件，同时在`OD`的命令行中输入: "`bp MessageBoxA`",在`OD`按`F9`使软件运行。在弹出的窗口中任意输入一些注册码。按“Register"按钮，程序中断下来；
4. 这时候我们在`OD`右下角的堆栈中给我们提示，代码以下

```
012F7BC   0056A7B2  /CALL 到 MessageBoxA 来自 RegOpt.0056A7AD
0012F7C0   00080754  |hOwner = 00080754 ('Register Information',class='TfrmRegister',parent=000907FE)
0012F7C4   01F3F0B8  |Text = "Sorry, that is an invalid license key. Please ensure you have entered the license key exactly as provided."
0012F7C8   01F9EDB8  |Title = "Information"
0012F7CC   00000040  \Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
```

5.我们从上图中这句代码： `0012F7BC   0056A7B2  /CALL 到 MessageBoxA 来自RegOpt.0056A7AD`

知，关键点在`0056A7AD`这个地址。在`OD`中按`Ctrl+G`输入这地址。确定后到我们指定的地方。如下

```
0056A7A4  |.  50            push    eax
0056A7A5  |.  8BC3          mov     eax, ebx
0056A7A7  |.  E8 7040EFFF   call    0045E81C
0056A7AC  |.  50            push    eax                              ; |hOwner
0056A7AD  |.  E8 8ADBE9FF   call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA
0056A7B2  |.  E9 C1010000   jmp     0056A978
0056A7B7  |>  8B75 F8       mov     esi, dword ptr [ebp-8]
0056A7BA  |.  81C6 01020000 add     esi, 201
0056A7C0  |.  0FAFF7        imul    esi, edi
```

6. 通用，算法的核心部分就上面，我们逆上找，在`0056A3A9`这个地址我下断的地方,代码如下:

我们按`F8`单步调试,如下

```
0056A389      00            db      00
0056A38A      00            db      00
0056A38B      00            db      00
0056A38C  /.  55            push    ebp
0056A38D  |.  8BEC          mov     ebp, esp
0056A38F  |.  B9 0E000000   mov     ecx, 0E
0056A394  |>  6A 00         /push    0
0056A396  |.  6A 00         |push    0
0056A398  |.  49            |dec     ecx
0056A399  |.^ 75 F9         \jnz     short 0056A394
0056A39B  |.  51            push    ecx
0056A39C  |.  53            push    ebx
0056A39D  |.  56            push    esi
0056A39E  |.  57            push    edi
0056A39F  |.  8BD8          mov     ebx, eax
0056A3A1  |.  33C0          xor     eax, eax
0056A3A3  |.  55            push    ebp
0056A3A4  |.  68 E4A95600   push    0056A9E4
0056A3A9  |.  64:FF30       push    dword ptr fs:[eax]       ;这是我下断点地方
0056A3AC  |.  64:8920       mov     dword ptr fs:[eax], esp
0056A3AF  |.  8D45 FC       lea     eax, dword ptr [ebp-4]
0056A3B2  |.  8B15 84055A00 mov     edx, dword ptr [5A0584]   ;  RegOpt.005A55E0
0056A3B8 >|.  8B92 70080000 mov     edx, dword ptr [edx+870]  ;  加载错误注册码后，提示错误信息
0056A3BE  |.  E8 6DAEE9FF   call    00405230
0056A3C3  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
0056A3C6  |.  8B83 A4030000 mov     eax, dword ptr [ebx+3A4]
0056A3CC  |.  E8 4BC9EEFF   call    00456D1C                  ;  取输入第一组注册码
0056A3D1  |.  8B45 F4       mov     eax, dword ptr [ebp-C]    ;  将输入第一组注册码放在EAX中
0056A3D4  |.  E8 9B45F6FF   call    004CE974                  ;  检查第一组注册码中是不是都是数字,都是数字,返回al=1
0056A3D9  |.  84C0          test    al, al                    ；
0056A3DB  |.  75 2E         jnz     short 0056A40B            ;  都是数字，继续验证
0056A3DD  |.  6A 40         push    40
0056A3DF  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
0056A3E4  |.  8B80 6C080000 mov     eax, dword ptr [eax+86C]
0056A3EA  |.  E8 6DB2E9FF   call    0040565C
0056A3EF  |.  50            push    eax
0056A3F0  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056A3F3  |.  E8 64B2E9FF   call    0040565C
0056A3F8  |.  50            push    eax
0056A3F9  |.  8BC3          mov     eax, ebx
0056A3FB  |.  E8 1C44EFFF   call    0045E81C
0056A400  |.  50            push    eax                       ; |hOwner
0056A401  |.  E8 36DFE9FF   call    <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A406  |.  E9 6D050000   jmp     0056A978
0056A40B  |>  8D55 F0       lea     edx, dword ptr [ebp-10]
0056A40E  |.  8B83 A8030000 mov     eax, dword ptr [ebx+3A8]
0056A414  |.  E8 03C9EEFF   call    00456D1C                  ;  取输入第二组注册码
0056A419  |.  8B45 F0       mov     eax, dword ptr [ebp-10]   ;  将输入第二组注册码放在EAX中
0056A41C  |.  E8 5345F6FF   call    004CE974                  ;  检查第二组注册码中是不是都是数字,都是数字,返回al=1
0056A421  |.  84C0          test    al, al                    ;
0056A423  |.  75 2E         jnz     short 0056A453            ； 都是数字，继续验证
0056A425  |.  6A 40         push    40
0056A427  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
0056A42C  |.  8B80 6C080000 mov     eax, dword ptr [eax+86C]
0056A432  |.  E8 25B2E9FF   call    0040565C
0056A437  |.  50            push    eax
0056A438  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056A43B  |.  E8 1CB2E9FF   call    0040565C
0056A440  |.  50            push    eax
0056A441  |.  8BC3          mov     eax, ebx
0056A443  |.  E8 D443EFFF   call    0045E81C
0056A448  |.  50            push    eax                       ; |hOwner
0056A449  |.  E8 EEDEE9FF   call    <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A44E  |.  E9 25050000   jmp     0056A978
0056A453  |>  8D55 EC       lea     edx, dword ptr [ebp-14]
0056A456  |.  8B83 AC030000 mov     eax, dword ptr [ebx+3AC]
0056A45C  |.  E8 BBC8EEFF   call    00456D1C                  ;  取输入第三组注册码
0056A461  |.  8B45 EC       mov     eax, dword ptr [ebp-14]
0056A464  |.  E8 0B45F6FF   call    004CE974                  ;  检查第三组注册码中是不是都是数字
0056A469  |.  84C0          test    al, al
0056A46B  |.  75 2E         jnz     short 0056A49B            ;  都是数字，继续验证
0056A46D  |.  6A 40         push    40
0056A46F  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
0056A474  |.  8B80 6C080000 mov     eax, dword ptr [eax+86C]
0056A47A  |.  E8 DDB1E9FF   call    0040565C
0056A47F  |.  50            push    eax
0056A480  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056A483  |.  E8 D4B1E9FF   call    0040565C
0056A488  |.  50            push    eax
0056A489  |.  8BC3          mov     eax, ebx
0056A48B  |.  E8 8C43EFFF   call    0045E81C
0056A490  |.  50            push    eax                       ; |hOwner
0056A491  |.  E8 A6DEE9FF   call    <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A496  |.  E9 DD040000   jmp     0056A978
0056A49B  |>  8D55 E8       lea     edx, dword ptr [ebp-18]
0056A49E  |.  8B83 B0030000 mov     eax, dword ptr [ebx+3B0]
0056A4A4  |.  E8 73C8EEFF   call    00456D1C                  ;  取输入第4组注册码
0056A4A9  |.  8B45 E8       mov     eax, dword ptr [ebp-18]
0056A4AC  |.  E8 C344F6FF   call    004CE974                  ;  检查第4组注册码中是不是都是数字
0056A4B1  |.  84C0          test    al, al
0056A4B3  |.  75 2E         jnz     short 0056A4E3            ;  都是数字，继续验证
0056A4B5  |.  6A 40         push    40
0056A4B7  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
0056A4BC  |.  8B80 6C080000 mov     eax, dword ptr [eax+86C]
0056A4C2  |.  E8 95B1E9FF   call    0040565C
0056A4C7  |.  50            push    eax
0056A4C8  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056A4CB  |.  E8 8CB1E9FF   call    0040565C
0056A4D0  |.  50            push    eax
0056A4D1  |.  8BC3          mov     eax, ebx
0056A4D3  |.  E8 4443EFFF   call    0045E81C
0056A4D8  |.  50            push    eax                       ; |hOwner
0056A4D9  |.  E8 5EDEE9FF   call    <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A4DE  |.  E9 95040000   jmp     0056A978
0056A4E3  |>  8D55 E4       lea     edx, dword ptr [ebp-1C]
```
