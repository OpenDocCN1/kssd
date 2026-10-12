
# XnView v1.91 最新版-软柿子算法分析及注册机

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-171.htm`](https://www.pediy.com/kssd/pediy09/pediy09-171.htm)
>
> └文章标题┐:XnView v1.91 最新版-软柿子算法分析及注册机  
>
> └破文作者┐:大菜一号  
>
> └破解对象┐:XnView v1.91  
>
> └下载地址┐:自已搜吧  
>
> └对象大小┐:不知  
>
> └加壳方式┐:aspack  
>
> └保护方式┐:普通软柿子算法  
>
> └编写语言┐:vc6  
>
> └使用工具┐:OD,peid  
>
> └破解平台┐:D-XP  
>
> └破解声明┐:软柿子真好~~~^-^  
>
> \----------------------------------------------------------------------------------  
>
> └破解过程┐:  
>
> 嘿嘿``近来嫌 ACDsee5.0 速度太慢，看个图也要几秒！就下了个 XnView! 呵呵`虽说它不能简单地编辑一个图，不过看图速度较快，还有十六进制什么什么的！！饿也没用过啦~~西西~~~~~~~~~~  
>
> 好了``aspack 的壳`，Peid 帮我们搞定了！下断 GetDlgItemText, name: jiangwu, serial: 123456!, 点注册后断在:

```
0052E0AA  .  56            push    esi
0052E0AB  .  8BB424 700100> mov    esi, dword ptr [esp+170]
0052E0B2  .  57            push    edi
0052E0B3  .  8B3D 70366600 mov    edi, dword ptr [<&USER32.GetDlgI>;  USER32.GetDlgItemTextA
0052E0B9  .  68 00010000  push    100                     ; /Count = 100 (256.)
0052E0BE  .  50            push    eax                     ; |Buffer
0052E0BF  .  68 D0070000  push    7D0                     ; |ControlID = 7D0 (2000.)
0052E0C4  .  56            push    esi                     ; |hWnd
0052E0C5  .  FFD7         call    edi                     ; \GetDlgItemTextA<=============断在这里,取注册名
0052E0C7  .  8D4C24 10     lea     ecx, dword ptr [esp+10]
0052E0CB  .  6A 20         push    20                      ; /Count = 20 (32.)
0052E0CD  .  51            push    ecx                     ; |Buffer
0052E0CE  .  68 D1070000  push    7D1                     ; |ControlID = 7D1 (2001.)
0052E0D3  .  56            push    esi                     ; |hWnd
0052E0D4  .  FFD7         call    edi                     ; \GetDlgItemTextA<=============这里，取注册码
0052E0D6  .  8A4424 70     mov     al, byte ptr [esp+70]<============注册名第一位到 al
0052E0DA  .  84C0          test    al, al<===========================是否为空
0052E0DC  .  0F84 3A010000 je     0052E21C<=========================没输入注册名就出错
0052E0E2  .  8A4424 10     mov     al, byte ptr [esp+10]<============注册码第一位到 al
0052E0E6  .  84C0          test    al, al<===========================是否为空
0052E0E8  .  0F84 2E010000 je     0052E21C<=========================没输入注册码就出错
0052E0EE  .  8D5424 08     lea     edx, dword ptr [esp+8]
0052E0F2  .  8D4424 70     lea     eax, dword ptr [esp+70]
0052E0F6  .  52            push    edx
0052E0F7  .  50            push    eax<==============================注册名地址进栈
0052E0F8  .  E8 E32CFAFF   call    004D0DE0<=========================关键 call
0052E0FD  .  8D4C24 18     lea     ecx, dword ptr [esp+18]
0052E101  .  51            push    ecx
0052E102  .  E8 EC5F0200   call    005540F3
0052E107  .  8B4C24 14     mov     ecx, dword ptr [esp+14]
0052E10B  .  83C4 0C       add     esp, 0C
0052E10E  .  3BC8          cmp     ecx, eax
0052E110  .  74 5D         je      short 0052E16F
0052E112  .  A1 582A6E00   mov     eax, dword ptr [6E2A58]
0052E117  .  8D5424 30     lea     edx, dword ptr [esp+30]
0052E11B  .  6A 40         push    40                      ; /Count = 40 (64.)
0052E11D  .  52            push    edx                     ; |Buffer
0052E11E  .  68 93130000   push    1393                    ; |RsrcID = STRING "Invalid registration"
0052E123  .  50            push    eax                     ; |hInst => 10000000
0052E124  .  FF15 08376600 call    dword ptr [<&USER32.LoadStringA>>; \LoadStringA<==========加载错误字符串
0052E12A  .  6A 10         push    10                      ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
0052E12C  .  8D4C24 34     lea     ecx, dword ptr [esp+34]   ; |
0052E130  .  68 9CAA6D00   push    006DAA9C                ; |Title = ""
0052E135  .  51            push    ecx                     ; |Text
0052E136  .  56            push    esi                     ; |hOwner
0052E137  .  FF15 90366600 call    dword ptr [<&USER32.MessageBoxA>>; \MessageBoxA<==========显示错误字符串
0052E13D  .  68 D0070000  push    7D0                     ; /ControlID = 7D0 (2000.)
0052E142  .  56            push    esi                     ; |hWnd
0052E143  .  FF15 88366600 call    dword ptr [<&USER32.GetDlgItem>] ; \GetDlgItem
0052E149  .  50            push    eax                     ; /hWnd
0052E14A  .  FF15 C8366600 call    dword ptr [<&USER32.SetFocus>]   ; \SetFocus
0052E150  .  68 9CAA6D00   push    006DAA9C                ; /Text = ""
0052E155  .  68 D1070000  push    7D1                     ; |ControlID = 7D1 (2001.)
0052E15A  .  56            push    esi                     ; |hWnd
0052E15B  .  FF15 84366600 call    dword ptr [<&USER32.SetDlgItemTe>; \SetDlgItemTextA<=====注册错误之后设置注册码输入框为空
0052E161  .  5F            pop     edi
0052E162  .  B8 01000000  mov     eax, 1
0052E167  .  5E            pop     esi
0052E168  .  81C4 68010000 add     esp, 168
0052E16E  .  C3            retn
0052E16F  >  8D5424 70     lea     edx, dword ptr [esp+70]
0052E173  .  68 00010000  push    100
0052E178  .  52            push    edx
0052E179  .  68 D0070000  push    7D0
0052E17E  .  56            push    esi
0052E17F  .  FFD7         call    edi
0052E181  .  8D4424 10     lea     eax, dword ptr [esp+10]
0052E185  .  6A 20         push    20
0052E187  .  50            push    eax
0052E188  .  68 D1070000  push    7D1
0052E18D  .  56            push    esi
0052E18E  .  FFD7         call    edi
0052E190  .  8D4C24 70     lea     ecx, dword ptr [esp+70]
0052E194  .  51            push    ecx
0052E195  .  68 84716B00   push    006B7184                ;  ASCII "LicenseName"
0052E19A  .  6A 00         push    0
0052E19C  .  E8 0FC1F7FF   call    004AA2B0
0052E1A1  .  8D5424 1C     lea     edx, dword ptr [esp+1C]
0052E1A5  .  52            push    edx
0052E1A6  .  68 74716B00   push    006B7174                ;  ASCII "LicenseNumber"
0052E1AB  .  6A 00         push    0
0052E1AD  .  E8 FEC0F7FF   call    004AA2B0
0052E1B2  .  A1 602A6E00   mov     eax, dword ptr [6E2A60]
0052E1B7  .  83C4 18       add     esp, 18
0052E1BA  .  C705 882A6E00> mov     dword ptr [6E2A88], 1
0052E1C4  .  6A 01         push    1                       ; /Flags = MF_BYCOMMAND|MF_GRAYED|MF_STRING
0052E1C6  .  68 F2000000  push    0F2                     ; |ItemID = F2 (242.)
0052E1CB  .  50            push    eax                     ; |/hWnd => 00180068 ('XnView - [浏览器 - E:\工具\Sk...',class='XmainClass')
0052E1CC  .  FF15 F4366600 call    dword ptr [<&USER32.GetMenu>]    ; |\GetMenu
0052E1D2  .  50            push    eax                     ; |hMenu
0052E1D3  .  FF15 DC366600 call    dword ptr [<&USER32.EnableMenuIt>; \EnableMenuItem
0052E1D9  .  8B15 582A6E00 mov     edx, dword ptr [6E2A58]     ;  xnviewzh.10000000
0052E1DF  .  8D4C24 30     lea     ecx, dword ptr [esp+30]
0052E1E3  .  6A 40         push    40                      ; /Count = 40 (64.)
0052E1E5  .  51            push    ecx                     ; |Buffer
0052E1E6  .  68 94130000  push    1394                    ; |RsrcID = STRING "Registration successful.
Thank you for purchasing XnView."
```



> 0052E1EB . 52 push edx ; |hInst => 10000000
> 0052E1EC . FF15 08376600 call dword ptr [&lt;USER32.LoadStringA&gt;>; \LoadStringA&lt;=========加载注册成功字符串
> 0052E1F2 . 6A 40 push 40 ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
> 0052E1F4 . 8D4424 34 lea eax, dword ptr [esp+34] ; |
> 0052E1F8 . 68 9CAA6D00 push 006DAA9C ; |Title = ""
> 0052E1FD . 50 push eax ; |Text
> 0052E1FE . 56 push esi ; |hOwner
> 0052E1FF . FF15 90366600 call dword ptr [&lt;USER32.MessageBoxA&gt;>; \MessageBoxA&lt;=========显示注册成功字符串
> 0052E205 . 6A 00 push 0 ; /Result = 0
> 0052E207 . 56 push esi ; |hWnd
> 0052E208 . FF15 74366600 call dword ptr [&lt;USER32.EndDialog&gt;] ; \EndDialog&lt;===========结束注册对话框
> 0052E20E . 5F pop edi
> 0052E20F . B8 01000000 mov eax, 1
> 0052E214 . 5E pop esi
> 0052E215 . 81C4 68010000 add esp, 168
> 0052E21B . C3 retn

> 0052E21C &gt; 8B15 582A6E00 mov edx, dword ptr [6E2A58] ; xnviewzh.10000000
> 0052E222 . 8D4C24 30 lea ecx, dword ptr [esp+30]
> 0052E226 . 6A 40 push 40 ; /Count = 40 (64.)
> 0052E228 . 51 push ecx ; |Buffer
> 0052E229 . 68 93130000 push 1393 ; |RsrcID = STRING "Invalid registration"
> 0052E22E . 52 push edx ; |hInst => 10000000
> 0052E22F . FF15 08376600 call dword ptr [&lt;USER32.LoadStringA&gt;>; \LoadStringA
> 0052E235 . 6A 10 push 10 ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
> 0052E237 . 8D4424 34 lea eax, dword ptr [esp+34] ; |
> 0052E23B . 68 9CAA6D00 push 006DAA9C ; |Title = ""
> 0052E240 . 50 push eax ; |Text
> 0052E241 . 56 push esi ; |hOwner
> 0052E242 . FF15 90366600 call dword ptr [&lt;USER32.MessageBoxA&gt;>; \MessageBoxA

关键 call:

F7 跟进后到这里:

```
004D0DE0 /$ 8B5424 04 mov edx, dword ptr [esp+4]
004D0DE4 |. 53 push ebx
004D0DE5 |. 55 push ebp
004D0DE6 |. 56 push esi
004D0DE7 |. 57 push edi
004D0DE8 |. 8BFA mov edi, edx
004D0DEA |. 83C9 FF or ecx, FFFFFFFF
004D0DED |. 33C0 xor eax, eax
004D0DEF |. F2:AE repne scas byte ptr es:[edi]
004D0DF1 |. F7D1 not ecx
004D0DF3 |. 49 dec ecx
004D0DF4 |. BE C0656B00 mov esi, 006B65C0
004D0DF9 |. 8BE9 mov ebp, ecx
004D0DFB |. B9 05000000 mov ecx, 5
004D0E00 |. BF 30C76D00 mov edi, 006DC730
004D0E05 |. F3:A5 rep movs dword ptr es:[edi], dword p>
004D0E07 |. 8BF0 mov esi, eax
004D0E09 |. 74 21 je short 004D0E2C
004D0E0B |&gt; 8A0C16 /mov cl, byte ptr [esi+edx]&lt;============注册名字符传到 cl
004D0E0E |. 8AD9 |mov bl, cl&lt;============================再传到 bl
004D0E10 |. 3298 30C76D00 |xor bl, byte ptr [eax+6DC730]&lt;=========与一串十六进制数异或
004D0E16 |. 40 |inc eax&lt;===============================eax+1
004D0E17 |. 83F8 05 |cmp eax, 5&lt;============================eax 和 5 比较
004D0E1A |. 881C16 |mov byte ptr [esi+edx], bl&lt;============异或结果传到 12f608
004D0E1D |. 8888 2FC76D00 |mov byte ptr [eax+6DC72F], cl&lt;=========原字符传到 6dc730
004D0E23 |. 75 02 |jnz short 004D0E27&lt;====================eax 不等于 5 就跳
004D0E25 |. 33C0 |xor eax, eax&lt;==========================eax 等于 5,eax 就为 0
004D0E27 |&gt; 46 |inc esi&lt;===============================计数器加一
004D0E28 |. 3BF5 |cmp esi, ebp
004D0E2A |.^ 72 DF \\jb short 004D0E0B&lt;====================没完跳上去
```

根据 `eax`，可以知道十六进制数串只算其前五位

```
004D0E2C |&gt; 33FF xor edi, edi
004D0E2E |. 33C9 xor ecx, ecx
004D0E30 |. 85ED test ebp, ebp
004D0E32 |. 76 26 jbe short 004D0E5A
004D0E34 |&gt; 8A9F 35C76D00 /mov bl, byte ptr [edi+6DC735]&lt;=========十六进制数串从第六位开始循环依次传到 bl
004D0E3A |. 8BF5 |mov esi, ebp
004D0E3C |. 2BF1 |sub esi, ecx&lt;==========================注册名长度减去计数器值
004D0E3E |. 4E |dec esi&lt;===============================结果减一(指向第一区循环结果的尾部)
004D0E3F |. 8A0416 |mov al, byte ptr [esi+edx]&lt;============从上面循环结果后面开始取字符到 al
004D0E42 |. 32D8 |xor bl, al&lt;============================两者异或
004D0E44 |. 47 |inc edi&lt;===============================edi+1
004D0E45 |. 881C16 |mov byte ptr [esi+edx], bl
004D0E48 |. 8887 34C76D00 |mov byte ptr [edi+6DC734], al
004D0E4E |. 83FF 05 |cmp edi, 5&lt;============================edi 是否为 5
004D0E51 |. 75 02 |jnz short 004D0E55
004D0E53 |. 33FF |xor edi, edi&lt;==========================edi 为 5,edi 就清零
004D0E55 |&gt; 41 |inc ecx
004D0E56 |. 3BCD |cmp ecx, ebp&lt;==========================是否算完
004D0E58 |.^ 72 DA \\jb short 004D0E34&lt;====================没完跳上去
```

根据 `edi`,可以知道十六进制数串在这个循环只算其第六到第十位

```
004D0E5A |&gt; 33F6 xor esi, esi
004D0E5C |. 33FF xor edi, edi
004D0E5E |. 85ED test ebp, ebp
004D0E60 |. 76 21 jbe short 004D0E83
004D0E62 |&gt; 8A0417 /mov al, byte ptr [edi+edx]&lt;============从上面循环结果的前面开始取字符
004D0E65 |. 8A8E 3AC76D00 |mov cl, byte ptr [esi+6DC73A]&lt;=========十六进制数串的一个字节到 cl
004D0E6B |. 32C8 |xor cl, al&lt;============================两者异或
004D0E6D |. 46 |inc esi&lt;===============================esi+1
004D0E6E |. 880C17 |mov byte ptr [edi+edx], cl&lt;============传回去
004D0E71 |. 8886 39C76D00 |mov byte ptr [esi+6DC739], al
004D0E77 |. 83FE 05 |cmp esi, 5&lt;============================esi 是否为 5
004D0E7A |. 75 02 |jnz short 004D0E7E
004D0E7C |. 33F6 |xor esi, esi&lt;==========================esi 为 5 时，esi 就清零
004D0E7E |&gt; 47 |inc edi
004D0E7F |. 3BFD |cmp edi, ebp&lt;==========================是否已算完
004D0E81 |.^ 72 DF \\jb short 004D0E62&lt;====================没完就跳上去
```

根据 `esi`,可以知道十六进制数串在这个循环不只算其第十一位到第十五位

```
004D0E83 |&gt; 33FF xor edi, edi
004D0E85 |. 33C9 xor ecx, ecx
004D0E87 |. 85ED test ebp, ebp
004D0E89 |. 76 26 jbe short 004D0EB1
004D0E8B |&gt; 8A9F 3FC76D00 /mov bl, byte ptr [edi+6DC73F]&lt;========= 从十六进制的第十五位开始取字符
004D0E91 |. 8BF5 |mov esi, ebp
004D0E93 |. 2BF1 |sub esi, ecx
004D0E95 |. 4E |dec esi
004D0E96 |. 8A0416 |mov al, byte ptr [esi+edx]&lt;============从上面循环结果的尾部开始向前取字符
004D0E99 |. 32D8 |xor bl, al&lt;============================两者异或
004D0E9B |. 47 |inc edi&lt;===============================edi+1
004D0E9C |. 881C16 |mov byte ptr [esi+edx], bl
```



### 汇编代码

```assembly
004D0E9F  |. 8887 3EC76D00 |mov     byte ptr [edi+6DC73E], al  
004D0EA5  |. 83FF 05       |cmp     edi, 5<============================edi 是否为 5  
004D0EA8  |. 75 02         |jnz     short 004D0EAC  
004D0EAA  |. 33FF          |xor     edi, edi<==========================edi 为 5，edi 就清零  
004D0EAC  |>  41            |inc     ecx  
004D0EAD  |. 3BCD          |cmp     ecx, ebp<==========================是否已算完  
004D0EAF  |.^ 72 DA        \jb     short 004D0E8B<====================没完就跳上去  
004D0EB1  |>  8B7C24 18     mov     edi, dword ptr [esp+18]  
004D0EB5  |. 33C0          |xor     eax, eax  
004D0EB7  |. 85ED          |test    ebp, ebp  
004D0EB9  |. C707 00000000 mov     dword ptr [edi], 0  
004D0EBF  |. 76 17         |jbe     short 004D0ED8  
004D0EC1  |>  8BC8          /mov    ecx, eax<==========================计数器值传到 ecx  
004D0EC3  |. 83E1 03       |and     ecx, 3<============================ecx 与上 3  
004D0EC6  |. 8A1C39        |mov     bl, byte ptr [ecx+edi]<============根据与的结果从上面循环结果中取出字符  
004D0EC9  |. 8D3439        |lea     esi, dword ptr [ecx+edi]<==========把该位字符相应的地址传到 esi  
004D0ECC  |. 8A0C10        |mov     cl, byte ptr [eax+edx]<============从上面循环结果的前面开始取字符  
004D0ECF  |. 02D9          |add     bl, cl<============================两者相加  
004D0ED1  |. 40            |inc     eax <==============================计数器加一  
004D0ED2  |. 3BC5          |cmp     eax, ebp  
004D0ED4  |. 881E          |mov     byte ptr [esi], bl<================结果传到与 bl 字符相应的内存地址  
004D0ED6  |.^ 72 E9        \jb     short 004D0EC1<====================没完跳上去  
004D0ED8  |>  5F            pop     edi  
004D0ED9  |. 5E            pop     esi  
004D0EDA  |. 5D            pop     ebp  
004D0EDB  |. 5B            pop     ebx  
004D0EDC  \. C3            retn  
```

### 算法总结

其实是简单的 `xor` 加密，呵呵！！不过就是内存位置乱了点~~~

还是不要总结了！！自已看了都烦~~哦呵呵呵！！反正就是异或异或，传回去传过来！取前面，取后面！`就是这样了！~~自己看清楚一点吧~~`

呵呵`

### 注册机代码

注册机还是要写滴！`mfc` 的 `button` 事件代码！！：

```c
 char name[100],code[100],a[10]; 
 long int key[]={0xaa,0x89,0xc4,0xfe,0x46,0x78,0xf0, 
   0xd0,0x03,0xe7,0xf7,0xfd,0xf4,0xe7,0xb9,0xb5,0x1b,//对了`这就是那个传说中的十六进制数串 
   0xc9,0x50,0x73}; 
 int i,len=0,k=0; 
 unsigned  int num=0,max1=0,max2=0; 
 DWORD* p; 
 GetDlgItemText(IDC_EDIT1,name,-1); 
 len=strlen(name); 
 i=0; 
 for(;i<len;i++) 
 { 
   num=name[i]^key[k]; 
   k++; 
   key[k-1]=name[i]; 
   name[i]=num; 
   if(5==k) 
     k=0; 
 } 
 k=0; 
 for(i=0;i<len;i++) 
 { 
   max1=(BYTE)name[len-i-1]; 
   max2=(BYTE)key[k+5]; 
   num=max1^max2; 
   key[k+5]=max1; 
   name[len-i-1]=num; 
   k++; 
   if(5==k) 
     k=0; 
 } 
 k=0; 
 for(i=0;i<len;i++) 
 { 
   max1=(BYTE)name[i]; 
   max2=(BYTE)key[k+10]; 
   num=max1^max2; 
   key[k+10]=name[i]; 
   name[i]=num; 
   k++; 
   if(5==k) 
     k=0; 
 } 
 k=0; 
 for(i=0;i<len;i++) 
 { 
   max1=(BYTE)name[len-i-1]; 
   max2=(BYTE)key[k+15]; 
   num=max1^max2; 
   key[k+15]=name[len-i-1]; 
   name[len-i-1]=num; 
   k++; 
   if(5==k) 
     k=0; 
 } 
 k=0; 
 for(i=0;i<len;i++) 
 { 
   k=i&3; 
   max1=a[k]; 
   max2=(BYTE)name[i]; 
   num=max1+max2; 
   a[k]=(BYTE)num; 
 } 
 p=(DWORD*)a; 
 sprintf(name,"%d",*p); 
 SetDlgItemText(IDC_EDIT2,name); 
}
```

其实注册机可以写得非常简洁``因为想让大家看得更清楚一点这个算法，所以，原封不动地还原了反汇编代码！

### 经验总结

嘿嘿``原来 `crack` 一个软件也不难嘛！至少这个是这样~~呵呵~

### 版权声明

本文原创于看雪软件安全论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 6 月 6 日 22:28:16

4dd466d0e1e0cc123afbe504da2462ca
