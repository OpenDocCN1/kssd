
# netsowell 的壳的 IAT 简要分析的修复【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-552.htm`](https://www.pediy.com/kssd/pediy08/pediy8-552.htm)

netsowell 的壳的 IAT 简要分析的修复

原贴在这里

`http://bbs.pediy.com/showthread.php?s=&threadid=22815`

原程序下载：`ennotepad.rar`

由于没有花指令，分析起来好看多了，感谢 netsowell 大侠给我等菜鸟调试分析的机会

野猪力量 --- 注入!

```
      002C0264            8945 90           mov dword ptr ss:[ebp-70],eax
      002C0267            33C0             xor eax,eax
      002C0269            8945 8C          mov dword ptr ss:[ebp-74],eax
      002C026C            6A 2C            push 2C
      002C026E            8D85 D0FEFFFF    lea eax,dword ptr ss:[ebp-130]               //这里的 ebp-130 是每次处理时在堆栈中的数据结构的起始
      002C0274            50             push eax
      002C0275            53             push ebx
      002C0276            FFD6           call esi                                        //从这里到最后所有的 esi 都是 memcpy(src, dst, size)
      002C0278            E9 CC020000    jmp       002C0549
```

这个是准备处理第一个调用输入表变形的地方， 也就是把 `call [IAT]`，变成 `call 2X0000`

壳的思路就是，将一块一块关于变形 `call` 的数据结构 copy 到堆栈的局部变量中，根据数据结构完成每一个变形

这个数据结构大致是这样的

```
struct ss
{
    BYTE      key;          //解密输入表 dll 和函数名字的密钥，算法是异或
    DWORD     IATRva;       //相关的在输入表(IAT)中的位置的 RVA
    DWORD     IsOrdinal;    //1= 通过 Ordinal 取导入函数, 0=通过名字取导入函数
    DWORD     Unknown1;
    DWORD     Ordinal;      //Ordinal 的值，如果没有，这里是-1
    BYTE      code1[7];     //1 个 E8 的 call, 目的地址是处理输入表变形的公共部分
                            BYTE[7]的起始地址是 IAT 中的值
    WORD      temp1;        //临时写入 E9
    DWORD     ff25addr;     //如果不为 0，则这个值加上主代码段起始地址得到需要变形
                            的地址，且这个变形是 FF25 的变形
    WORD      temp2;        //临时写入 90 E8
    DWORD     ff15addr;     //如果不为 0，则这个值加上主代码段起始地址得到需要变形
                            的地址，且这个变形是 FF15 的变形
    DWORD     UnKnown2;
    //以上共 0x2C 长
    enum
    {
        BYTE      EnDllName[];  //加密的 dll 的名字，密钥是上面的 key, 算法是异或
        DWORD     EnFunAddr;    //将* (LPDWORD)code1[1]和导入函数的地址异或
    }
    BYTE      EnFunName[];  //加密的导入函数的名字，密钥是上面的 key, 算法是异或
}
```

