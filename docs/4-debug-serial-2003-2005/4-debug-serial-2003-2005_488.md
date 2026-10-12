
# 数据库大师 1.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6344.htm`](https://www.pediy.com/kssd/pediy06/pediy6344.htm)

用 `od`，是个明码比较。

先用 `peid` 查：无壳。`:)` 是 `delphi`

试运行。注册。注册错误时提示：密码错误。关闭。

用 `w32asm` 反汇编。

参考串中找到密码错误。双击，找到其在代码中的位置。

```
* Possible StringData Ref from Code Obj ->"警告"
                         |
:004E0BCE B9800C4E00             mov ecx, 004E0C80

* Possible StringData Ref from Code Obj ->"密码不正确！"
                         |
:004E0BD3 BA880C4E00             mov edx, 004E0C88     ***********这里
```

记住地址。

用 `dede` 反编译。找到注册窗体。找到"确定"键按下时的代码：

```
004E0ADC   55                  push    ebp  ***********这里,下断
004E0ADD   8BEC                mov     ebp, esp
004E0ADF   33C9                xor     ecx, ecx
004E0AE1   51                  push    ecx
004E0AE2   51                  push    ecx
004E0AE3   51                  push    ecx
004E0AE4   51                  push    ecx
```

用 `od` 加载。粗略的跟一遍，发现是明码比较。然后找关键 `call`（在哪里生成注册码，即上面的断点处）。

如果只要注册码而不求算法。就不用这么做。在堆栈中可以看到注册码。

`F9`，注册。

用户名：`asasasas`

试验码：`987654321`

注册。

停在断点处。

`F8`，慢行。

