
```asm
005A415B  |.  50               push eax

005A415C  |.  8D55 C8          lea edx,[local.14]

005A415F  |.  B8 38425A00      mov eax,NetUSB\_e.005A4238            ;  ASCII

"qvFntWKEjVV\_gG\_LpGk@zgS@iKK=eiVchyV\_gIWTpHGPny?LjkJyfySyngronXvfqvFntYK?jvZ\_dL"

005A4164 >|.  E8 373AFFFF      call NetUSB\_e.00597BA0               ;  ->:THttpCli.\_PROC\_00597BA0()

005A4169  |.  8B45 C8          mov eax,[local.14]

005A416C >|.  E8 170DE6FF      call NetUSB\_e.00404E88               ;  ->System.@LStrToPChar(String):PAnsiChar;

005A4171  |.  8BD0             mov edx,eax

005A4173  |.  A1 4C2A5C00      mov eax,dword ptr ds:[5C2A4C]

005A4178  |.  8B00             mov eax,dword ptr ds:[eax]

005A417A  |.  59               pop ecx

005A417B >|.  E8 70B2ECFF      call NetUSB\_e.0046F3F0               ;

->Forms.TApplication.MessageBox(TApplication;PChar;PChar;Longint):Integer;

005A4180  |.  C783 4C020000 01>mov dword ptr ds:[ebx+24C],1

005A418A  |.  8BC3             mov eax,ebx

005A418C >|.  E8 9379ECFF      call NetUSB\_e.0046BB24               ;  ->Forms.TCustomForm.Close(TCustomForm);

005A4191  |>  33C0             xor eax,eax

005A4193  |.  5A               pop edx

005A4194  |.  59               pop ecx

005A4195  |.  59               pop ecx

005A4196  |.  64:8910          mov dword ptr fs:[eax],edx

005A4199  |.  68 E8415A00      push NetUSB\_e.005A41E8

005A419E  |>  8D45 C8          lea eax,[local.14]

005A41A1  |.  BA 05000000      mov edx,5

005A41A6 >|.  E8 4108E6FF      call NetUSB\_e.004049EC               ;  ->System.@LStrArrayClr(void;void;Integer);

005A41AB  |.  8D45 DC          lea eax,[local.9]

005A41AE >|.  E8 1508E6FF      call NetUSB\_e.004049C8               ;  ->System.@LStrClr(void;void);

005A41B3  |.  8D45 E0          lea eax,[local.8]

005A41B6 >|.  E8 0D08E6FF      call NetUSB\_e.004049C8               ;  ->System.@LStrClr(void;void);

005A41BB  |.  8D45 E4          lea eax,[local.7]

005A41BE >|.  E8 0508E6FF      call NetUSB\_e.004049C8               ;  ->System.@LStrClr(void;void);

005A41C3  |.  8D45 E8          lea eax,[local.6]

005A41C6  |.  BA 04000000      mov edx,4

005A41CB >|.  E8 1C08E6FF      call NetUSB\_e.004049EC               ;  ->System.@LStrArrayClr(void;void;Integer);

005A41D0  |.  8D45 F8          lea eax,[local.2]

005A41D3 >|.  E8 F007E6FF      call NetUSB\_e.004049C8               ;  ->System.@LStrClr(void;void);

005A41D8  |.  8D45 FC          lea eax,[local.1]

005A41DB >|.  E8 E807E6FF      call NetUSB\_e.004049C8               ;  ->System.@LStrClr(void;void);

005A41E0  \.  C3               retn

005A41E1 > .^ E9 4A01E6FF      jmp NetUSB\_e.00404330                ;  ->System.@HandleFinally;

005A41E6   .^ EB B6            jmp short NetUSB\_e.005A419E

005A41E8   .  5B               pop ebx

005A41E9   .  8BE5             mov esp,ebp

005A41EB   .  5D               pop ebp

005A41EC   .  C3               retn
```

从上面代码可以很明显的看出:

1、在005A40CA处 `call NetUSB_e.00597BA0` 对试炼码进行处理

2、在005A4126处 `call NetUSB_e.004EF6BC` 是关键部分，需要跟进，

3、在005A412D处 跳转,爆破此处仅仅是注册验证爆破，重启验证还是通不过的!

接下来开始分析一下上面的两了函数吧……

1、`call NetUSB_e.00597BA0`

---

