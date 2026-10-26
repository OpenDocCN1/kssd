
```
00401E08    C64424 4C 04    mov byte ptr ss:[esp+4C],4
00401E0D    E8 BE0C2D00     call <jmp.&MFC42.#800_CString::~CString>
00401E12    8D4C24 18       lea ecx,dword ptr ss:[esp+18]
00401E16    C64424 4C 03    mov byte ptr ss:[esp+4C],3
00401E1B    E8 B00C2D00     call <jmp.&MFC42.#800_CString::~CString>
00401E20    8D4C24 1C       lea ecx,dword ptr ss:[esp+1C]
00401E24    C64424 4C 02    mov byte ptr ss:[esp+4C],2
00401E29    E8 A20C2D00     call <jmp.&MFC42.#800_CString::~CString>
00401E2E    8D4C24 20       lea ecx,dword ptr ss:[esp+20]
00401E32    C64424 4C 01    mov byte ptr ss:[esp+4C],1
00401E37    E8 940C2D00     call <jmp.&MFC42.#800_CString::~CString>
00401E3C    8D4C24 24       lea ecx,dword ptr ss:[esp+24]
00401E40    C64424 4C 00    mov byte ptr ss:[esp+4C],0
00401E45    E8 860C2D00     call <jmp.&MFC42.#800_CString::~CString>
00401E4A    8D4C24 58       lea ecx,dword ptr ss:[esp+58]
00401E4E    C74424 4C FFFFF>mov dword ptr ss:[esp+4C],-1
00401E56    E8 750C2D00     call <jmp.&MFC42.#800_CString::~CString>
00401E5B    8B4C24 44       mov ecx,dword ptr ss:[esp+44]
00401E5F    5E              pop esi
00401E60    64:890D 0000000>mov dword ptr fs:[0],ecx
00401E67    5B              pop ebx
00401E68    83C4 48         add esp,48
00401E6B    C3              retn
```

现在我们来看看这段程序：在地址`00401B60`处的`call`会产生4个ASCII码：`"5243"`、`"a524"`、`"5652"`、`"2722"`，通过跟入`call G2F.00401E70`和`call G2F.004021E0`（代码太长了，而且还有子`call`，就不一一说明了），拿试炼码来做说明如下：

（一）、第一次循环，先取试炼码前24位的前8位，即`"12345678"`，再分为`"1234"`和`"5678"`两部分，计为试炼码①部分和试炼码②部分，计算如下：

（1）、先将`0x5678`（试炼码②部分）与`0x2722`相加，得`0x7d9a`，再与`0x1234`（试炼码①部分）异或得`0x6fae`；将`0x6fae`与`0x5652`相加，得`0xc600`，再与`0x5678`（试炼码②部分）异或得`0x9078`。通过`call G2F.004021E0`将`0x6fae`和`0x9078`转换为`0xea6f`和`0x7890`。

（2）、将`0xae6f`与`0xa524`相加得`0x5393`，再与`0x7890`异或得`0x2b03`；将`0x2b03`与`0x5243`相加得`0x7d46`，再与`0xae6f`异或得`0xd329`。通过`call G2F.004021E0`将`0x2b03`和`0xd329`转换为`0x032b`和`0x29d3`。

（3）、将`0x032b`与`0x5243`相加得`0x556e`，再与`0x29d3`异或得`0x7cbd`；将`0x7cbd`与`0xa524`相加得`0x21e1`，再与`0x032b`异或得`0x22ca`。通过`call G2F.004021E0`将`0x7cbd`和`0x22ca`转换为`0xbd7c`和`0xca22`。

（4）、将`0xbd7c`与`0x5652`相加得`0x13ce`，再与`0xca22`异或得`0xd9ec`；将`0xd9ec`与`0x2722`相加得`0x010e`，再与`0xbd7c`异或得`0xbc72`。最后将`0xd9ec`和`0xbc72`合并起来组成`0xbc72d9ec`。

（二）、第二次循环，取得试炼码前24位中间的8位，即`"90123456"`，分为`"9012"`和`"3456"`两部分，计为试炼码③部分和试炼码④部分，计算如下：

