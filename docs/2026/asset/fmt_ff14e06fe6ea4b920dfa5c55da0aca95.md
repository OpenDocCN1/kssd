
4、取五十进制数`RSuhU`的`h`位，查表，发现五十进制的`h`对应十进制的`6`。

用`6`乘以`50`的3次方，得到`750000`。

5、取五十进制数`RSuhU`的`U`位，查表，发现五十进制的`U`对应十进制的`39`。

用`39`乘以`50`的4次方，得到`243750000`。

将上述5个乘积累加，得到`244541886`。十进制数`244541886`的十六进制形式为`0E9369BE`，故五十进制数`RSuhU`对应的十六进制数为`0E9369BE`（当然前提是遵循Opera的作者所制定的转换规则，即使用上面的那张表）。

同理可把`8bbTm`、`taXah`、`v3uMt`、`EDrKv`转换为对应的十六进制数：`04023177`、`023DCE97`、`05D430D9`、`068EFB70`。

除了转换之外，它还限制转换后的十六进制数不得大于`0x0FFFFFFF`，即最高4 bit必须为0，这点在后面会用到。

``` 
:004F3F39 8B4508                  mov eax, dword ptr [ebp+08]
:004F3F3C 8A0E                    mov cl, byte ptr [esi]          //注册码的第1部分
:004F3F3E 8365FC00                and dword ptr [ebp-04], 00000000
:004F3F42 8D7E02                  lea edi, dword ptr [esi+02]
:004F3F45 8808                    mov byte ptr [eax], cl          //把注册码的第1部分搬到另外的地方
:004F3F47 83C004                  add eax, 00000004
:004F3F4A 894508                  mov dword ptr [ebp+08], eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F3FA7(C)
|
:004F3F4D 33DB                    xor ebx, ebx                    //累加和清0
:004F3F4F C7450C01000000          mov [ebp+0C], 00000001          //50的0次方
:004F3F56 33F6                    xor esi, esi                    //循环控制变量清0
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F3F8A(C)
|
:004F3F58 0FBE0437                movsx eax, byte ptr [edi+esi]   //取五十进制数的一位
:004F3F5C 50                      push eax
:004F3F5D FF3584D35300            push dword ptr [0053D384]
:004F3F63 E888AA0100              call 0050E9F0                   //查表
:004F3F68 59                      pop ecx
:004F3F69 85C0                    test eax, eax
:004F3F6B 59                      pop ecx
:004F3F6C 743F                    je 004F3FAD
:004F3F6E 2A0584D35300            sub al, byte ptr [0053D384]
:004F3F74 0FB6C0                  movzx eax, al                   //查表所得的下标
:004F3F77 0FAF450C                imul eax, dword ptr [ebp+0C]    //乘以50的n次方
:004F3F7B 03D8                    add ebx, eax                    //累加到ebx中
:004F3F7D 8B450C                  mov eax, dword ptr [ebp+0C]
:004F3F80 6BC032                  imul eax, 00000032              //50的n次方再乘以50
:004F3F83 46                      inc esi                         //循环控制变量++
:004F3F84 89450C                  mov dword ptr [ebp+0C], eax
:004F3F87 83FE05                  cmp esi, 00000005               //5位全转换完了？
:004F3F8A 7CCC                    jl 004F3F58                     //内循环
:004F3F8C F7C3000000F0            test ebx, F0000000              //转换结果不得大于0x0FFFFFFF
:004F3F92 7519                    jne 004F3FAD                    //jump if bad guy
:004F3F94 8B4508                  mov eax, dword ptr [ebp+08]
:004F3F97 FF45FC                  inc [ebp-04]
:004F3F9A 83450804                add dword ptr [ebp+08], 00000004
:004F3F9E 83C706                  add edi, 00000006
:004F3FA1 837DFC05                cmp dword ptr [ebp-04], 00000005 //注册码的后5部分全转换完了？
:004F3FA5 8918                    mov dword ptr [eax], ebx
:004F3FA7 7CA4                    jl 004F3F4D                     //外循环
:004F3FA9 32C0                    xor al, al                      //good guy
:004F3FAB EB02                    jmp 004F3FAF
```

