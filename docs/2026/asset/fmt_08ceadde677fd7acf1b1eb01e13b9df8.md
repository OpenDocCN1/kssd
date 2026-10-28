
```


// 仅 CmUnRegisterCallback 可以定位



// 1. 由 CmUnRegisterCallback 定位 nt!CallbackListHead 链表地址

0: kd> uf CmUnRegisterCallback

nt!CmUnRegisterCallback:

fffff805`2e38bd50 4c8bdc          mov     r11,rsp

fffff805`2e38bd53 53              push    rbx

fffff805`2e38bd54 56              push    rsi

fffff805`2e38bd55 57              push    rdi

fffff805`2e38bd56 4154            push    r12

fffff805`2e38bd58 4155            push    r13

fffff805`2e38bd5a 4156            push    r14

fffff805`2e38bd5c 4157            push    r15

fffff805`2e38bd5e 4881ec80000000  sub     rsp,80h

fffff805`2e38bd65 488bd9          mov     rbx,rcx

fffff805`2e38bd68 be0d0000c0      mov     esi,0C000000Dh

fffff805`2e38bd6d 89b424d8000000  mov     dword ptr [rsp+0D8h],esi

fffff805`2e38bd74 33c0            xor     eax,eax

fffff805`2e38bd76 498943b0        mov     qword ptr [r11-50h],rax

fffff805`2e38bd7a 498943b8        mov     qword ptr [r11-48h],rax

fffff805`2e38bd7e 498943c0        mov     qword ptr [r11-40h],rax

fffff805`2e38bd82 49214380        and     qword ptr [r11-80h],rax

fffff805`2e38bd86 65488b042588010000 mov   rax,qword ptr gs:[188h]

fffff805`2e38bd8f 4183ccff        or      r12d,0FFFFFFFFh

fffff805`2e38bd93 664401a0e4010000 add     word ptr [rax+1E4h],r12w

fffff805`2e38bd9b 33d2            xor     edx,edx

fffff805`2e38bd9d 4c8d35ecf3ccff  lea     r14,[nt!CmpCallbackListLock (fffff805`2e05b190)]

fffff805`2e38bda4 498bce          mov     rcx,r14

fffff805`2e38bda7 e894f094ff      call    nt!ExAcquirePushLockExclusiveEx (fffff805`2dcdae40)

fffff805`2e38bdac 41bf00000080    mov     r15d,80000000h



nt!CmUnRegisterCallback+0x62:

fffff805`2e38bdb2 4533c0          xor     r8d,r8d

fffff805`2e38bdb5 488d542438      lea     rdx,[rsp+38h]

fffff805`2e38bdba 488d0ddff3ccff  lea     rcx,[nt!CallbackListHead (fffff805`2e05b1a0)]

fffff805`2e38bdc1 e82a94e1ff      call    nt!CmListGetNextElement (fffff805`2e1a51f0)

fffff805`2e38bdc6 488bf8          mov     rdi,rax

fffff805`2e38bdc9 4889442440      mov     qword ptr [rsp+40h],rax

