
## 汇编代码分析

```
:00403D66 888BA0224500            mov byte ptr [ebx+004522A0], cl
:00403D6C 889320224500            mov byte ptr [ebx+00452220], dl <-把结果写在[4522A0]
:00403D72 43                      inc ebx
:00403D73 888F20214500            mov byte ptr [edi+00452120], cl
:00403D79 3BDE                    cmp ebx, esi
:00403D7B 889721214500            mov byte ptr [edi+00452121], dl <-再写在[452120]
:00403D81 7CAF                    jl 00403D32

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403D30(C)

:00403D83 56                      push esi
:00403D84 55                      push ebp
:00403D85 6820224500              push 00452220
:00403D8A E8C1540000              call 00409250
:00403D8F 8DBE20214500            lea edi, dword ptr [esi+00452120]
:00403D95 56                      push esi
:00403D96 BBA0224500              mov ebx, 004522A0
:00403D9B 57                      push edi
:00403D9C 53                      push ebx
:00403D9D E8AE540000              call 00409250
:00403DA2 56                      push esi
:00403DA3 53                      push ebx
:00403DA4 55                      push ebp
:00403DA5 E8A6540000              call 00409250
:00403DAA 56                      push esi
:00403DAB 6820224500              push 00452220
:00403DB0 57                      push edi
:00403DB1 E89A540000              call 00409250 <--又是同样的方法把前半段 ch1 和后半段 ch2 再调换
:00403DB6 83C430                  add esp, 00000030
:00403DB9 33C0                    xor eax, eax
:00403DBB 85F6                    test esi, esi
:00403DBD 5D                      pop ebp
:00403DBE 5B                      pop ebx
:00403DBF 7E1C                    jle 00403DDD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403DDB(C) <--第三处循环计算

:00403DC1 8A0C4521214500          mov cl, byte ptr [2*eax+00452121] <--从[452120]处每次取两个值
:00403DC8 C0E104                  shl cl, 04                          即第二次计算的结果,把后一值
:00403DCB 0A0C4520214500          or cl, byte ptr [2*eax+00452120]    左移四位后与前值进行 OR 运算,
:00403DD2 40                      inc eax                               结果写在[4523A0]
:00403DD3 3BC6                    cmp eax, esi
:00403DD5 88889F234500            mov byte ptr [eax+0045239F], cl
:00403DDB 7CE4                    jl 00403DC1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403DBF(C)

:00403DDD 80A6A023450000          and byte ptr [esi+004523A0], 00
:00403DE4 5F                      pop edi
:00403DE5 5E                      pop esi
:00403DE6 C3                      ret
```

进行比较的那个 `CALL` 我就不写了(因为这篇文章太长了),但它好像只比较前四位,也就是说只要你的 `EMAIL` 前四位与`[4523A0]`处的结果一致就可以.可我随意填的假码经反算后根本不是可显示字符,要想写出注册机必须把算法逆回去,我想了好半天也不知道这个算法怎样逆运算,看着清清楚楚的代码就是写不出注册机,痛苦中.....

哪位大哥帮我分析一下,我的数学太烂了...

## 标题

终于把注册机写出来了,现在破一个软件不写注册机的话很不爽..... (2千字)

**发信人**:`RoBa`

**时间**:`2003-10-09 13:02:30`

### 详细信息

**程序判断注册的思路:**

1. 先将 `EMAIL` 中的`'@'`、`'.'`字符去掉,如果小于 `12` 个字符就在后面补`'0'`.然后把前六位和后六位颠倒过来.

2. 取注册码(24 位)按下表计算: (假设输入的是 `ABCDEFGHIJKLMNOPQRSTUVWX`)

| 假码 | A | B | C | D | E | F | G | H | I | J | K | L |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Num | 0 | 1 | 2 | 3 | 4 | 5 | 3 | 4 | 5 | 6 | 7 | 8 |

| 假码 | M | N | O | P | Q | R | S | T | U | V | W | X |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Num | 8 | 7 | 6 | 5 | 4 | 3 | 5 | 4 | 3 | 2 | 1 | 0 |

| Nxor | 1 | 3 | 7 | D | 5 | F | B | 9 | 9 | B | F | 5 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| N | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |

其中第 `N` 列的 `Nxor` 值计算方法是`((N-1)*N+1) mod 10h`

判断时将每列上一格里的(注册码-`Num`-`41h`) `xor` 该格对应列的 `Nxor` 值,记结果为 `S1`,用同样方法算出该列下一格的值 `S2`,计算(`S2 SHR 4`) `OR` `S1`. 12 列的结果组成的字串如果和 `EMAIL` 经处理后的字串一致就成功.

因为程序将注册码反算,所以注册机要求它的逆运算.逆运算的关键是如何把 `EMAIL` 中的一个值拆分成 `S1` 和 `S2`,因为这个逆运算产生的结果不惟一,但要保证计算出的注册码是可显示字符.不妨把一个两位的十六进制值拆成高位和低位.如字母`'R'=52h`,可拆成 `S1=2`,`S2=5`,这样计算出来的注册码在`'A'-'X'`之间,完全符合要求.

### 注册机: (`Borland Pascal 7.0`)

```pascal
Program CrackZbook;

var 
  fmail, mail, code, temp: string;
  p, p1, s1, s2, s : integer;
  Num: array[1..24] of integer;
  Nxor: array[1..12] of integer;

begin
  Num[1]:=0; Num[2]:=1; Num[3]:=2; Num[4]:=3;
  Num[5]:=4; Num[6]:=5; Num[7]:=3; Num[8]:=4;
  Num[9]:=5; Num[10]:=6; Num[11]:=7; Num[12]:=8;
  Num[13]:=8; Num[14]:=7; Num[15]:=6; Num[16]:=5;
  Num[17]:=4; Num[18]:=3; Num[19]:=5; Num[20]:=4;
  Num[21]:=3; Num[22]:=2; Num[23]:=1; Num[24]:=0;
  
  Nxor[1]:=1; Nxor[2]:=3; Nxor[3]:=7; Nxor[4]:=13;
  Nxor[5]:=5; Nxor[6]:=15; Nxor[7]:=11; Nxor[8]:=9;
  Nxor[9]:=9; Nxor[10]:=11; Nxor[11]:=15; Nxor[12]:=5; //初始化计算表
  
  p1:=1;
  repeat
    write('Please input your Email:');
    readln(fmail);
  until length(fmail)>=4;
  
  for p:=1 to length(fmail) do
    if (fmail[p]<>'.') and (fmail[p]<>'@') then
    begin
      mail[p1]:=fmail[p];
      inc(p1);
    end;
    
  for p:=p1 to 12 do mail[p]:='0';
  
  for p:=1 to 6 do temp[p]:=mail[p];
  for p:=7 to 12 do mail[p-6]:=mail[p];
  for p:=7 to 12 do mail[p]:=temp[p-6]; //对 EMAIL 的处理
  
  for p:=1 to 12 do
  begin
    s:=ord(mail[p]);
    s1:=s mod 16;
    s2:=s div 16;
    s1:=s1 xor Nxor[p];
    s2:=s2 xor Nxor[p];
    s1:=s1+$41+Num[p];
    s2:=s2+$41+Num[p+12];
    code[p]:=chr(s1);
    code[p+12]:=chr(s2);
  end;
  
  for p:=1 to 24 do write(code[p]);
  writeln;
  writeln('Crack by RoBa     Thank you');
end.
```

一个可用的注册码:

`EMAIL`:`RoBa`

`CODE`:`BEJQJUMKQQWNKHKTKPTTQPNG`

`4dd466d0e1e0cc123afbe504da2462ca`
