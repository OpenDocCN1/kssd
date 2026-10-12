
## 004733AA - 004734CD

```
004733AA  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]-------->注册码地址   
004733AD  |.  8B55 EC       |MOV EDX,DWORD PTR SS:[EBP-14]------->计数器   
004733B0  |.  8A4410 FF     |MOV AL,BYTE PTR DS:[EAX+EDX-1]------>取注册码的下一位(相对于上面)   
004733B4  |.  8845 E4       |MOV BYTE PTR SS:[EBP-1C],AL\--------->取得的这一位注册码   
004733B7  |.  807D E4 24    |CMP BYTE PTR SS:[EBP-1C],24--------->取得的这位字符与"$"比较   
004733BB  |.  75 27         |JNZ SHORT UNPACKED.004733E4--------->不相等则跳   
004733BD  |.  8B45 E8       |MOV EAX,DWORD PTR SS:[EBP-18]   
004733C0  |.  48            |DEC EAX   
004733C1  |.  3B45 EC       |CMP EAX,DWORD PTR SS:[EBP-14]   
004733C4  |.  74 09         |JE SHORT UNPACKED.004733CF   
004733C6  |.  C645 F3 04    |MOV BYTE PTR SS:[EBP-D],4   
004733CA  |.  E9 CD000000   |JMP UNPACKED.0047349C   
004733CF  |>  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]   
004733D2  |.  8B55 EC       |MOV EDX,DWORD PTR SS:[EBP-14]   
004733D5  |.  803C10 24     |CMP BYTE PTR DS:[EAX+EDX],24   
004733D9  |.  74 50         |JE SHORT UNPACKED.0047342B   
004733DB  |.  C645 F3 05    |MOV BYTE PTR SS:[EBP-D],5   
004733DF  |.  E9 B8000000   |JMP UNPACKED.0047349C   
004733E4  |>  8D4D E7       |LEA ECX,DWORD PTR SS:[EBP-19]------>再取新地址   
004733E7  |.  8A55 E4       |MOV DL,BYTE PTR SS:[EBP-1C]-------->取得的这位注册码   
004733EA  |.  8B45 FC       |MOV EAX,DWORD PTR SS:[EBP-4]------->ASSII 07,"TBase64"   
004733ED  |.  E8 16FBFFFF   |CALL UNPACKED.00472F08------------->关键运算三◎◎◎   
004733F2  |.  84C0          |TEST AL,AL\------------------------->作为标志   
004733F4  |.  75 09         |JNZ SHORT UNPACKED.004733FF-------->合法则跳走   
004733F6  |.  C645 F3 02    |MOV BYTE PTR SS:[EBP-D],2   
004733FA  |.  E9 9D000000   |JMP UNPACKED.0047349C   
004733FF  |>  8A45 E6       |MOV AL,BYTE PTR SS:[EBP-1A]-------->第二位的结果   
00473402  |.  C1E0 04       |SHL EAX,4-------------------------->逻辑左移 4 位   
00473405  |.  33D2          |XOR EDX,EDX\------------------------>EDX 清零作准备   
00473407  |.  8A55 E7       |MOV DL,BYTE PTR SS:[EBP-19]-------->第三位的结果   
0047340A  |.  C1EA 02       |SHR EDX,2-------------------------->逻辑右移 2 位   
0047340D  |.  02C2          |ADD AL,DL\-------------------------->结果相加   
0047340F  |.  8845 E5       |MOV BYTE PTR SS:[EBP-1B],AL\-------->【保存结果】   
00473412  |.  8D45 D8       |LEA EAX,DWORD PTR SS:[EBP-28]------>【新地址】   
00473415  |.  8A55 E5       |MOV DL,BYTE PTR SS:[EBP-1B]-------->前面三位的运算结果   
00473418  |.  E8 F30BF9FF   |CALL UNPACKED.00404010------------->结果保存于上面的新地址   
0047341D  |.  8B55 D8       |MOV EDX,DWORD PTR SS:[EBP-28]------>前面三位的运算结果   
00473420  |.  8B45 F4       |MOV EAX,DWORD PTR SS:[EBP-C]------->前两位的运算结果   
00473423  |.  E8 C80CF9FF   |CALL UNPACKED.004040F0------------->上面两个结果连在一起   
00473428  |.  8B45 F4       |MOV EAX,DWORD PTR SS:[EBP-C]------->上面两个结果连接后的结果   
0047342B  |>  FF45 EC       |INC DWORD PTR SS:[EBP-14]---------->计数器+1   
0047342E  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]------->输入的注册码   
00473431  |.  8B55 EC       |MOV EDX,DWORD PTR SS:[EBP-14]------>计数器   
00473434  |.  8A4410 FF     |MOV AL,BYTE PTR DS:[EAX+EDX-1]----->取下一位注册码   
00473438  |.  8845 E4       |MOV BYTE PTR SS:[EBP-1C],AL\-------->保存结果   
0047343B  |.  807D E4 24    |CMP BYTE PTR SS:[EBP-1C],24-------->取得的这位字符于"$"比较   
0047343F  |.  75 0E         |JNZ SHORT UNPACKED.0047344F-------->不相等则跳   
00473441  |.  8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]   
00473444  |.  3B45 E8       |CMP EAX,DWORD PTR SS:[EBP-18]   
00473447  |.  74 43         |JE SHORT UNPACKED.0047348C   
00473449  |.  C645 F3 04    |MOV BYTE PTR SS:[EBP-D],4   
0047344D  |.  EB 4D         |JMP SHORT UNPACKED.0047349C   
0047344F  |>  8D4D E6       |LEA ECX,DWORD PTR SS:[EBP-1A]----->再取新地址   
00473452  |.  8A55 E4       |MOV DL,BYTE PTR SS:[EBP-1C]------->取得的这位字符   
00473455  |.  8B45 FC       |MOV EAX,DWORD PTR SS:[EBP-4]------>ASSII 07,"TBase64"   
00473458  |.  E8 ABFAFFFF   |CALL UNPACKED.00472F08------------>关键运算三◎◎◎   
0047345D  |.  84C0          |TEST AL,AL\------------------------>作为标志   
0047345F  |.  75 06         |JNZ SHORT UNPACKED.00473467------->合法则跳走   
00473461  |.  C645 F3 02    |MOV BYTE PTR SS:[EBP-D],2   
00473465  |.  EB 35         |JMP SHORT UNPACKED.0047349C   
00473467  |>  8A45 E7       |MOV AL,BYTE PTR SS:[EBP-19]------->第三位运算的结果   
0047346A  |.  C1E0 06       |SHL EAX,6------------------------->逻辑左移 6 位   
0047346D  |.  0245 E6       |ADD AL,BYTE PTR SS:[EBP-1A]------->结果相加   
00473470  |.  8845 E5       |MOV BYTE PTR SS:[EBP-1B],AL\------->【保存结果】   
00473473  |.  8D45 D4       |LEA EAX,DWORD PTR SS:[EBP-2C]----->【新地址】   
00473476  |.  8A55 E5       |MOV DL,BYTE PTR SS:[EBP-1B]------->运算结果   
00473479  |.  E8 920BF9FF   |CALL UNPACKED.00404010------------>保存于上面的新地址   
0047347E  |.  8B55 D4       |MOV EDX,DWORD PTR SS:[EBP-2C]----->运算结果   
00473481  |.  8B45 F4       |MOV EAX,DWORD PTR SS:[EBP-C]------>前面几位的运算结果   
00473484  |.  E8 670CF9FF   |CALL UNPACKED.004040F0------------>连接到一起   
00473489  |.  8B45 F4       |MOV EAX,DWORD PTR SS:[EBP-C]------>连接后的结果-------->◎◎   
0047348C  |>  8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]----->计数器   
0047348F  |.  3B45 E8       |CMP EAX,DWORD PTR SS:[EBP-18]----->计数器的值和注册码的长度比较   
00473492  |.^ 0F8C 8DFEFFFF JL UNPACKED.00473325-------------->没有结束则跳到上面继续进行运算   
00473498  |.  C645 F3 00    MOV BYTE PTR SS:[EBP-D],0   
0047349C  |>  33C0          XOR EAX,EAX   
0047349E  |.  5A            POP EDX   
0047349F  |.  59            POP ECX   
004734A0  |.  59            POP ECX   
004734A1  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX   
004734A4  |.  68 C6344700   PUSH UNPACKED.004734C6   
004734A9  |>  8D45 D4       LEA EAX,DWORD PTR SS:[EBP-2C]   
004734AC  |.  BA 04000000   MOV EDX,4   
004734B1  |.  E8 D609F9FF   CALL UNPACKED.00403E8C   
004734B6  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]   
004734B9  |.  E8 AA09F9FF   CALL UNPACKED.00403E68   
004734BE  .  C3            RETN   
004734BF   .^ E9 B803F9FF   JMP UNPACKED.0040387C   
004734C4   .^ EB E3         JMP SHORT UNPACKED.004734A9   
004734C6   .  8A45 F3       MOV AL,BYTE PTR SS:[EBP-D]   
004734C9   .  5B            POP EBX   
004734CA   .  8BE5          MOV ESP,EBP   
004734CC   .  5D            POP EBP   
004734CD   .  C3            RETN   
```