经过上面的转换，注册码的后5部分被转换成了5个十六进制数。显然上面的转换是可逆的，即给定一个十六进制数，我们可以写出其五十进制的表示。这对写注册机是有用的。下面我们继续看它如何处理这5个数。用数组`a[5]`表示这5个数，则下面的汇编代码可以翻译为对应的C程序：

```
for (k = 0; k < 4; k++)
{
    a[k] ^= a[k+1];
}
```

``` 
* Referenced by a CALL at Address:
|:004174A3
|
:004174F3 8B442404                mov eax, dword ptr [esp+04]
:004174F7 56                      push esi
:004174F8 6A04                    push 00000004                  //循环4次
:004174FA 83C004                  add eax, 00000004
:004174FD 5A                      pop edx
:004174FE 8B7004                  mov esi, dword ptr [eax+04]    //取出a[k+1]
:00417501 8D4804                  lea ecx, dword ptr [eax+04]
:00417504 3130                    xor dword ptr [eax], esi      //与a[k]异或
:00417506 4A                      dec edx
:00417507 8BC1                    mov eax, ecx
:00417509 75F3                    jne 004174FE                  //循环
:0041750B 32C0                    xor al, al
:0041750D 5E                      pop esi
:0041750E C3                      ret
```

由于异或运算是可逆的，因此上面的变换也可逆。我们继续看它如何处理异或过的这5个数。接下来将是一个大的异或变换，不过不用担心，这个变换仍是可逆的。如下：

``` 
* Referenced by a CALL at Address:
|:004174B4
|
:004174C4 56                      push esi
:004174C5 57                      push edi
:004174C6 33FF                    xor edi, edi                  //循环控制变量edi清0
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004174EC(C)
|
:004174C8 6A05                    push 00000005
:004174CA 8BC7                    mov eax, edi
:004174CC 33D2                    xor edx, edx
:004174CE 59                      pop ecx
:004174CF F7F1                    div ecx                       //循环控制变量edi对5求余
:004174D1 8B44240C                mov eax, dword ptr [esp+0C]
:004174D5 6A00                    push 00000000
:004174D7 8D749004                lea esi, dword ptr [eax+4*edx+04]
:004174DB E870FFFFFF              call 00417450                 //取常数数列的下一项
:004174E0 25FFFFFF0F              and eax, 0FFFFFFF             //屏蔽掉高4个bit
:004174E5 59                      pop ecx
:004174E6 3106                    xor dword ptr [esi], eax      //与a[edi % 5]异或
:004174E8 47                      inc edi                       //循环控制变量加1
:004174E9 83FF65                  cmp edi, 00000065
:004174EC 72DA                    jb 004174C8                   //继续循环
:004174EE 5F                      pop edi
:004174EF 32C0                    xor al, al
:004174F1 5E                      pop esi
:004174F2 C3                      ret
```

上面的汇编代码可翻译为如下的伪码：

```
for (edi = 0; edi < 0x65; edi++)
{
    NextNumber = GetNumberFromSequence();
    NextNumber &= 0x0FFFFFFF;
    a[edi % 5] ^= NextNumber;
}
```

即用一个常数数列的前`0x65`项来分别与我们在上一步所得到的5个数进行异或。这个循环的异或过程可以等价于：

```
a[0] ^= 常数1；
a[1] ^= 常数2；
a[2] ^= 常数3；
a[3] ^= 常数4；
a[4] ^= 常数5；
```

其中上述5个常数是可以求出来的，因为所用到的常数数列是确定的，跟进`call 00417450`就可以看清常数数列的计算过程。

用`Sequence[n]`表示该数列的第n项，则`call 00417450`的作用就是根据`Sequence[n]`计算出`Sequence[n+1]`来，相当于数列的递推公式，如下所示。

