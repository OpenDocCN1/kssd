
# [原创]分享几个C语言花指令宏，希望论坛里的破解、脱壳和逆向的大牛们进来给个意见

男人，看待事情不求过程只求讲究结果。我也不想说C语言花指令宏生成器的实现，我只分享几个C语言花指令宏，可以扰乱IDA的静态反汇编结果。

因为我原来的文件“花指令c语言宏`Win32RandJmpCodeDef.h`”就达到186 MB没有办法完全上传给大家所以附件上的文件也就是样本。

希望论坛里的破解、脱壳和逆向的大牛们进来给个意见，谈谈关于`Win32RandJmpCodeDef.h`的花指令宏在编译后的“`空控制台程序.exe`”里有没有办法用IDC把花指令去除干净，这些宏是我以前做的，如果有人做到了用IDC把花指令去除干净，我将再给出现在我使用的更花的花指令程序给爱分析逆向的大牛们过瘾。

直接贴个具体的C语言花指令宏：

```c
#define WIN32_RAND_JMP_CODE_DEF_0(TempEax,TempEcx,TempEdx) \
        __asm MOV TempEax,EAX \
        __asm MOV TempEcx,ECX \
        __asm MOV TempEdx,EDX \
/* 本次产生的随机跳转类型齐全 */ \
__asm Win32_Rand_Jmp_Code_Def_0_Start: /* 随机跳转框架开始 AsmSub_Start 类型代码 */ \
/* 以下开辟了24字节堆栈伪造调用参数 */ \
        __asm PUSH ECX \
        __asm PUSH EAX \
        __asm PUSH ESI \
        __asm PUSH ESP \
        __asm PUSH EBX \
        __asm PUSH ESI \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_2 /* 调用AsmSub_2类型代码 */ \
        __asm _emit 0xC2 /* 不会被执行的花指令，伪造函数返回 */ \
        __asm _emit 80  \
        __asm _emit 0  \
/* 随机产生的垃圾数据，反汇编器将错误反汇编或无法再继续反汇编 */ \
        __asm _emit 0x16 \
__asm Win32_Rand_Jmp_Code_Def_0_1:/* AsmSub_1类型代码入口 */ \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x0F \
        __asm _emit 0x8D \
        __asm SUB ESP,8 /*随机跳转入口*/ \
/*__asm Win32_Rand_Jmp_Code_Def_0_F_6_0:*/ \
        __asm JNS Win32_Rand_Jmp_Code_Def_0_F_6_4 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x35 \
__asm Win32_Rand_Jmp_Code_Def_0_F_6_1: \
        __asm JNB Win32_Rand_Jmp_Code_Def_0_F_6_2 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x81 \
        __asm _emit 0xEF \
__asm Win32_Rand_Jmp_Code_Def_0_F_6_2: \
        __asm PUSH ESI \
        __asm PUSH ECX \
        __asm PUSH ESI \
        __asm PUSH EAX \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_F_6_3 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x0F \
        __asm _emit 0x8C \
__asm Win32_Rand_Jmp_Code_Def_0_F_6_3: \
        __asm PUSH EBX \
        __asm SUB ESP,20 \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_F_6_End \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0xB9 \
__asm Win32_Rand_Jmp_Code_Def_0_F_6_4: \
        __asm JNS Win32_Rand_Jmp_Code_Def_0_F_6_1 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x81 \
        __asm _emit 0xC2 \
__asm Win32_Rand_Jmp_Code_Def_0_F_6_End: \
        /* 随机跳转出口,开辟了堆栈帧 56 字节*/ \
        __asm SUB DWORD PTR[ESP + 100],OFFSET Win32_Rand_Jmp_Code_Def_0_1/* 运行地址 - 编译地址（需要修正）*/ \
        __asm SUB ESP,4 /*随机跳转入口*/ \
/*__asm Win32_Rand_Jmp_Code_Def_0_F_7_0:*/ \
        __asm JNC Win32_Rand_Jmp_Code_Def_0_F_7_4 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x0F \
        __asm _emit 0x8C \
__asm Win32_Rand_Jmp_Code_Def_0_F_7_1: \
        __asm JNZ Win32_Rand_Jmp_Code_Def_0_F_7_3 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x81 \
        __asm _emit 0xEB \
__asm Win32_Rand_Jmp_Code_Def_0_F_7_2: \
        __asm POP EDX \
        __asm JNS Win32_Rand_Jmp_Code_Def_0_F_7_1 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x81 \
        __asm _emit 0xC1 \
__asm Win32_Rand_Jmp_Code_Def_0_F_7_3: \
        __asm JNZ Win32_Rand_Jmp_Code_Def_0_F_7_End \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0xBF \
__asm Win32_Rand_Jmp_Code_Def_0_F_7_4: \
        __asm PUSH ESP \
        __asm PUSH ECX \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_F_7_2 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0xFF \
        __asm _emit 0x25 \
__asm Win32_Rand_Jmp_Code_Def_0_F_7_End: \
        /* 随机跳转出口,开辟了堆栈帧 12 字节*/ \
        __asm SUB DWORD PTR[ESP + 112],2/* 运行地址 - 编译地址（内嵌宏进行编译后的修正值[esp + x]为0） */ \
        __asm SUB ESP,8 /*随机跳转入口*/ \
/*__asm Win32_Rand_Jmp_Code_Def_0_F_8_0:*/ \
        __asm JNL Win32_Rand_Jmp_Code_Def_0_F_8_2 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x05 \
__asm Win32_Rand_Jmp_Code_Def_0_F_8_1: \
        __asm PUSH ESI \
        __asm PUSH ESI \
        __asm PUSHFD \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_F_8_3 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0xB9 \
__asm Win32_Rand_Jmp_Code_Def_0_F_8_2: \
        __asm JNO Win32_Rand_Jmp_Code_Def_0_F_8_4 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x81 \
        __asm _emit 0xC1 \
__asm Win32_Rand_Jmp_Code_Def_0_F_8_3: \
        __asm PUSH EBP \
        __asm PUSH EDX \
        __asm PUSH EDI \
        __asm PUSH EBX \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_F_8_End \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x0F \
        __asm _emit 0x85 \
__asm Win32_Rand_Jmp_Code_Def_0_F_8_4: \
        __asm JNC Win32_Rand_Jmp_Code_Def_0_F_8_1 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x64 \
        __asm _emit 0xA3 \
__asm Win32_Rand_Jmp_Code_Def_0_F_8_End: \
        /* 随机跳转出口,开辟了堆栈帧 44 字节*/ \
        __asm ADD DWORD PTR[ESP + 156],OFFSET Win32_Rand_Jmp_Code_Def_0_3/* AsmSub_3 类型代码的编译地址 + （运行地址 - 编译地址）得到AsmSub_3 类型代码的运行地 */ \
        __asm SUB ESP,4 /*随机跳转入口*/ \
/*__asm Win32_Rand_Jmp_Code_Def_0_F_9_0:*/ \
        __asm JA Win32_Rand_Jmp_Code_Def_0_F_9_2 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0xE9 \
__asm Win32_Rand_Jmp_Code_Def_0_F_9_1: \
        __asm JNB Win32_Rand_Jmp_Code_Def_0_F_9_3 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x0F \
        __asm _emit 0x89 \
__asm Win32_Rand_Jmp_Code_Def_0_F_9_2: \
        __asm JNZ Win32_Rand_Jmp_Code_Def_0_F_9_4 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x81 \
        __asm _emit 0xC2 \
__asm Win32_Rand_Jmp_Code_Def_0_F_9_3: \
        __asm PUSH EBP \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_F_9_End \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0xB9 \
__asm Win32_Rand_Jmp_Code_Def_0_F_9_4: \
        __asm JG Win32_Rand_Jmp_Code_Def_0_F_9_1 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x0F \
        __asm _emit 0x85 \
__asm Win32_Rand_Jmp_Code_Def_0_F_9_End: \
        /* 随机跳转出口,开辟了堆栈帧 12 字节*/ \
/* 跳到 AsmSub_3 类型代码执行 */ \
        __asm JMP DWORD PTR[ESP + 168] \
/* 随机产生的垃圾数据，反汇编器将错误反汇编或无法再继续反汇编 */ \
        __asm _emit 0xE8 \
/* 随机产生x个0 填充，触使反汇编器错误识别为遇到数据段了 */ \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
        __asm _emit 0 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0xBA \
__asm Win32_Rand_Jmp_Code_Def_0_2: /* AsmSub_2 类型代码入口 */ \
        __asm SUB ESP,8 /*随机跳转入口*/ \
/*__asm Win32_Rand_Jmp_Code_Def_0_F_4_0:*/ \
        __asm JNZ Win32_Rand_Jmp_Code_Def_0_F_4_4 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
        __asm _emit 0x81 \
        __asm _emit 0xE7 \
__asm Win32_Rand_Jmp_Code_Def_0_F_4_1: \
        __asm PUSH EBX \
        __asm PUSH EAX \
        __asm CALL Win32_Rand_Jmp_Code_Def_0_F_4_2 \
/* 随机产生的多字节指令前缀汇编代码，导致后续反汇编将很可能会出错 */ \
```