本端对输入的 16 位注册码分 4 次循环分别进行运算，第四次到◎◎时，下指令 `d [eax]`得到的结果就是关键运算一的结果，具体计算过程描述如下(以第一次循环为例)：

ASCII 表示 `"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"`

取输入注册码的首位，计算其在 ASCII 中的位置(首位位置为 0)，然后计算如下：

①：首位的位置值×4＋第二位的位置值÷16

②：第二位的位置值×16＋第三位的位置值÷4

③：①和②连接保存到一个新地址

④：第三位的结果×64＋第四位的位置值

连接③和④地结果

内存中形式如下：① ② ④

我输入注册码后，四次循环时内存中内容分别如下：

```
01E21054  F7 CD B5 00 1E 00 00 00 01 00 00 00 0C 00 00 00   魍?..........   
01E140BC  F7 CD B5 D3 5D F5 00 00 6A 00 00 00 08 D9 42 00   魍涤\]?.j...貰.   
01E27864  F7 CD B5 D3 5D F5 DB 7E 39 00 00 00 1A 00 00 00   魍涤\]踣~9......   
01E1FE10  F7 CD B5 D3 5D F5 DB 7E 39 EB BD 78 00 00 00 00   魍涤\]踣~9 虢 x....   
```

最后返回第四次循环后的值

## 关键二

『前面关键运算一根据输入的注册码得出一个结果，本段总共调用 4 次，每次比较一个字符，分别为`"F"`、`"S"`、`"Z"`、`"L"`关键一运算的结果里面必须含有这四个字符。』

『返回时 `EAX` 不可为零』

```
004043D4  /$  85C0          TEST EAX,EAX\---------------------------------->程序固定??   
004043D6  |.  74 40         JE SHORT UNPACKED.00404418-------------------->不可为空   
004043D8  |.  85D2          TEST EDX,EDX\---------------------------------->关键一的结果   
004043DA  |.  74 31         JE SHORT UNPACKED.0040440D-------------------->不可为空   
004043DC  |.  53            PUSH EBX\-------------------------   
004043DD  |.  56            PUSH ESI                      |保存   
004043DE  |.  57            PUSH EDI\-------------------------/   
```
