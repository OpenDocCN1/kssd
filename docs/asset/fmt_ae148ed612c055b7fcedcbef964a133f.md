
# 代码分析

`- 上面的 CALL 结束,省略了一部分代码!`

```
004C6A31  |>  33C0          XOR     EAX, EAX  
004C6A33  |.   5A          POP     EDX                          ;  MoneyHom.004C6A8A  
004C6A34  |.   59          POP     ECX                          ;  MoneyHom.004C6A8A  
004C6A35  |.   59          POP     ECX                          ;  MoneyHom.004C6A8A  
004C6A36  |.   64:8910     MOV     DWORD PTR FS:[EAX], EDX  
004C6A39  |.   68 4E6A4C00 PUSH    MoneyHom.004C6A4E  
004C6A3E  |>   8D45 FC     LEA     EAX, [LOCAL.1]  
004C6A41  |.   E8 0AD6F3FF CALL    MoneyHom.00404050  
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

`004C69F6 |. E8 91FDFFFF CALL MoneyHom.004C678C` 这个 `CALL` 的代码如下，主要是了解对机器码的处理：

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

004C678C  /$  55           PUSH    EBP  
004C678D  |.   8BEC        MOV     EBP, ESP  
004C678F  |.   83C4 F8     ADD     ESP, -8  
004C6792  |.   53           PUSH    EBX  
004C6793  |.   56           PUSH    ESI  
004C6794  |.   33D2        XOR     EDX, EDX  
004C6796  |.   8955 F8     MOV     [LOCAL.2], EDX  
004C6799  |.   8945 FC     MOV     [LOCAL.1], EAX  
004C679C  |.   8B45 FC     MOV     EAX, [LOCAL.1]  
004C679F  |.   E8 F0DCF3FF CALL    MoneyHom.00404494  
004C67A4  |.   33C0        XOR     EAX, EAX  
004C67A6  |.   55           PUSH    EBP  
004C67A7  |.   68 66684C00 PUSH    MoneyHom.004C6866  
004C67AC  |.   64:FF30     PUSH    DWORD PTR FS:[EAX]  
004C67AF  |.   64:8920     MOV     DWORD PTR FS:[EAX], ESP  
004C67B2  |.   8B55 FC     MOV     EDX, [LOCAL.1]  
004C67B5  |.   B8 80684C00 MOV     EAX, MoneyHom.004C6880  
004C67BA  |.   E8 0DDEF3FF CALL    MoneyHom.004045CC  
004C67BF  |.   85C0        TEST    EAX, EAX  
004C67C1  |.   76 1F       JBE     SHORT MoneyHom.004C67E2  
004C67C3  |>   8D55 FC     /LEA     EDX, [LOCAL.1]  
004C67C6  |.   B9 01000000 |MOV     ECX, 1  
004C67CB  |.   92           |XCHG    EAX, EDX  
004C67CC  |.   E8 57DDF3FF |CALL    MoneyHom.00404528======>此 CALL 干什么?把机器码中间的"-"去掉,变成联在一起的 8 位数!  
004C67D1  |.   8B55 FC     |MOV     EDX, [LOCAL.1]=========>处理后的机器码入 EDX,即 EDX 的值为 92269973!  
004C67D4  |.   B8 80684C00 |MOV     EAX, MoneyHom.004C6880  
004C67D9  |.   E8 EEDDF3FF |CALL    MoneyHom.004045CC  
004C67DE  |.   85C0        |TEST    EAX, EAX  
004C67E0  |.^ 77 E1        \JA      SHORT MoneyHom.004C67C3  
004C67E2  |>   8B45 FC     MOV     EAX, [LOCAL.1]  
004C67E5  |.   E8 F6DAF3FF CALL    MoneyHom.004042E0  
004C67EA  |.   83F8 08     CMP     EAX, 8=================>比较是否是 8 位!  
004C67ED  |.   74 04       JE      SHORT MoneyHom.004C67F3====>相等就跳,这里当然会跳,因为处理过后的机器码就是 8 位!  
004C67EF  |.   33DB        XOR     EBX, EBX  
004C67F1  |.   EB 58       JMP     SHORT MoneyHom.004C684B  
004C67F3  |>   8D55 F8     LEA     EDX, [LOCAL.2]=============>跳到此处![LOCAL.2]存放经过处理后的 8 位机器码!  
004C67F6  |.   8B45 FC     MOV     EAX, [LOCAL.1]  
004C67F9  |.   E8 BA36F4FF CALL    MoneyHom.00409EB8  
004C67FE  |.   33C9        XOR     ECX, ECX  
004C6800  |.   33F6        XOR     ESI, ESI  
004C6802  |.   B8 06000000 MOV     EAX, 6=====================>把 6 送入 EAX,即是说取 6 位机器码运算!  
004C6807  |>   8B55 F8     /MOV     EDX, [LOCAL.2]============>机器码放入 EDX!  
004C680A  |.   0FB65402 FF |MOVZX    EDX, BYTE PTR DS:[EDX+EAX-1]  
004C680F  |.   83EA 30     |SUB      EDX, 30  
004C6812  |.   03F2        |ADD      ESI, EDX  
004C6814  |.   03C9        |ADD      ECX, ECX  
004C6816  |.   8D0C89       |LEA      ECX, DWORD PTR DS:[ECX+ECX*4]  
004C6819  |.   03D1        |ADD      EDX, ECX  
004C681B  |.   8BCA        |MOV      ECX, EDX  
004C681D  |.   48           |DEC      EAX  
004C681E  |.   85C0        |TEST     EAX, EAX  
004C6820  |.^ 75 E5        \JNZ      SHORT MoneyHom.004C6807===>这里是 6 次循环!对前 6 位机器码处理!如何处理?取前 6 位机器码,  
\======================================>把它反过来,即前 6 位机器码是:922699,反过来是:996229,然后把它转化为十六进制数值,就  
\======================================>是:F3385,然后把它放入 EDX 和 ECX 当中!  
004C6822  |.   33D2        XOR     EDX, EDX  
004C6824  |.   B8 08000000 MOV     EAX, 8  
004C6829  |>   03D2        /ADD     EDX, EDX  
004C682B  |.   8D1492       |LEA      EDX, DWORD PTR DS:[EDX+EDX*4]  
004C682E  |.   8B5D F8     |MOV      EBX, [LOCAL.2]               ;  MoneyHom.004C69CA  
004C6831  |.   0FB65C03 FF |MOVZX    EBX, BYTE PTR DS:[EBX+EAX-1]  
004C6836  |.   83EB 30     |SUB      EBX, 30  
004C6839  |.   03D3        |ADD      EDX, EBX  
004C683B  |.   48           |DEC      EAX  
004C683C  |.   83F8 06     |CMP      EAX, 6  
004C683F  |.^ 75 E8        \JNZ      SHORT MoneyHom.004C6829======>这里又是两次循环,对最后两位机器码处理,同上!即 73 反过来  
\======================================>是 37,转化为十六进制值是:25,放入 EDX 中!然后我们回到算法 CALL(1)当中去看!看上面!  
004C6841  |.   3BF2        CMP     ESI, EDX  
004C6843  |.   74 04       JE      SHORT MoneyHom.004C6849  
004C6845  |.   33DB        XOR     EBX, EBX  
004C6847  |.   EB 02       JMP     SHORT MoneyHom.004C684B  
004C6849  |>   8BD9        MOV     EBX, ECX  
004C684B  |>   33C0        XOR     EAX, EAX  
004C684D  |.   5A          POP     EDX                          ;  MoneyHom.004C69FB  
004C684E  |.   59          POP     ECX                          ;  MoneyHom.004C69FB  
004C684F  |.   59          POP     ECX                          ;  MoneyHom.004C69FB  
004C6850  |.   64:8910     MOV     DWORD PTR FS:[EAX], EDX  
004C6853  |.   68 6D684C00 PUSH    MoneyHom.004C686D  
004C6858  |>   8D45 F8     LEA     EAX, [LOCAL.2]  
004C685B  |.   BA 02000000 MOV     EDX, 2  
004C6860  |.   E8 0FD8F3FF CALL    MoneyHom.00404074
```