（1）、先将`0x3456`（试炼码④部分）与`0x2722`相加，得`0x5b78`，再与`0x9012`（试炼码③部分）异或得`0xcb6a`；将`0xcb6a`与`0x5652`相加，得`0x21bc`，再与`0x3456`（试炼码④部分）异或得`0x15ea`。通过`call G2F.004021E0`将`0xcb6a`和`0x15ea`转换为`0x6acb`和`0xea15`。

（2）、将`0x6acb`与`0xa524`相加得`0x0fef`，再与`0xea15`异或得`0xe5fa`；将`0xe5fa`与`0x5243`相加得`0x383d`，再与`0x6acb`异或得`0x52f6`。通过`call G2F.004021E0`将`0xe5fa`和`0x52f6`转换为`0xfae5`和`0xf652`。

（3）、将`0xfae5`与`0x5243`相加得`0x4d28`，再与`0xf652`异或得`0xbb7a`；将`0xbb7a`与`0xa524`相加得`0x609e`，再与`0xfae5`异或得`0x9a7b`。通过`call G2F.004021E0`将`0xbb7a`和`0x9a7b`转换为`0x7abb`和`0x7b9a`。

（4）、将`0x7abb`与`0x5652`相加得`0xd10d`，再与`0x7b9a`异或得`0xaa97`；将`0xaa97`与`0x2722`相加得`0xd1b9`，再与`0x7abb`异或得`0xab02`。最后将`0xab02`和`0xaa97`合并起来组成`0xab02aa97`，与第一次循环的结果`0xbc72d9ec`合并为`0xbc72d9ecab02aa97`。

（三）、第三次循环，取得试炼码前24位的后8位，即`"78901234"`，再分为`"7890"`和`"1234"`两部分，计为试炼码⑤部分和试炼码⑥部分，计算如下：

（1）、先将`0x1234`（试炼码⑥部分）与`0x2722`相加，得`0x3956`，再与`0x7890`（试炼码⑤部分）异或得`0x41c6`；将`0x41c6`与`0x5652`相加，得`0x9818`，再与`0x1234`（试炼码⑥部分）异或得`0x8a2c`。通过`call G2F.004021E0`将`0x41c6`和`0x8a2c`转换为`0xc641`和`0x2c8a`。

（2）、将`0xc641`与`0xa524`相加得`0x6b65`，再与`0x2c8a`异或得`0x47ef`；将`0x47ef`与`0x5243`相加得`0x9a32`，再与`0xc641`异或得`0x5c73`。通过`call G2F.004021E0`将`0x47ef`和`0x5c73`转换为`0xef47`和`0x735c`。

（3）、将`0xef47`与`0x5243`相加得`0x418a`，再与`0x735c`异或得`0x32d6`；将`0x32d6`与`0xa524`相加得`0xd7fa`，再与`0xef47`异或得`0x38bd`。通过`call G2F.004021E0`将`0x32d6`和`0x38bd`转换为`0xd632`和`0xbd38`。

（4）、将`0xd632`与`0x5652`相加得`0x2c84`，再与`0xbd38`异或得`0x91bc`；将`0x91bc`与`0x2722`相加得`0xb8de`，再与`0xd632`异或得`0x6eec`。最后将`0x91bc`和`0x6eec`合并起来组成`0x6eec91bc`，再与前两次循环的结果合并为`0xbc72d9ecab02aa976eec91bc`。