fffff805`2e38bdce 4885c0          test    rax,rax

fffff805`2e38bdd1 0f84cf000000    je      nt!CmUnRegisterCallback+0x156 (fffff805`2e38bea6)  Branch





// 2. 显示 CMREG_CALLBACK 结构（nt!CallbackListHead）

0: kd> dq nt!CallbackListHead

fffff805`2e05b1a0  ffff9705`3572b420 ffff9705`3572ba20

fffff805`2e05b1b0  00000000`00000000 01d9e206`c817750c

fffff805`2e05b1c0  fffff805`2e05b1c0 fffff805`2e05b1c0

fffff805`2e05b1d0  00000000`00000000 00000000`00000000

fffff805`2e05b1e0  00000000`00060001 fffff805`2e05b1e8

fffff805`2e05b1f0  fffff805`2e05b1e8 00000000`00000000

fffff805`2e05b200  00000000`00060001 fffff805`2e05b208

fffff805`2e05b210  fffff805`2e05b208 00000000`00000000





// 3. 显示 nt!CallbackListHead 下一个结点

0: kd> dq ffff9705`3572ba20

ffff9705`3572ba20  fffff805`2e05b1a0 ffff9705`35449660

ffff9705`3572ba30  00000000`00000000 01d9e206`c817750b

ffff9705`3572ba40  00000000`00000000 fffff805`3015c728

ffff9705`3572ba50  00000000`000c000c ffff9705`3572a530

ffff9705`3572ba60  ffff9705`3572ba60 ffff9705`3572ba60

ffff9705`3572ba70  74705041`03060000 00000000`00000000

ffff9705`3572ba80  ffff9705`35729ae0 00000000`00000000

ffff9705`3572ba90  00000000`00000000 00000000`00000000





// 4. 偏移 0x28 位置 的 8 字节指针（回调函数）所在模块

0: kd> lm a fffff805`3015c728

Browse full module list

start             end                 module name

fffff805`30150000 fffff805`301a5000   360qpesv64   (deferred)
```

## 07 — 删除杀软回调的方法

删除上述四大系统回调的方法被分为两类，“创建进程通知回调”、“创建线程通知回调”、“加载镜像通知回调”被分为一类，为数组类；“注册表通知回调”被分为一类，为双向循环链表类，以下简称链表类。

数组类删除方法是：找到驱动对应数组中的元素，将该元素内核地址赋值为 0；

链表类删除方法是：找到驱动对应的链表中的结点，利用数据结构中双向循环链表删除结点的方法，删除杀软驱动对应的结点。

这里需要注意，数组类中：

“创建进程通知回调”、“创建线程通知回调”，在 win7 及以上的个人系统和 Server 系统中，通常情况下，数组大小为 64 个元素；而“加载镜像通知回调”中，通常情况下，win10 全系列及以上，数组大小为 64 个元素，win7-7601、win8-9200、win8.1-9600系统中，可能为 8 个或 64 个元素，为不定值，相关资料可以参考微软官方文档，解决方法是，通过 `nt!PspLoadImageNotifyRoutineCount` 确定当前注册回调个数。

## 08 — 删除杀软回调效果

- 运行 Mimikatz 测试：

  这里测试国内外6款主流杀软，国内两款：某0 数字卫士和某绒，国外四款：卡巴斯基、Windows Defender、eset-nod32、avast；由于该项目主要消弱杀软或EDR的动态查杀能力，这里，使用内存加载 Mimikatz 的方式进行测试，注意该项目需要免杀或者重新构思编码，比如：换个合法漏洞驱动，编写自定义驱动使用 kdmapper 改良版等项目内存加载等，总之，首先要保证你的测试程序在杀软环境下要免杀而且能正常执行功能。在这里，只考虑删除杀软回调对杀软的影响，其他相关因素需要通过某些方法进行消除处理。

  测试结果：

  - eset-nod32、avast、某绒、某0 数字卫士：无需删除杀软回调，内存加载 Mimikatz，就可以正常执行功能；

  - 卡巴斯基、Windows Defender：删除全部回调前，内存加载 Mimikatz 被查杀；删除全部回调后，内存加载 Mimikatz 正常执行功能。

- 注册表修改测试：

  - 删除全部回调前，某些注册表键值不可修改；

  - 删除全部回调后，某些注册表键值现在已经可以修改。

## 09 — 备注

如果，需要该项目 Windows 全版本兼容，则需要将所有版本的特征码一一找出进行总结适配，下面有一个github项目可以节省你很多时间。

3f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1K9h3N6T1j5h3&6Y4z5e0g2Q4x3V1k6F1N6r3!0K6K9%4u0F1L8l9`.`.

## 10 — 参考链接

文章：

[对抗临近 | 红队大佬的私人秘籍：EDR绕过技术曝光！](https://bbs.kanxue.com/elink@12bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9K6c8W2)9#2k6W2)9#2k6X3u0A6P5W2)9K6c8p5#2*7g2e0y4a6g2p5q4&6e0@1c8C8y4p5#2c8i4K6y4p5i4K6y4p5i4K6t1$3L8h3W2V1i4K6y4p5x3U0t1@1y4K6b7^5y4U0f1@1y4q4)9J5y4X3W2V1P5q4)9K6c8o6q4Q4x3U0k6K6L8W2)9K6c8o6p5H3x3o6x3J5k6U0S2T1z5e0k6X3y4U0j5%4y4h3f1%4y4o6p5^5y4$3q4S2k6e0x3J5j5U0t1@1x3e0g2X3i4K6t1$3M7$3y4W2L8X3g2Q4x3@1b7J5x3g2)9J5x3%4N6W2j5$3S2S2N6q4)9#2k6Y4u0W2k6r3W2J5k6h3y4@1)（e20K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3A6$3j5e0u0V1z5r3&6x3P5U0k6@1K9e0S2X3e0q4c8d9x3p5!0Q4x3X3c8%4f1g2!0q4c8W2!0n7b7#2)9^5z5b7`.`.

ef9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1M7W2)9J5k6s2y4F1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6d9k6h3#2G2N6X3W2F1k6#2)9J5k6p5E0W2M7X3&6W2L8q4)9J5k6p5y4S2L8r3I4T1j5h3y4C8M7#2)9J5k6q4g2K6K9h3&6Y4i4K6u0V1f1$3W2Y4L8X3g2V1i4K6u0V1c8s2u0A6N6X3g2J5M7#2)9J5c8R3`.`.

项目：

5d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7W2)9J5k6s2y4F1i4K6u0r3b7$3S2W2k6h3E0&6b7X3I4A6L8X3c8W2M7R3`.`.

37aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5i4N6A6k6i4b7@1y4#2)9J5c8W2y4f1c8W2g2q4c8q4t1`.

## 11 — 声明

本文所述方法，仅供安全研究使用。凡擅自用于违法用途，将被追究法律责任。其违法行为均与本人无关。特此声明！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