```
004E0ADC  /. 55               PUSH EBP
004E0ADD  |. 8BEC             MOV EBP,ESP
004E0ADF  |. 33C9             XOR ECX,ECX
004E0AE1  |. 51               PUSH ECX
004E0AE2  |. 51               PUSH ECX
004E0AE3  |. 51               PUSH ECX
004E0AE4  |. 51               PUSH ECX
004E0AE5  |. 56               PUSH ESI
004E0AE6  |. 8BF0             MOV ESI,EAX
004E0AE8  |. 33C0             XOR EAX,EAX
004E0AEA  |. 55               PUSH EBP
004E0AEB  |. 68 190C4E00      PUSH SJK.004E0C19
004E0AF0  |. 64:FF30          PUSH DWORD PTR FS:[EAX]
004E0AF3  |. 64:8920          MOV DWORD PTR FS:[EAX],ESP
004E0AF6  |. 8D55 FC          LEA EDX,DWORD PTR SS:[EBP-4]
004E0AF9  |. 8B86 DC020000    MOV EAX,DWORD PTR DS:[ESI+2DC]
004E0AFF  |. E8 6432F5FF      CALL SJK.00433D68***********读取用户名.F8 掠过时,用户名在堆栈出现
004E0B04  |. 8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
004E0B07  |. E8 5035F2FF      CALL SJK.0040405C  ***********计算用户名长度
004E0B0C  |. 83F8 04         CMP EAX,4  ***********与 4 比较
004E0B0F  |. 7D 39           JGE SHORT SJK.004E0B4A********大于,跳
004E0B11  |. 6A 40           PUSH 40
004E0B13  |. B9 280C4E00     MOV ECX,SJK.004E0C28
004E0B18  |. BA 300C4E00     MOV EDX,SJK.004E0C30
004E0B1D  |. A1 08904E00     MOV EAX,DWORD PTR DS:[4E9008]
004E0B22  |. 8B00            MOV EAX,DWORD PTR DS:[EAX]
004E0B24  |. E8 D715F7FF     CALL SJK.00452100
004E0B29  |. 8B86 DC020000   MOV EAX,DWORD PTR DS:[ESI+2DC]
004E0B2F  |. 8B10            MOV EDX,DWORD PTR DS:[EAX]
004E0B31  |. FF92 CC000000   CALL DWORD PTR DS:[EDX+CC]
004E0B37  |. 8B86 E8020000   MOV EAX,DWORD PTR DS:[ESI+2E8]
004E0B3D  |. 8B10            MOV EDX,DWORD PTR DS:[EAX]
004E0B3F  |. FF92 CC000000   CALL DWORD PTR DS:[EDX+CC]
004E0B45  |. E9 A4000000     JMP SJK.004E0BEE
004E0B4A  |> 8D55 F8         LEA EDX,DWORD PTR SS:[EBP-8]  ********跳到这里
004E0B4D  |. 8B86 E8020000   MOV EAX,DWORD PTR DS:[ESI+2E8]
004E0B53  |. E8 1032F5FF     CALL SJK.00433D68********读取试验码
004E0B58  |. 8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
004E0B5B  |. 50              PUSH EAX
004E0B5C  |. 8D55 F0         LEA EDX,DWORD PTR SS:[EBP-10]
004E0B5F  |. 8B86 DC020000   MOV EAX,DWORD PTR DS:[ESI+2DC]
004E0B65  |. E8 FE31F5FF     CALL SJK.00433D68
004E0B6A  |. 8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]
004E0B6D  |. 8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
004E0B70  |. E8 FBFCFFFF     CALL SJK.004E0870   *************注册码生成的地方,F7 跟进
004E0B75  |. 8B55 F4         MOV EDX,DWORD PTR SS:[EBP-C]
004E0B78  |. 58              POP EAX
004E0B79  |. E8 EE35F2FF     CALL SJK.0040416C
004E0B7E  |. 75 33           JNZ SHORT SJK.004E0BB3
004E0B80  |. BA 0B000000     MOV EDX,0B
004E0B85  |. B8 580C4E00     MOV EAX,SJK.004E0C58                    ;  ASCII "Yire"
004E0B8A  |. E8 1DFCFFFF     CALL SJK.004E07AC
004E0B8F  |. 6A 40           PUSH 40
004E0B91  |. B9 600C4E00     MOV ECX,SJK.004E0C60
004E0B96  |. BA 680C4E00     MOV EDX,SJK.004E0C68
004E0B9B  |. A1 08904E00     MOV EAX,DWORD PTR DS:[4E9008]
004E0BA0  |. 8B00            MOV EAX,DWORD PTR DS:[EAX]
004E0BA2  |. E8 5915F7FF     CALL SJK.00452100
004E0BA7  |. A1 E4AB4E00     MOV EAX,DWORD PTR DS:[4EABE4]
004E0BAC  |. E8 F3E1F6FF     CALL SJK.0044EDA4
004E0BB1  |. EB 3B           JMP SHORT SJK.004E0BEE
004E0BB3  |> A1 5C924E00     MOV EAX,DWORD PTR DS:[4E925C]
004E0BB8  |. 8338 1E         CMP DWORD PTR DS:[EAX],1E
004E0BBB  |. 7C 0F           JL SHORT SJK.004E0BCC
004E0BBD  |. BA 01000000     MOV EDX,1
004E0BC2  |. B8 580C4E00     MOV EAX,SJK.004E0C58                    ;  ASCII "Yire"
004E0BC7  |. E8 E0FBFFFF     CALL SJK.004E07AC
004E0BCC  |> 6A 40           PUSH 40
004E0BCE  |. B9 800C4E00     MOV ECX,SJK.004E0C80
004E0BD3  |. BA 880C4E00     MOV EDX,SJK.004E0C88
004E0BD8  |. A1 08904E00     MOV EAX,DWORD PTR DS:[4E9008]
004E0BDD  |. 8B00            MOV EAX,DWORD PTR DS:[EAX]
004E0BDF  |. E8 1C15F7FF     CALL SJK.00452100
004E0BE4  |. A1 E4AB4E00     MOV EAX,DWORD PTR DS:[4EABE4]
004E0BE9  |. E8 B6E1F6FF     CALL SJK.0044EDA4
004E0BEE  |> 33C0            XOR EAX,EAX
004E0BF0  |. 5A              POP EDX
004E0BF1  |. 59              POP ECX
004E0BF2  |. 59              POP ECX
004E0BF3  |. 64:8910         MOV DWORD PTR FS:[EAX],EDX
004E0BF6  |. 68 200C4E00     PUSH SJK.004E0C20
004E0BFB  |> 8D45 F0         LEA EAX,DWORD PTR SS:[EBP-10]
004E0BFE  |. E8 D931F2FF     CALL SJK.00403DDC
004E0C03  |. 8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]
004E0C06  |. E8 D131F2FF     CALL SJK.00403DDC
004E0C0B  |. 8D45 F8         LEA EAX,DWORD PTR SS:[EBP-8]
004E0C0E  |. BA 02000000     MOV EDX,2
004E0C13  |. E8 E831F2FF     CALL SJK.00403E00
004E0C18  . C3              RETN
004E0C19   .^E9 D22BF2FF     JMP SJK.004037F0
004E0C1E   .^EB DB           JMP SHORT SJK.004E0BFB
004E0C20   . 5E              POP ESI
004E0C21   . 8BE5            MOV ESP,EBP
004E0C23   . 5D              POP EBP
004E0C24   . C3              RETN
```