## 算法总结

算法基本搞清楚，下面进行总结：

1. 机器码的形式是：`XXXX-YYYY`，把中间的 `-` 去掉，连接成 `8` 位数，即：`XXXXYYYY`，只取前六位机器码计算注册码，后两位不参与。前 `6` 位机器码取反，转换成十六进制形式，减去常数：`034B263C`，得一十六进制值，此值除 `1`，再乘常数：`0.9498864000000000`，取整数，再除 `1000000`，取余数，再取反得到前 `6` 位注册码，后两位注册码是余数逐位相加（注意余数被视作是十六进制值），然后转化成十进制值，取反后，即为后两位注册码，连接起来，四位一组，中间加 `-` 即为注册码！

计算公式如下：设机器码的前 `6` 位取反后转换成十六进值为 `A`，前 `6` 位注册码是 `SN(1)`，后两位注册码是 `sn(2)`，设六位余数是：`abcdef`，余数逐位相加后转化成十进制的值为 `gh`

* `SN(1) = ((A - 034B263C) / 1 * 0.9498864000000000) / 1000000` 整除后余数取反 = `fedcba`
* `sn(2) = a + b + c + d + e + f` 的十六进值转化成十进值 = `gh` 取反 = `hg`，注册码的形式是：`fedc-bahg`

2. 例证：我的机器码是：`9226-9973`，即：`92269973`，取前 `6` 位是：`922699`，取反是：`996229`，转成十六进值是：`F3385`

* `((A - 034B263C) / 1 * 0.9498864000000000) / 1000000`
* `= ((F3385 - 34B263C) / 1 * 0.9498864000000000) / 1000000`
* `= 4028.189800`
* `sn(1) = 008981`
* `1 + 8 + 9 + 8 + 0 + 0 = 1A (26)` 取反即为 `62`，所以 `sn(2) = 62`，我的注册码是：`0089-8162`

注册信息存放在：`HKEY_LOCAL_MACHINE\SOFTWARE\Moneywise\MoneyHome\RegisterCode`

要变成未注册版，先把未注册的主程序备份，删除注册表中的健值后，再把备份重新覆盖，即可变为未注册版！

`4dd466d0e1e0cc123afbe504da2462ca`