以上做加法运算时都忽略了最高位的进位。在将试炼码转换为相应的ASCII码时，将数字`0－9`转换为相应十六进制`0－9`，`a-f`和`A-F`转换为相应十六进制的`A-F`，其余字母则转换为数字`0`，再做加法或异或运算，有兴趣的可以跟进各个`call`看看，不过里面还有几个子`call`，看的头都晕了:(   现在我们再看看地址`00401B60`处的`call`是如何得到那4个ASCII码的：

```
00401840    6A FF           push -1
00401842    68 684C7900     push G2F.00794C68
00401847    64:A1 00000000  mov eax,dword ptr fs:[0]
0040184D    50              push eax
0040184E    64:8925 0000000>mov dword ptr fs:[0],esp
00401855    83EC 0C         sub esp,0C
00401858    53              push ebx
00401859    56              push esi
0040185A    8D4424 24       lea eax,dword ptr ss:[esp+24]
0040185E    8D4C24 0C       lea ecx,dword ptr ss:[esp+C]
00401862    50              push eax
00401863    C74424 20 00000>mov dword ptr ss:[esp+20],0
0040186B    E8 E4122D00     call <jmp.&MFC42.#535_CString::CString>
00401870    8D4C24 08       lea ecx,dword ptr ss:[esp+8]
00401874    C64424 1C 01    mov byte ptr ss:[esp+1C],1
00401879    E8 64122D00     call <jmp.&MFC42.#540_CString::CString>
0040187E    8B4C24 24       mov ecx,dword ptr ss:[esp+24]                 ; 取得ASCII"2r%e*RE$"
00401882    B3 02           mov bl,2
00401884    885C24 1C       mov byte ptr ss:[esp+1C],bl
00401888    8B41 F8         mov eax,dword ptr ds:[ecx-8]                  ; 获得ASCII"2r%e*RE$"的位数
0040188B    83F8 08         cmp eax,8
0040188E    7D 18           jge short G2F.004018A8                        ; 是否大于等于8，是则跳到004018A8处
00401890    8D4C24 0C       lea ecx,dword ptr ss:[esp+C]
00401894    51              push ecx
00401895    8D4C24 28       lea ecx,dword ptr ss:[esp+28]
00401899    E8 BC122D00     call <jmp.&MFC42.#939_CString::operator+=>
0040189E    8B4C24 24       mov ecx,dword ptr ss:[esp+24]
004018A2    8379 F8 08      cmp dword ptr ds:[ecx-8],8
004018A6  ^ 7C E8           jl short G2F.00401890
004018A8    8379 F8 08      cmp dword ptr ds:[ecx-8],8
004018AC    7E 2C           jle short G2F.004018DA                        ; 位数是否小于等于8，是则跳到004018DA
004018AE    8D5424 10       lea edx,dword ptr ss:[esp+10]
004018B2    6A 08           push 8
004018B4    52              push edx
004018B5    8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]
004018B9    E8 BA122D00     call <jmp.&MFC42.#4129_CString::Left>
004018BE    50              push eax
004018BF    8D4C24 28       lea ecx,dword ptr ss:[esp+28]
004018C3    C64424 20 03    mov byte ptr ss:[esp+20],3
004018C8    E8 7B122D00     call <jmp.&MFC42.#858_CString::operator=>
004018CD    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
004018D1    885C24 1C       mov byte ptr ss:[esp+1C],bl
004018D5    E8 F6112D00     call <jmp.&MFC42.#800_CString::~CString>
004018DA    33F6            xor esi,esi                                   ; esi清零
004018DC    8B4424 24       mov eax,dword ptr ss:[esp+24]                 ; 取得ASCII"2r%e*RE$"
004018E0    0FBE0406        movsx eax,byte ptr ds:[esi+eax]               ; 取得第一位2的ASCII码32H
004018E4    85C0            test eax,eax
004018E6    7D 05           jge short G2F.004018ED                        ; 大于等于0则跳走
004018E8    05 00010000     add eax,100
004018ED    50              push eax
004018EE    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
004018F2    68 58598700     push G2F.00875958                             ; ASCII "%x"
004018F7    51              push ecx
004018F8    E8 2D122D00     call <jmp.&MFC42.#2818_CString::Format>       ; 格式化32H成ASCII"32"
004018FD    83C4 0C         add esp,0C
00401900    8D5424 0C       lea edx,dword ptr ss:[esp+C]
00401904    8D4C24 08       lea ecx,dword ptr ss:[esp+8]
00401908    52              push edx
00401909    E8 4C122D00     call <jmp.&MFC42.#939_CString::operator+=>
0040190E    46              inc esi
0040190F    83FE 08         cmp esi,8
00401912  ^ 7C C8           jl short G2F.004018DC                         ; 共执行8次，依次取得字符串"2r%e*RE$"的ASCII码，组成"327225652a524524"
00401914    8D4424 10       lea eax,dword ptr ss:[esp+10]
00401918    6A 01           push 1
0040191A    50              push eax
0040191B    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
0040191F    E8 54122D00     call <jmp.&MFC42.#4129_CString::Left>         ; 取得ASCII"327225652a524524"左边一位"3"
```
