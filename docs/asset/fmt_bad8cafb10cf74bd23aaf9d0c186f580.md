
004DA7EB `|.` 8D>lea eax,dword ptr ss:\[ebp-1050\] `;` 密码表的第 1 组数据
004DA7F1 `|.` 50 push eax `;` /Arg1
004DA7F2 `|.` 8D>lea edx,dword ptr ss:\[ebp-8\] `;` |输入码的后 8（反）
004DA7F5 `|.` 8D>lea eax,dword ptr ss:\[ebp-4\] `;` |输入码的前 8（反）
004DA7F8 `|.` 8D>lea ecx,dword ptr ss:\[ebp-50\] `;` |最后 18 位的头地址
004DA7FB `|.` E8>call unpacked.004DA6BC `;` \\注册码处理的关键地方！！
004DA800 `|.` 8B>mov eax,ebx
004DA802 `|.` 8D>lea edx,dword ptr ds:\[ebx+4\]
004DA805 `|.` 8B>mov ecx,dword ptr ss:\[ebp-4\]
004DA808 `|.` 89>mov dword ptr ds:\[eax\],ecx `;` 将计算出来的结果赋值
004DA80A `|.` 8B>mov eax,dword ptr ss:\[ebp-8\]
004DA80D `|.` 89>mov dword ptr ds:\[edx\],eax `;` 将计算出来的结果赋值
004DA80F `|.` 5F pop edi
004DA810 `|.` 5E pop esi
004DA811 `|.` 5B pop ebx
004DA812 `|.` 8B>mov esp,ebp
004DA814 `|.` 5D pop ebp
004DA815 `\\.` C2>retn 4

---

进入 `004DA7FB` `|.` `E8>call unpacked.004DA6BC` `;` \\注册码处理的关键地方

这部分，有三处是一样的，只是分成两组数据（共 16 位，分前 8，后 8，前 8 分别拆成 2 位 2 位，进行取值

运算来改变后 8，然后交换后 8 改变前 8

---