```asm
00597BA0  /$  55               push ebp                             ;  字符串解密函数

00597BA1  |.  8BEC             mov ebp,esp

00597BA3  |.  83C4 D8          add esp,-28

00597BA6  |.  53               push ebx

00597BA7  |.  56               push esi

00597BA8  |.  57               push edi

00597BA9  |.  33C9             xor ecx,ecx

00597BAB  |.  894D D8          mov [local.10],ecx

00597BAE  |.  894D DC          mov [local.9],ecx

00597BB1  |.  894D E0          mov [local.8],ecx

00597BB4  |.  8955 F8          mov [local.2],edx

00597BB7  |.  8945 FC          mov [local.1],eax

00597BBA  |.  8B45 FC          mov eax,[local.1]

00597BBD  |.  E8 B6D2E6FF      call NetUSB\_e.00404E78

00597BC2  |.  33C0             xor eax,eax

00597BC4  |.  55               push ebp

00597BC5  |.  68 047D5900      push NetUSB\_e.00597D04

00597BCA  |.  64:FF30          push dword ptr fs:[eax]

00597BCD  |.  64:8920          mov dword ptr fs:[eax],esp

00597BD0  |.  8B45 F8          mov eax,[local.2]

00597BD3  |.  E8 F0CDE6FF      call NetUSB\_e.004049C8

00597BD8  |.  8D45 E0          lea eax,[local.8]

00597BDB  |.  E8 E8CDE6FF      call NetUSB\_e.004049C8

00597BE0  |.  C645 EA FC       mov byte ptr ss:[ebp-16],0FC

00597BE4  |.  C645 EC F0       mov byte ptr ss:[ebp-14],0F0

00597BE8  |.  C645 EE C0       mov byte ptr ss:[ebp-12],0C0

00597BEC  |.  33C0             xor eax,eax

00597BEE  |.  8945 F0          mov [local.4],eax

00597BF1  |.  C645 E6 00       mov byte ptr ss:[ebp-1A],0

00597BF5  |.  BB 02000000      mov ebx,2

00597BFA  |.  33FF             xor edi,edi

00597BFC  |.  33F6             xor esi,esi

00597BFE  |.  8B45 FC          mov eax,[local.1]

00597C01  |.  E8 8AD0E6FF      call NetUSB\_e.00404C90

00597C06  |.  8945 F4          mov [local.3],eax

00597C09  |.  3B75 F4          cmp esi,[local.3]

00597C0C  |.  0F8D B9000000    jge NetUSB\_e.00597CCB

00597C12  |>  8B45 FC          /mov eax,[local.1]

00597C15  |.  0FB60430         |movzx eax,byte ptr ds:[eax+esi]

00597C19  |.  83E8 3C          |sub eax,3C

00597C1C  |.  79 2A            |jns short NetUSB\_e.00597C48

00597C1E  |.  8D45 DC          |lea eax,[local.9]

00597C21  |.  8B55 FC          |mov edx,[local.1]

00597C24  |.  8A1432           |mov dl,byte ptr ds:[edx+esi]

00597C27  |.  E8 7CCFE6FF      |call NetUSB\_e.00404BA8

00597C2C  |.  8B55 DC          |mov edx,[local.9]

00597C2F  |.  8D45 E0          |lea eax,[local.8]

00597C32  |.  E8 61D0E6FF      |call NetUSB\_e.00404C98

00597C37  |.  FF45 F0          |inc [local.4]

00597C3A  |.  46               |inc esi

00597C3B  |.  C645 E6 00       |mov byte ptr ss:[ebp-1A],0

00597C3F  |.  BB 02000000      |mov ebx,2

00597C44  |.  33FF             |xor edi,edi

00597C46  |.  EB 7A            |jmp short NetUSB\_e.00597CC2

00597C48  |>  8B45 FC          |mov eax,[local.1]

00597C4B  |.  8A0430           |mov al,byte ptr ds:[eax+esi]

00597C4E  |.  2C 3C            |sub al,3C

00597C50  |.  8845 E7          |mov byte ptr ss:[ebp-19],al

00597C53  |.  8B45 F0          |mov eax,[local.4]

00597C56  |.  3B45 F4          |cmp eax,[local.3]

00597C59  |.  7D 70            |jge short NetUSB\_e.00597CCB

00597C5B  |.  8D47 06          |lea eax,dword ptr ds:[edi+6]

00597C5E  |.  83F8 08          |cmp eax,8

00597C61  |.  7C 44            |jl short NetUSB\_e.00597CA7

00597C63  |.  8A45 E7          |mov al,byte ptr ss:[ebp-19]

00597C66  |.  24 3F            |and al,3F

00597C68  |.  8BF8             |mov edi,eax

00597C6A  |.  81E7 FF000000    |and edi,0FF

00597C70  |.  B9 06000000      |mov ecx,6

00597C75  |.  2BCB             |sub ecx,ebx

00597C77  |.  D3EF             |shr edi,cl

00597C79  |.  8D45 D8          |lea eax,[local.10]

00597C7C  |.  33D2             |xor edx,edx

00597C7E  |.  8A55 E6          |mov dl,byte ptr ss:[ebp-1A]

00597C81  |.  0BD7             |or edx,edi

00597C83  |.  E8 20CFE6FF      |call NetUSB\_e.00404BA8

00597C88  |.  8B55 D8          |mov edx,[local.10]

00597C8B  |.  8D45 E0          |lea eax,[local.8]

00597C8E  |.  E8 05D0E6FF      |call NetUSB\_e.00404C98

00597C93  |.  FF45 F0          |inc [local.4]

00597C96  |.  33FF             |xor edi,edi

00597C98  |.  83FB 06          |cmp ebx,6

00597C9B  |.  7C 07            |jl short NetUSB\_e.00597CA4

00597C9D  |.  BB 02000000      |mov ebx,2

00597CA2  |.  EB 1D            |jmp short NetUSB\_e.00597CC1

00597CA4  |>  83C3 02          |add ebx,2

00597CA7  |>  8BCB             |mov ecx,ebx

00597CA9  |.  8A45 E7          |mov al,byte ptr ss:[ebp-19]

00597CAC  |.  D2E0             |shl al,cl

00597CAE  |.  8845 E6          |mov byte ptr ss:[ebp-1A],al

00597CB1  |.  8A441D E8        |mov al,byte ptr ss:[ebp+ebx-18]

00597CB5  |.  2045 E6          |and byte ptr ss:[ebp-1A],al
```
