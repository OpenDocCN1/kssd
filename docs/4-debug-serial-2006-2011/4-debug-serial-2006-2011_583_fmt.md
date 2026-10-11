
# iovsoft Blu-ray Copy 算法分析

> 原文：[`www.pediy.com/kssd/pediy12/132860.html`](https://www.pediy.com/kssd/pediy12/132860.html)

这些时间一直在玩 `CrackMe` 学习算法分析~，今天找个国外的软件来分析分析算法。

现在不找我国内的软件了~~~~！！！！！！！！！！！！！

因为软件编写是花大量的精神的~~，把他们的算法公布出来不太好~~~

呵呵不多说了大家看分析。

我的用户名是：`wangwei`

我输入的假码是：`123456789`

```
00546FC3      51                push ecx
00546FC4      53                push ebx
00546FC5      56                push esi
00546FC6      57                push edi
00546FC7      8945 FC           mov dword ptr ss:[ebp-4],eax
00546FCA      33C0              xor eax,eax
00546FCC      55                push ebp
00546FCD      68 A6755400       push iovsoft\_.005475A6
00546FD2      64:FF30           push dword ptr fs:[eax]
00546FD5      64:8920           mov dword ptr fs:[eax],esp
00546FD8      C605 90F9BE00 01  mov byte ptr ds:[BEF990],1
00546FDF      8D95 D0FEFFFF     lea edx,dword ptr ss:[ebp-130]
00546FE5      8B45 FC           mov eax,dword ptr ss:[ebp-4]
00546FE8      8B80 F8020000     mov eax,dword ptr ds:[eax+2F8]
00546FEE      E8 E554F0FF       call iovsoft\_.0044C4D8
00546FF3      8B85 D0FEFFFF     mov eax,dword ptr ss:[ebp-130]       ; ASCII "wangwei"
00546FF9      8D55 F4           lea edx,dword ptr ss:[ebp-C]
00546FFC      E8 7730ECFF       call iovsoft\_.0040A078
00547001      8D95 CCFEFFFF     lea edx,dword ptr ss:[ebp-134]
00547007      8B45 F4           mov eax,dword ptr ss:[ebp-C]
0054700A      E8 D530ECFF       call iovsoft\_.0040A0E4
0054700F      8B95 CCFEFFFF     mov edx,dword ptr ss:[ebp-134]
00547015      8D45 F4           lea eax,dword ptr ss:[ebp-C]
00547018      E8 6FDEEBFF       call iovsoft\_.00404E8C
0054701D      8D95 C8FEFFFF     lea edx,dword ptr ss:[ebp-138]
00547023      8B45 FC           mov eax,dword ptr ss:[ebp-4]
00547026      8B80 FC020000     mov eax,dword ptr ds:[eax+2FC]
0054702C      E8 A754F0FF       call iovsoft\_.0044C4D8
00547031      8B85 C8FEFFFF     mov eax,dword ptr ss:[ebp-138]       ; ASCII "123456789"
00547037      8D55 F0           lea edx,dword ptr ss:[ebp-10]
0054703A      E8 3930ECFF       call iovsoft\_.0040A078
0054703F      8D95 C4FEFFFF     lea edx,dword ptr ss:[ebp-13C]
00547045      8B45 F0           mov eax,dword ptr ss:[ebp-10]
00547048      E8 9730ECFF       call iovsoft\_.0040A0E4
0054704D      8B95 C4FEFFFF     mov edx,dword ptr ss:[ebp-13C]
00547053      8D45 F0           lea eax,dword ptr ss:[ebp-10]
00547056      E8 31DEEBFF       call iovsoft\_.00404E8C
0054705B      8D95 C0FEFFFF     lea edx,dword ptr ss:[ebp-140]
00547061      A1 A4F9BE00       mov eax,dword ptr ds:[BEF9A4]
00547066      E8 552BECFF       call iovsoft\_.00409BC0
0054706B      8B95 C0FEFFFF     mov edx,dword ptr ss:[ebp-140]
00547071      B8 C0755400       mov eax,iovsoft\_.005475C0          ; ASCII "123"
00547076      E8 7DE3EBFF       call iovsoft\_.004053F8
0054707B      85C0              test eax,eax
0054707D      7E 4B             jle short iovsoft\_.005470CA
0054707F      837D F0 00        cmp dword ptr ss:[ebp-10],0
00547083      0F85 8E000000     jnz iovsoft\_.00547117
00547089      6A 00             push 0
0054708B      8D8D B8FEFFFF     lea ecx,dword ptr ss:[ebp-148]
00547091      BA CC755400       mov edx,iovsoft\_.005475CC          ; ASCII "Invalid register code! Please retry!"
00547096      B8 01000000       mov eax,1
0054709B      E8 48AA0100       call iovsoft\_.00561AE8
005470A0      8B95 B8FEFFFF     mov edx,dword ptr ss:[ebp-148]
005470A6      8D85 BCFEFFFF     lea eax,dword ptr ss:[ebp-144]
005470AC      E8 CBDFEBFF       call iovsoft\_.0040507C
005470B1      8B85 BCFEFFFF     mov eax,dword ptr ss:[ebp-144]
005470B7      66:8B0D F4755400  mov cx,word ptr ds:[5475F4]
005470BE      B2 02             mov dl,2
005470C0      E8 1BE8EFFF       call iovsoft\_.004458E0
005470C5      E9 EA030000       jmp iovsoft\_.005474B4
005470CA      837D F4 00        cmp dword ptr ss:[ebp-C],0
005470CE      74 06             je short iovsoft\_.005470D6
005470D0      837D F0 00        cmp dword ptr ss:[ebp-10],0
005470D4      75 41             jnz short iovsoft\_.00547117
005470D6      6A 00             push 0
005470D8      8D8D B0FEFFFF     lea ecx,dword ptr ss:[ebp-150]
005470DE      BA CC755400       mov edx,iovsoft\_.005475CC          ; ASCII "Invalid register code! Please retry!"
005470E3      B8 01000000       mov eax,1
005470E8      E8 FBA90100       call iovsoft\_.00561AE8
005470ED      8B95 B0FEFFFF     mov edx,dword ptr ss:[ebp-150]
005470F3      8D85 B4FEFFFF     lea eax,dword ptr ss:[ebp-14C]
005470F9      E8 7EDFEBFF       call iovsoft\_.0040507C
005470FE      8B85 B4FEFFFF     mov eax,dword ptr ss:[ebp-14C]
00547104      66:8B0D F4755400  mov cx,word ptr ds:[5475F4]
0054710B      B2 02             mov dl,2
0054710D      E8 CEE7EFFF       call iovsoft\_.004458E0
00547112      E9 9D030000       jmp iovsoft\_.005474B4
00547117      8D95 A8FEFFFF     lea edx,dword ptr ss:[ebp-158]
0054711D      A1 64835700       mov eax,dword ptr ds:[578364]
00547122      8B00              mov eax,dword ptr ds:[eax]
00547124      E8 4365F2FF       call iovsoft\_.0046D66C
00547129      8B85 A8FEFFFF     mov eax,dword ptr ss:[ebp-158]
0054712F      8D95 ACFEFFFF     lea edx,dword ptr ss:[ebp-154]
00547135      E8 663AECFF       call iovsoft\_.0040ABA0
0054713A      8D85 ACFEFFFF     lea eax,dword ptr ss:[ebp-154]
00547140      BA 00765400       mov edx,iovsoft\_.00547600          ; ASCII "\\Config.ini"
00547145      E8 72DFEBFF       call iovsoft\_.004050BC
0054714A      8B8D ACFEFFFF     mov ecx,dword ptr ss:[ebp-154]
00547150      B2 01             mov dl,1
00547152      A1 B4194700       mov eax,dword ptr ds:[4719B4]
00547157      E8 08A9F2FF       call iovsoft\_.00471A64
0054715C      8BD8              mov ebx,eax
0054715E      6A 00             push 0
00547160      8D45 DC           lea eax,dword ptr ss:[ebp-24]
00547163      50                push eax
00547164      B9 14765400       mov ecx,iovsoft\_.00547614          ; ASCII "ProductID"
00547169      BA 28765400       mov edx,iovsoft\_.00547628          ; ASCII "FirstRun"
0054716E      8BC3              mov eax,ebx
00547170      8B30              mov esi,dword ptr ds:[eax]
00547172      FF16              call dword ptr ds:[esi]
00547174      6A 00             push 0
00547176      8D45 D8           lea eax,dword ptr ss:[ebp-28]
00547179      50                push eax
0054717A      B9 3C765400       mov ecx,iovsoft\_.0054763C          ; ASCII "VerifyWebAddr"
0054717F      BA 28765400       mov edx,iovsoft\_.00547628          ; ASCII "FirstRun"
00547184      8BC3              mov eax,ebx
00547186      8B30              mov esi,dword ptr ds:[eax]
00547188      FF16              call dword ptr ds:[esi]
0054718A      8BC3              mov eax,ebx
0054718C      E8 13CDEBFF       call iovsoft\_.00403EA4
00547191      837D DC 00        cmp dword ptr ss:[ebp-24],0
00547195      74 0A             je short iovsoft\_.005471A1
00547197      837D D8 00        cmp dword ptr ss:[ebp-28],0
0054719B      0F85 D9010000     jnz iovsoft\_.0054737A
005471A1      8B45 F0           mov eax,dword ptr ss:[ebp-10]
005471A4      E8 0BDFEBFF       call iovsoft\_.004050B4
005471A9      8BC8              mov ecx,eax
005471AB      85C9              test ecx,ecx
005471AD      7E 5C             jle short iovsoft\_.0054720B
005471AF      B8 01000000       mov eax,1
005471B4      8B55 F0           mov edx,dword ptr ss:[ebp-10]       ; --------------->>
```



### 反汇编代码分析

```asm
005471B7     0FB65402 FF       movzx edx,byte ptr ds:[edx+eax-1]    ; 检查我们输入的注册码是否为 0-9 的数字
005471BC     83FA 30           cmp edx,30
005471BF     7C 05             jl short iovsoft_.005471C6
005471C1     83FA 39           cmp edx,39
005471C4     7E 41             jle short iovsoft_.00547207
005471C6     6A 00             push 0
005471C8     8D8D A0FEFFFF     lea ecx,dword ptr ss:[ebp-160]
005471CE     BA CC755400       mov edx,iovsoft_.005475CC            ; ASCII "Invalid register code! Please retry!"
005471D3     B8 01000000       mov eax,1
005471D8     E8 0BA90100       call iovsoft_.00561AE8
005471DD     8B95 A0FEFFFF     mov edx,dword ptr ss:[ebp-160]
005471E3     8D85 A4FEFFFF     lea eax,dword ptr ss:[ebp-15C]
005471E9     E8 8EDEEBFF       call iovsoft_.0040507C
005471EE     8B85 A4FEFFFF     mov eax,dword ptr ss:[ebp-15C]
005471F4     66:8B0D F4755400  mov cx,word ptr ds:[5475F4]
005471FB     B2 02             mov dl,2
005471FD     E8 DEE6EFFF       call iovsoft_.004458E0
00547202     E9 AD020000       jmp iovsoft_.005474B4

00547207     40                inc eax
00547208     49                dec ecx
00547209     75 A9             jnz short iovsoft_.005471B4          ; <---------------
0054720B     C745 E0 00000000  mov dword ptr ss:[ebp-20],0
00547212     C745 E4 00000000  mov dword ptr ss:[ebp-1C],0
00547219     8B45 F4           mov eax,dword ptr ss:[ebp-C]         ; ASCII "wangwei"
0054721C     E8 93DEEBFF       call iovsoft_.004050B4              ; 这个 CALL 是取我们的用户名位数相当 strlen()
00547221     8BC8              mov ecx,eax                         ; 把我们用户名的位数给 ECX，
00547223     85C9              test ecx,ecx                        ；测试用户名位是否为空
00547225     7E 1E             jle short iovsoft_.00547245         ；为空就是实现就会提示出错
00547227     BE 01000000       mov esi,1                           ; -----------------》
0054722C     8B45 F4           mov eax,dword ptr ss:[ebp-C]        ； 实现用户名的 ASCII 的累加
0054722F     0FB64430 FF       movzx eax,byte ptr ds:[eax+esi-1]   ；逐位取用户名的 ASCII 给 EAX
00547234     99                cdq                                 ；EDX 清 0
00547235     0345 E0           add eax,dword ptr ss:[ebp-20]       ；ss:[ebp-20]初始值是 0
00547238     1355 E4           adc edx,dword ptr ss:[ebp-1C]
0054723B     8945 E0           mov dword ptr ss:[ebp-20],eax       ；把 EAX+ss:[ebp-20]的值保存到 ss:[ebp-20]中
0054723E     8955 E4           mov dword ptr ss:[ebp-1C],edx
00547241     46                inc esi                             ；ESI++它控制我们取用户名第 N 个字符
00547242     49                dec ecx                             ；ECX--它控制 JNZ 是否实现
00547243     75 E7             jnz short iovsoft_.0054722C         ； <----------------------

00547245     8D45 F8           lea eax,dword ptr ss:[ebp-8]
00547248     8B15 A4F9BE00     mov edx,dword ptr ds:[BEF9A4]
0054724E     E8 39DCEBFF       call iovsoft_.00404E8C
00547253     C745 E8 00000000  mov dword ptr ss:[ebp-18],0
0054725A     C745 EC 00000000  mov dword ptr ss:[ebp-14],0
00547261     8B45 F8           mov eax,dword ptr ss:[ebp-8]
00547264     E8 4BDEEBFF       call iovsoft_.004050B4              ；它取 ASCII "iovsoft Blu-ray Copy"字符串的位数
00547269     8BC8              mov ecx,eax                         ；把它的位数给 ECX
0054726B     85C9              test ecx,ecx
0054726D     7E 1E             jle short iovsoft_.0054728D
0054726F     BE 01000000       mov esi,1
00547274     8B45 F8           mov eax,dword ptr ss:[ebp-8]        ； ASCII "iovsoft Blu-ray Copy"
00547277     0FB64430 FF       movzx eax,byte ptr ds:[eax+esi-1]   ； ----------------->
0054727C     99                cdq                                 ； 实现字符串"iovsoft Blu-ray Copy" ASCII 的累加
0054727D     0345 E8           add eax,dword ptr ss:[ebp-18]
00547280     1355 EC           adc edx,dword ptr ss:[ebp-14]
00547283     8945 E8           mov dword ptr ss:[ebp-18],eax
00547286     8955 EC           mov dword ptr ss:[ebp-14],edx
00547289     46                inc esi
0054728A     49                dec ecx
0054728B     75 E7             jnz short iovsoft_.00547274         ； <-------------------

0054728D     FF75 EC           push dword ptr ss:[ebp-14]
00547290     FF75 E8           push dword ptr ss:[ebp-18]          ； 保存的是 ASCII "iovsoft Blu-ray Copy"自加的结果
00547293     FF75 E4           push dword ptr ss:[ebp-1C]
00547296     FF75 E0           push dword ptr ss:[ebp-20]          ； 保存是用户名自加的结果
00547299     8B45 E8           mov eax,dword ptr ss:[ebp-18]
0054729C     8B55 EC           mov edx,dword ptr ss:[ebp-14]
0054729F     E8 74EEEBFF       call iovsoft_.00406118              ； 用户名的自加结果*实现字符串"iovsoft Blu-ray Copy" ASCII 的累加给 EAX
005472A4     83C0 20           add eax,20                          ； EAX=EAX+20
005472A7     83D2 00           adc edx,0
005472AA     E8 69EEEBFF       call iovsoft_.00406118              ； 把的结果*实现字符串"iovsoft Blu-ray Copy" ASCII 的累加
005472AF     8945 E0           mov dword ptr ss:[ebp-20],eax
005472B2     8955 E4           mov dword ptr ss:[ebp-1C],edx
005472B5     8B45 F0           mov eax,dword ptr ss:[ebp-10]
005472B8     E8 4333ECFF       call iovsoft_.0040A600              ； 把我们输入的 10 进制数转化为 16 进制数
005472BD     3B55 E4           cmp edx,dword ptr ss:[ebp-1C]
005472C0     75 77             jnz short iovsoft_.00547339
005472C2     3B45 E0           cmp eax,dword ptr ss:[ebp-20]       ； EAX=075BCD15 也就 10 进的 123456789，堆栈 ss:[EBP-20]=A5D9A112 就是我们用户名运算的结果
005472C5     75 72             jnz short iovsoft_.00547339         ；这个 JNZ 是关键。 相同就不是实现，哪就注册成功了
005472C7     6A 00             push 0
005472C9     8D8D 98FEFFFF     lea ecx,dword ptr ss:[ebp-168]
005472CF     BA 54765400       mov edx,iovsoft_.00547654           ； ASCII "Congratuation! You have successfully registered!"
005472D4     B8 02000000       mov eax,2
005472D9     E8 0AA80100       call iovsoft_.00561AE8
005472DE     8B95 98FEFFFF     mov edx,dword ptr ss:[ebp-168]
005472E4     8D85 9CFEFFFF     lea eax,dword ptr ss:[ebp-164]
005472EA     E8 8DDDEBFF       call iovsoft_.0040507C
005472EF     8B85 9CFEFFFF     mov eax,dword ptr ss:[ebp-164]
005472F5     66:8B0D F4755400  mov cx,word ptr ds:[5475F4]
005472FC     B2 02             mov dl,2
005472FE     E8 DDE5EFFF       call iovsoft_.004458E0
00547303     C605 A0F9BE00 01  mov byte ptr ds:[BEF9A0],1
0054730A     33C9              xor ecx,ecx
0054730C     33D2              xor edx,edx
0054730E     A1 A8F9BE00       mov eax,dword ptr ds:[BEF9A8]
00547313     8B18              mov ebx,dword ptr ds:[eax]
00547315     FF53 14           call dword ptr ds:[ebx+14]
00547318     BA A0F9BE00       mov edx,iovsoft_.00BEF9A0
0054731D     B9 01000000       mov ecx,1
00547322     A1 A8F9BE00       mov eax,dword ptr ds:[BEF9A8]
00547327     E8 34AFEDFF       call iovsoft_.00422260
0054732C     8B45 FC           mov eax,dword ptr ss:[ebp-4]
0054732F     E8 9C23F2FF       call iovsoft_.004696D0
00547334     E9 7B010000       jmp iovsoft_.005474B4

00547339     6A 00             push 0
0054733B     8D8D 90FEFFFF     lea ecx,dword ptr ss:[ebp-170]
00547341     BA CC755400       mov edx,iovsoft_.005475CC           ； ASCII "Invalid register code! Please retry!"
```

### 算法总结

1. 把用户名逐位取 ASCII 值进行累加结果为 `A`。
2. 把软件名 `iovsoft Blu-ray Copy` 也进行逐位取值累加结果为 `B`。
3. 在让 `A`, `B` 进行下面的运算：(`A` * `B` + `0x20`) * `B` = `C`。
4. 把 `A` 和 `B` 运算的结果 `C` 与我们输入的注册码转化为 16 进制数比较。相同就注册成功。

### C 语言注册机代码

这是我用 C 写的一个简单的算法注册机！！可能没有写的很完整。

```c
#include <stdio.h>

/*
    写一个字串自加函数 zifuadd(char *s)
*/
```



```c
int zifuadd(char *s)  

{  

  int sum = 0;  

  while (*s != '\0')  

  {  

        sum +=*s;  

    s++;  

  }  

  return sum;  

}  

main(void)  

{  

  int up, zp;                   
  
  long mp;  

    char *dp = "iovsoftBlu-rayCopy";    
  
  char name[30];                   //定义一个用户名字符数组用来接收用户名；  

  scanf("%s", &name);  

  up = zifuadd(dp);                    // 自加的结果给 UP  

    zp = zifuadd(name);                  //用户名自加的结果给 ZP  

//    printf("%x\n", up);  

//  printf("%x\n", zp);  

    mp = zp*up + 0X20;                   //  

  mp = mp*up;  

  printf("%X\n", mp);  

}
```