004DA6BC `/$` 55 push ebp `;` 对注册码的处理的地方！！！
004DA6BD `|.` 8B>mov ebp,esp
004DA6BF `|.` 81>add esp,-0FFC
004DA6C5 `|.` 50 push eax
004DA6C6 `|.` 83>add esp,-54
004DA6C9 `|.` 53 push ebx
004DA6CA `|.` 56 push esi
004DA6CB `|.` 57 push edi
004DA6CC `|.` 8B>mov esi,dword ptr ss:\[ebp+8\]
004DA6CF `|.` 8D>lea edi,dword ptr ss:\[ebp-1054\]
004DA6D5 `|.` 51 push ecx
004DA6D6 `|.` B9>mov ecx,400
004DA6DB `|.` F3>rep movs dword ptr es:\[edi\],dword ptr d `;` 拷贝 1024 组密码表数据不包括前 18 组
004DA6DD `|.` 59 pop ecx
004DA6DE `|.` 8B>mov esi,ecx
004DA6E0 `|.` 8D>lea edi,dword ptr ss:\[ebp-54\]
004DA6E3 `|.` B9>mov ecx,12
004DA6E8 `|.` F3>rep movs dword ptr es:\[edi\],dword ptr d `;` 拷贝前 18 位密码表数据放在上面密码表的后面
004DA6EA `|.` 89>mov dword ptr ss:\[ebp-8\],edx
004DA6ED `|.` 89>mov dword ptr ss:\[ebp-4\],eax
004DA6F0 `|.` 8B>mov ebx,dword ptr ss:\[ebp-4\]
004DA6F3 `|.` 8B>mov ebx,dword ptr ds:\[ebx\] `;` 前 8 位注册码（反）
004DA6F5 `|.` 8B>mov edi,dword ptr ss:\[ebp-8\]
004DA6F8 `|.` 8B>mov edi,dword ptr ds:\[edi\] `;` 后 8 位注册码（反）
004DA6FA `|.` C6>mov byte ptr ss:\[ebp-9\],0F0
004DA6FE `|.` 8D>lea esi,dword ptr ss:\[ebp-10\]
004DA701 `>` 33>/xor ebx,dword ptr ds:\[esi\] `;` 取最后的密码表数据 60CBF06A 与前 8 位异或
004DA703 `|.` 8D>|lea edx,dword ptr ss:\[ebp-1054\] `;` 取密码表
004DA709 `|.` 8B>|mov eax,ebx
004DA70B `|.` E8>|call unpacked.004DA5E4 `;` 把异或后的值在密码表中取值运算结果 EAX 中
004DA710 `|.` 33>|xor edi,eax `;` 运算的结果与注册码的后 8 位（反着放)异或
004DA712 `|.` 8B>|mov eax,ebx `;` 又取得前 8 位异或得到的值
004DA714 `|.` 8B>|mov ebx,edi `;` 刚才的结果给了 EBX
004DA716 `|.` 8B>|mov edi,eax
004DA718 `|.` 83>|sub esi,4 `;` 取上一个值用于异或的值
004DA71B `|.` FE>|inc byte ptr ss:\[ebp-9\] `;` 一共循环 16 次
004DA71E `|.^\` 75>\\jnz short unpacked.004DA701
004DA720 `|.` 8B>mov eax,ebx
004DA722 `|.` 8B>mov ebx,edi
004DA724 `|.` 8B>mov edi,eax
004DA726 `|.` 33>xor edi,dword ptr ss:\[ebp-50\] `;` 与密码表中的第 2 组数据 D9DB2B90 异或得到最终结果
004DA729 `|.` 33>xor ebx,dword ptr ss:\[ebp-54\] `;` 与密码表中的第 1 组数据 6A113CAB 异或得到标志位结果
004DA72C `|.` 8B>mov eax,dword ptr ss:\[ebp-4\]
004DA72F `|.` 89>mov dword ptr ds:\[eax\],ebx `;` 存储了
004DA731 `|.` 8B>mov eax,dword ptr ss:\[ebp-8\]
004DA734 `|.` 89>mov dword ptr ds:\[eax\],edi `;` 存储
004DA736 `|.` 5F pop edi
004DA737 `|.` 5E pop esi
004DA738 `|.` 5B pop ebx
004DA739 `|.` 8B>mov esp,ebp
004DA73B `|.` 5D pop ebp
004DA73C `\\.` C2>retn 4

---

进入 取值运算 `004DA70B` `|.` `E8>|call unpacked.004DA5E4`

---

004DA604 `|.` 8A>mov dl,byte ptr ds:\[eax+3\] `;` 取刚才异或的的前 2 位
004DA607 `|.` 8B>mov edx,dword ptr ss:\[esp+edx\*4+4\] `;` 取值
004DA60B `|.` 33>xor ecx,ecx
004DA60D `|.` 8A>mov cl,byte ptr ds:\[eax+2\] `;` 取刚才异或的的 3，4 位
004DA610 `|.` 03>add edx,dword ptr ss:\[esp+ecx\*4+404\] `;` 取值加上原来的值
004DA617 `|.` 33>xor ecx,ecx
004DA619 `|.` 8A>mov cl,byte ptr ds:\[eax+1\] `;` 取刚才异或的的 5，6 位
004DA61C `|.` 33>xor edx,dword ptr ss:\[esp+ecx\*4+804\] `;` 取值后异或原来的值
004DA623 `|.` 0F>movzx eax,byte ptr ds:\[eax\] `;` 取刚才异或的的 7，8
004DA626 `|.` 03>add edx,dword ptr ss:\[esp+eax\*4+C04\] `;` 取值加上原来的值
004DA62D `|.` 8B>mov eax,edx
004DA62F `|.` 81>add esp,1004
004DA635 `|.` 5F pop edi
004DA636 `|.` 5E pop esi
004DA637 `\\.` C3 retn

---

进入 `004DAD5A` `|.` `E8>call unpacked.004DA8A4` `;` 处理用户名的地方！

---

004DA8A4 `/$` 55 push ebp
004DA8A5 `|.` 8B>mov ebp,esp
004DA8A7 `|.` 81>add esp,-110
004DA8AD `|.` 53 push ebx
004DA8AE `|.` 56 push esi
004DA8AF `|.` 89>mov dword ptr ss:\[ebp-C\],ecx
004DA8B2 `|.` 89>mov dword ptr ss:\[ebp-8\],edx
004DA8B5 `|.` 89>mov dword ptr ss:\[ebp-4\],eax
004DA8B8 `|.` 8B>mov eax,dword ptr ss:\[ebp-8\]
004DA8BB `|.` E8>call unpacked.004042D4 `;` 没什么
004DA8C0 `|.` 33>xor eax,eax
004DA8C2 `|.` 55 push ebp
004DA8C3 `|.` 68>push unpacked.004DAA41
004DA8C8 `|.` 64>push dword ptr fs:\[eax\]
004DA8CB `|.` 64>mov dword ptr fs:\[eax\],esp
004DA8CE `|.` 8B>mov eax,dword ptr ss:\[ebp-8\]
004DA8D1 `|.` E8>call unpacked.00404120 `;` 得到用户名位数
004DA8D6 `|.` 8B>mov ebx,eax `;` 位数
004DA8D8 `|.` 85>test ebx,ebx
004DA8DA `|.` 75>jnz short unpacked.004DA8E3
004DA8DC `|.` 33>xor ebx,ebx
004DA8DE `|.` E9>jmp unpacked.004DAA2B
004DA8E3 `>` B9>mov ecx,unpacked.005E6A7C
004DA8E8 `|.` BA>mov edx,unpacked.005E6A34
004DA8ED `|.` A1>mov eax,dword ptr ds:\[5E36E8\] `;` 得到一串字符串 另一个
004DA8F2 `|.` E8>call unpacked.004DA3EC `;` 将字符串处理成为密码表
004DA8F7 `|.` A1>mov eax,dword ptr ds:\[5E36F0\]
004DA8FC `|.` 03>add eax,ebx
004DA8FE `|.` 48 dec eax
004DA8FF `|.` 99 cdq
004DA900 `|.` F7>idiv dword ptr ds:\[5E36F0\] `;` 除以 8
004DA906 `|.` 89>mov dword ptr ss:\[ebp-10\],eax `;` 得到商
004DA909 `|.` 8B>mov eax,dword ptr ss:\[ebp-10\]
004DA90C `|.` F7>imul dword ptr ds:\[5E36F0\] `;` 商乘以 8
004DA912 `|.` 8B>mov ebx,eax
004DA914 `|.` 4B dec ebx `;` 位数减 1
004DA915 `|.` 85>test ebx,ebx
004DA917 `|.` 7C>jl short unpacked.004DA927
004DA919 `|.` 43 inc ebx
004DA91A `|.` 8D>lea eax,dword ptr ss:\[ebp-110\] `;` 得到密码表地址
004DA920 `>` C6>/mov byte ptr ds:\[eax\],0 `;` 密码表前 16 个（前 4 组）为 0
004DA923 `|.` 40 |inc eax
004DA924 `|.` 4B |dec ebx
004DA925 `|.^\` 75>\\jnz short unpacked.004DA920
004DA927 `>` 8D>lea eax,dword ptr ss:\[ebp-110\]
004DA92D `|.` B9>mov ecx,0FF
004DA932 `|.` 8B>mov edx,dword ptr ss:\[ebp-8\] `;` 用户名地址
004DA935 `|.` E8>call unpacked.0040A4B0 `;` 拷贝一下用户名
004DA93A `|.` 8B>mov ebx,dword ptr ss:\[ebp-10\]
004DA93D `|.` 4B dec ebx
004DA93E `|.` 85>test ebx,ebx
004DA940 `|.` 7C>jl short unpacked.004DA96B
004DA942 `|.` 43 inc ebx
004DA943 `|.` 33>xor esi,esi
004DA945 `>` 8D>/lea eax,dword ptr ss:\[ebp-110\] `;` 用户名地址
004DA94B `|.` 8B>|mov edx,esi
004DA94D `|.` 0F>|imul edx,dword ptr ds:\[5E36F0\] `;` 乘以 8
004DA954 `|.` 03>|add eax,edx
