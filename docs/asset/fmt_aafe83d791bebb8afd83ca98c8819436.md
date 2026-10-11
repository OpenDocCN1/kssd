
## C++ 代码片段

```cpp
    0x2E, 0x7A, 0x66, 0xB3, 0xB8, 0x4A, 0x61, 0xC4,
    0x02, 0x1B, 0x68, 0x5D, 0x94, 0x2B, 0x6F, 0x2A,
    0x37, 0xBE, 0x0B, 0xB4, 0xA1, 0x8E, 0x0C, 0xC3,
    0x1B, 0xDF, 0x05, 0x5A, 0x8D, 0xEF, 0x02, 0x2D,
};

cout<<endl<<"              Iparmor(木马克星)5.33.31/32/34 keygen by TAE!\[CCG\]"<<endl;

cout<<endl<<"Please input your name: ";

cin>>name;

l=strlen(name);

for(i=0;i<l;i++)
    name[i]=toupper(name[i]);

char name1[72];
name1[0]=l;

for (i=0;i<0x47;i++)
{
    name1[i+1]=name[i];
    if (i>=l) name1[i+1]='*';
}

edx=0xEFCA99;
for (i=0;i<0x47;i++)
{
    ebx=edx;
    ebx=ebx>>8;
    ebx=ebx&0x00FFFFFF;
    esi=name1[i];
    edx=edx^esi;
    edx=edx&0x000000FF;
    edx1=table[edx*4];
    edx2=table[edx*4+1];
    edx3=table[edx*4+2];
    edx4=table[edx*4+3];
    edx=edx1+edx2*0x100+edx3*0x10000+edx4*0x1000000;
    ebx=ebx^edx;
    edx=ebx;
}

ebx=ebx+0x4EA;
if (ebx<0){ebx=ebx^0xffffffff;(ebx=ebx-0xffffffff);}

cout<<endl<<"Your registration code: "<<ebx;

getchar();

return 0;
}
```

使用了几次发现这个程序还有几个很黑的暗桩，写出来和大家交流一下

## 汇编代码分析

```asm
:004F6CD2 E8E5C8F0FF            call 004035BC〈--以使用次数为参数的运算

:004F6CD7 83F803                cmp eax, 00000003

:004F6CDA 0F8512080000          jne 004F74F2〈--如出口参数 EAX 不等于 3 就跳过程序再次检测密码

:004F6CE0 8BC3                  mov eax, ebx

:004F6CE2 E8F10C0000            call 004F79D8

:004F6CE7 E906080000            jmp 004F74F2
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|
:004F6C57(C)
|

```asm
:004F6CEC 8D55F4                lea edx, dword ptr [ebp-0C]

:004F6CEF 8B45FC                mov eax, dword ptr [ebp-04]

:004F6CF2 E855FAFFFF            call 004F674C
```

其实程序要经过很多个地方（使用很多种算法）检测密码的，

每一个正确都是会使程序跳到 `004F74F2` 的，

我们从下面这段文字就能看到了

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|
:004F6CDA(C), :004F6CE7(U), :004F6DA1(C), :004F6DAE(U), :004F6E5D(C)
:004F6E6A(U), :004F6F19(C), :004F6F26(U), :004F6FD5(C), :004F6FE2(U)
:004F7091(C), :004F709E(U), :004F7152(C), :004F7158(U), :004F7207(C)
:004F7214(U), :004F72C1(C), :004F72CE(U), :004F737D(C), :004F738A(U)
|

```asm
:004F74F2 33C0                  xor eax, eax

:004F74F4 5A                    pop edx
```

但我们只需要把第一个比较的地方改为跳就可了

也就是把

```asm
:004F6CDA                jne 004F74F2
```

改为

```asm
:004F6CDA                jmp 004F74F2
```

再补上一个 `NOP`（使程序字节数一样）就能破解掉这些暗桩了 `4dd466d0e1e0cc123afbe504da2462ca`
