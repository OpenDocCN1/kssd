
```
0101DA5F    FF53 20         call    [ebx+20]                  ; 取得输入的字符串，返回长度

0101DA62    85C0            test    eax, eax

0101DA64    0F84 97000000   je      0101DB01

0101DA6A    8BF8            mov     edi, eax

0101DA6C    8D55 F0         lea     edx, [ebp-10]

0101DA6F    52              push    edx                       ; 参数3：

0101DA70    50              push    eax                       ; 参数2：输入的长度，7

0101DA71    FF73 04         push    dword ptr [ebx+4]         ; 参数1：输入的字符串

0101DA74  FF53 0C       call   [ebx+C]                ；单向的加密函数

0101DA77    8D8E B8B5B200   lea     ecx, [esi+B2B5B8]         ; ecx 为标准的密文

0101DA7D    C745 CC 0400000>mov     dword ptr [ebp-34], 4     ; 存放比较次数

0101DA84    8D55 F0         lea     edx, [ebp-10]             ; edx为unsigned数组，长度4

0101DA87    8B02            mov     eax, [edx]                ；这里开始的3句nop掉

0101DA89    3901            cmp     [ecx], eax                ；修改为mov eax, [ecx]；mov [edx], eax

0101DA8B    75 3E           jnz     short 0101DACB            ；这样虽然可以把密文盗窃过来，但还是不能解决问题

0101DA8D    83C2 04         add     edx, 4

0101DA90    83C1 04         add     ecx, 4

0101DA93    FF4D CC         dec     dword ptr [ebp-34]        ; 比较的次数减1

0101DA96  ^ 75 EF           jnz     short 0101DA87            ; 继续比较（这7个指令的功能是比较密文）

0101DA98    8D45 D0         lea     eax, [ebp-30]

0101DA9B    50              push    eax

0101DA9C    57              push    edi

0101DA9D    FF73 04         push    dword ptr [ebx+4]

0101DAA0    FF53 58         call    [ebx+58]

0101DAA3    8DBE B7BAB200   lea     edi, [esi+B2BAB7]

0101DAA9    B9 04000000     mov     ecx, 4

0101DAAE    8D55 D0         lea     edx, [ebp-30]

0101DAB1    8B02            mov     eax, [edx]

0101DAB3    3342 04         xor     eax, [edx+4]

0101DAB6    3107            xor     [edi], eax

0101DAB8    83C7 04         add     edi, 4

0101DABB    83C2 08         add     edx, 8

0101DABE    49              dec     ecx

0101DABF  ^ 75 F0           jnz     short 0101DAB1

0101DAC1    33C0            xor     eax, eax

0101DAC3    40              inc     eax

0101DAC4    5F              pop     edi

0101DAC5    5B              pop     ebx

0101DAC6    5E              pop     esi

0101DAC7    C9              leave

0101DAC8    C2 0400         retn    4
```

(12)再次绝望了,彻底的绝望

软件需要输入正确的密码才能正确执行，否则立即结束

验证密码正确性的算法为单向的（MD5，RSA）之类算法，所以在软件中只出现明文

代码的加密/解密算法是分组密码算法，需要正确的密码才能解密

这个问题，从理论上都是无法解决的。

带着希望来，带着失望去。

有了希望，可以风雨无阻；

没了希望，人生就走到了终点~~

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`obsidium_1.0.0.69_demo_anlayse.rar`](javascript:void(0);)

  （112.76kb，21次下载）