``` 
:0041745C 8B0D50625400            mov ecx, dword ptr [00546250]   //取出Sequence[n]
:00417462 8BC1                    mov eax, ecx
:00417464 8D1409                  lea edx, dword ptr [ecx+ecx]
:00417467 24FE                    and al, FE
:00417469 33C2                    xor eax, edx
:0041746B 8BD1                    mov edx, ecx
:0041746D D1E0                    shl eax, 1
:0041746F 83E2FC                  and edx, FFFFFFFC
:00417472 33C2                    xor eax, edx
:00417474 8BD1                    mov edx, ecx
:00417476 C1E002                  shl eax, 02
:00417479 83E2F0                  and edx, FFFFFFF0
:0041747C 33C2                    xor eax, edx
:0041747E 8BD1                    mov edx, ecx
:00417480 C1E002                  shl eax, 02
:00417483 83E2C0                  and edx, FFFFFFC0
:00417486 33C2                    xor eax, edx
:00417488 8BD1                    mov edx, ecx
:0041748A C1E019                  shl eax, 19
:0041748D 81E200000080            and edx, 80000000
:00417493 33C2                    xor eax, edx
:00417495 D1E9                    shr ecx, 1
:00417497 0BC1                    or eax, ecx
:00417499 A350625400              mov dword ptr [00546250], eax   //保存Sequence[n+1]
:0041749E C3                      ret
```

看一下`DS:[00546250]`中存放的初始值，就知道`Sequence[0] = 0x01A26A75`。根据异或运算的结合律，可以知道：

```
常数1 = (Sequence[1] ^ Sequence[6] ^ ... ^ Sequence[96] ^ Sequence[101]) & 0x0FFFFFFF
常数2 = (Sequence[2] ^ Sequence[7] ^ ... ^ Sequence[97]) & 0x0FFFFFFF
常数3 = (Sequence[3] ^ Sequence[8] ^ ... ^ Sequence[98]) & 0x0FFFFFFF
常数4 = (Sequence[4] ^ Sequence[9] ^ ... ^ Sequence[99]) & 0x0FFFFFFF
常数5 = (Sequence[5] ^ Sequence[10] ^ ... ^ Sequence[100]) & 0x0FFFFFFF
```

经过了上面的循环异或后，我们再来看看它下面是怎么处理的。运算了半天，总该判断一下了吧。

在如下的代码中，它要判断注册码的第2部分。

``` 
* Referenced by a CALL at Address:
|:004F4107
|
:004F3FB4 55                      push ebp
:004F3FB5 8BEC                    mov ebp, esp
:004F3FB7 83EC18                  sub esp, 00000018
:004F3FBA 8B4508                  mov eax, dword ptr [ebp+08]
:004F3FBD 56                      push esi
:004F3FBE 8B750C                  mov esi, dword ptr [ebp+0C]
:004F3FC1 57                      push edi
:004F3FC2 BA000000F0              mov edx, F0000000               //掩码，用来提取一个数的高4bit（即28～31bit）
:004F3FC7 8B4E08                  mov ecx, dword ptr [esi+08]     //取出a[1]
:004F3FCA 89480C                  mov dword ptr [eax+0C], ecx
:004F3FCD 8B4E0C                  mov ecx, dword ptr [esi+0C]     //取出a[2]
:004F3FD0 894810                  mov dword ptr [eax+10], ecx
:004F3FD3 8B4E10                  mov ecx, dword ptr [esi+10]     //取出a[3]
:004F3FD6 894814                  mov dword ptr [eax+14], ecx
:004F3FD9 8B4E14                  mov ecx, dword ptr [esi+14]     //取出a[4]
:004F3FDC 894818                  mov dword ptr [eax+18], ecx
:004F3FDF 8B4E04                  mov ecx, dword ptr [esi+04]     //取出a[0]
:004F3FE2 C1E104
```
