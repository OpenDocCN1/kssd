
# [原创]华琦库管精灵注册分析--VB与函数的嵌套附算法程序代码

## 破解信息

- 【破解作者】 `jsliyangsj`
- 【作者邮箱】 `sjcrack@yahoo.com.cn`
- 【使用工具】 `peid`、`OllyDbg1.10`
- 【破解平台】 Winxp
- 【软件名称】 华琦库管精灵
- 【软件地址】 `f3bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6G2j5$3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3X3g2Z5N6r3@1`
- 【编写语言】 VB
- 【软件介绍】 库管精灵是我们推出的一款构思奇巧、结构严谨的实用精品软件。该软件具有独到的数据分段检索、自定义特殊处理、智能辅助输入、智能成本核算、分级+分组权限控制、外部数据导入、多种数据格式转换等神奇功能，并采用了数据批量传送、多重数据加密、强力数据压缩、并发冲突降解等多种先进技术。界面简洁、功能强大，是典型的傻瓜+实用型的进销存一体化设计的仓储及营销管理工具！软件中全面集成了库存物资的入库、出库、调拨、处理、利润核算、盘库、盘库盈亏核算等数据的录入、维护、汇总、统计、查询、过滤、报表打印及数据格式转换（支持Microsoft Excel、Microsoft Access、Dbase、Foxpro、HTML、纯文本等格式）等系列功能。因界面简介而貌似“迷你”，实际上功能相当全面--甚至集成了大型进销存软件都不敢随意问津的任意组合查询，并进而独创了强大的直接SQL查询！内嵌了自主开发的HK Script脚本语言解释器！

## 过程

用暂停法，在堆栈中找到返回主程序的地址堆栈，直接返回即可到达主程序的错误窗口点。

```asm
005DEB58    .  E>call <jmp.&MSVBVM60.\_\_vbaStrMove>
005DEB5D    .  F>push dword ptr ss:[ebp-28]
005DEB60    .  E>call <jmp.&MSVBVM60.\_\_vbaLenBstr>         ;  得到注册码的长度
005DEB65    .  8>test eax,eax                              ;  测试是否输入了注册码
005DEB67    .  0>jnz stmagic.005DEC1D
005DEB6D    .  E>call <jmp.&MSVBVM60.#685>
005DEB72    .  5>push eax
005DEB73    .  8>lea eax,dword ptr ss:[ebp-48]
…………省略………………
005DEC1D    > \F>push dword ptr ss:[ebp-28]
005DEC20    .  E>call stmagic.004F307B                     ;  关键点，进入！！  计算和比较都在里面
005DEC25    .  6>test ax,ax
005DEC28    .  8>mov dword ptr ss:[ebp-8C],edi
005DEC2E    .  8>mov dword ptr ss:[ebp-94],esi
005DEC34    .  8>mov dword ptr ss:[ebp-7C],edi
005DEC37    .  8>mov dword ptr ss:[ebp-84],esi
005DEC3D    .  8>mov dword ptr ss:[ebp-6C],edi
005DEC40    .  8>mov dword ptr ss:[ebp-74],esi
005DEC43    .  7>jnz short stmagic.005DEC85                ;  如果比较不相等，错误
005DEC45    .  C>mov dword ptr ss:[ebp-BC],stmagic.0043CDE>
005DEC4F    .  8>mov dword ptr ss:[ebp-C4],ebx
005DEC55    .  8>lea edx,dword ptr ss:[ebp-C4]
005DEC5B    .  8>lea ecx,dword ptr ss:[ebp-64]
005DEC5E    .  E>call <jmp.&MSVBVM60.\_\_vbaVarDup>
005DEC63    .  8>lea eax,dword ptr ss:[ebp-94]
005DEC69    .  5>push eax
005DEC6A    .  8>lea eax,dword ptr ss:[ebp-84]
005DEC70    .  5>push eax
005DEC71    .  8>lea eax,dword ptr ss:[ebp-74]
005DEC75    .  6>push 40
005DEC77    .  8>lea eax,dword ptr ss:[ebp-64]
005DEC7A    .  5>push eax
005DEC7B    .  E>call <jmp.&MSVBVM60.#595>                 ;  错误窗口！！！
```

