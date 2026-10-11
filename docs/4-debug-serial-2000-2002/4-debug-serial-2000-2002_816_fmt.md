
# GoldView32 2.0 Build168

> 原文：[`www.pediy.com/kssd/pediy04/kanxue361.htm`](https://www.pediy.com/kssd/pediy04/kanxue361.htm)

## GoldView32 2.0 build 168 破解过程

### 软件简介
是一个对文件的所在结构进行浏览的软件。就是能显示文件有关结构和对象的相关信息，是一种只能显示不能修改的工具，对于不想查询数据词典而想知道有关信息的人非常有用，能让人们安全浏览文件。

### 下载地址
`http://newhua.infosail.com/Goldview.htm`

### 破解工具
`Language2000`、`DeDe 3.0`、`Ollydbg 1.4`

### 文章作者
`eCool[BCG][FCG][CNCG]`

### 出版时间
2002 年 1 月 24 日

### 郑重声明
转载请保持文章完整，谢谢。

很久没有搞破解了有点手痒，所以立马去华军一趟，随便选了个软件下载，结果就搞到了这个东东，安装完成后，运行，注册，原来是单 Serial 型的注册，看来有点困难，随便输入一个注册码，点 `Ok`，出来一个错误对话框，呵呵，有戏看了。退出程序，立即用 `Language2000` 分析一下 `GoldView.exe`，不错没有加壳，而且是 `DeDe` 的，so，用 `Dede` 分析一下，很快在 `Nag Unit` 中找到个 `RegisterButtonClick`，进入这个事件,来到这里：

`Reference to field TNagDlg.OFFS_0314`

```
004E0626  8B9314030000           mov     edx, [ebx+$0314]

004E062C  8D45FC                 lea     eax, [ebp-$04]

004E062F  E8B445F2FF             call    00404BE8

004E0634  8B45FC                 mov     eax, [ebp-$04]

004E0637  E894F8FFFF             call    004DFED0

004E063C  84C0                   test    al, al  经典的比较啊，因此在 4E0637 处下断。
004E063E  743C                   jz      004E067C
004E0640  8D55F4                 lea     edx, [ebp-$0C]
```

好的，马上运行 `Ollydbg`，载入，运行，输入假注册码 `AB1234`，然后在 `4E0637` 下断，点 `OK` 按钮，马上背拦下，按 `F7` 跟入。

```
004DFED0  55                     push    ebp
004DFED1  8BEC                   mov     ebp, esp
004DFED3  33C9                   xor     ecx, ecx
004DFED5  51                     push    ecx
004DFED6  51                     push    ecx
004DFED7  51                     push    ecx
004DFED8  51                     push    ecx
004DFED9  51                     push    ecx
004DFEDA  51                     push    ecx
004DFEDB  53                     push    ebx
004DFEDC  56                     push    esi
004DFEDD  8945FC                 mov     [ebp-$04], eax
004DFEE0  8B45FC                 mov     eax, [ebp-$04]

004DFEE3  E89C4EF2FF             call    00404D84

004DFEE8  33C0                   xor     eax, eax
004DFEEA  55                     push    ebp

Possible String Reference to: '閹 B?脬奅鸮[嬪]脥@'

004DFEEB  68F5FF4D00             push    $004DFFF5
```
***** TRY

```
004DFEF0  64FF30                 push    dword ptr fs:[eax]
004DFEF3  648920                 mov     fs:[eax], esp
004DFEF6  C645FB00               mov     byte ptr [ebp-$05], $00
004DFEFA  8D55EC                 lea     edx, [ebp-$14]
004DFEFD  8B45FC                 mov     eax, [ebp-$04]

004DFF00  E8DB8EF2FF             call    00408DE0  取字符串"Goldview32"
004DFF05  8B55EC                 mov     edx, [ebp-$14]
004DFF08  8D45FC                 lea     eax, [ebp-$04]

004DFF0B  E86C4AF2FF             call    0040497C  将"Goldview32"加上我们的注册码
004DFF10  8D45F0                 lea     eax, [ebp-$10] 所以变成"Goldview32AB1234"

004DFF13  E8CC49F2FF             call    004048E4  转换成大写
004DFF18  8B45FC                 mov     eax, [ebp-$04] "GOLDVIEW32AB1234"

004DFF1B  E87C4CF2FF             call    00404B9C
004DFF20  8BD8                   mov     ebx, eax
004DFF22  EB01                   jmp     004DFF25
004DFF24  4B                     dec     ebx
004DFF25  8B45FC                 mov     eax, [ebp-$04]              -+
004DFF28  8A4418FF               mov     al, byte ptr [eax+ebx-$01] | 这段代码的作用是
004DFF2C  04D0                   add     al, -$30                   | 判断输入的注册码的
004DFF2E  2C0A                   sub     al, $0A                    | 最后几位是不是数字。
004DFF30  7304                   jnb     004DFF36                   | （最起码最后一位必须
004DFF32  85DB                   test    ebx, ebx                   | 是数字）
004DFF34  7FEE                   jnle    004DFF24                   -+
004DFF36  8D45F0                 lea     eax, [ebp-$10]
004DFF39  50                     push    eax
004DFF3A  8B45FC                 mov     eax, [ebp-$04]

004DFF3D  E85A4CF2FF             call    00404B9C
004DFF42  8BC8                   mov     ecx, eax
004DFF44  2BCB                   sub     ecx, ebx
004DFF46  8D5301                 lea     edx, [ebx+$01]
004DFF49  8B45FC                 mov     eax, [ebp-$04]

004DFF4C  E8A34EF2FF             call    00404DF4
004DFF51  8D45F4                 lea     eax, [ebp-$0C]
004DFF54  50                     push    eax
004DFF55  8B45FC                 mov     eax, [ebp-$04]

004DFF58  E83F4CF2FF             call    00404B9C
004DFF5D  50                     push    eax
004DFF5E  8B45F0                 mov     eax, [ebp-$10]

004DFF61  E8364CF2FF             call    00404B9C
004DFF66  59                     pop     ecx
004DFF67  2BC8                   sub     ecx, eax
004DFF69  BA01000000             mov     edx, $00000001
004DFF6E  8B45FC                 mov     eax, [ebp-$04]

004DFF71  E87E4EF2FF             call    00404DF4
004DFF76  33F6                   xor     esi, esi
004DFF78  8B45F4                 mov     eax, [ebp-$0C]

004DFF7B  E81C4CF2FF             call    00404B9C
004DFF80  85C0                   test    eax, eax
004DFF82  7E4E                   jle     004DFFD2
004DFF84  8B45F4                 mov     eax, [ebp-$0C]

004DFF87  E8104CF2FF             call    00404B9C
004DFF8C  85C0                   test    eax, eax
004DFF8E  7E42                   jle     004DFFD2
004DFF90  8B45F4                 mov     eax, [ebp-$0C]

004DFF93  E8044CF2FF             call    00404B9C  取去掉最后几位数字的字符
004DFF98  85C0                   test    eax, eax      这里是 GOLDVIEW32AB
004DFF9A  7E1B                   jle     004DFFB7
004DFF9C  BB01000000             mov     ebx, $00000001
004DFFA1  8B55F4                 mov     edx, [ebp-$0C]
004DFFA4  0FB6541AFF             movzx   edx, byte ptr [edx+ebx-$01] -+ 对 GOLDVIEW32AB 进行运算
004DFFA9  0FAFD3                 imul    edx, ebx                     | s="GOLDVIEW32AB":y=0
004DFFAC  6BCB0B                 imul    ecx, ebx, $0B                | for i=1 to len(s)
004DFFAF  03D1                   add     edx, ecx                     |   x=asc(mid$(s,i,1))
004DFFB1  03F2                   add     esi, edx                     |   x=x*i
004DFFB3  43                     inc     ebx                          |   x=i*&hb+x
004DFFB4  48                     dec     eax                          |   y=y+x
004DFFB5  75EA                   jnz     004DFFA1                     -+ next i
004DFFB7  8D55E8                 lea     edx, [ebp-$18]
004DFFBA  8BC6                   mov     eax, esi

004DFFBC  E80395F2FF             call    004094C4  获得最后几位是数字的字符，这里是 1234
004DFFC1  8B45E8                 mov     eax, [ebp-$18]  eax=y＝6100
004DFFC4  8B55F0                 mov     edx, [ebp-$10]  edx＝1234

004DFFC7  E8144DF2FF             call    00404CE0  比较是否相等
004DFFCC  7504                   jnz     004DFFD2
004DFFCE  C645FB01               mov     byte ptr [ebp-$05], $01
004DFFD2  33C0                   xor     eax, eax
004DFFD4  5A                     pop     edx
004DFFD5  59                     pop     ecx
004DFFD6  59                     pop     ecx
004DFFD7  648910                 mov     fs:[eax], edx
```
***** FINALLY

`Possible String Reference to: '奅鸮[嬪]脥@'`

```
004DFFDA  68FCFF4D00             push    $004DFFFC
004DFFDF  8D45E8                 lea     eax, [ebp-$18]
004DFFE2  BA04000000             mov     edx, $00000004

004DFFE7  E81C49F2FF             call    00404908
004DFFEC  8D45FC                 lea     eax, [ebp-$04]

004DFFEF  E8F048F2FF             call    004048E4
004DFFF4  C3                     ret
004DFFF5  E98E42F2FF             jmp     00404288
```



```
004DFFFA  EBE3                  jmp   004DFFDF

****** END

|

004DFFFC  8A45FB               mov   al, byte ptr [ebp-$05]

004DFFFF  5E                   pop   esi

004E0000  5B                   pop   ebx

004E0001  8BE5                 mov   esp, ebp

004E0003  5D                   pop   ebp

004E0004  C3                   ret
```

到此，我们可以得到一个合法的注册码是：`AB6100. 4dd466d0e1e0cc123afbe504da2462ca`
