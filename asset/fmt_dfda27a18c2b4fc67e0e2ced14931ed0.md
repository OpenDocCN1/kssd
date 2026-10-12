
### 汇编代码分析

```
004AB9F0     8D45 A8           lea eax,dword ptr ss:[ebp-58]  
004AB9F3     8B0D 903E4D00    mov ecx,dword ptr ds:[4D3E90]        ; "685741366c409b32b56387ffcf1be72a")  
004AB9F9     8B55 E8          mov edx,dword ptr ss:[ebp-18]        ; "59a1f9766f8bea7003f5669b189dc515")  
004AB9FC     E8 0793F5FF      call dumped_.00404D08                ; 连接  
004ABA01     8B45 A8          mov eax,dword ptr ss:[ebp-58]  
004ABA04     8D55 B4          lea edx,dword ptr ss:[ebp-4C]  
004ABA07     E8 CCF0FFFF      call dumped_.004AAAD8                ; MD5  
004ABA0C     8D45 B4          lea eax,dword ptr ss:[ebp-4C]  
004ABA0F     8D55 E4          lea edx,dword ptr ss:[ebp-1C]  
004ABA12     E8 35F1FFFF      call dumped_.004AAB4C  
004ABA17     8D45 A4          lea eax,dword ptr ss:[ebp-5C]  
004ABA1A     8B0D 3C404D00    mov ecx,dword ptr ds:[4D403C]        ; "d83aa6e66c68a865110c2f51e8495ced")  
004ABA20     8B55 E4          mov edx,dword ptr ss:[ebp-1C]        ; "54528f8555c8f0a3c3e44bad293df596")  
004ABA23     E8 E092F5FF      call dumped_.00404D08  
004ABA28     8B45 A4          mov eax,dword ptr ss:[ebp-5C]  
004ABA2B     8D55 B4          lea edx,dword ptr ss:[ebp-4C]  
004ABA2E     E8 A5F0FFFF      call dumped_.004AAAD8                ; MD5  
004ABA33     8D45 B4          lea eax,dword ptr ss:[ebp-4C]  
004ABA36     8D55 E0          lea edx,dword ptr ss:[ebp-20]  
004ABA39     E8 0EF1FFFF      call dumped_.004AAB4C  
004ABA3E     8D45 A0          lea eax,dword ptr ss:[ebp-60]  
004ABA41     8B0D 7C414D00    mov ecx,dword ptr ds:[4D417C]        ; "d6cdf360ef1fbe03d6c5f5b27fc0cb59")  
004ABA47     8B55 E0          mov edx,dword ptr ss:[ebp-20]        ;  "485be39a626dfb63b41cb5dd6d4962db")  
004ABA4A     E8 B992F5FF      call dumped_.00404D08  
004ABA4F     8B45 A0          mov eax,dword ptr ss:[ebp-60]  
004ABA52     8D55 B4          lea edx,dword ptr ss:[ebp-4C]  
004ABA55     E8 7EF0FFFF      call dumped_.004AAAD8                ; MD5  
004ABA5A     8D45 B4          lea eax,dword ptr ss:[ebp-4C]  
004ABA5D     8D55 DC          lea edx,dword ptr ss:[ebp-24]  
004ABA60     E8 E7F0FFFF      call dumped_.004AAB4C  
004ABA65     8D4D 9C          lea ecx,dword ptr ss:[ebp-64]  
004ABA68     BA 01000000      mov edx,1  
004ABA6D     8B45 EC          mov eax,dword ptr ss:[ebp-14]        ; "0bc71bcce533bff7ee6fa25cc5a6fa3b")  
004ABA70     E8 BBF1FFFF      call dumped_.004AAC30  
004ABA75     8B55 9C          mov edx,dword ptr ss:[ebp-64]  
004ABA78     8D45 EC          lea eax,dword ptr ss:[ebp-14]  
004ABA7B     E8 1C90F5FF      call dumped_.00404A9C  
004ABA80     8D4D 98          lea ecx,dword ptr ss:[ebp-68]  
004ABA83     BA 01000000      mov edx,1  
004ABA88     8B45 E8          mov eax,dword ptr ss:[ebp-18]        ; "59a1f9766f8bea7003f5669b189dc515")  
004ABA8B     E8 A0F1FFFF      call dumped_.004AAC30  
004ABA90     8B55 98          mov edx,dword ptr ss:[ebp-68]  
004ABA93     8D45 E8          lea eax,dword ptr ss:[ebp-18]  
004ABA96     E8 0190F5FF      call dumped_.00404A9C  
004ABA9B     8D4D 94          lea ecx,dword ptr ss:[ebp-6C]  
004ABA9E     BA 01000000      mov edx,1  
004ABAA3     8B45 E4          mov eax,dword ptr ss:[ebp-1C]        ; "54528f8555c8f0a3c3e44bad293df596")  
004ABAA6     E8 85F1FFFF      call dumped_.004AAC30  
004ABAAB     8B55 94          mov edx,dword ptr ss:[ebp-6C]  
004ABAAE     8D45 E4          lea eax,dword ptr ss:[ebp-1C]  
004ABAB1     E8 E68FF5FF      call dumped_.00404A9C  
004ABAB6     8D4D 90          lea ecx,dword ptr ss:[ebp-70]  
004ABAB9     BA 01000000      mov edx,1  
004ABABE     8B45 E0          mov eax,dword ptr ss:[ebp-20]        ; "485be39a626dfb63b41cb5dd6d4962db")  
004ABAC1     E8 6AF1FFFF      call dumped_.004AAC30  
004ABAC6     8B55 90          mov edx,dword ptr ss:[ebp-70]  
004ABAC9     8D45 E0          lea eax,dword ptr ss:[ebp-20]  
004ABACC     E8 CB8FF5FF      call dumped_.00404A9C  
004ABAD1     8D4D 8C          lea ecx,dword ptr ss:[ebp-74]  
004ABAD4     BA 01000000      mov edx,1  
004ABAD9     8B45 DC          mov eax,dword ptr ss:[ebp-24]        ; "07274d30ea3d54bb3435d9f68c657d4e")  
004ABADC     E8 4FF1FFFF      call dumped_.004AAC30  
004ABAE1     8B55 8C          mov edx,dword ptr ss:[ebp-74]  
004ABAE4     8D45 DC          lea eax,dword ptr ss:[ebp-24]  
004ABAE7     E8 B08FF5FF      call dumped_.00404A9C  
004ABAEC     FF75 EC          push dword ptr ss:[ebp-14]  
004ABAEF     FF75 E8          push dword ptr ss:[ebp-18]  
004ABAF2     FF75 E4          push dword ptr ss:[ebp-1C]  
004ABAF5     FF75 E0          push dword ptr ss:[ebp-20]  
004ABAF8     FF75 DC          push dword ptr ss:[ebp-24]  
004ABAFB     8D45 D8          lea eax,dword ptr ss:[ebp-28]  
004ABAFE     BA 05000000      mov edx,5  
004ABB03     E8 7492F5FF      call dumped_.00404D7C  
004ABB08     8D4D CC          lea ecx,dword ptr ss:[ebp-34]  
004ABB0B     BA 05000000      mov edx,5  
004ABB10     8B45 C8          mov eax,dword ptr ss:[ebp-38]  
004ABB13     E8 18F1FFFF      call dumped_.004AAC30               ; 取注册码前 5 位＝87654  
004ABB18     8D4D D4          lea ecx,dword ptr ss:[ebp-2C]  
004ABB1B     BA 03000000      mov edx,3  
004ABB20     8B45 C8          mov eax,dword ptr ss:[ebp-38]  
004ABB23     E8 60F1FFFF      call dumped_.004AAC88               ; ？？  
004ABB28     8D4D 84          lea ecx,dword ptr ss:[ebp-7C]  
004ABB2B     BA 1E000000      mov edx,1E  
004ABB30     8B45 C8          mov eax,dword ptr ss:[ebp-38]  
004ABB33     E8 F8F0FFFF      call dumped_.004AAC30               ; 取注册码前 30 位  
004ABB38     8B45 84          mov eax,dword ptr ss:[ebp-7C]  
004ABB3B     8D55 B4          lea edx,dword ptr ss:[ebp-4C]  
004ABB3E     E8 95EFFFFF      call dumped_.004AAAD8  
004ABB43     8D45 B4          lea eax,dword ptr ss:[ebp-4C]  
004ABB46     8D55 88          lea edx,dword ptr ss:[ebp-78]  
004ABB49     E8 FEEFFFFF      call dumped_.004AAB4C               ; MD5（注册码前 30 位）  
004ABB4E     8B45 88          mov eax,dword ptr ss:[ebp-78]       ; "5e8667a439c68f5145dd2fcbecf02209")  
004ABB51     8D4D D0          lea ecx,dword ptr ss:[ebp-30]  
004ABB54     BA 03000000      mov edx,3  
004ABB59     E8 D2F0FFFF      call dumped_.004AAC30               ; ？？  
004ABB5E     8B45 D8          mov eax,dword ptr ss:[ebp-28]       ; 05540  
004ABB61     8B55 CC          mov edx,dword ptr ss:[ebp-34]       ; 87654  
004ABB64     E8 9792F5FF      call dumped_.00404E00  
004ABB69     75 11           jnz short dumped_.004ABB7C  
004ABB6B     8B45 D4          mov eax,dword ptr ss:[ebp-2C]       ; 321  
004ABB6E     8B55 D0          mov edx,dword ptr ss:[ebp-30]       ; 5e8  
004ABB71     E8 8A92F5FF      call dumped_.00404E00  
004ABB76     75 04           jnz short dumped_.004ABB7C  
004ABB78     C645 F7 01       mov byte ptr ss:[ebp-9],1            ; 它的赋值决定返回后的跳转  
004ABB7C     33C0            xor eax,eax  
004ABB7E     5A             pop edx  
004ABB7F     59             pop ecx  
004ABB80     59             pop ecx  
004ABB81     64:8910        mov dword ptr fs:[eax],edx  
004ABB84     68 91BB4A00    push dumped_.004ABB91  
004ABB89     C3             retn  
```