生成注册码的 `call`

```
004E0870  /$ 55               PUSH EBP
004E0871  |. 8BEC             MOV EBP,ESP
004E0873  |. B9 04000000      MOV ECX,4
004E0878  |> 6A 00           /PUSH 0
004E087A  |. 6A 00           |PUSH 0
004E087C  |. 49              |DEC ECX
004E087D  |. 75 F9           JNZ SHORT SJK.004E0878
004E087F  |. 51              PUSH ECX
004E0880  |. 53              PUSH EBX
004E0881  |. 56              PUSH ESI
004E0882  |. 57              PUSH EDI
004E0883  |. 8955 F8         MOV DWORD PTR SS:[EBP-8],EDX
004E0886  |. 8945 FC         MOV DWORD PTR SS:[EBP-4],EAX
004E0889  |. 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004E088C  |. E8 7F39F2FF      CALL SJK.00404210
004E0891  |. 33C0            XOR EAX,EAX
004E0893  |. 55              PUSH EBP
004E0894  |. 68 320A4E00      PUSH SJK.004E0A32
004E0899  |. 64:FF30          PUSH DWORD PTR FS:[EAX]
004E089C  |. 64:8920          MOV DWORD PTR FS:[EAX],ESP
004E089F  |. 8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]
004E08A2  |. 8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]
004E08A5  |. E8 CA35F2FF      CALL SJK.00403E74
004E08AA  |. 8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
004E08AD  |. E8 AA37F2FF      CALL SJK.0040405C
004E08B2  |. 8BF0            MOV ESI,EAX
004E08B4  |. 85F6            TEST ESI,ESI
004E08B6  |. 7E 70           JLE SHORT SJK.004E0928
```