进入`call stmagic.004F307B`

```asm
004F30E8    .  E>call <jmp.&MSVBVM60.\_\_vbaLenBstr>         ;  得到输入码的长度
004F30ED    .  8>cmp eax,21                                ;  输入码是否位33位，不是就完
004F30F0    .  0>jnz stmagic.004F32C3
004F30F6    .  C>mov dword ptr ss:[ebp-6C],stmagic.0043CE8>
004F30FD    .  C>mov dword ptr ss:[ebp-74],8
004F3104    .  8>lea edx,dword ptr ss:[ebp-74]
004F3107    .  8>lea ecx,dword ptr ss:[ebp-54]
004F310A    .  E>call <jmp.&MSVBVM60.\_\_vbaVarDup>
004F310F    .  6>push 1
004F3111    .  6>push -1
004F3113    .  8>lea eax,dword ptr ss:[ebp-54]
004F3116    .  5>push eax
004F3117    .  F>push dword ptr ss:[ebp-24]
004F311A    .  8>lea eax,dword ptr ss:[ebp-64]
004F311D    .  5>push eax
004F311E    .  E>call <jmp.&MSVBVM60.#711>
004F3123    .  8>lea eax,dword ptr ss:[ebp-64]
004F3126    .  5>push eax
004F3127    .  6>push 2008
004F312C    .  E>call <jmp.&MSVBVM60.\_\_vbaAryVar>
004F3131    .  8>mov dword ptr ss:[ebp-78],eax
004F3134    .  8>lea eax,dword ptr ss:[ebp-78]
004F3137    .  5>push eax
004F3138    .  8>lea eax,dword ptr ss:[ebp-28]
004F313B    .  5>push eax
004F313C    .  E>call <jmp.&MSVBVM60.\_\_vbaAryCopy>
004F3141    .  8>lea eax,dword ptr ss:[ebp-64]
004F3144    .  5>push eax
004F3145    .  8>lea eax,dword ptr ss:[ebp-54]
004F3148    .  5>push eax
004F3149    .  6>push 2
004F314B    .  E>call <jmp.&MSVBVM60.\_\_vbaFreeVarList>
004F3150    .  8>add esp,0C
004F3153    .  F>push dword ptr ss:[ebp-28]
004F3156    .  6>push 1
004F3158    .  E>call <jmp.&MSVBVM60.\_\_vbaUbound>
004F315D    .  8>cmp eax,5                                 ;  上面是检查输入码中是否有5个“－”
004F3160    .  0>jnz stmagic.004F32C3                      ;  不等于就完
```