```
      002C027D            8BC3             mov eax,ebx
      002C027F            0345 8C          add eax,dword ptr ss:[ebp-74]
      002C0282            83C0 11          add eax,11
      002C0285            8B55 90          mov edx,dword ptr ss:[ebp-70]
      002C0288            2BD0            sub edx,eax
      002C028A            83EA 05          sub edx,5
      002C028D            8955 88          mov dword ptr ss:[ebp-78],edx
      002C0290            C685 E1FEFFFF E8 mov byte ptr ss:[ebp-11F],0E8
      002C0297            8B45 88          mov eax,dword ptr ss:[ebp-78]
      002C029A            8985 E2FEFFFF     mov dword ptr ss:[ebp-11E],eax             //完成对数据结构 ss 中 ss.code1 的计算
      002C02A0            6A 2C            push 2C
      002C02A2            8BC3             mov eax,ebx
      002C02A4            0345 8C          add eax,dword ptr ss:[ebp-74]
      002C02A7            50             push eax
      002C02A8            8D85 D0FEFFFF    lea eax,dword ptr ss:[ebp-130]
      002C02AE            50             push eax
      002C02AF            FFD6           call esi                                       //将更新后的 ss.code1 部分写回原来的地方
      002C02B1            8BC3             mov eax,ebx
      002C02B3            0345 8C          add eax,dword ptr ss:[ebp-74]
      002C02B6            83C0 11          add eax,11
      002C02B9            8945 88          mov dword ptr ss:[ebp-78],eax
      002C02BC            6A 04            push 4
      002C02BE            8B45 AC          mov eax,dword ptr ss:[ebp-54]                //程序基址
      002C02C1            0385 D1FEFFFF    add eax,dword ptr ss:[ebp-12F]               //再加上 ss.IATRva 就是这个变形相关的 IAT 的地址
      002C02C7            50             push eax
      002C02C8            8D45 88          lea eax,dword ptr ss:[ebp-78]
      002C02CB            50             push eax
      002C02CC            FFD6           call esi                                       //将 ss.code1 的起始地址写入相关的 IAT 中
                                                                                       //这样以后每次 call [IAT]，相当于 call ss.code1
                                                                                       //ss.code1 再 call 入处理变形的公共代码部分
      002C02CE            8345 8C 2C       add dword ptr ss:[ebp-74],2C
      002C02D2            8B45 8C          mov eax,dword ptr ss:[ebp-74]
      002C02D5            03C3             add eax,ebx
      002C02D7            8945 A8          mov dword ptr ss:[ebp-58],eax
      002C02DA            83BD 54FFFFFF 00 cmp dword ptr ss:[ebp-AC],0
      002C02E1            0F85 F6010000    jnz       002C04DD
      002C02E7            8BFB             mov edi,ebx
      002C02E9            037D 8C          add edi,dword ptr ss:[ebp-74]
      002C02EC            0FB685 D0FEFFFF  movzx eax,byte ptr ss:[ebp-130]             //取出 ss.key 密钥
      002C02F3            50             push eax
      002C02F4            57             push edi                                    //ss.EnDllName
      002C02F5            E8 6D020000    call      002C0567                         //解出 dll 的名字，返回名字的长度
      002C02FA            8945 80          mov dword ptr ss:[ebp-80],eax
      002C02FD            8B45 80          mov eax,dword ptr ss:[ebp-80]
      002C0300            0145 8C          add dword ptr ss:[ebp-74],eax
      002C0303            57             push edi
      002C0304            FF55 98          call dword ptr ss:[ebp-68]              //Call GetModuleHandle，取 dll 的基础
      002C0307            8945 84          mov dword ptr ss:[ebp-7C],eax
      002C030A            837D 84 00       cmp dword ptr ss:[ebp-7C],0
      002C030E            75 07            jnz short      002C0317
      002C0310            57             push edi
      002C0311            FF55 A0          call dword ptr ss:[ebp-60]              //如果 dll 不存在，则 LoadLibrary 这个 dll
      002C0314            8945 84          mov dword ptr ss:[ebp-7C],eax
      002C0317            57             push edi
      002C0318            E8 96020000    call      002C05B3                         //取得 dll 基址后，将 dll 的名字加密回去
      002C031D            837D 84 00       cmp dword ptr ss:[ebp-7C],0
      002C0321            0F84 11020000    je      002C0538
      002C0327            83BD D5FEFFFF 00 cmp dword ptr ss:[ebp-12B],0           //ss.IsOrdinal 是否为 1
      002C032E            75 2F            jnz short      002C035F
      002C0330            8BFB             mov edi,ebx
      002C0332            037D 8C          add edi,dword ptr ss:[ebp-74]
      002C0335            0FB685 D0FEFFFF  movzx eax,byte ptr ss:[ebp-130]
      002C033C            50             push eax
      002C033D            57             push edi
      002C033E            E8 24020000    call      002C0567                         //ss.IsOrdinal 为 0 则解密导入函数的名字
      002C0343            8945 80          mov dword ptr ss:[ebp-80],eax
      002C0346            57             push edi
      002C0347            8B45 84          mov eax,dword ptr ss:[ebp-7C]
      002C034A            50             push eax
      002C034B            FF55 9C          call dword ptr ss:[ebp-64]              //GetProcAddress 取导入函数地址
      002C034E            8945 84          mov dword ptr ss:[ebp-7C],eax
      002C0351            57             push edi
      002C0352            E8 5C020000    call      002C05B3                         //将导入函数名字加密回去
```
