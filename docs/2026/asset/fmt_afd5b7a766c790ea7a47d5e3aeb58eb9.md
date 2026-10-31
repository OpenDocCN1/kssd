
## 功能1 添加机器人

```c
unsigned __int64 sub_400DF8()
{
    _WORD *v0; // rax
    char *v1; // rax
    _DWORD *v2; // rax
    char *v3; // rax
    _QWORD *v4; // rax
    char *v5; // rax
    unsigned int v7; // [rsp+8h] [rbp-18h]
    unsigned int v8; // [rsp+8h] [rbp-18h]
    unsigned int v9; // [rsp+8h] [rbp-18h]
    int v10; // [rsp+Ch] [rbp-14h]
    char s[8]; // [rsp+10h] [rbp-10h] BYREF
    unsigned __int64 v12; // [rsp+18h] [rbp-8h]

    v12 = __readfsqword(0x28u);
    sub_400D83("Which robot do you want to add to the wheel?");
    printf("Your choice :");
    memset(&unk_603110, 0, 4uLL);
    v10 = sub_400A36(&unk_603110, 5uLL);
    if ((unsigned __int64)qword_603130 <= 2) {
        switch (v10) {
        case 1:
            if (!dword_603120) {
                buf = calloc(1uLL, 0x14uLL);
                dword_603120 = 1;
                v0 = buf;
                *(_QWORD *)buf = 'iT ynniT';
                v0[4] = 109;
                ++qword_603130;
            }
            break;
        case 2:
            if (!dword_603114) {
                printf("Increase Bender's intelligence: ");
                memset(s, 0, 5uLL);
                v7 = sub_400A36(s, 5uLL);
                if (v7 > 4) {
                    puts("Sorry impossible to make bender as smart!");
                    v7 = 2;
                }
                qword_6030F0 = calloc(1uLL, 20 * v7);
                qword_603138 = v7;
                dword_603114 = 1;
                v1 = (char *)qword_6030F0;
                *(_DWORD *)qword_6030F0 = 'dneB';
                strcpy(v1 + 4, "er");
                ++qword_603130;
            }
            break;
        case 3:
            if (!dword_603124) {
                printf("Increase Robot Devil's cruelty: ");
                memset(s, 0, 5uLL);
                v8 = sub_400A36(s, 5uLL);
                if (v8 > 0x63) {
                    puts("You are crazy!!");
                    v8 = 20;
                }
                qword_603100 = calloc(1uLL, 20 * v8);
                qword_603140 = v8;
                dword_603124 = 1;
                v2 = qword_603100;
                *(_QWORD *)qword_603100 = 'eD toboR';
                v2[2] = 'liv';
                ++qword_603130;
            }
            break;
        case 4:
            if (!dword_603118) {
                qword_6030E0 = calloc(1uLL, 0xFA0uLL);
                v3 = (char *)qword_6030E0;
                *(_QWORD *)qword_6030E0 = 'mS niahC';
                strcpy(v3 + 8, "oker");
                dword_603118 = 1;
                ++qword_603130;
            }
            break;
        case 5:
            if (!dword_603128) {
                qword_603108 = calloc(1uLL, 0x9C40uLL);
                v4 = qword_603108;
                *(_QWORD *)qword_603108 = 0x616E6F696C6C6942LL;
                v4[1] = 0x746F4220657269LL;
                dword_603128 = 1;
                ++qword_603130;
            }
            break;
        case 6:
            if (!dword_60311C) {
                printf("Increase Destructor's powerful: ");
                memset(s, 0, 5uLL);
                v9 = sub_400A36(s, 5uLL);
                qword_6030E8 = calloc(1uLL, 20 * v9);
                qword_603148 = v9;
                dword_60311C = 1;
                v5 = (char *)qword_6030E8;
                *(_QWORD *)qword_6030E8 = 'tcurtseD';
                strcpy(v5 + 8, "or");
                ++qword_603130;
            }
            break;
        default:
            return __readfsqword(0x28u) ^ v12;
        }
    } else {
        puts("Wheel Of Robots is Full!");
    }
    return __readfsqword(0x28u) ^ v12;
}
```

`v10 = sub_400A36(&unk_603110, 5uLL);`函数内容见上文，可以看到读入5个字节，但是`unk_603110`是用4个字节存储，如果我们读入5个字节，最后一个字节就会溢出到`dword_603114`变量，造成off by one漏洞，而`dword_603114`这个变量其实就是case2 bender机器人的申请记录，我们可以通过off by one篡改。

ps:这些变量名都对应其地址

![](https://bbs.kanxue.com/upload/attach/202603/1002337_53NFZHKXCD37C7X.png)

## 功能2 回收内存

看功能2，这里就是回收六个内存的功能，注意到六个内存在`free`之后并没有设置为`null`，而且内存大小也没设置为0，但是申请记录变量有设为0，而回看刚才的功能1，每个内存在申请时都会检测申请记录，如果我们能够修改这个值（比如利用off by one）就可以造成use after free漏洞。

```c
__int64 sub_40120C()
{
    __int64 result;
    // rax

    sub_400D83("Which robot do you want to remove from the wheel?");
    printf("Your choice :");
    memset(&unk_603110, 0, 4uLL);
    result = (unsigned int)sub_400A36(&unk_603110, 4uLL);
    switch ((int)result) {
    case 1:
        result = (unsigned int)dword_603120;
        if (dword_603120) {
            free(buf);
            dword_603120 = 0;
            result = --qword_603130;
        }
        break;
    case 2:
        result = (unsigned int)dword_603114;
        if (dword_603114) {
            free(qword_6030F0);
            dword_603114 = 0;
            result = --qword_603130;
        }
        break;
    case 3:
        result = (unsigned int)dword_603124;
        if (dword_603124) {
            free(qword_603100);
            dword_603124 = 0;
            result = --qword_603130;
        }
        break;
    case 4:
        result = (unsigned int)dword_603118;
        if (dword_603118) {
            free(qword_6030E0);
            dword_603118 = 0;
            result = --qword_603130;
        }
        break;
    case 5:
        result = (unsigned int)dword_603128;
        if (dword_603128) {
            free(qword_603108);
            dword_603128 = 0;
            result = --qword_603130;
        }
        break;
    case 6:
        result = (unsigned int)dword_60311C;
        if (dword_60311C) {
            free(qword_6030E8);
            dword_60311C = 0;
            result = --qword_603130;
        }
        break;
    default:
        return result;
    }
    return result;
}
```

## 功能3 写入内容

看功能3，就是往申请的内存里写内容，会检查内存是否已申请。
