
### 汇编代码段 1

```
CODE:004D5A47                     pop     ecx  
CODE:004D5A48                     mov     fs:[eax], edx  
CODE:004D5A4B                     jmp     short loc_4D5A66  
CODE:004D5A4D                     jmp     @System@@HandleAnyException$qqrv ;  
CODE:004D5A52                     mov     ebx, 0C9h  
CODE:004D5A57                     call    @@DoneExcept$qqrv  
CODE:004D5A5C                     jmp     loc_4D5B02  
CODE:004D5A61                     call    @@DoneExcept$qqrv  
CODE:004D5A66                     test    esi, esi        //测试 esi 是否为 0  
CODE:004D5A68                     jnz     short loc_4D5A95 //如果不是则出错，后面程序总是返回 0C9H。  
CODE:004D5A6A                     lea     eax, [ebp+var_4]  
CODE:004D5A6D                     push    eax  
CODE:004D5A6E                     mov     edx, [ebx+6Ch]  
CODE:004D5A71                     mov     eax, [ebx+34h]  
CODE:004D5A74                     mov     ecx, esi  
CODE:004D5A76                     call    sub_4D36CC  
CODE:004D5A7B                     mov     eax, [ebp+var_4]   
CODE:004D5A7E                     mov     edx, edi  
CODE:004D5A80                     call    @System@@LStrCmp$qqrv ; //在这里 d eax 看到真注册码，d edi 看到假码。此 CALL 就是关键，为了看清如何比较，我也把这个 CALL 代码放在下面了。不过没有多大意义 了。：）  
CODE:004D5A85                     jnz     short loc_4D5A8E  
CODE:004D5A87                     mov     ebx, 0C8h    //如果注册码正确，则 ebx=C8H  
CODE:004D5A8C                     jmp     short loc_4D5B02  
```

*略掉 10 多行*

```
CODE:004D5AB0                     jnz     short loc_4D5AFD  
CODE:004D5AB2                     inc     dword ptr [ebx+44h]  
CODE:004D5AB5                     lea     edx, [ebp+var_8]  
CODE:004D5AB8                     mov     eax, ebx  
CODE:004D5ABA                     call    sub_4D6A94  
CODE:004D5ABF                     mov     edx, [ebp+var_8]  
CODE:004D5AC2                     lea     eax, [ebx+34h]  
```

========================================

### 4. 在 `004D5A80` 设断点，`F8` 跟入：

```
CODE:004041F4                     push    ebx  
CODE:004041F5                     push    esi  
CODE:004041F6                     push    edi  
CODE:004041F7                     mov     esi, eax  
CODE:004041F9                     mov     edi, edx  
CODE:004041FB                     cmp     eax, edx  
CODE:004041FD                     jz      loc_404292  
CODE:00404203                     test    esi, esi  
CODE:00404205                     jz      short loc_40426F  
CODE:00404207                     test    edi, edi  
CODE:00404209                     jz      short loc_404276  
CODE:0040420B                     mov     eax, [esi-4]  
CODE:0040420E                     mov     edx, [edi-4]  
CODE:00404211                     sub     eax, edx           //计算真假注册码的长度之差  
CODE:00404213                     ja      short loc_404217  
CODE:00404215                     add     edx, eax  
CODE:00404217                     push    edx  
CODE:00404218                     shr     edx, 2  
CODE:0040421B                     jz      short loc_404243  
CODE:0040421D                     mov     ecx, [esi]  
CODE:0040421F                     mov     ebx, [edi]  
CODE:00404221                     cmp     ecx, ebx    //此处 d esi 看到真注册码，d edi 看到你的假注册码！  
CODE:00404223                     jnz     short loc_40427D  
CODE:00404225                     dec     edx  
CODE:00404226                     jz      short loc_40423D  
CODE:00404228                     mov     ecx, [esi+4]  
CODE:0040422B                     mov     ebx, [edi+4] //比较真假注册码  
CODE:0040422E                     cmp     ecx, ebx  
CODE:00404230                     jnz     short loc_40427D  
CODE:00404232                     add     esi, 8  
CODE:00404235                     add     edi, 8  
CODE:00404238                     dec     edx  
CODE:00404239                     jnz     short loc_40421D  //没有结束，返回上面，继续比较。  
CODE:0040423B                     jmp     short loc_404243  
CODE:0040423D                     add     esi, 4  
CODE:00404240                     add     edi, 4  
CODE:00404243                     pop     edx  
CODE:00404244                     and     edx, 3  
CODE:00404247                     jz      short loc_40426B  //此处自动跳走，不要修改 Z 标志。  
CODE:00404249                     mov     ecx, [esi]  
CODE:0040424B                     mov     ebx, [edi]  
CODE:0040424D                     cmp     cl, bl  
CODE:0040424F                     jnz     short loc_404292  
CODE:00404251                     dec     edx  
CODE:00404252                     jz      short loc_40426B  
CODE:00404254                     cmp     ch, bh  
CODE:00404256                     jnz     short loc_404292  
CODE:00404258                     dec     edx  
CODE:00404259                     jz      short loc_40426B  
```

*略掉 10 多行*

```
CODE:0040428C                     cmp     cl, bl  
CODE:0040428E                     jnz     short loc_404292  
CODE:00404290                     cmp     ch, bh  
CODE:00404292                     pop     edi  
CODE:00404293                     pop     esi  
CODE:00404294                     pop     ebx  
CODE:00404295                     retn  
```

========================================

### 后记：

1. 本来我已经正确找到了 3.8 版的注册码了，我又安装了这个新版 4.1 实验一下找注册码，结果发现这个 4.1 版用 `ASPACK2.11C` 加壳了，我用 `ASPACK DIE` 脱壳成功，可是无法用 `W32DASM` 打开查看，因为打开后只有前面的文件头部分，没有代码部分。我用 `IDA` 打开可以看到代码。

2. 在输入注册码的时候，必须使最后 2 位是 `0`，我就是因为忽视了这个问题，最后实验了好多次总是返回 `CA`，感到纳闷，后来发现一个 `CALL` 就是验证这个最后 2 个位的。

3. 如果想让原来注册码作废，可以重新输入用户名，其他的系列号和注册码即可使原先正确的注册码作废。便于继续深入调试。

4. 注册信息如下：

   - `User Name`: `bigpig`
   - `Serial Number`: `123456`
   - `Register Number`: `2E1F0C4F1B00` （注册码好像与系列号和用户名无关？）

5. 同样的程序可能在不同机器上的地址是不同的，菜鸟在学习的时候不要用文章中显示的地址去和你电脑上出现的实际地址比较，您应该关注代码是否相同才行。在不同电脑上，同样的代码可能地址不同。

耽误您的时间阅读，非常感谢！讨论技术问题 EMAIL TO:

`qduwg@163.com`

2005/12/30

`4dd466d0e1e0cc123afbe504da2462ca`