```assembly
004E08B8   .  BB 01000000    MOV EBX,1

004E08BD   |> 8BCB           /MOV ECX,EBX******* 这个循环生成注册码(4e08bd---------4e0926) 8)

004E08BF   .  0FAFCB         |IMUL ECX,EBX

004E08C2   .  8BC1           |MOV EAX,ECX

004E08C4   .  F7EB           |IMUL EBX

004E08C6   .  8D53 14        |LEA EDX,DWORD PTR DS:[EBX+14]

004E08C9   .  8BFA           |MOV EDI,EDX

004E08CB   .  99             |CDQ

004E08CC   .  F7FF           |IDIV EDI

004E08CE   .  8BFA           |MOV EDI,EDX

004E08D0   .  8BC1           |MOV EAX,ECX

004E08D2   .  8D53 0A        |LEA EDX,DWORD PTR DS:[EBX+A]

004E08D5   .  8BCA           |MOV ECX,EDX

004E08D7   .  99             |CDQ

004E08D8   .  F7F9           |IDIV ECX

004E08DA   .  03FA           |ADD EDI,EDX

004E08DC   .  8BC7           |MOV EAX,EDI

004E08DE   .  8BD3           |MOV EDX,EBX

004E08E0   .  03D2           |ADD EDX,EDX

004E08E2   .  03C2           |ADD EAX,EDX

004E08E4   .  83C0 02        |ADD EAX,2   **********到这里,第一次 eax=4,第二次 eax=10,然后是 13,1a,14,18,30,22

004E08E7   .  8B55 F4        |MOV EDX,DWORD PTR SS:[EBP-C]

004E08EA   .  0FB6541A FF    |MOVZX EDX,BYTE PTR DS:[EDX+EBX-1] 用户名的第一个字符=>edx

004E08EF   .  03C2           |ADD EAX,EDX *****6+61(a)

004E08F1   .  83F8 7A        |CMP EAX,7A ******与'z'比较

004E08F4   .  7E 08          |JLE SHORT SJK.004E08FE****小于,跳

004E08F6   |> 83E8 0A        |/SUB EAX,0A******eax-a,直到小于'z'

004E08F9   .  83F8 7A        ||CMP EAX,7A

004E08FC   .  7F F8          |JG SHORT SJK.004E08F6

004E08FE   |> 83F8 61        |CMP EAX,61          *******与'a'比较

004E0901   .  7D 08          |JGE SHORT SJK.004E090B****大于,跳

004E0903   |> 83C0 0A        |/ADD EAX,0A*********eax+a,直到大于'a'

004E0906   .  83F8 61        ||CMP EAX,61

004E0909   .  7C F8          |JL SHORT SJK.004E0903

004E090B   |> 8845 EB        |MOV BYTE PTR SS:[EBP-15],AL  *****6+61=g 存入 0075ee4c

004E090E   .  8D45 E4        |LEA EAX,DWORD PTR SS:[EBP-1C]

004E0911   .  8A55 EB        |MOV DL,BYTE PTR SS:[EBP-15]

004E0914   .  E8 6B36F2FF    |CALL SJK.00403F84

004E0919   .  8B55 E4        |MOV EDX,DWORD PTR SS:[EBP-1C]

004E091C   .  8D45 F0        |LEA EAX,DWORD PTR SS:[EBP-10]

004E091F   .  E8 4037F2FF    |CALL SJK.00404064

004E0924   .  43             |INC EBX

004E0925   .  4E             |DEC ESI

004E0926   .  75 95          JNZ SHORT SJK.004E08BD

004E0928   |> 8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]

004E092B   .  E8 2C37F2FF    CALL SJK.0040405C

004E0930   .  83F8 0A        CMP EAX,0A

004E0933   .  0F8D 84000000  JGE SJK.004E09BD

004E0939   .  8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]

004E093C   .  E8 1B37F2FF    CALL SJK.0040405C

004E0941   .  50             PUSH EAX

004E0942   .  B8 0A000000    MOV EAX,0A

004E0947   .  5A             POP EDX

004E0948   .  2BC2           SUB EAX,EDX

004E094A   .  8BF0           MOV ESI,EAX

004E094C   .  85F6           TEST ESI,ESI

004E094E   .  7E 6D          JLE SHORT SJK.004E09BD

004E0950   .  BB 01000000    MOV EBX,1

004E0955   |> 8BCB           /MOV ECX,EBX

004E0957   .  0FAFCB         |IMUL ECX,EBX

004E095A   .  8BC1           |MOV EAX,ECX

004E095C   .  F7EB           |IMUL EBX

004E095E   .  8D53 14        |LEA EDX,DWORD PTR DS:[EBX+14]

004E0961   .  8BFA           |MOV EDI,EDX

004E0963   .  99             |CDQ

004E0964   .  F7FF           |IDIV EDI

004E0966   .  8BFA           |MOV EDI,EDX

004E0968   .  8BC1           |MOV EAX,ECX

004E096A   .  8D53 0A        |LEA EDX,DWORD PTR DS:[EBX+A]

004E096D   .  8BCA           |MOV ECX,EDX

004E096F   .  99             |CDQ

004E0970   .  F7F9           |IDIV ECX

004E0972   .  03FA           |ADD EDI,EDX

004E0974   .  8BC7           |MOV EAX,EDI

004E0976   .  8BD3           |MOV EDX,EBX

004E0978   .  03D2           |ADD EDX,EDX

004E097A   .  03C2           |ADD EAX,EDX

004E097C   .  40             |INC EAX

004E097D   .  8B55 F4        |MOV EDX,DWORD PTR SS:[EBP-C]

004E0980   .  0FB652 01      |MOVZX EDX,BYTE PTR DS:[EDX+1]

004E0984   .  03C2           |ADD EAX,EDX

004E0986   .  83F8 7A        |CMP EAX,7A

004E0989   .  7E 08          |JLE SHORT SJK.004E0993

004E098B   |> 83E8 0A        |/SUB EAX,0A

004E098E   .  83F8 7A        ||CMP EAX,7A

004E0991   .  7F F8          |JG SHORT SJK.004E098B

004E0993   |> 83F8 61        |CMP EAX,61

004E0996   .  7D 08          |JGE SHORT SJK.004E09A0

004E0998   |> 83C0 0A        |/ADD EAX,0A

004E099B   .  83F8 61        ||CMP EAX,61

004E099E   .  7C F8          |JL SHORT SJK.004E0998

004E09A0   |> 8845 EB        |MOV BYTE PTR SS:[EBP-15],AL

004E09A3   .  8D45 E0        |LEA EAX,DWORD PTR SS:[EBP-20]

004E09A6   .  8A55 EB        |MOV DL,BYTE PTR SS:[EBP-15]

004E09A9   .  E8 D635F2FF    |CALL SJK.00403F84

004E09AE   .  8B55 E0        |MOV EDX,DWORD PTR SS:[EBP-20]

004E09B1   .  8D45 F0        |LEA EAX,DWORD PTR SS:[EBP-10]

004E09B4   .  E8 AB36F2FF    |CALL SJK.00404064

004E09B9   .  43             |INC EBX

004E09BA   .  4E             |DEC ESI

004E09BB   .  75 98          JNZ SHORT SJK.004E0955

004E09BD   |> 8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]

004E09C0   .  E8 9736F2FF    CALL SJK.0040405C

004E09C5   .  8BF0           MOV ESI,EAX

004E09C7   .  85F6           TEST ESI,ESI

004E09C9   .  7E 2C          JLE SHORT SJK.004E09F7

004E09CB   .  BB 01000000    MOV EBX,1

004E09D0   |> 8B45 F0        /MOV EAX,DWORD PTR SS:[EBP-10]

004E09D3   .  E8 8436F2FF    |CALL SJK.0040405C

004E09D8   .  2BC3           |SUB EAX,EBX

004E09DA   .  8B55 F0        |MOV EDX,DWORD PTR SS:[EBP-10]

004E09DD   .  8A1402         |MOV DL,BYTE PTR DS:[EDX+EAX]

004E09E0   .  8D45 DC        |LEA EAX,DWORD PTR SS:[EBP-24]

004E09E3   .  E8 9C35F2FF    |CALL SJK.00403F84

004E09E8   .  8B55 DC        |MOV EDX,DWORD PTR SS:[EBP-24]

004E09EB   .  8D45 EC        |LEA EAX,DWORD PTR SS:[EBP-14]

004E09EE   .  E8 7136F2FF    |CALL SJK.00404064

004E09F3   .  43             |INC EBX

004E09F4   .  4E             |DEC ESI

004E09F5   .  75 D9          JNZ SHORT SJK.004E09D0

004E09F7   |> 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]

004E09FA   .  8B55 EC        MOV EDX,DWORD PTR SS:[EBP-14]

004E09FD   .  E8 2E34F2FF    CALL SJK.00403E30

004E0A02   .  33C0           XOR EAX,EAX

004E0A04   .  5A             POP EDX

004E0A05   .  59             POP ECX

004E0A06   .  59             POP ECX

004E0A07   .  64:8910        MOV DWORD PTR FS:[EAX],EDX

004E0A0A   .  68 390A4E00    PUSH SJK.004E0A39

004E0A0F   |> 8D45 DC        LEA EAX,DWORD PTR SS:[EBP-24]

004E0A12   .  BA 03000000    MOV EDX,3

004E0A17   .  E8 E433F2FF    CALL SJK.00403E00

004E0A1C   .  8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]

004E0A1F   .  BA 03000000    MOV EDX,3

004E0A24   .  E8 D733F2FF    CALL SJK.00403E00

004E0A29   .  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]

004E0A2C   .  E8 AB33F2FF    CALL SJK.00403DDC

004E0A31   .  C3             RETN
```