```asm
004F3168    > /6>push 4
004F316A    . |5>pop eax
004F316B    . |3>cmp esi,eax
004F316D    . |F>push dword ptr ss:[ebp-34]
004F3170    . |0>jg stmagic.004F322B
004F3176    . |6>push 1
004F3178    . |5>push esi
004F3179    . |F>push dword ptr ss:[ebp-28]
004F317C    . |E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>
004F3181    . |F>push dword ptr ds:[eax]                   ;  看到第一段
004F3183    . |E>call <jmp.&MSVBVM60.#616>                 ;  从左边取第一位
004F3188    . |8>mov edx,eax
004F318A    . |8>lea ecx,dword ptr ss:[ebp-3C]
004F318D    . |E>call <jmp.&MSVBVM60.\_\_vbaStrMove>
004F3192    . |5>push eax
004F3193    . |E>call <jmp.&MSVBVM60.\_\_vbaStrCat>          ;  把每段的第一位及最后一段的全部组合在一起用于计算出注册码
004F3198    . |8>mov edx,eax
004F319A    . |8>lea ecx,dword ptr ss:[ebp-34]
004F319D    . |E>call <jmp.&MSVBVM60.\_\_vbaStrMove>
004F31A2    . |8>lea ecx,dword ptr ss:[ebp-3C]
004F31A5    . |E>call <jmp.&MSVBVM60.\_\_vbaFreeStr>
004F31AA    . |C>mov dword ptr ss:[ebp-4C],80020004
004F31B1    . |C>mov dword ptr ss:[ebp-54],0A
004F31B8    . |8>lea eax,dword ptr ss:[ebp-54]
004F31BB    . |5>push eax
004F31BC    . |6>push 2
004F31BE    . |5>push esi
004F31BF    . |F>push dword ptr ss:[ebp-28]
004F31C2    . |E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>
004F31C7    . |F>push dword ptr ds:[eax]
004F31C9    . |E>call <jmp.&MSVBVM60.#631>
004F31CE    . |8>mov edx,eax
004F31D0    . |8>lea ecx,dword ptr ss:[ebp-3C]
004F31D3    . |E>call <jmp.&MSVBVM60.\_\_vbaStrMove>
004F31D8    . |8>mov ebx,eax
004F31DA    . |5>push esi
004F31DB    . |F>push dword ptr ss:[ebp-28]
004F31DE    . |E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>
004F31E3    . |8>mov ecx,eax
004F31E5    . |8>mov edx,ebx
004F31E7    . |E>call <jmp.&MSVBVM60.\_\_vbaStrCopy>
004F31EC    . |8>lea ecx,dword ptr ss:[ebp-3C]
004F31EF    . |E>call <jmp.&MSVBVM60.\_\_vbaFreeStr>
004F31F4    . |8>lea ecx,dword ptr ss:[ebp-54]
004F31F7    . |E>call <jmp.&MSVBVM60.\_\_vbaFreeVar>
004F31FC    . |F>push dword ptr ss:[ebp-38]
004F31FF    . |5>push esi
004F3200    . |F>push dword ptr ss:[ebp-28]
004F3203    . |E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>
004F3208    . |F>push dword ptr ds:[eax]
004F320A    . |E>call <jmp.&MSVBVM60.\_\_vbaStrCat>          ;  把去掉每段第一位和“－”的输入码重新组合0015eeec中
004F320F    . |8>mov edx,eax
004F3211    . |8>lea ecx,dword ptr ss:[ebp-38]
004F3214    . |E>call <jmp.&MSVBVM60.\_\_vbaStrMove>
004F3219    . |6>push 1
004F321B    . |5>pop eax
004F321C    . |0>add eax,esi
004F321E    . |0>jo stmagic.004F333D
004F3224    . |8>mov esi,eax
004F3226    .^\E>jmp stmagic.004F3168
004F322B    >  6>push 5                                    ;  上面一段是把各个段的第一个输入码组合在一起
004F322D    .  F>push dword ptr ss:[ebp-28]
004F3230    .  E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>
004F3235    .  F>push dword ptr ds:[eax]
004F3237    .  E>call <jmp.&MSVBVM60.\_\_vbaStrCat>          ;  把最后一段的输入码与刚才的组合
004F323C    .  8>mov edx,eax
004F323E    .  8>lea ecx,dword ptr ss:[ebp-34]
004F3241    .  E>call <jmp.&MSVBVM60.\_\_vbaStrMove>
004F3246    .  F>push dword ptr ss:[ebp-34]
```

此段是：输入码应该写成：`输入码1-输入码2-输入码3-输入码4-输入码5-输入码6`；总长度为33

并把输入码1、输入码2、输入码3、输入码4、输入码5的第一个字符，及输入码6的全部字符组合为一组参与运算其他的注册码。

也把去掉上面组合起来的字符，及5个“-”，在组合为另一组用与比较参与计算的输入码计算出来的字符。

输入码有要求：实际上，最后一段输入码6只能是三位，因为：

```asm
004F3246    .  F>push dword ptr ss:[ebp-34]
004F3249    .  E>call stmagic.004F2F98                     ;  把参与计算的输入码压入，计算注册码
004F324E    .  8>mov edx,eax
004F3250    .  8>lea ecx,dword ptr ss:[ebp-40]
004F3253    .  E>call <jmp.&MSVBVM60.\_\_vbaStrMove>
004F3258    .  8>mov edx,dword ptr ss:[ebp-40]
004F325B    .  8>mov dword ptr ss:[ebp-40],edi
004F325E    .  6>push 14                                   ;  20
004F3260    .  8>lea ecx,dword ptr ss:[ebp-3C]
```