流程很明显，都是 `MD5` 的运算，参与运算的两个参数，一个用户名 `gao@263.com`，一个是注册码（因为注册码以“－”号形式连接，所以只取“－”号前面的，后面的不参与）。

### 5. 算法总结

#### 5.1 程序对输入的用户名及注册码有要求

用户名大于 6 位，注册码要 33 位或者 39 位。

对用户名进行计算：

- 顺取用户名 `ASCII` 码 `XOR 8 XOR 4`，得到的值转换成字符，连接起来＝ `"6B6D634C3E3A3F226F6361"`
- 头尾转换 `A3F226F63616B6D634C3E3`，随机连接上 2 个字节＝`XXA3F226F63616B6D634C3E3XX`
- 然后连接字符串：`30383B383B3A38383E77`；`C43C3F3C383E3C3C3A06`；和输入的注册码 `87654321-12345678-12345678-761103`：
  `"30383B383B3A38383E77-C43C3F3C383E3C3C3A06-XXA3F226F63616B6D634C3E3XX-87654321-12345678-12345678-761103"`
- 写入注册表。

#### 5.2 从注册表里取出信息

以“－”号连接的，第一第二组为程序加上去的，第 3 组为用户名的计算值，第 4 组为注册码，如果注册码也是以`-`号连接的话只取到`-`号为止。

- 先取第 3 组，还原成用户名；再取第 4 组注册码（只取“－”号前的字符）；
- 用户名与字符串 `9af19b63a9db476829d7f04d9632fd56` 连接：