此时注册码以生成: `0075EE54 00C911E8 ASCII "gqvqwyuy"`

`ascii` 依次为: `67,71,76,71,77,79,75,79`

`eax` 存的数: `4,10,13,1a,14,18,30,22`

`eax+2=:` `6,12,15,1c,16,1a,32,24`
`61,5f,61,55,61,5f,43,55`

`ascii:` `a - a U a - C U`

`s=73`

`s+12=85-a=7b-a=71`
`s+1c=8f-a=85-a=7b-a=71`
`s+1a=8d-a=83-a=79`
`s+24=97-a=8d-a=83-a=79`
`a=61`
`a+32=93-a=89-a=7f-a=75`

换一个用户名试一试.

`asdfasdfasdf`

`eax=4,10,13,1a,14,18,30,22,1b,1e,43,24,`

注册码: `0075EE54 00C9AA4C ASCII "gqyxwyxvtuwx"`

`eax` 中数据计算如下:



```
004E08BD   |> 8BCB          /MOV ECX,EBX**********初始 ebx=1(递增)==>ecx   

004E08BF   . 0FAFCB        |IMUL ECX,EBX        ************* ebx*ecx=>ecx   

004E08C2   . 8BC1          |MOV EAX,ECX     ********        cx=>eax   

004E08C4   . F7EB          |IMUL EBX           **********     ebx*eax=>eax   

004E08C6   . 8D53 14       |LEA EDX,DWORD PTR DS:[EBX+14]  *************edx=15(初始,然后递增)   

004E08C9   . 8BFA          |MOV EDI,EDX  ***********   edx=>edi   

004E08CB   . 99            |CDQ            *********     edx=0   

004E08CC   . F7FF          |IDIV EDI      ***********       edi/ebx   

004E08CE   . 8BFA          |MOV EDI,EDX  ********************   商=>edi   

004E08D0   . 8BC1          |MOV EAX,ECX      ******* ecx=>eax   

004E08D2   . 8D53 0A       |LEA EDX,DWORD PTR DS:[EBX+A]  ************** 初值 b,递增   

004E08D5   . 8BCA          |MOV ECX,EDX   *****************  edx=>ecx   

004E08D7   . 99            |CDQ             ********************       edx=0   

004E08D8   . F7F9          |IDIV ECX          *************************    ecx/ebx   

004E08DA   . 03FA          |ADD EDI,EDX     *****************   初 1+1=>edi   

004E08DC   . 8BC7          |MOV EAX,EDI       *****************     edi=>eax   

004E08DE   . 8BD3          |MOV EDX,EBX        *****************    ebx=>edx   

004E08E0   . 03D2          |ADD EDX,EDX     *****************************     edx*2=>edx   

004E08E2   . 03C2          |ADD EAX,EDX          *******     edx+eax=>eax   

004E08E4   . 83C0 02       |ADD EAX,2         **************************     eax+2
```

注册机可编程.

完工.

`4dd466d0e1e0cc123afbe504da2462ca`
